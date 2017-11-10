# ROOT证书、CA证书和使用CA签发的X.509证书

[TOC]

## 简介

日常开发中，我们程序员不怎么会接触证书相关的问题，对信息安全领域相关的内容知之甚少。因为平时主要实现的业务很少要直接面向底层的通信，也就很少关注这证书这样的知识。在一般情况下，我们仅仅只是在使用一些高层的依赖中会引入证书、加密相关的依赖包，比如：
``` xml
<!-- https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on -->
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.58</version>
</dependency>
```
然而， bouncy castle这样广泛使用证书密码项目([Github](https://github.com/bcgit/bc-java))并不是广受关注的明星项目，文档也就不怎么完备，甚至有点儿法语焉不详，不够系统完整。而且，这方面的内容包含了很多密码学方面专业名词，文档读起来就更晦涩难懂。可以参与bouncy castle的[WIKI](http://bouncycastle.org/wiki/)。我将大致的列举一下BC在证书方面的使用方法。

## 概念

ROOT证书、CA证书和使用CA证签发的X.509证书之间的联系，其实就是一个使用私钥签发（issue）的关系。使用ROOT证书的私钥签发CA证书，再使用CA证书签发其他的X.509证书，这样就形成了一条可以信的Path。在被签发的证书中含有使用签发者使用自己私钥加过密的签名（sign）、签名算法（比如Sha256WithHash）以及签名者的一些身份信息，这个我们可以在[验证签名的过程](https://docs.oracle.com/cd/E19424-01/820-4811/6ng8i26av/index.html)中得过启发。

从上面可以看出，我们可以把一个证书主要分为两类：自签名和Ca签名，并且我在上面提到的证书中只有ROOT的是自签名的。

## 使用Bouncy Castle

### 生成一个签名证书

ROOT证书是用来签发和验证CA证书的，在整个可信证书链条中处于最上层的地位，它是一个自签名的证书。签名的作用是防止在不知道签名者私钥的情况下恶意篡改证书的内容，这是证书可信保证重要的步骤。如下代码是生成一个签名的证书：

``` java
// must add BC Provider before use it in code
Security.addProvider(new BouncyCastleProvider());

// generate EC key pair
ECGenParameterSpec ecGenSpec = new ECGenParameterSpec("prime192v1");
KeyPairGenerator generator = KeyPairGenerator.getInstance("ECDSA", "BC");
generator.initialize(ecGenSpec, new SecureRandom());
KeyPair kp = generator.genKeyPair();

Date startDate = new Date(System.currentTimeMillis() - 24 * 60 * 60 * 1000);
Date endDate = new Date(System.currentTimeMillis() + 365 * 24 * 60 * 60 * 1000);

// create x.509 certificate build
X509v3CertificateBuilder v3CertGen = new JcaX509v3CertificateBuilder(
  new X500Principal("CN=Test"),
  BigInteger.valueOf(System.currentTimeMillis()),
  startDate, endDate,
  new X500Principal("CN=Test"),
  kp.getPublic());

// Content Signer
ContentSigner signer = new JcaContentSignerBuilder("SHA256withECDSA")
  .setProvider("BC").build(kp.getPrivate());

// build x.509 certificate
X509CertificateHolder holder = v3CertGen.build(signer);
```

holder即我们生成的X.509证书，对于生成的证书我们可以使用org.bouncycastle.openssl.jcajce.JcaPEMWriter转化成字符串：

```java
// certificate to write to string
X509CertificateHolder holder = ...;

ByteArrayOutputStream bos = new ByteArrayOutputStream();
JcaPEMWriter jcaPEMWriter = new JcaPEMWriter(new PrintWriter(bos));
jcaPEMWriter.writeObject(holder);
jcaPEMWriter.flush();

System.out.printf(bos.toString());
```

这个我们生成的证书的内容：

```
-----BEGIN CERTIFICATE-----
MIHwMIGmoAMCAQICBgFfoSV4vTAKBggqhkjOPQQDAjAPMQ0wCwYDVQQDEwRUZXN0
MB4XDTE3MTEwODE0MTgyOFoXDTE3MTEyNjE0NTg1N1owDzENMAsGA1UEAxMEVGVz
dDBJMBMGByqGSM49AgEGCCqGSM49AwEBAzIABLgAmZuZxd6IFZ7tJS8H0VY1Wify
2i+YTIwF5cw0O7t1pcAh6dkF021bh+W0gk/rzzAKBggqhkjOPQQDAgM5ADA2AhkA
6Vb1VWTQrwcrEr2+7eTkAcwiHJ+YqGFoAhkAxWvJu2L+8dQbXXa6bd48zvR0ifX8
849E
-----END CERTIFICATE-----
```

### 验证证书的签名

以下代码用于验证我们的签名是否合法。

```java
// public key from issuer, used to verify signature
PublicKey publicKey = ...;

ContentVerifierProvider contentVerifierProvider = new JcaContentVerifierProviderBuilder()
  .setProvider("BC").build(publicKey);

if (!certHolder.isSignatureValid(contentVerifierProvider)) {
  System.err.println("signature invalid");
} else {
  System.out.printf("signature valid");
}
```

## 使用来自Oracle的证书支持

oracle Java使用keystore来存储证书及其私钥，并提供一个命令行工具——keytool。

```bash
//generate key pair with alias test
keytool -genkeypair -alias test -keyalg EC -validity 365 -keypass password -keysize 571 -keystore ./test.jks  -storepass password

//generate certificate with public key from keypair test
keytool -certreq -alias test -sigalg SHA256WithECDSA -keypass password -keysize 571 -keystore ./test.jks  -storepass password

//list jdk content
keytool -list -rfc  -keystore ./test.jks
```

### 加载Keystore文件

使用java.security.KeyStore加载Keystore文件：

```java
KeyStore keyStore = KeyStore.getInstance("jks");
        keyStore.load(new FileInputStream("C:\\...\\test.jks"), "password".toCharArray());
        Certificate certificate = keyStore.getCertificate("test");
```

可以看到X.509的证书结构：

```
[
[
  Version: V3
  Subject: CN=yinkn, OU=ericsson, O=ericsson, L=GZ, ST=GD, C=CN
  Signature Algorithm: SHA256withECDSA, OID = 1.2.840.10045.4.3.2

  Key:  Sun EC public key, 571 bits
  public x coord: 3002605725884979889022476092292003776673075892528685922593430921126421431956026876782257630852231195990030412442522229709626574940546289855595376742293701765308565676110538
  public y coord: 3969672869627949486403678776244029006985355959068388562809699610619530481229193224794938137041924106460645494573259209667845661791858569871701054704389588094823968191257337
  parameters: sect571k1 [NIST K-571] (1.3.132.0.38)
  Validity: [From: Thu Nov 09 23:02:59 GMT+08:00 2017,
               To: Fri Nov 09 23:02:59 GMT+08:00 2018]
  Issuer: CN=yinkn, OU=ericsson, O=ericsson, L=GZ, ST=GD, C=CN
  SerialNumber: [    1ae30239]

Certificate Extensions: 1
[1]: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 02 50 11 0F 4C F1 C3 FA   FC BB 0F 19 5C 83 D6 36  .P..L.......\..6
0010: 95 C3 E3 3E                                        ...>
]
]

]
  Algorithm: [SHA256withECDSA]
  Signature:
0000: 30 81 93 02 48 00 9B D1   92 9E 7B 95 FC 0E 11 19  0...H...........
0010: 28 25 C2 C2 35 2E D5 B1   74 5C F4 EA A9 D8 71 95  (%..5...t\....q.
0020: F1 72 BE 8B 53 A2 8D CA   AC 88 96 A5 86 B3 46 14  .r..S.........F.
0030: 1B B1 8B A5 12 08 B7 7D   ED 23 BF 64 DD 46 E2 33  .........#.d.F.3
0040: E0 6B C1 62 14 25 BE 5C   B9 AB 45 04 AD 02 47 6F  .k.b.%.\..E...Go
0050: D0 B9 B2 9E A1 D3 D6 98   8A 56 16 38 6B BB 27 B1  .........V.8k.'.
0060: 42 53 D7 27 95 2F 96 2E   E4 39 19 3D A3 54 54 8B  BS.'./...9.=.TT.
0070: 71 DA 0E 91 54 5F 78 37   CA 03 58 4F 56 FD C1 BD  q...T_x7..XOV...
0080: 43 9D 06 CF DF 6A B1 C6   4D 5C 0B 06 62 FF DF 8C  C....j..M\..b...
0090: 33 0B A7 62 0A ED                                  3..b..

]
```

### 验证签名是否合法

代码来自项目Californiumnn，类org.eclipse.californium.scandium.dtls.CertificateVerify：

``` java
/**
  * Tries to verify the client's signature contained in the CertificateVerify
  * message.
  * 
  * @param clientPublicKey
  *            the client's public key.
  * @param handshakeMessages
  *            the handshake messages exchanged so far.
  * @throws HandshakeException if the signature could not be verified.
  */
public void verifySignature(PublicKey clientPublicKey, byte[] handshakeMessages) throws HandshakeException {
  boolean verified = false;
  try {
    Signature signature = Signature.getInstance(signatureAndHashAlgorithm.jcaName());
    signature.initVerify(clientPublicKey);

    signature.update(handshakeMessages);

    verified = signature.verify(signatureBytes);

  } catch (SignatureException | InvalidKeyException | NoSuchAlgorithmException e) {
    LOGGER.log(Level.SEVERE,"Could not verify the client's signature.", e);
  }
  
  if (!verified) {
    String message = "The client's CertificateVerify message could not be verified.";
    AlertMessage alert = new AlertMessage(AlertLevel.FATAL, AlertDescription.HANDSHAKE_FAILURE, getPeer());
    throw new HandshakeException(message, alert);
  }
}
```

### 验证Trust Chain

```java
/**
  * Validates the X.509 certificate chain provided by the the peer as part of this message.
  * 
  * This method checks
  * <ol>
  * <li>that each certificate's issuer DN equals the subject DN of the next certiciate in the chain</li>
  * <li>that each certificate is currently valid according to its validity period</li>
  * <li>that the chain is rooted at a trusted CA</li>
  * </ol>
  * 
  * @param certificate certificate to be verified 
  * @param trustedCertificates the list of trusted root CAs
  * 
  * @throws HandshakeException if any of the checks fails
  */
public void verifyCertificate(X509Certificate certificate, X509Certificate[] trustedCertificates) throws HandshakeException {
  CertificateFactory factory = CertificateFactory.getInstance("X.509");
  CertPath certPath = factory.generateCertPath(Arrays.asList(certificate));

  Set<TrustAnchor> trustAnchors = getTrustAnchors(trustedCertificates);

  try {
    PKIXParameters params = new PKIXParameters(trustAnchors);
    // TODO: implement alternative means of revocation checking
    params.setRevocationEnabled(false);

    CertPathValidator validator = CertPathValidator.getInstance("PKIX");
    validator.validate(certPath, params);

  } catch (GeneralSecurityException e) {
    if (LOGGER.isLoggable(Level.FINEST)) {
      LOGGER.log(Level.FINEST, "Certificate validation failed", e);
    } else if (LOGGER.isLoggable(Level.FINE)) {
      LOGGER.log(Level.FINE, "Certificate validation failed due to {0}", e.getMessage());
    }
    AlertMessage alert = new AlertMessage(AlertLevel.FATAL, AlertDescription.BAD_CERTIFICATE, getPeer());
    throw new HandshakeException("Certificate chain could not be validated", alert);
  }
}

private static Set<TrustAnchor> getTrustAnchors(X509Certificate[] trustedCertificates) {
  Set<TrustAnchor> result = new HashSet<>();
  if (trustedCertificates != null) {
    for (X509Certificate cert : trustedCertificates) {
      result.add(new TrustAnchor((X509Certificate) cert, null));
    }
  }
  return result;
}
```

## 结尾

到现在为止，本文涉及的只是安全甚至证书领域的一鳞半爪，也只是一些简单的内容。要深刻理解这方面的内容，必须深入实际的应用。
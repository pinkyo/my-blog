# 在Spring-boot项目中自定一个Validation Annotaion

在`javax.validation:validation-api`中`javax.validation.constraints`定义诸如`Null`, `Max`, `Min`这样的这样的Annotation。在大多数情况下，这些Annotation已经够用了，但是我们还是要了解一下如何自己定义一个Annotaion来实现自己想要的验证方式。

## 定义一个自己的验证的Annotaion

```java
package my.pinkyo.demo.validation;

import org.hibernate.validator.constraints.NotBlank;

import javax.validation.Constraint;
import javax.validation.Payload;
import javax.validation.constraints.NotNull;
import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Target( { ElementType.METHOD, ElementType.FIELD })
@Retention(RUNTIME)
@Constraint(validatedBy = AlphaDigitOnlyValidator.class)
@Documented
@NotBlank
public @interface AlphaDigitOnly {
    String message() default "can only have alpha digit.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};

    /**
     * Defines several {@link NotNull} annotations on the same element.
     *
     * @see NotNull
     */
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @Documented
    @interface List {

        NotNull[] value();
    }
}
```

定义完Annotation就要实现验证这个字段的方式，就是完成`my.pinkyo.demo.validation.AlphaDigitOnlyValidator`，代码如下：

```java
package my.pinkyo.demo.validation;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.Objects;

public class AlphaDigitOnlyValidator implements ConstraintValidator<AlphaDigitOnly, CharSequence> {
    @Override
    public void initialize(AlphaDigitOnly constraintAnnotation) {
        // do nothing.
    }

    @Override
    public boolean isValid(CharSequence value, ConstraintValidatorContext context) {
        if (Objects.nonNull(value)
            && !value.toString().matches("[a-zA-Z0-9]*")) {
            return false;
        }
        return true;
    }
}
```

在这个验证中，要求被验证的字段是字符串，不为且只能由`字母数字`组成。

一般情况下，到这一步我们定义自已的验证Annotation事情已经做完了。但是如果你的这个Annotation要和group一起使用就要再多点儿考虑了。

## 定义自己的Group

在Hibernate-Validator中，Group必须为interface，所以我们可以定义一个Group如下：

```java
package my.pinkyo.demo.valiation;

public interface Insert {
}
```

这样我们就可以在Modal里面使用这个Group了，例如：

```java
public class User {
    ...
    @AlphaDigitOnly(groups = {Update.class, Insert.class})
    private String name;
    ...
}
```

但是我们要考虑一个惯例：如果一个Annotation没有设置Group，则应该对于所有的Group都要进行字段的验证。但是，如上的定义完以后的效果却是，`如果不定义Group，则对所有的Group都不生效，与惯例不一样，要特别注意`。
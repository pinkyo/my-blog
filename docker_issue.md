# Error response from daemon: Driver devicemapper failed to remove root filesystem e2d5e867c98dac2d71e5fc6e61eecc4e1dad6882ac1d4f580f4702b50d698c5e: remove /var/lib/docker/devicemapper/mnt/41a6d174876cad7213cdf814229846352f342af779c52b7e66945288e34ed2e2: device or resource busy

由返回的错误日志，可以知道出现这种错误的原因是文件被多个进程挂载，所以docker rm无法删除相应的文件。

```
[root@bay208v6 ~]#]# find /proc/*/mounts | xargs grep -E "e2d5e867c9"
/proc/17259/mounts:shm /var/lib/docker/containers/e2d5e867c98dac2d71e5fc6e61eecc4e1dad6882ac1d4f580f4702b50d698c5e/shm tmpfs rw,nosuid,nodev,noexec,relatime,size=65536k 0 0
[root@bay208v6 ~]# ps -eaf | grep 17259
root     17259     1  0 15:33 ?        00:00:00 /usr/lib/systemd/systemd-machined
root     21692 24601  0 15:38 pts/2    00:00:00 grep --color=auto 17259
```

至此，可以知道是systemd-machined这个进程出的问题。执行:

```
[root@bay208v6 ~] kill -9 17259
[root@bay208v6 ~] docker system prune
```

这样，docker环境恢复正常。
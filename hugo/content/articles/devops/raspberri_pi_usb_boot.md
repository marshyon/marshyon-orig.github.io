

### 32Gig SSD:

```
pi@raspberrypi:~ $ mkdir tmp
pi@raspberrypi:~ $ cd tmp
pi@raspberrypi:~/tmp $ ls
pi@raspberrypi:~/tmp $ sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 33.5763 s, 32.0 MB/s
pi@raspberrypi:~/tmp $ dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 2.54035 s, 423 MB/s
pi@raspberrypi:~/tmp $ sudo /sbin/sysctl -w vm.drop_caches=3
vm.drop_caches = 3
pi@raspberrypi:~/tmp $ dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 23.561 s, 45.6 MB/s
```


### 1TB Samsung spinning rust

```
pi@raspberrypi:~ $ mkdir tmp
pi@raspberrypi:~ $ cd tmp
pi@raspberrypi:~/tmp $ ls
pi@raspberrypi:~/tmp $ sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 9.64114 s, 111 MB/s
pi@raspberrypi:~/tmp $ dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 1.91001 s, 562 MB/s
pi@raspberrypi:~/tmp $ sudo /sbin/sysctl -w vm.drop_caches=3
vm.drop_caches = 3
pi@raspberrypi:~/tmp $ dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 9.07368 s, 118 MB/s
```

### 240 GB SSD

```
pi@raspberrypi:~ $ mkdir tmp
pi@raspberrypi:~ $ cd tmp/
pi@raspberrypi:~/tmp $ sync; dd if=/dev/zero of=tempfile bs=1M count=1024; sync
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 4.87453 s, 220 MB/s
pi@raspberrypi:~/tmp $ dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 1.8912 s, 568 MB/s
pi@raspberrypi:~/tmp $ sudo /sbin/sysctl -w vm.drop_caches=3
vm.drop_caches = 3
pi@raspberrypi:~/tmp $ dd if=tempfile of=/dev/null bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 3.18565 s, 337 MB/s
```




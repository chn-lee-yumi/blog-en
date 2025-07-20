---
title: "The Difference Between Buffer and Cache in Linux 'free' Command"
description: "In Linux, 'buffer' refers to block device cache, while 'cache' refers to filesystem cache."
date: 2023-01-13T17:35:40+08:00
categories:
  - Learning
tags:
  - Linux
---

## Conclusion

* **Buffer** refers to cache for **block devices**.
* **Cache** refers to cache for the **filesystem**.

These caches include both read and write operations.

## Experimental Proof

First, check the values of buffer and cache:

```shell
root@localhost:~# cat /proc/meminfo | grep -E 'Buffers|Cache'
Buffers:          238024 kB
Cached:          1041304 kB
SwapCached:        88328 kB
```

Then use `dd` to read 100MB from a block device. As shown in the result, the buffer increased by 100MB.

```shell
root@localhost:~# dd if=/dev/vda of=/dev/null bs=1M count=100
记录了100+0 的读入
记录了100+0 的写出
104857600字节（105 MB，100 MiB）已复制，0.334603 s，313 MB/s
root@localhost:~# cat /proc/meminfo |grep -E 'Buffers|Cache'
Buffers:          341480 kB
Cached:          1042240 kB
SwapCached:        88328 kB
```

Reading the same block device again shows a significantly higher speed, indicating the data was retrieved directly from the buffer.

```shell
root@localhost:~# dd if=/dev/vda of=/dev/null bs=1M count=100
记录了100+0 的读入
记录了100+0 的写出
104857600字节（105 MB，100 MiB）已复制，0.0202984 s，5.2 GB/s
```

Next, read a 33MB file using `cat`. As expected, the cache increased by 33MB.

```shell
root@localhost:~# ls -lh /var/log
......
-rw-r-----  1 xrdp        adm              33M 11月  6 15:26 xrdp.log
......
root@localhost:~# cat /var/log/xrdp.log > /dev/null
root@localhost:~# cat /proc/meminfo |grep -E 'Buffers|Cache'
Buffers:          341648 kB
Cached:          1075616 kB
SwapCached:        88328 kB
```

Use `dd` to write a 100MB file, then execute `sync`, while observing `vmstat`.

```shell
root@localhost:~# dd if=/dev/zero of=test.img bs=1M count=100
记录了100+0 的读入
记录了100+0 的写出
104857600字节（105 MB，100 MiB）已复制，0.0703571 s，1.5 GB/s
root@localhost:~# sync
```

The `dd` command writes data very quickly, and the cache increased by 100MB, but the data wasn’t immediately written to disk. It was only after executing `sync` that the data was flushed to disk.

```shell
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b 交换 空闲 缓冲 缓存   si   so    bi    bo   in   cs us sy id wa st
 0  0 214316 281576 338164 1019048    0    0     1    14    2    1  0  1 99  0  0
 0  0 214316 281416 338164 1019060    0    0     0     0  961  286  0  0 100  0  0
 0  0 214316 280808 338164 1019072    0    0     0     0 1111  294  0  1 99  0  0
 0  0 214316 280232 338164 1019080    0    0     0     0  951  294  1  1 98  0  0
 0  0 214316 280040 338164 1019088    0    0     0     0  765  279  1  0 99  0  0
 0  0 214316 176712 338164 1121500    0    0     0    44  988  352  0  7 93  0  0
 0  0 214316 176328 338164 1121508    0    0     0   300 1043  308  0  1 99  0  0
 0  0 214316 176392 338164 1121520    0    0     0     8  960  314  1  0 99  0  0
 0  0 214316 176296 338164 1121524    0    0     0     0  843  275  0  0 100  0  0
 0  0 214316 175944 338164 1121532    0    0     0   124  682  355  0  1 98  1  0
 0  0 214316 175616 338164 1121540    0    0     0 102760 1015  508  1  2 76 21  0
 0  0 214316 175752 338164 1121556    0    0     0     0  971  343  0  1 99  0  0
```

While writing to a block device with `dd`, the buffer also increases. Due to the lack of a proper test environment at hand, this part is not demonstrated (though it's known that both buffer and cache can grow).
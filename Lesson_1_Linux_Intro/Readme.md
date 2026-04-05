**Занятие 1. С чего начинается Linux**


```
user@ubuntu-server:~$ uname -r
6.8.0-107-generic
```

```
user@ubuntu-server:~$ uname -p
x86_64
```

```
user@ubuntu-server:~$ mkdir kernel && cd kernel
user@ubuntu-server:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
--2026-04-05 06:58:48--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.76, 185.125.189.74, 185.125.189.75
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3694072 (3.5M) [application/x-debian-package]
Saving to: ‘linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’

linux-headers-6.13.2-061302-gen 100%[====================================================>]   3.52M  6.13MB/s    in 0.6s

2026-04-05 06:58:49 (6.13 MB/s) - ‘linux-headers-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’ saved [3694072/3694072]

user@ubuntu-server:~/kernel$  wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb
--2026-04-05 06:59:24--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.75, 185.125.189.76, 185.125.189.74
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.75|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 13875326 (13M) [application/x-debian-package]
Saving to: ‘linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb’

linux-headers-6.13.2-061302_6.1 100%[====================================================>]  13.23M  15.4MB/s    in 0.9s

2026-04-05 06:59:25 (15.4 MB/s) - ‘linux-headers-6.13.2-061302_6.13.2-061302.202502081010_all.deb’ saved [13875326/13875326]

user@ubuntu-server:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
--2026-04-05 06:59:44--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.74, 185.125.189.75, 185.125.189.76
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.74|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15677632 (15M) [application/x-debian-package]
Saving to: ‘linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’

linux-image-unsigned-6.13.2-061 100%[====================================================>]  14.95M  16.6MB/s    in 0.9s

2026-04-05 06:59:45 (16.6 MB/s) - ‘linux-image-unsigned-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’ saved [15677632/15677632]

user@ubuntu-server:~/kernel$ wget https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
--2026-04-05 06:59:59--  https://kernel.ubuntu.com/mainline/v6.13.2/amd64/linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb
Resolving kernel.ubuntu.com (kernel.ubuntu.com)... 185.125.189.76, 185.125.189.74, 185.125.189.75
Connecting to kernel.ubuntu.com (kernel.ubuntu.com)|185.125.189.76|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 184576192 (176M) [application/x-debian-package]
Saving to: ‘linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’

linux-modules-6.13.2-061302-gen 100%[====================================================>] 176.03M  7.80MB/s    in 17s

2026-04-05 07:00:17 (10.4 MB/s) - ‘linux-modules-6.13.2-061302-generic_6.13.2-061302.202502081010_amd64.deb’ saved [184576192/184576192]
```



# Занятие 7. Управление пакетами. Дистрибьюция софта

## Задание
1) Создать свой RPM пакет (можно взять свое приложение, либо собрать, например,
Apache с определенными опциями).
2) Создать свой репозиторий и разместить там ранее собранный RPM.

## Стенд
Виртуальная машина: AlmaLinux 9.7, 2Gb RAM, 2 CPU, 10Gb HDD.
Гипервизор: Virtual Box 7.2.8 r173730 (Qt6.8.0 on cocoa)

```
[user@localhost ~]$ hostnamectl
   Static hostname: (unset)
Transient hostname: localhost
         Icon name: computer-vm
           Chassis: vm 🖴
        Machine ID: f717e2108a9c4f78bbcc329a10646f3b
           Boot ID: 37153698f12449aeab578cb0f84ac5b2
    Virtualization: vmware
  Operating System: AlmaLinux 9.7 (Moss Jungle Cat)
       CPE OS Name: cpe:/o:almalinux:almalinux:9::baseos
            Kernel: Linux 5.14.0-611.5.1.el9_7.x86_64
      Architecture: x86-64
   Hardware Vendor: VMware, Inc.
    Hardware Model: VMware Virtual Platform
  Firmware Version: 6.00
```

## 1) Создание своего RPM пакета
Установим пакеты необходимые для выполнения задания:

```
[root@localhost user]# yum install -y wget rpmdevtools rpm-build createrepo yum-utils cmake gcc git nano
AlmaLinux 9 - AppStream                                                                      6.1 MB/s | 9.8 MB     00:01
AlmaLinux 9 - BaseOS                                                                         3.0 MB/s | 2.7 MB     00:00
AlmaLinux 9 - Extras                                                                          35 kB/s |  22 kB     00:00
Dependencies resolved.
=============================================================================================================================
 Package                                 Architecture        Version                            Repository              Size
=============================================================================================================================
Installing:
 cmake                                   x86_64              3.31.8-3.el9                       appstream               13 M
 createrepo_c                            x86_64              0.20.1-4.el9                       appstream               71 k
 gcc                                     x86_64              11.5.0-14.el9.alma.1               appstream               32 M
 git                                     x86_64              2.52.0-1.el9                       appstream               39 k
 nano                                    x86_64              5.6.1-7.el9                        baseos                 692 k
 rpm-build                               x86_64              4.16.1.3-40.el9                    appstream               58 k
 rpmdevtools                             noarch              9.5-1.el9                          appstream               75 k
 wget                                    x86_64              1.21.1-8.el9_4                     appstream              768 k
 yum-utils                               noarch              4.3.0-26.el9                       baseos                  34 k
Upgrading:
 bzip2-libs                              x86_64              1.0.8-11.el9                       baseos                  39 k
 dnf-plugins-core                        noarch              4.3.0-26.el9                       baseos                  35 k
 elfutils-debuginfod-client              x86_64              0.194-1.el9.alma.1                 baseos                  43 k
 elfutils-libelf                         x86_64              0.194-1.el9.alma.1                 baseos                 201 k
 elfutils-libs                           x86_64              0.194-1.el9.alma.1                 baseos                 268 k
 glibc                                   x86_64              2.34-266.el9_8                     baseos                 2.0 M
 glibc-common                            x86_64              2.34-266.el9_8                     baseos                 300 k
 glibc-gconv-extra                       x86_64              2.34-266.el9_8                     baseos                 1.6 M
 glibc-minimal-langpack                  x86_64              2.34-266.el9_8                     baseos                  27 k
 libgcc                                  x86_64              11.5.0-14.el9.alma.1               baseos                  84 k
 libgomp                                 x86_64              11.5.0-14.el9.alma.1               baseos                 256 k
 python3-dnf-plugins-core                noarch              4.3.0-26.el9                       baseos                 245 k
 python3-rpm                             x86_64              4.16.1.3-40.el9                    baseos                  64 k
 rpm                                     x86_64              4.16.1.3-40.el9                    baseos                 484 k
 rpm-build-libs                          x86_64              4.16.1.3-40.el9                    baseos                  88 k
 rpm-libs                                x86_64              4.16.1.3-40.el9                    baseos                 307 k
 rpm-plugin-audit                        x86_64              4.16.1.3-40.el9                    baseos                  15 k
 rpm-plugin-selinux                      x86_64              4.16.1.3-40.el9                    baseos                  15 k
 rpm-sign-libs                           x86_64              4.16.1.3-40.el9                    baseos                  20 k
Installing dependencies:
 annobin                                 x86_64              12.98-2.el9                        appstream              1.1 M
 bzip2                                   x86_64              1.0.8-11.el9                       baseos                  51 k
 cmake-data                              noarch              3.31.8-3.el9                       appstream              1.9 M
 cmake-filesystem                        x86_64              3.31.8-3.el9                       appstream               11 k
 cmake-rpm-macros                        noarch              3.31.8-3.el9                       appstream               10 k
 cpp                                     x86_64              11.5.0-14.el9.alma.1               appstream               11 M
 createrepo_c-libs                       x86_64              0.20.1-4.el9                       appstream               97 k
 debugedit                               x86_64              5.0-11.el9                         appstream               76 k
 dwz                                     x86_64              0.16-1.el9                         appstream              133 k
 ed                                      x86_64              1.14.2-12.el9                      baseos                  74 k
 efi-srpm-macros                         noarch              6-4.el9                            appstream               21 k
 elfutils                                x86_64              0.194-1.el9.alma.1                 baseos                 592 k
 emacs-filesystem                        noarch              1:27.2-18.el9                      appstream              8.2 k
 fonts-srpm-macros                       noarch              1:2.0.5-7.el9.1                    appstream               27 k
 gcc-plugin-annobin                      x86_64              11.5.0-14.el9.alma.1               appstream               36 k
 gdb-minimal                             x86_64              16.3-3.el9                         appstream              4.4 M
 ghc-srpm-macros                         noarch              1.5.0-6.el9                        appstream              7.8 k
 git-core                                x86_64              2.52.0-1.el9                       appstream              5.0 M
 git-core-doc                            noarch              2.52.0-1.el9                       appstream              2.8 M
 glibc-devel                             x86_64              2.34-266.el9_8                     appstream               37 k
 glibc-headers                           x86_64              2.34-266.el9_8                     appstream              443 k
 go-srpm-macros                          noarch              3.8.1-1.el9                        appstream               27 k
 info                                    x86_64              6.7-15.el9                         baseos                 224 k
 kernel-headers                          x86_64              5.14.0-687.5.3.el9_8               appstream              2.5 M
 kernel-srpm-macros                      noarch              1.0-14.el9                         appstream               14 k
 libmpc                                  x86_64              1.2.1-4.el9                        appstream               61 k
 libpkgconf                              x86_64              1.7.3-10.el9                       baseos                  35 k
 libuv                                   x86_64              1:1.42.0-2.el9_4                   appstream              146 k
 libxcrypt-devel                         x86_64              4.4.18-3.el9                       appstream               28 k
 llvm-filesystem                         x86_64              21.1.8-2.el9                       appstream               13 k
 llvm-libs                               x86_64              21.1.8-2.el9                       appstream               31 M
 lua-srpm-macros                         noarch              1-6.el9                            appstream              8.4 k
 make                                    x86_64              1:4.3-8.el9                        baseos                 530 k
 ocaml-srpm-macros                       noarch              6-6.el9                            appstream              7.7 k
 openblas-srpm-macros                    noarch              2-11.el9                           appstream              7.3 k
 patch                                   x86_64              2.7.6-16.el9                       appstream              127 k
 perl-AutoLoader                         noarch              5.74-481.1.el9_6                   appstream               20 k
 perl-B                                  x86_64              1.80-481.1.el9_6                   appstream              177 k
 perl-Carp                               noarch              1.50-460.el9                       appstream               29 k
 perl-Class-Struct                       noarch              0.66-481.1.el9_6                   appstream               21 k
 perl-Data-Dumper                        x86_64              2.174-462.el9                      appstream               55 k
 perl-Digest                             noarch              1.19-4.el9                         appstream               25 k
 perl-Digest-MD5                         x86_64              2.58-4.el9                         appstream               36 k
 perl-DynaLoader                         x86_64              1.47-481.1.el9_6                   appstream               24 k
 perl-Encode                             x86_64              4:3.08-462.el9                     appstream              1.7 M
 perl-Errno                              x86_64              1.30-481.1.el9_6                   appstream               13 k
 perl-Error                              noarch              1:0.17029-7.el9                    appstream               41 k
 perl-Exporter                           noarch              5.74-461.el9                       appstream               31 k
 perl-Fcntl                              x86_64              1.13-481.1.el9_6                   appstream               19 k
 perl-File-Basename                      noarch              2.85-481.1.el9_6                   appstream               16 k
 perl-File-Path                          noarch              2.18-4.el9                         appstream               35 k
 perl-File-Temp                          noarch              1:0.231.100-4.el9                  appstream               59 k
 perl-File-stat                          noarch              1.09-481.1.el9_6                   appstream               16 k
 perl-FileHandle                         noarch              2.03-481.1.el9_6                   appstream               14 k
 perl-Getopt-Long                        noarch              1:2.52-4.el9                       appstream               59 k
 perl-Getopt-Std                         noarch              1.12-481.1.el9_6                   appstream               14 k
 perl-Git                                noarch              2.52.0-1.el9                       appstream               37 k
 perl-HTTP-Tiny                          noarch              0.076-462.el9                      appstream               53 k
 perl-IO                                 x86_64              1.43-481.1.el9_6                   appstream               85 k
 perl-IO-Socket-IP                       noarch              0.41-5.el9                         appstream               42 k
 perl-IO-Socket-SSL                      noarch              2.073-2.el9                        appstream              214 k
 perl-IPC-Open3                          noarch              1.21-481.1.el9_6                   appstream               21 k
 perl-MIME-Base64                        x86_64              3.16-4.el9                         appstream               30 k
 perl-Mozilla-CA                         noarch              20200520-6.el9                     appstream               12 k
 perl-Net-SSLeay                         x86_64              1.94-3.el9                         appstream              391 k
 perl-POSIX                              x86_64              1.94-481.1.el9_6                   appstream               95 k
 perl-PathTools                          x86_64              3.78-461.el9                       appstream               85 k
 perl-Pod-Escapes                        noarch              1:1.07-460.el9                     appstream               20 k
 perl-Pod-Perldoc                        noarch              3.28.01-461.el9                    appstream               83 k
 perl-Pod-Simple                         noarch              1:3.42-4.el9                       appstream              215 k
 perl-Pod-Usage                          noarch              4:2.01-4.el9                       appstream               40 k
 perl-Scalar-List-Utils                  x86_64              4:1.56-462.el9                     appstream               70 k
 perl-SelectSaver                        noarch              1.02-481.1.el9_6                   appstream               10 k
 perl-Socket                             x86_64              4:2.031-4.el9                      appstream               54 k
 perl-Storable                           x86_64              1:3.21-460.el9                     appstream               95 k
 perl-Symbol                             noarch              1.08-481.1.el9_6                   appstream               13 k
 perl-Term-ANSIColor                     noarch              5.01-461.el9                       appstream               48 k
 perl-Term-Cap                           noarch              1.17-460.el9                       appstream               22 k
 perl-TermReadKey                        x86_64              2.38-11.el9                        appstream               36 k
 perl-Text-ParseWords                    noarch              3.30-460.el9                       appstream               16 k
 perl-Text-Tabs+Wrap                     noarch              2013.0523-460.el9                  appstream               22 k
 perl-Time-Local                         noarch              2:1.300-7.el9                      appstream               33 k
 perl-URI                                noarch              5.09-3.el9                         appstream              108 k
 perl-base                               noarch              2.27-481.1.el9_6                   appstream               15 k
 perl-constant                           noarch              1.33-461.el9                       appstream               23 k
 perl-if                                 noarch              0.60.800-481.1.el9_6               appstream               12 k
 perl-interpreter                        x86_64              4:5.32.1-481.1.el9_6               appstream               69 k
 perl-lib                                x86_64              0.65-481.1.el9_6                   appstream               13 k
 perl-libnet                             noarch              3.13-4.el9                         appstream              125 k
 perl-libs                               x86_64              4:5.32.1-481.1.el9_6               appstream              2.0 M
 perl-mro                                x86_64              1.23-481.1.el9_6                   appstream               27 k
 perl-overload                           noarch              1.31-481.1.el9_6                   appstream               44 k
 perl-overloading                        noarch              0.02-481.1.el9_6                   appstream               11 k
 perl-parent                             noarch              1:0.238-460.el9                    appstream               14 k
 perl-podlators                          noarch              1:4.14-460.el9                     appstream              111 k
 perl-srpm-macros                        noarch              1-41.el9                           appstream              8.1 k
 perl-subs                               noarch              1.03-481.1.el9_6                   appstream               10 k
 perl-vars                               noarch              1.05-481.1.el9_6                   appstream               11 k
 pkgconf                                 x86_64              1.7.3-10.el9                       baseos                  40 k
 pkgconf-m4                              noarch              1.7.3-10.el9                       baseos                  14 k
 pkgconf-pkg-config                      x86_64              1.7.3-10.el9                       baseos                 9.9 k
 pyproject-srpm-macros                   noarch              1.18.5-1.el9                       appstream               12 k
 python-srpm-macros                      noarch              3.9-54.el9                         appstream               16 k
 python3-argcomplete                     noarch              1.12.0-5.el9                       appstream               61 k
 python3-chardet                         noarch              4.0.0-5.el9                        baseos                 209 k
 python3-idna                            noarch              2.10-7.el9_4.1                     baseos                  96 k
 python3-pysocks                         noarch              1.7.1-12.el9                       baseos                  34 k
 python3-requests                        noarch              2.25.1-10.el9_6                    baseos                 115 k
 python3-urllib3                         noarch              1.26.5-6.el9_7.1                   baseos                 191 k
 qt5-srpm-macros                         noarch              5.15.9-1.el9                       appstream              7.8 k
 redhat-rpm-config                       noarch              210-1.el9.alma.1                   appstream               64 k
 rust-srpm-macros                        noarch              17-4.el9                           appstream              9.2 k
 tar                                     x86_64              2:1.34-11.el9                      baseos                 876 k
 unzip                                   x86_64              6.0-60.el9                         baseos                 180 k
 vim-filesystem                          noarch              2:8.2.2637-26.el9_8.4              baseos                  10 k
 zip                                     x86_64              3.0-35.el9                         baseos                 263 k
 zstd                                    x86_64              1.5.5-1.el9                        baseos                 462 k
Installing weak dependencies:
 perl-NDBM_File                          x86_64              1.15-481.1.el9_6                   appstream               21 k

Transaction Summary
=============================================================================================================================
Install  127 Packages
Upgrade   19 Packages

Total download size: 125 M
Downloading Packages:
(1/146): annobin-12.98-2.el9.x86_64.rpm                                                      4.6 MB/s | 1.1 MB     00:00
(2/146): cmake-data-3.31.8-3.el9.noarch.rpm                                                  6.9 MB/s | 1.9 MB     00:00
(3/146): cmake-filesystem-3.31.8-3.el9.x86_64.rpm                                            239 kB/s |  11 kB     00:00
(4/146): cmake-rpm-macros-3.31.8-3.el9.noarch.rpm                                            795 kB/s |  10 kB     00:00
(5/146): createrepo_c-0.20.1-4.el9.x86_64.rpm                                                2.3 MB/s |  71 kB     00:00
(6/146): createrepo_c-libs-0.20.1-4.el9.x86_64.rpm                                           2.4 MB/s |  97 kB     00:00
(7/146): debugedit-5.0-11.el9.x86_64.rpm                                                     1.2 MB/s |  76 kB     00:00
(8/146): dwz-0.16-1.el9.x86_64.rpm                                                           1.0 MB/s | 133 kB     00:00
(9/146): efi-srpm-macros-6-4.el9.noarch.rpm                                                  642 kB/s |  21 kB     00:00
(10/146): emacs-filesystem-27.2-18.el9.noarch.rpm                                            325 kB/s | 8.2 kB     00:00
(11/146): fonts-srpm-macros-2.0.5-7.el9.1.noarch.rpm                                         506 kB/s |  27 kB     00:00
(12/146): cpp-11.5.0-14.el9.alma.1.x86_64.rpm                                                6.3 MB/s |  11 MB     00:01
(13/146): gcc-plugin-annobin-11.5.0-14.el9.alma.1.x86_64.rpm                                 773 kB/s |  36 kB     00:00
(14/146): cmake-3.31.8-3.el9.x86_64.rpm                                                      5.2 MB/s |  13 MB     00:02
(15/146): ghc-srpm-macros-1.5.0-6.el9.noarch.rpm                                             129 kB/s | 7.8 kB     00:00
(16/146): git-2.52.0-1.el9.x86_64.rpm                                                        795 kB/s |  39 kB     00:00
(17/146): gdb-minimal-16.3-3.el9.x86_64.rpm                                                  4.9 MB/s | 4.4 MB     00:00
(18/146): git-core-2.52.0-1.el9.x86_64.rpm                                                   6.3 MB/s | 5.0 MB     00:00
(19/146): git-core-doc-2.52.0-1.el9.noarch.rpm                                               4.8 MB/s | 2.8 MB     00:00
(20/146): glibc-devel-2.34-266.el9_8.x86_64.rpm                                              757 kB/s |  37 kB     00:00
(21/146): go-srpm-macros-3.8.1-1.el9.noarch.rpm                                              660 kB/s |  27 kB     00:00
(22/146): glibc-headers-2.34-266.el9_8.x86_64.rpm                                            5.8 MB/s | 443 kB     00:00
(23/146): kernel-srpm-macros-1.0-14.el9.noarch.rpm                                           326 kB/s |  14 kB     00:00
(24/146): libmpc-1.2.1-4.el9.x86_64.rpm                                                      2.5 MB/s |  61 kB     00:00
(25/146): libuv-1.42.0-2.el9_4.x86_64.rpm                                                    1.9 MB/s | 146 kB     00:00
(26/146): libxcrypt-devel-4.4.18-3.el9.x86_64.rpm                                            1.2 MB/s |  28 kB     00:00
(27/146): llvm-filesystem-21.1.8-2.el9.x86_64.rpm                                            466 kB/s |  13 kB     00:00
(28/146): kernel-headers-5.14.0-687.5.3.el9_8.x86_64.rpm                                     7.4 MB/s | 2.5 MB     00:00
(29/146): lua-srpm-macros-1-6.el9.noarch.rpm                                                 279 kB/s | 8.4 kB     00:00
(30/146): ocaml-srpm-macros-6-6.el9.noarch.rpm                                               312 kB/s | 7.7 kB     00:00
(31/146): openblas-srpm-macros-2-11.el9.noarch.rpm                                           237 kB/s | 7.3 kB     00:00
(32/146): patch-2.7.6-16.el9.x86_64.rpm                                                      2.4 MB/s | 127 kB     00:00
(33/146): perl-AutoLoader-5.74-481.1.el9_6.noarch.rpm                                        589 kB/s |  20 kB     00:00
(34/146): gcc-11.5.0-14.el9.alma.1.x86_64.rpm                                                9.2 MB/s |  32 MB     00:03
(35/146): perl-B-1.80-481.1.el9_6.x86_64.rpm                                                 1.5 MB/s | 177 kB     00:00
(36/146): perl-Carp-1.50-460.el9.noarch.rpm                                                  2.1 MB/s |  29 kB     00:00
(37/146): perl-Data-Dumper-2.174-462.el9.x86_64.rpm                                          2.0 MB/s |  55 kB     00:00
(38/146): perl-Class-Struct-0.66-481.1.el9_6.noarch.rpm                                      424 kB/s |  21 kB     00:00
(39/146): perl-Digest-1.19-4.el9.noarch.rpm                                                  2.0 MB/s |  25 kB     00:00
(40/146): perl-DynaLoader-1.47-481.1.el9_6.x86_64.rpm                                        858 kB/s |  24 kB     00:00
(41/146): perl-Digest-MD5-2.58-4.el9.x86_64.rpm                                              784 kB/s |  36 kB     00:00
(42/146): perl-Errno-1.30-481.1.el9_6.x86_64.rpm                                             293 kB/s |  13 kB     00:00
(43/146): perl-Error-0.17029-7.el9.noarch.rpm                                                648 kB/s |  41 kB     00:00
(44/146): perl-Exporter-5.74-461.el9.noarch.rpm                                              1.7 MB/s |  31 kB     00:00
(45/146): perl-Encode-3.08-462.el9.x86_64.rpm                                                9.7 MB/s | 1.7 MB     00:00
(46/146): perl-File-Basename-2.85-481.1.el9_6.noarch.rpm                                     1.4 MB/s |  16 kB     00:00
(47/146): perl-Fcntl-1.13-481.1.el9_6.x86_64.rpm                                             378 kB/s |  19 kB     00:00
(48/146): perl-File-Path-2.18-4.el9.noarch.rpm                                               2.1 MB/s |  35 kB     00:00
(49/146): perl-File-Temp-0.231.100-4.el9.noarch.rpm                                          2.4 MB/s |  59 kB     00:00
(50/146): perl-File-stat-1.09-481.1.el9_6.noarch.rpm                                         543 kB/s |  16 kB     00:00
(51/146): perl-Getopt-Long-2.52-4.el9.noarch.rpm                                             4.7 MB/s |  59 kB     00:00
(52/146): perl-Getopt-Std-1.12-481.1.el9_6.noarch.rpm                                        1.4 MB/s |  14 kB     00:00
(53/146): perl-Git-2.52.0-1.el9.noarch.rpm                                                   3.5 MB/s |  37 kB     00:00
(54/146): perl-HTTP-Tiny-0.076-462.el9.noarch.rpm                                            2.8 MB/s |  53 kB     00:00
(55/146): perl-IO-1.43-481.1.el9_6.x86_64.rpm                                                6.4 MB/s |  85 kB     00:00
(56/146): perl-FileHandle-2.03-481.1.el9_6.noarch.rpm                                        154 kB/s |  14 kB     00:00
(57/146): perl-IO-Socket-IP-0.41-5.el9.noarch.rpm                                            3.6 MB/s |  42 kB     00:00
(58/146): perl-IO-Socket-SSL-2.073-2.el9.noarch.rpm                                          4.9 MB/s | 214 kB     00:00
(59/146): perl-MIME-Base64-3.16-4.el9.x86_64.rpm                                             2.1 MB/s |  30 kB     00:00
(60/146): perl-Mozilla-CA-20200520-6.el9.noarch.rpm                                          691 kB/s |  12 kB     00:00
(61/146): perl-NDBM_File-1.15-481.1.el9_6.x86_64.rpm                                         1.0 MB/s |  21 kB     00:00
(62/146): perl-IPC-Open3-1.21-481.1.el9_6.noarch.rpm                                         233 kB/s |  21 kB     00:00
(63/146): perl-Net-SSLeay-1.94-3.el9.x86_64.rpm                                              5.7 MB/s | 391 kB     00:00
(64/146): perl-PathTools-3.78-461.el9.x86_64.rpm                                             3.2 MB/s |  85 kB     00:00
(65/146): perl-POSIX-1.94-481.1.el9_6.x86_64.rpm                                             931 kB/s |  95 kB     00:00
(66/146): perl-Pod-Perldoc-3.28.01-461.el9.noarch.rpm                                        7.0 MB/s |  83 kB     00:00
(67/146): perl-Pod-Escapes-1.07-460.el9.noarch.rpm                                           451 kB/s |  20 kB     00:00
(68/146): perl-Pod-Simple-3.42-4.el9.noarch.rpm                                              8.5 MB/s | 215 kB     00:00
(69/146): perl-Pod-Usage-2.01-4.el9.noarch.rpm                                               2.8 MB/s |  40 kB     00:00
(70/146): perl-SelectSaver-1.02-481.1.el9_6.noarch.rpm                                       714 kB/s |  10 kB     00:00
(71/146): perl-Scalar-List-Utils-1.56-462.el9.x86_64.rpm                                     2.0 MB/s |  70 kB     00:00
(72/146): perl-Storable-3.21-460.el9.x86_64.rpm                                              3.6 MB/s |  95 kB     00:00
(73/146): perl-Socket-2.031-4.el9.x86_64.rpm                                                 1.6 MB/s |  54 kB     00:00
(74/146): perl-Term-ANSIColor-5.01-461.el9.noarch.rpm                                        1.0 MB/s |  48 kB     00:00
(75/146): perl-Term-Cap-1.17-460.el9.noarch.rpm                                              1.6 MB/s |  22 kB     00:00
(76/146): perl-Symbol-1.08-481.1.el9_6.noarch.rpm                                            175 kB/s |  13 kB     00:00
(77/146): perl-Text-ParseWords-3.30-460.el9.noarch.rpm                                       1.5 MB/s |  16 kB     00:00
(78/146): perl-Text-Tabs+Wrap-2013.0523-460.el9.noarch.rpm                                   1.8 MB/s |  22 kB     00:00
(79/146): perl-TermReadKey-2.38-11.el9.x86_64.rpm                                            851 kB/s |  36 kB     00:00
(80/146): perl-Time-Local-1.300-7.el9.noarch.rpm                                             3.0 MB/s |  33 kB     00:00
(81/146): perl-URI-5.09-3.el9.noarch.rpm                                                     2.5 MB/s | 108 kB     00:00
(82/146): perl-base-2.27-481.1.el9_6.noarch.rpm                                              297 kB/s |  15 kB     00:00
(83/146): perl-constant-1.33-461.el9.noarch.rpm                                              764 kB/s |  23 kB     00:00
(84/146): perl-if-0.60.800-481.1.el9_6.noarch.rpm                                            227 kB/s |  12 kB     00:00
(85/146): perl-interpreter-5.32.1-481.1.el9_6.x86_64.rpm                                     1.9 MB/s |  69 kB     00:00
(86/146): perl-lib-0.65-481.1.el9_6.x86_64.rpm                                               605 kB/s |  13 kB     00:00
(87/146): perl-libnet-3.13-4.el9.noarch.rpm                                                  2.6 MB/s | 125 kB     00:00
(88/146): perl-mro-1.23-481.1.el9_6.x86_64.rpm                                               704 kB/s |  27 kB     00:00
(89/146): perl-overload-1.31-481.1.el9_6.noarch.rpm                                          1.2 MB/s |  44 kB     00:00
(90/146): perl-overloading-0.02-481.1.el9_6.noarch.rpm                                       269 kB/s |  11 kB     00:00
(91/146): perl-parent-0.238-460.el9.noarch.rpm                                               423 kB/s |  14 kB     00:00
(92/146): perl-libs-5.32.1-481.1.el9_6.x86_64.rpm                                            7.6 MB/s | 2.0 MB     00:00
(93/146): perl-podlators-4.14-460.el9.noarch.rpm                                             1.0 MB/s | 111 kB     00:00
(94/146): perl-srpm-macros-1-41.el9.noarch.rpm                                               307 kB/s | 8.1 kB     00:00
(95/146): perl-vars-1.05-481.1.el9_6.noarch.rpm                                              1.0 MB/s |  11 kB     00:00
(96/146): pyproject-srpm-macros-1.18.5-1.el9.noarch.rpm                                      1.0 MB/s |  12 kB     00:00
(97/146): python-srpm-macros-3.9-54.el9.noarch.rpm                                           1.3 MB/s |  16 kB     00:00
(98/146): python3-argcomplete-1.12.0-5.el9.noarch.rpm                                        4.5 MB/s |  61 kB     00:00
(99/146): qt5-srpm-macros-5.15.9-1.el9.noarch.rpm                                            792 kB/s | 7.8 kB     00:00
(100/146): perl-subs-1.03-481.1.el9_6.noarch.rpm                                             141 kB/s |  10 kB     00:00
(101/146): redhat-rpm-config-210-1.el9.alma.1.noarch.rpm                                     4.8 MB/s |  64 kB     00:00
(102/146): rpm-build-4.16.1.3-40.el9.x86_64.rpm                                              1.6 MB/s |  58 kB     00:00
(103/146): rpmdevtools-9.5-1.el9.noarch.rpm                                                  1.5 MB/s |  75 kB     00:00
(104/146): rust-srpm-macros-17-4.el9.noarch.rpm                                              317 kB/s | 9.2 kB     00:00
(105/146): bzip2-1.0.8-11.el9.x86_64.rpm                                                     558 kB/s |  51 kB     00:00
(106/146): wget-1.21.1-8.el9_4.x86_64.rpm                                                    4.8 MB/s | 768 kB     00:00
(107/146): ed-1.14.2-12.el9.x86_64.rpm                                                       899 kB/s |  74 kB     00:00
(108/146): info-6.7-15.el9.x86_64.rpm                                                        4.1 MB/s | 224 kB     00:00
(109/146): libpkgconf-1.7.3-10.el9.x86_64.rpm                                                1.5 MB/s |  35 kB     00:00
(110/146): elfutils-0.194-1.el9.alma.1.x86_64.rpm                                            3.9 MB/s | 592 kB     00:00
(111/146): make-4.3-8.el9.x86_64.rpm                                                         3.3 MB/s | 530 kB     00:00
(112/146): nano-5.6.1-7.el9.x86_64.rpm                                                       4.4 MB/s | 692 kB     00:00
(113/146): pkgconf-1.7.3-10.el9.x86_64.rpm                                                   645 kB/s |  40 kB     00:00
(114/146): pkgconf-m4-1.7.3-10.el9.noarch.rpm                                                483 kB/s |  14 kB     00:00
(115/146): pkgconf-pkg-config-1.7.3-10.el9.x86_64.rpm                                        457 kB/s | 9.9 kB     00:00
(116/146): python3-idna-2.10-7.el9_4.1.noarch.rpm                                            2.5 MB/s |  96 kB     00:00
(117/146): python3-chardet-4.0.0-5.el9.noarch.rpm                                            3.2 MB/s | 209 kB     00:00
(118/146): python3-pysocks-1.7.1-12.el9.noarch.rpm                                           2.4 MB/s |  34 kB     00:00
(119/146): python3-requests-2.25.1-10.el9_6.noarch.rpm                                       2.4 MB/s | 115 kB     00:00
(120/146): python3-urllib3-1.26.5-6.el9_7.1.noarch.rpm                                       3.1 MB/s | 191 kB     00:00
(121/146): unzip-6.0-60.el9.x86_64.rpm                                                       2.6 MB/s | 180 kB     00:00
(122/146): vim-filesystem-8.2.2637-26.el9_8.4.noarch.rpm                                     122 kB/s |  10 kB     00:00
(123/146): yum-utils-4.3.0-26.el9.noarch.rpm                                                 2.0 MB/s |  34 kB     00:00
(124/146): tar-1.34-11.el9.x86_64.rpm                                                        4.0 MB/s | 876 kB     00:00
(125/146): zstd-1.5.5-1.el9.x86_64.rpm                                                       4.1 MB/s | 462 kB     00:00
(126/146): bzip2-libs-1.0.8-11.el9.x86_64.rpm                                                1.4 MB/s |  39 kB     00:00
(127/146): dnf-plugins-core-4.3.0-26.el9.noarch.rpm                                          851 kB/s |  35 kB     00:00
(128/146): elfutils-debuginfod-client-0.194-1.el9.alma.1.x86_64.rpm                          2.0 MB/s |  43 kB     00:00
(129/146): elfutils-libelf-0.194-1.el9.alma.1.x86_64.rpm                                     2.6 MB/s | 201 kB     00:00
(130/146): elfutils-libs-0.194-1.el9.alma.1.x86_64.rpm                                       2.9 MB/s | 268 kB     00:00
(131/146): zip-3.0-35.el9.x86_64.rpm                                                         650 kB/s | 263 kB     00:00
(132/146): glibc-common-2.34-266.el9_8.x86_64.rpm                                            4.5 MB/s | 300 kB     00:00
(133/146): glibc-2.34-266.el9_8.x86_64.rpm                                                   4.9 MB/s | 2.0 MB     00:00
(134/146): glibc-minimal-langpack-2.34-266.el9_8.x86_64.rpm                                  943 kB/s |  27 kB     00:00
(135/146): libgcc-11.5.0-14.el9.alma.1.x86_64.rpm                                            2.3 MB/s |  84 kB     00:00
(136/146): glibc-gconv-extra-2.34-266.el9_8.x86_64.rpm                                       3.8 MB/s | 1.6 MB     00:00
(137/146): python3-dnf-plugins-core-4.3.0-26.el9.noarch.rpm                                  5.9 MB/s | 245 kB     00:00
(138/146): python3-rpm-4.16.1.3-40.el9.x86_64.rpm                                            4.8 MB/s |  64 kB     00:00
(139/146): libgomp-11.5.0-14.el9.alma.1.x86_64.rpm                                           3.2 MB/s | 256 kB     00:00
(140/146): rpm-build-libs-4.16.1.3-40.el9.x86_64.rpm                                         1.9 MB/s |  88 kB     00:00
(141/146): llvm-libs-21.1.8-2.el9.x86_64.rpm                                                 8.4 MB/s |  31 MB     00:03
(142/146): rpm-libs-4.16.1.3-40.el9.x86_64.rpm                                               2.1 MB/s | 307 kB     00:00
(143/146): rpm-4.16.1.3-40.el9.x86_64.rpm                                                    2.3 MB/s | 484 kB     00:00
(144/146): rpm-sign-libs-4.16.1.3-40.el9.x86_64.rpm                                          698 kB/s |  20 kB     00:00
(145/146): rpm-plugin-selinux-4.16.1.3-40.el9.x86_64.rpm                                     485 kB/s |  15 kB     00:00
(146/146): rpm-plugin-audit-4.16.1.3-40.el9.x86_64.rpm                                       374 kB/s |  15 kB     00:00
-----------------------------------------------------------------------------------------------------------------------------
Total                                                                                         14 MB/s | 125 MB     00:08
AlmaLinux 9 - AppStream                                                                      1.5 MB/s | 3.1 kB     00:00
Importing GPG key 0xB86B3716:
 Userid     : "AlmaLinux OS 9 <packager@almalinux.org>"
 Fingerprint: BF18 AC28 7617 8908 D6E7 1267 D36C B86C B86B 3716
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-9
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                     1/1
  Upgrading        : glibc-common-2.34-266.el9_8.x86_64                                                                1/165
  Upgrading        : glibc-gconv-extra-2.34-266.el9_8.x86_64                                                           2/165
  Running scriptlet: glibc-gconv-extra-2.34-266.el9_8.x86_64                                                           2/165
  Upgrading        : glibc-minimal-langpack-2.34-266.el9_8.x86_64                                                      3/165
  Upgrading        : libgcc-11.5.0-14.el9.alma.1.x86_64                                                                4/165
  Running scriptlet: libgcc-11.5.0-14.el9.alma.1.x86_64                                                                4/165
  Running scriptlet: glibc-2.34-266.el9_8.x86_64                                                                       5/165
  Upgrading        : glibc-2.34-266.el9_8.x86_64                                                                       5/165
  Running scriptlet: glibc-2.34-266.el9_8.x86_64                                                                       5/165
  Upgrading        : bzip2-libs-1.0.8-11.el9.x86_64                                                                    6/165
  Upgrading        : rpm-4.16.1.3-40.el9.x86_64                                                                        7/165
  Upgrading        : rpm-libs-4.16.1.3-40.el9.x86_64                                                                   8/165
  Upgrading        : elfutils-libelf-0.194-1.el9.alma.1.x86_64                                                         9/165
  Upgrading        : elfutils-libs-0.194-1.el9.alma.1.x86_64                                                          10/165
  Upgrading        : elfutils-debuginfod-client-0.194-1.el9.alma.1.x86_64                                             11/165
  Installing       : gdb-minimal-16.3-3.el9.x86_64                                                                    12/165
  Installing       : elfutils-0.194-1.el9.alma.1.x86_64                                                               13/165
  Installing       : dwz-0.16-1.el9.x86_64                                                                            14/165
  Installing       : cmake-rpm-macros-3.31.8-3.el9.noarch                                                             15/165
  Installing       : unzip-6.0-60.el9.x86_64                                                                          16/165
  Installing       : git-core-2.52.0-1.el9.x86_64                                                                     17/165
  Installing       : libmpc-1.2.1-4.el9.x86_64                                                                        18/165
  Installing       : make-1:4.3-8.el9.x86_64                                                                          19/165
  Upgrading        : libgomp-11.5.0-14.el9.alma.1.x86_64                                                              20/165
  Upgrading        : rpm-build-libs-4.16.1.3-40.el9.x86_64                                                            21/165
  Installing       : python3-idna-2.10-7.el9_4.1.noarch                                                               22/165
  Installing       : emacs-filesystem-1:27.2-18.el9.noarch                                                            23/165
  Installing       : cmake-filesystem-3.31.8-3.el9.x86_64                                                             24/165
  Installing       : cpp-11.5.0-14.el9.alma.1.x86_64                                                                  25/165
  Installing       : git-core-doc-2.52.0-1.el9.noarch                                                                 26/165
  Installing       : zip-3.0-35.el9.x86_64                                                                            27/165
  Installing       : debugedit-5.0-11.el9.x86_64                                                                      28/165
  Installing       : createrepo_c-libs-0.20.1-4.el9.x86_64                                                            29/165
  Upgrading        : rpm-sign-libs-4.16.1.3-40.el9.x86_64                                                             30/165
  Upgrading        : python3-rpm-4.16.1.3-40.el9.x86_64                                                               31/165
  Installing       : efi-srpm-macros-6-4.el9.noarch                                                                   32/165
  Installing       : lua-srpm-macros-1-6.el9.noarch                                                                   33/165
  Installing       : bzip2-1.0.8-11.el9.x86_64                                                                        34/165
  Installing       : glibc-headers-2.34-266.el9_8.x86_64                                                              35/165
  Installing       : libuv-1:1.42.0-2.el9_4.x86_64                                                                    36/165
  Installing       : perl-Digest-1.19-4.el9.noarch                                                                    37/165
  Installing       : perl-Digest-MD5-2.58-4.el9.x86_64                                                                38/165
  Installing       : perl-B-1.80-481.1.el9_6.x86_64                                                                   39/165
  Installing       : perl-FileHandle-2.03-481.1.el9_6.noarch                                                          40/165
  Installing       : perl-Data-Dumper-2.174-462.el9.x86_64                                                            41/165
  Installing       : perl-libnet-3.13-4.el9.noarch                                                                    42/165
  Installing       : perl-base-2.27-481.1.el9_6.noarch                                                                43/165
  Installing       : perl-URI-5.09-3.el9.noarch                                                                       44/165
  Installing       : perl-AutoLoader-5.74-481.1.el9_6.noarch                                                          45/165
  Installing       : perl-Mozilla-CA-20200520-6.el9.noarch                                                            46/165
  Installing       : perl-if-0.60.800-481.1.el9_6.noarch                                                              47/165
  Installing       : perl-IO-Socket-IP-0.41-5.el9.noarch                                                              48/165
  Installing       : perl-Time-Local-2:1.300-7.el9.noarch                                                             49/165
  Installing       : perl-File-Path-2.18-4.el9.noarch                                                                 50/165
  Installing       : perl-Pod-Escapes-1:1.07-460.el9.noarch                                                           51/165
  Installing       : perl-Text-Tabs+Wrap-2013.0523-460.el9.noarch                                                     52/165
  Installing       : perl-IO-Socket-SSL-2.073-2.el9.noarch                                                            53/165
  Installing       : perl-Net-SSLeay-1.94-3.el9.x86_64                                                                54/165
  Installing       : perl-Class-Struct-0.66-481.1.el9_6.noarch                                                        55/165
  Installing       : perl-POSIX-1.94-481.1.el9_6.x86_64                                                               56/165
  Installing       : perl-Term-ANSIColor-5.01-461.el9.noarch                                                          57/165
  Installing       : perl-IPC-Open3-1.21-481.1.el9_6.noarch                                                           58/165
  Installing       : perl-subs-1.03-481.1.el9_6.noarch                                                                59/165
  Installing       : perl-File-Temp-1:0.231.100-4.el9.noarch                                                          60/165
  Installing       : perl-Term-Cap-1.17-460.el9.noarch                                                                61/165
  Installing       : perl-Pod-Simple-1:3.42-4.el9.noarch                                                              62/165
  Installing       : perl-HTTP-Tiny-0.076-462.el9.noarch                                                              63/165
  Installing       : perl-Socket-4:2.031-4.el9.x86_64                                                                 64/165
  Installing       : perl-SelectSaver-1.02-481.1.el9_6.noarch                                                         65/165
  Installing       : perl-Symbol-1.08-481.1.el9_6.noarch                                                              66/165
  Installing       : perl-File-stat-1.09-481.1.el9_6.noarch                                                           67/165
  Installing       : perl-podlators-1:4.14-460.el9.noarch                                                             68/165
  Installing       : perl-Pod-Perldoc-3.28.01-461.el9.noarch                                                          69/165
  Installing       : perl-Fcntl-1.13-481.1.el9_6.x86_64                                                               70/165
  Installing       : perl-Text-ParseWords-3.30-460.el9.noarch                                                         71/165
  Installing       : perl-mro-1.23-481.1.el9_6.x86_64                                                                 72/165
  Installing       : perl-IO-1.43-481.1.el9_6.x86_64                                                                  73/165
  Installing       : perl-overloading-0.02-481.1.el9_6.noarch                                                         74/165
  Installing       : perl-Pod-Usage-4:2.01-4.el9.noarch                                                               75/165
  Installing       : perl-Errno-1.30-481.1.el9_6.x86_64                                                               76/165
  Installing       : perl-File-Basename-2.85-481.1.el9_6.noarch                                                       77/165
  Installing       : perl-Getopt-Std-1.12-481.1.el9_6.noarch                                                          78/165
  Installing       : perl-MIME-Base64-3.16-4.el9.x86_64                                                               79/165
  Installing       : perl-Scalar-List-Utils-4:1.56-462.el9.x86_64                                                     80/165
  Installing       : perl-constant-1.33-461.el9.noarch                                                                81/165
  Installing       : perl-Storable-1:3.21-460.el9.x86_64                                                              82/165
  Installing       : perl-overload-1.31-481.1.el9_6.noarch                                                            83/165
  Installing       : perl-parent-1:0.238-460.el9.noarch                                                               84/165
  Installing       : perl-vars-1.05-481.1.el9_6.noarch                                                                85/165
  Installing       : perl-Getopt-Long-1:2.52-4.el9.noarch                                                             86/165
  Installing       : perl-Carp-1.50-460.el9.noarch                                                                    87/165
  Installing       : perl-Exporter-5.74-461.el9.noarch                                                                88/165
  Installing       : perl-NDBM_File-1.15-481.1.el9_6.x86_64                                                           89/165
  Installing       : perl-PathTools-3.78-461.el9.x86_64                                                               90/165
  Installing       : perl-Encode-4:3.08-462.el9.x86_64                                                                91/165
  Installing       : perl-libs-4:5.32.1-481.1.el9_6.x86_64                                                            92/165
  Installing       : perl-interpreter-4:5.32.1-481.1.el9_6.x86_64                                                     93/165
  Installing       : kernel-srpm-macros-1.0-14.el9.noarch                                                             94/165
  Installing       : perl-DynaLoader-1.47-481.1.el9_6.x86_64                                                          95/165
  Installing       : perl-TermReadKey-2.38-11.el9.x86_64                                                              96/165
  Installing       : perl-Error-1:0.17029-7.el9.noarch                                                                97/165
  Installing       : perl-lib-0.65-481.1.el9_6.x86_64                                                                 98/165
  Installing       : perl-Git-2.52.0-1.el9.noarch                                                                     99/165
  Installing       : git-2.52.0-1.el9.x86_64                                                                         100/165
  Installing       : info-6.7-15.el9.x86_64                                                                          101/165
  Installing       : ed-1.14.2-12.el9.x86_64                                                                         102/165
  Installing       : patch-2.7.6-16.el9.x86_64                                                                       103/165
  Installing       : libpkgconf-1.7.3-10.el9.x86_64                                                                  104/165
  Installing       : pkgconf-1.7.3-10.el9.x86_64                                                                     105/165
  Installing       : tar-2:1.34-11.el9.x86_64                                                                        106/165
  Installing       : zstd-1.5.5-1.el9.x86_64                                                                         107/165
  Upgrading        : python3-dnf-plugins-core-4.3.0-26.el9.noarch                                                    108/165
  Upgrading        : dnf-plugins-core-4.3.0-26.el9.noarch                                                            109/165
  Installing       : vim-filesystem-2:8.2.2637-26.el9_8.4.noarch                                                     110/165
  Installing       : cmake-data-3.31.8-3.el9.noarch                                                                  111/165
  Installing       : cmake-3.31.8-3.el9.x86_64                                                                       112/165
  Installing       : python3-pysocks-1.7.1-12.el9.noarch                                                             113/165
  Installing       : python3-urllib3-1.26.5-6.el9_7.1.noarch                                                         114/165
  Installing       : python3-chardet-4.0.0-5.el9.noarch                                                              115/165
  Installing       : python3-requests-2.25.1-10.el9_6.noarch                                                         116/165
  Installing       : pkgconf-m4-1.7.3-10.el9.noarch                                                                  117/165
  Installing       : pkgconf-pkg-config-1.7.3-10.el9.x86_64                                                          118/165
  Installing       : rust-srpm-macros-17-4.el9.noarch                                                                119/165
  Installing       : qt5-srpm-macros-5.15.9-1.el9.noarch                                                             120/165
  Installing       : python3-argcomplete-1.12.0-5.el9.noarch                                                         121/165
  Installing       : perl-srpm-macros-1-41.el9.noarch                                                                122/165
  Installing       : openblas-srpm-macros-2-11.el9.noarch                                                            123/165
  Installing       : ocaml-srpm-macros-6-6.el9.noarch                                                                124/165
  Installing       : llvm-filesystem-21.1.8-2.el9.x86_64                                                             125/165
  Installing       : llvm-libs-21.1.8-2.el9.x86_64                                                                   126/165
  Installing       : kernel-headers-5.14.0-687.5.3.el9_8.x86_64                                                      127/165
  Installing       : libxcrypt-devel-4.4.18-3.el9.x86_64                                                             128/165
  Installing       : glibc-devel-2.34-266.el9_8.x86_64                                                               129/165
  Installing       : gcc-11.5.0-14.el9.alma.1.x86_64                                                                 130/165
  Installing       : annobin-12.98-2.el9.x86_64                                                                      131/165
  Installing       : gcc-plugin-annobin-11.5.0-14.el9.alma.1.x86_64                                                  132/165
  Installing       : ghc-srpm-macros-1.5.0-6.el9.noarch                                                              133/165
  Installing       : fonts-srpm-macros-1:2.0.5-7.el9.1.noarch                                                        134/165
  Installing       : go-srpm-macros-3.8.1-1.el9.noarch                                                               135/165
  Installing       : python-srpm-macros-3.9-54.el9.noarch                                                            136/165
  Installing       : pyproject-srpm-macros-1.18.5-1.el9.noarch                                                       137/165
  Installing       : redhat-rpm-config-210-1.el9.alma.1.noarch                                                       138/165
  Running scriptlet: redhat-rpm-config-210-1.el9.alma.1.noarch                                                       138/165
  Installing       : rpm-build-4.16.1.3-40.el9.x86_64                                                                139/165
  Installing       : rpmdevtools-9.5-1.el9.noarch                                                                    140/165
  Installing       : yum-utils-4.3.0-26.el9.noarch                                                                   141/165
  Installing       : createrepo_c-0.20.1-4.el9.x86_64                                                                142/165
  Upgrading        : rpm-plugin-audit-4.16.1.3-40.el9.x86_64                                                         143/165
  Upgrading        : rpm-plugin-selinux-4.16.1.3-40.el9.x86_64                                                       144/165
  Installing       : wget-1.21.1-8.el9_4.x86_64                                                                      145/165
  Installing       : nano-5.6.1-7.el9.x86_64                                                                         146/165
  Cleanup          : dnf-plugins-core-4.3.0-24.el9_7.noarch                                                          147/165
  Cleanup          : elfutils-debuginfod-client-0.193-1.el9.alma.1.x86_64                                            148/165
  Cleanup          : python3-rpm-4.16.1.3-39.el9.x86_64                                                              149/165
  Cleanup          : rpm-build-libs-4.16.1.3-39.el9.x86_64                                                           150/165
  Cleanup          : elfutils-libs-0.193-1.el9.alma.1.x86_64                                                         151/165
  Cleanup          : rpm-sign-libs-4.16.1.3-39.el9.x86_64                                                            152/165
  Cleanup          : libgomp-11.5.0-11.el9.alma.1.x86_64                                                             153/165
  Cleanup          : elfutils-libelf-0.193-1.el9.alma.1.x86_64                                                       154/165
  Cleanup          : rpm-plugin-selinux-4.16.1.3-39.el9.x86_64                                                       155/165
  Cleanup          : rpm-plugin-audit-4.16.1.3-39.el9.x86_64                                                         156/165
  Cleanup          : python3-dnf-plugins-core-4.3.0-24.el9_7.noarch                                                  157/165
  Cleanup          : rpm-4.16.1.3-39.el9.x86_64                                                                      158/165
  Cleanup          : rpm-libs-4.16.1.3-39.el9.x86_64                                                                 159/165
  Cleanup          : bzip2-libs-1.0.8-10.el9_5.x86_64                                                                160/165
  Cleanup          : glibc-2.34-231.el9_7.2.x86_64                                                                   161/165
  Cleanup          : glibc-minimal-langpack-2.34-231.el9_7.2.x86_64                                                  162/165
  Cleanup          : glibc-gconv-extra-2.34-231.el9_7.2.x86_64                                                       163/165
  Running scriptlet: glibc-gconv-extra-2.34-231.el9_7.2.x86_64                                                       163/165
  Cleanup          : glibc-common-2.34-231.el9_7.2.x86_64                                                            164/165
  Cleanup          : libgcc-11.5.0-11.el9.alma.1.x86_64                                                              165/165
  Running scriptlet: libgcc-11.5.0-11.el9.alma.1.x86_64                                                              165/165
  Running scriptlet: rpm-4.16.1.3-40.el9.x86_64                                                                      165/165
  Running scriptlet: libgcc-11.5.0-11.el9.alma.1.x86_64                                                              165/165
warning: %transfiletriggerpostun(info-6.7-15.el9.x86_64) scriptlet failed, exit status 1

Error in <unknown> scriptlet in rpm package libgcc
  Verifying        : annobin-12.98-2.el9.x86_64                                                                        1/165
  Verifying        : cmake-3.31.8-3.el9.x86_64                                                                         2/165
  Verifying        : cmake-data-3.31.8-3.el9.noarch                                                                    3/165
  Verifying        : cmake-filesystem-3.31.8-3.el9.x86_64                                                              4/165
  Verifying        : cmake-rpm-macros-3.31.8-3.el9.noarch                                                              5/165
  Verifying        : cpp-11.5.0-14.el9.alma.1.x86_64                                                                   6/165
  Verifying        : createrepo_c-0.20.1-4.el9.x86_64                                                                  7/165
  Verifying        : createrepo_c-libs-0.20.1-4.el9.x86_64                                                             8/165
  Verifying        : debugedit-5.0-11.el9.x86_64                                                                       9/165
  Verifying        : dwz-0.16-1.el9.x86_64                                                                            10/165
  Verifying        : efi-srpm-macros-6-4.el9.noarch                                                                   11/165
  Verifying        : emacs-filesystem-1:27.2-18.el9.noarch                                                            12/165
  Verifying        : fonts-srpm-macros-1:2.0.5-7.el9.1.noarch                                                         13/165
  Verifying        : gcc-11.5.0-14.el9.alma.1.x86_64                                                                  14/165
  Verifying        : gcc-plugin-annobin-11.5.0-14.el9.alma.1.x86_64                                                   15/165
  Verifying        : gdb-minimal-16.3-3.el9.x86_64                                                                    16/165
  Verifying        : ghc-srpm-macros-1.5.0-6.el9.noarch                                                               17/165
  Verifying        : git-2.52.0-1.el9.x86_64                                                                          18/165
  Verifying        : git-core-2.52.0-1.el9.x86_64                                                                     19/165
  Verifying        : git-core-doc-2.52.0-1.el9.noarch                                                                 20/165
  Verifying        : glibc-devel-2.34-266.el9_8.x86_64                                                                21/165
  Verifying        : glibc-headers-2.34-266.el9_8.x86_64                                                              22/165
  Verifying        : go-srpm-macros-3.8.1-1.el9.noarch                                                                23/165
  Verifying        : kernel-headers-5.14.0-687.5.3.el9_8.x86_64                                                       24/165
  Verifying        : kernel-srpm-macros-1.0-14.el9.noarch                                                             25/165
  Verifying        : libmpc-1.2.1-4.el9.x86_64                                                                        26/165
  Verifying        : libuv-1:1.42.0-2.el9_4.x86_64                                                                    27/165
  Verifying        : libxcrypt-devel-4.4.18-3.el9.x86_64                                                              28/165
  Verifying        : llvm-filesystem-21.1.8-2.el9.x86_64                                                              29/165
  Verifying        : llvm-libs-21.1.8-2.el9.x86_64                                                                    30/165
  Verifying        : lua-srpm-macros-1-6.el9.noarch                                                                   31/165
  Verifying        : ocaml-srpm-macros-6-6.el9.noarch                                                                 32/165
  Verifying        : openblas-srpm-macros-2-11.el9.noarch                                                             33/165
  Verifying        : patch-2.7.6-16.el9.x86_64                                                                        34/165
  Verifying        : perl-AutoLoader-5.74-481.1.el9_6.noarch                                                          35/165
  Verifying        : perl-B-1.80-481.1.el9_6.x86_64                                                                   36/165
  Verifying        : perl-Carp-1.50-460.el9.noarch                                                                    37/165
  Verifying        : perl-Class-Struct-0.66-481.1.el9_6.noarch                                                        38/165
  Verifying        : perl-Data-Dumper-2.174-462.el9.x86_64                                                            39/165
  Verifying        : perl-Digest-1.19-4.el9.noarch                                                                    40/165
  Verifying        : perl-Digest-MD5-2.58-4.el9.x86_64                                                                41/165
  Verifying        : perl-DynaLoader-1.47-481.1.el9_6.x86_64                                                          42/165
  Verifying        : perl-Encode-4:3.08-462.el9.x86_64                                                                43/165
  Verifying        : perl-Errno-1.30-481.1.el9_6.x86_64                                                               44/165
  Verifying        : perl-Error-1:0.17029-7.el9.noarch                                                                45/165
  Verifying        : perl-Exporter-5.74-461.el9.noarch                                                                46/165
  Verifying        : perl-Fcntl-1.13-481.1.el9_6.x86_64                                                               47/165
  Verifying        : perl-File-Basename-2.85-481.1.el9_6.noarch                                                       48/165
  Verifying        : perl-File-Path-2.18-4.el9.noarch                                                                 49/165
  Verifying        : perl-File-Temp-1:0.231.100-4.el9.noarch                                                          50/165
  Verifying        : perl-File-stat-1.09-481.1.el9_6.noarch                                                           51/165
  Verifying        : perl-FileHandle-2.03-481.1.el9_6.noarch                                                          52/165
  Verifying        : perl-Getopt-Long-1:2.52-4.el9.noarch                                                             53/165
  Verifying        : perl-Getopt-Std-1.12-481.1.el9_6.noarch                                                          54/165
  Verifying        : perl-Git-2.52.0-1.el9.noarch                                                                     55/165
  Verifying        : perl-HTTP-Tiny-0.076-462.el9.noarch                                                              56/165
  Verifying        : perl-IO-1.43-481.1.el9_6.x86_64                                                                  57/165
  Verifying        : perl-IO-Socket-IP-0.41-5.el9.noarch                                                              58/165
  Verifying        : perl-IO-Socket-SSL-2.073-2.el9.noarch                                                            59/165
  Verifying        : perl-IPC-Open3-1.21-481.1.el9_6.noarch                                                           60/165
  Verifying        : perl-MIME-Base64-3.16-4.el9.x86_64                                                               61/165
  Verifying        : perl-Mozilla-CA-20200520-6.el9.noarch                                                            62/165
  Verifying        : perl-NDBM_File-1.15-481.1.el9_6.x86_64                                                           63/165
  Verifying        : perl-Net-SSLeay-1.94-3.el9.x86_64                                                                64/165
  Verifying        : perl-POSIX-1.94-481.1.el9_6.x86_64                                                               65/165
  Verifying        : perl-PathTools-3.78-461.el9.x86_64                                                               66/165
  Verifying        : perl-Pod-Escapes-1:1.07-460.el9.noarch                                                           67/165
  Verifying        : perl-Pod-Perldoc-3.28.01-461.el9.noarch                                                          68/165
  Verifying        : perl-Pod-Simple-1:3.42-4.el9.noarch                                                              69/165
  Verifying        : perl-Pod-Usage-4:2.01-4.el9.noarch                                                               70/165
  Verifying        : perl-Scalar-List-Utils-4:1.56-462.el9.x86_64                                                     71/165
  Verifying        : perl-SelectSaver-1.02-481.1.el9_6.noarch                                                         72/165
  Verifying        : perl-Socket-4:2.031-4.el9.x86_64                                                                 73/165
  Verifying        : perl-Storable-1:3.21-460.el9.x86_64                                                              74/165
  Verifying        : perl-Symbol-1.08-481.1.el9_6.noarch                                                              75/165
  Verifying        : perl-Term-ANSIColor-5.01-461.el9.noarch                                                          76/165
  Verifying        : perl-Term-Cap-1.17-460.el9.noarch                                                                77/165
  Verifying        : perl-TermReadKey-2.38-11.el9.x86_64                                                              78/165
  Verifying        : perl-Text-ParseWords-3.30-460.el9.noarch                                                         79/165
  Verifying        : perl-Text-Tabs+Wrap-2013.0523-460.el9.noarch                                                     80/165
  Verifying        : perl-Time-Local-2:1.300-7.el9.noarch                                                             81/165
  Verifying        : perl-URI-5.09-3.el9.noarch                                                                       82/165
  Verifying        : perl-base-2.27-481.1.el9_6.noarch                                                                83/165
  Verifying        : perl-constant-1.33-461.el9.noarch                                                                84/165
  Verifying        : perl-if-0.60.800-481.1.el9_6.noarch                                                              85/165
  Verifying        : perl-interpreter-4:5.32.1-481.1.el9_6.x86_64                                                     86/165
  Verifying        : perl-lib-0.65-481.1.el9_6.x86_64                                                                 87/165
  Verifying        : perl-libnet-3.13-4.el9.noarch                                                                    88/165
  Verifying        : perl-libs-4:5.32.1-481.1.el9_6.x86_64                                                            89/165
  Verifying        : perl-mro-1.23-481.1.el9_6.x86_64                                                                 90/165
  Verifying        : perl-overload-1.31-481.1.el9_6.noarch                                                            91/165
  Verifying        : perl-overloading-0.02-481.1.el9_6.noarch                                                         92/165
  Verifying        : perl-parent-1:0.238-460.el9.noarch                                                               93/165
  Verifying        : perl-podlators-1:4.14-460.el9.noarch                                                             94/165
  Verifying        : perl-srpm-macros-1-41.el9.noarch                                                                 95/165
  Verifying        : perl-subs-1.03-481.1.el9_6.noarch                                                                96/165
  Verifying        : perl-vars-1.05-481.1.el9_6.noarch                                                                97/165
  Verifying        : pyproject-srpm-macros-1.18.5-1.el9.noarch                                                        98/165
  Verifying        : python-srpm-macros-3.9-54.el9.noarch                                                             99/165
  Verifying        : python3-argcomplete-1.12.0-5.el9.noarch                                                         100/165
  Verifying        : qt5-srpm-macros-5.15.9-1.el9.noarch                                                             101/165
  Verifying        : redhat-rpm-config-210-1.el9.alma.1.noarch                                                       102/165
  Verifying        : rpm-build-4.16.1.3-40.el9.x86_64                                                                103/165
  Verifying        : rpmdevtools-9.5-1.el9.noarch                                                                    104/165
  Verifying        : rust-srpm-macros-17-4.el9.noarch                                                                105/165
  Verifying        : wget-1.21.1-8.el9_4.x86_64                                                                      106/165
  Verifying        : bzip2-1.0.8-11.el9.x86_64                                                                       107/165
  Verifying        : ed-1.14.2-12.el9.x86_64                                                                         108/165
  Verifying        : elfutils-0.194-1.el9.alma.1.x86_64                                                              109/165
  Verifying        : info-6.7-15.el9.x86_64                                                                          110/165
  Verifying        : libpkgconf-1.7.3-10.el9.x86_64                                                                  111/165
  Verifying        : make-1:4.3-8.el9.x86_64                                                                         112/165
  Verifying        : nano-5.6.1-7.el9.x86_64                                                                         113/165
  Verifying        : pkgconf-1.7.3-10.el9.x86_64                                                                     114/165
  Verifying        : pkgconf-m4-1.7.3-10.el9.noarch                                                                  115/165
  Verifying        : pkgconf-pkg-config-1.7.3-10.el9.x86_64                                                          116/165
  Verifying        : python3-chardet-4.0.0-5.el9.noarch                                                              117/165
  Verifying        : python3-idna-2.10-7.el9_4.1.noarch                                                              118/165
  Verifying        : python3-pysocks-1.7.1-12.el9.noarch                                                             119/165
  Verifying        : python3-requests-2.25.1-10.el9_6.noarch                                                         120/165
  Verifying        : python3-urllib3-1.26.5-6.el9_7.1.noarch                                                         121/165
  Verifying        : tar-2:1.34-11.el9.x86_64                                                                        122/165
  Verifying        : unzip-6.0-60.el9.x86_64                                                                         123/165
  Verifying        : vim-filesystem-2:8.2.2637-26.el9_8.4.noarch                                                     124/165
  Verifying        : yum-utils-4.3.0-26.el9.noarch                                                                   125/165
  Verifying        : zip-3.0-35.el9.x86_64                                                                           126/165
  Verifying        : zstd-1.5.5-1.el9.x86_64                                                                         127/165
  Verifying        : bzip2-libs-1.0.8-11.el9.x86_64                                                                  128/165
  Verifying        : bzip2-libs-1.0.8-10.el9_5.x86_64                                                                129/165
  Verifying        : dnf-plugins-core-4.3.0-26.el9.noarch                                                            130/165
  Verifying        : dnf-plugins-core-4.3.0-24.el9_7.noarch                                                          131/165
  Verifying        : elfutils-debuginfod-client-0.194-1.el9.alma.1.x86_64                                            132/165
  Verifying        : elfutils-debuginfod-client-0.193-1.el9.alma.1.x86_64                                            133/165
  Verifying        : elfutils-libelf-0.194-1.el9.alma.1.x86_64                                                       134/165
  Verifying        : elfutils-libelf-0.193-1.el9.alma.1.x86_64                                                       135/165
  Verifying        : elfutils-libs-0.194-1.el9.alma.1.x86_64                                                         136/165
  Verifying        : elfutils-libs-0.193-1.el9.alma.1.x86_64                                                         137/165
  Verifying        : glibc-2.34-266.el9_8.x86_64                                                                     138/165
  Verifying        : glibc-2.34-231.el9_7.2.x86_64                                                                   139/165
  Verifying        : glibc-common-2.34-266.el9_8.x86_64                                                              140/165
  Verifying        : glibc-common-2.34-231.el9_7.2.x86_64                                                            141/165
  Verifying        : glibc-gconv-extra-2.34-266.el9_8.x86_64                                                         142/165
  Verifying        : glibc-gconv-extra-2.34-231.el9_7.2.x86_64                                                       143/165
  Verifying        : glibc-minimal-langpack-2.34-266.el9_8.x86_64                                                    144/165
  Verifying        : glibc-minimal-langpack-2.34-231.el9_7.2.x86_64                                                  145/165
  Verifying        : libgcc-11.5.0-14.el9.alma.1.x86_64                                                              146/165
  Verifying        : libgcc-11.5.0-11.el9.alma.1.x86_64                                                              147/165
  Verifying        : libgomp-11.5.0-14.el9.alma.1.x86_64                                                             148/165
  Verifying        : libgomp-11.5.0-11.el9.alma.1.x86_64                                                             149/165
  Verifying        : python3-dnf-plugins-core-4.3.0-26.el9.noarch                                                    150/165
  Verifying        : python3-dnf-plugins-core-4.3.0-24.el9_7.noarch                                                  151/165
  Verifying        : python3-rpm-4.16.1.3-40.el9.x86_64                                                              152/165
  Verifying        : python3-rpm-4.16.1.3-39.el9.x86_64                                                              153/165
  Verifying        : rpm-4.16.1.3-40.el9.x86_64                                                                      154/165
  Verifying        : rpm-4.16.1.3-39.el9.x86_64                                                                      155/165
  Verifying        : rpm-build-libs-4.16.1.3-40.el9.x86_64                                                           156/165
  Verifying        : rpm-build-libs-4.16.1.3-39.el9.x86_64                                                           157/165
  Verifying        : rpm-libs-4.16.1.3-40.el9.x86_64                                                                 158/165
  Verifying        : rpm-libs-4.16.1.3-39.el9.x86_64                                                                 159/165
  Verifying        : rpm-plugin-audit-4.16.1.3-40.el9.x86_64                                                         160/165
  Verifying        : rpm-plugin-audit-4.16.1.3-39.el9.x86_64                                                         161/165
  Verifying        : rpm-plugin-selinux-4.16.1.3-40.el9.x86_64                                                       162/165
  Verifying        : rpm-plugin-selinux-4.16.1.3-39.el9.x86_64                                                       163/165
  Verifying        : rpm-sign-libs-4.16.1.3-40.el9.x86_64                                                            164/165
  Verifying        : rpm-sign-libs-4.16.1.3-39.el9.x86_64                                                            165/165

Upgraded:
  bzip2-libs-1.0.8-11.el9.x86_64                                    dnf-plugins-core-4.3.0-26.el9.noarch
  elfutils-debuginfod-client-0.194-1.el9.alma.1.x86_64              elfutils-libelf-0.194-1.el9.alma.1.x86_64
  elfutils-libs-0.194-1.el9.alma.1.x86_64                           glibc-2.34-266.el9_8.x86_64
  glibc-common-2.34-266.el9_8.x86_64                                glibc-gconv-extra-2.34-266.el9_8.x86_64
  glibc-minimal-langpack-2.34-266.el9_8.x86_64                      libgcc-11.5.0-14.el9.alma.1.x86_64
  libgomp-11.5.0-14.el9.alma.1.x86_64                               python3-dnf-plugins-core-4.3.0-26.el9.noarch
  python3-rpm-4.16.1.3-40.el9.x86_64                                rpm-4.16.1.3-40.el9.x86_64
  rpm-build-libs-4.16.1.3-40.el9.x86_64                             rpm-libs-4.16.1.3-40.el9.x86_64
  rpm-plugin-audit-4.16.1.3-40.el9.x86_64                           rpm-plugin-selinux-4.16.1.3-40.el9.x86_64
  rpm-sign-libs-4.16.1.3-40.el9.x86_64
Installed:
  annobin-12.98-2.el9.x86_64                                   bzip2-1.0.8-11.el9.x86_64
  cmake-3.31.8-3.el9.x86_64                                    cmake-data-3.31.8-3.el9.noarch
  cmake-filesystem-3.31.8-3.el9.x86_64                         cmake-rpm-macros-3.31.8-3.el9.noarch
  cpp-11.5.0-14.el9.alma.1.x86_64                              createrepo_c-0.20.1-4.el9.x86_64
  createrepo_c-libs-0.20.1-4.el9.x86_64                        debugedit-5.0-11.el9.x86_64
  dwz-0.16-1.el9.x86_64                                        ed-1.14.2-12.el9.x86_64
  efi-srpm-macros-6-4.el9.noarch                               elfutils-0.194-1.el9.alma.1.x86_64
  emacs-filesystem-1:27.2-18.el9.noarch                        fonts-srpm-macros-1:2.0.5-7.el9.1.noarch
  gcc-11.5.0-14.el9.alma.1.x86_64                              gcc-plugin-annobin-11.5.0-14.el9.alma.1.x86_64
  gdb-minimal-16.3-3.el9.x86_64                                ghc-srpm-macros-1.5.0-6.el9.noarch
  git-2.52.0-1.el9.x86_64                                      git-core-2.52.0-1.el9.x86_64
  git-core-doc-2.52.0-1.el9.noarch                             glibc-devel-2.34-266.el9_8.x86_64
  glibc-headers-2.34-266.el9_8.x86_64                          go-srpm-macros-3.8.1-1.el9.noarch
  info-6.7-15.el9.x86_64                                       kernel-headers-5.14.0-687.5.3.el9_8.x86_64
  kernel-srpm-macros-1.0-14.el9.noarch                         libmpc-1.2.1-4.el9.x86_64
  libpkgconf-1.7.3-10.el9.x86_64                               libuv-1:1.42.0-2.el9_4.x86_64
  libxcrypt-devel-4.4.18-3.el9.x86_64                          llvm-filesystem-21.1.8-2.el9.x86_64
  llvm-libs-21.1.8-2.el9.x86_64                                lua-srpm-macros-1-6.el9.noarch
  make-1:4.3-8.el9.x86_64                                      nano-5.6.1-7.el9.x86_64
  ocaml-srpm-macros-6-6.el9.noarch                             openblas-srpm-macros-2-11.el9.noarch
  patch-2.7.6-16.el9.x86_64                                    perl-AutoLoader-5.74-481.1.el9_6.noarch
  perl-B-1.80-481.1.el9_6.x86_64                               perl-Carp-1.50-460.el9.noarch
  perl-Class-Struct-0.66-481.1.el9_6.noarch                    perl-Data-Dumper-2.174-462.el9.x86_64
  perl-Digest-1.19-4.el9.noarch                                perl-Digest-MD5-2.58-4.el9.x86_64
  perl-DynaLoader-1.47-481.1.el9_6.x86_64                      perl-Encode-4:3.08-462.el9.x86_64
  perl-Errno-1.30-481.1.el9_6.x86_64                           perl-Error-1:0.17029-7.el9.noarch
  perl-Exporter-5.74-461.el9.noarch                            perl-Fcntl-1.13-481.1.el9_6.x86_64
  perl-File-Basename-2.85-481.1.el9_6.noarch                   perl-File-Path-2.18-4.el9.noarch
  perl-File-Temp-1:0.231.100-4.el9.noarch                      perl-File-stat-1.09-481.1.el9_6.noarch
  perl-FileHandle-2.03-481.1.el9_6.noarch                      perl-Getopt-Long-1:2.52-4.el9.noarch
  perl-Getopt-Std-1.12-481.1.el9_6.noarch                      perl-Git-2.52.0-1.el9.noarch
  perl-HTTP-Tiny-0.076-462.el9.noarch                          perl-IO-1.43-481.1.el9_6.x86_64
  perl-IO-Socket-IP-0.41-5.el9.noarch                          perl-IO-Socket-SSL-2.073-2.el9.noarch
  perl-IPC-Open3-1.21-481.1.el9_6.noarch                       perl-MIME-Base64-3.16-4.el9.x86_64
  perl-Mozilla-CA-20200520-6.el9.noarch                        perl-NDBM_File-1.15-481.1.el9_6.x86_64
  perl-Net-SSLeay-1.94-3.el9.x86_64                            perl-POSIX-1.94-481.1.el9_6.x86_64
  perl-PathTools-3.78-461.el9.x86_64                           perl-Pod-Escapes-1:1.07-460.el9.noarch
  perl-Pod-Perldoc-3.28.01-461.el9.noarch                      perl-Pod-Simple-1:3.42-4.el9.noarch
  perl-Pod-Usage-4:2.01-4.el9.noarch                           perl-Scalar-List-Utils-4:1.56-462.el9.x86_64
  perl-SelectSaver-1.02-481.1.el9_6.noarch                     perl-Socket-4:2.031-4.el9.x86_64
  perl-Storable-1:3.21-460.el9.x86_64                          perl-Symbol-1.08-481.1.el9_6.noarch
  perl-Term-ANSIColor-5.01-461.el9.noarch                      perl-Term-Cap-1.17-460.el9.noarch
  perl-TermReadKey-2.38-11.el9.x86_64                          perl-Text-ParseWords-3.30-460.el9.noarch
  perl-Text-Tabs+Wrap-2013.0523-460.el9.noarch                 perl-Time-Local-2:1.300-7.el9.noarch
  perl-URI-5.09-3.el9.noarch                                   perl-base-2.27-481.1.el9_6.noarch
  perl-constant-1.33-461.el9.noarch                            perl-if-0.60.800-481.1.el9_6.noarch
  perl-interpreter-4:5.32.1-481.1.el9_6.x86_64                 perl-lib-0.65-481.1.el9_6.x86_64
  perl-libnet-3.13-4.el9.noarch                                perl-libs-4:5.32.1-481.1.el9_6.x86_64
  perl-mro-1.23-481.1.el9_6.x86_64                             perl-overload-1.31-481.1.el9_6.noarch
  perl-overloading-0.02-481.1.el9_6.noarch                     perl-parent-1:0.238-460.el9.noarch
  perl-podlators-1:4.14-460.el9.noarch                         perl-srpm-macros-1-41.el9.noarch
  perl-subs-1.03-481.1.el9_6.noarch                            perl-vars-1.05-481.1.el9_6.noarch
  pkgconf-1.7.3-10.el9.x86_64                                  pkgconf-m4-1.7.3-10.el9.noarch
  pkgconf-pkg-config-1.7.3-10.el9.x86_64                       pyproject-srpm-macros-1.18.5-1.el9.noarch
  python-srpm-macros-3.9-54.el9.noarch                         python3-argcomplete-1.12.0-5.el9.noarch
  python3-chardet-4.0.0-5.el9.noarch                           python3-idna-2.10-7.el9_4.1.noarch
  python3-pysocks-1.7.1-12.el9.noarch                          python3-requests-2.25.1-10.el9_6.noarch
  python3-urllib3-1.26.5-6.el9_7.1.noarch                      qt5-srpm-macros-5.15.9-1.el9.noarch
  redhat-rpm-config-210-1.el9.alma.1.noarch                    rpm-build-4.16.1.3-40.el9.x86_64
  rpmdevtools-9.5-1.el9.noarch                                 rust-srpm-macros-17-4.el9.noarch
  tar-2:1.34-11.el9.x86_64                                     unzip-6.0-60.el9.x86_64
  vim-filesystem-2:8.2.2637-26.el9_8.4.noarch                  wget-1.21.1-8.el9_4.x86_64
  yum-utils-4.3.0-26.el9.noarch                                zip-3.0-35.el9.x86_64
  zstd-1.5.5-1.el9.x86_64

Complete!
```

В качестве примера соберем пакет Nginx с дополнительным модулем ngx_brotli.
Создадим отдельную директорию и загрузим SRPM пакет Nginx для дальнейшей работы над ним:

```
[root@localhost user]# mkdir rpm && cd rpm
[root@localhost rpm]# yumdownloader --source nginx
enabling appstream-source repository
enabling baseos-source repository
enabling extras-source repository
AlmaLinux 9 - AppStream - Source                                                             808 kB/s | 853 kB     00:01
AlmaLinux 9 - BaseOS - Source                                                                326 kB/s | 293 kB     00:00
AlmaLinux 9 - Extras - Source                                                                 12 kB/s | 9.7 kB     00:00
nginx-1.20.1-28.el9_8.2.alma.1.src.rpm                                                       1.3 MB/s | 1.1 MB     00:00
```

Установим пакет Nginx и зависимости необходимые для сборки:

```
[root@localhost rpm]# rpm -Uvh nginx-1.20.1-28.el9_8.2.alma.1.src.rpm
Updating / installing...
   1:nginx-2:1.20.1-28.el9_8.2.alma.1 ################################# [100%]

[root@localhost rpm]# yum-builddep nginx
enabling appstream-source repository
enabling baseos-source repository
enabling extras-source repository
enabling epel-source repository
enabling epel-cisco-openh264-source repository
Extra Packages for Enterprise Linux 9 - x86_64 - Source                                      3.2 MB/s | 4.3 MB     00:01
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64 - Source                948  B/s | 1.2 kB     00:01
Package make-1:4.3-8.el9.x86_64 is already installed.
Package gcc-11.5.0-14.el9.alma.1.x86_64 is already installed.
Package systemd-252-67.el9_8.2.alma.1.x86_64 is already installed.
Package systemd-rpm-macros-252-67.el9_8.2.alma.1.noarch is already installed.
Package gnupg2-2.3.3-4.el9.x86_64 is already installed.
Dependencies resolved.
=============================================================================================================================
 Package                                 Architecture       Version                              Repository             Size
=============================================================================================================================
Installing:
 gd-devel                                x86_64             2.3.2-3.el9                          appstream              37 k
 libxslt-devel                           x86_64             1.1.34-14.el9_7.1                    appstream             287 k
 openssl-devel                           x86_64             1:3.5.5-2.el9_8                      appstream             3.4 M
 pcre-devel                              x86_64             8.44-4.el9                           appstream             469 k
 perl-ExtUtils-Embed                     noarch             1.35-481.1.el9_6                     appstream              16 k
 perl-devel                              x86_64             4:5.32.1-481.1.el9_6                 appstream             659 k
 perl-generators                         noarch             1.13-1.el9                           appstream              15 k
 zlib-devel                              x86_64             1.2.11-40.el9                        appstream              44 k
Upgrading:
 glib2                                   x86_64             2.68.4-19.el9_8.1                    baseos                2.6 M
 gnupg2                                  x86_64             2.3.3-5.el9_7                        baseos                2.5 M
 libblkid                                x86_64             2.37.4-25.el9                        baseos                105 k
 libbrotli                               x86_64             1.0.9-9.el9_7                        baseos                311 k
 libfdisk                                x86_64             2.37.4-25.el9                        baseos                152 k
 libmount                                x86_64             2.37.4-25.el9                        baseos                133 k
 libsmartcols                            x86_64             2.37.4-25.el9                        baseos                 61 k
 libuuid                                 x86_64             2.37.4-25.el9                        baseos                 26 k
 libxml2                                 x86_64             2.9.13-14.el9_7                      baseos                746 k
 openssl                                 x86_64             1:3.5.5-2.el9_8                      baseos                1.4 M
 openssl-fips-provider                   x86_64             1:3.5.5-2.el9_8                      baseos                814 k
 openssl-libs                            x86_64             1:3.5.5-2.el9_8                      baseos                2.3 M
 util-linux                              x86_64             2.37.4-25.el9                        baseos                2.2 M
 util-linux-core                         x86_64             2.37.4-25.el9                        baseos                430 k
Installing dependencies:
 brotli                                  x86_64             1.0.9-9.el9_7                        appstream             311 k
 brotli-devel                            x86_64             1.0.9-9.el9_7                        appstream              30 k
 bzip2-devel                             x86_64             1.0.8-11.el9                         appstream             213 k
 cairo                                   x86_64             1.17.4-7.el9                         appstream             659 k
 dejavu-sans-fonts                       noarch             2.37-18.el9                          baseos                1.3 M
 fontconfig                              x86_64             2.14.0-2.el9_1                       appstream             274 k
 fontconfig-devel                        x86_64             2.14.0-2.el9_1                       appstream             127 k
 fonts-filesystem                        noarch             1:2.0.5-7.el9.1                      baseos                9.0 k
 freetype                                x86_64             2.10.4-10.el9_5                      baseos                385 k
 freetype-devel                          x86_64             2.10.4-10.el9_5                      appstream             1.1 M
 gd                                      x86_64             2.3.2-3.el9                          appstream             131 k
 glib2-devel                             x86_64             2.68.4-19.el9_8.1                    appstream             469 k
 graphite2                               x86_64             1.3.14-9.el9                         baseos                 94 k
 graphite2-devel                         x86_64             1.3.14-9.el9                         appstream              21 k
 harfbuzz                                x86_64             2.7.4-10.el9                         baseos                623 k
 harfbuzz-devel                          x86_64             2.7.4-10.el9                         appstream             304 k
 harfbuzz-icu                            x86_64             2.7.4-10.el9                         appstream              13 k
 jbigkit-libs                            x86_64             2.1-23.el9                           appstream              52 k
 langpacks-core-font-en                  noarch             3.0-16.el9                           appstream             9.4 k
 libICE                                  x86_64             1.0.10-8.el9                         appstream              70 k
 libSM                                   x86_64             1.2.3-10.el9                         appstream              41 k
 libX11                                  x86_64             1.8.12-1.el9                         appstream             647 k
 libX11-common                           noarch             1.8.12-1.el9                         appstream             144 k
 libX11-devel                            x86_64             1.8.12-1.el9                         appstream             910 k
 libX11-xcb                              x86_64             1.8.12-1.el9                         appstream              10 k
 libXau                                  x86_64             1.0.9-8.el9                          appstream              30 k
 libXau-devel                            x86_64             1.0.9-8.el9                          appstream              13 k
 libXext                                 x86_64             1.3.4-8.el9                          appstream              39 k
 libXpm                                  x86_64             3.5.13-10.el9                        appstream              57 k
 libXpm-devel                            x86_64             3.5.13-10.el9                        appstream              34 k
 libXrender                              x86_64             0.9.10-16.el9                        appstream              27 k
 libXt                                   x86_64             1.2.0-6.el9                          appstream             179 k
 libblkid-devel                          x86_64             2.37.4-25.el9                        appstream              15 k
 libffi-devel                            x86_64             3.4.2-8.el9                          appstream              28 k
 libgpg-error-devel                      x86_64             1.42-5.el9                           appstream              65 k
 libicu-devel                            x86_64             67.1-10.el9_6                        appstream             829 k
 libjpeg-turbo                           x86_64             2.0.90-7.el9                         appstream             174 k
 libjpeg-turbo-devel                     x86_64             2.0.90-7.el9                         appstream              98 k
 libmount-devel                          x86_64             2.37.4-25.el9                        appstream              16 k
 libpng                                  x86_64             2:1.6.37-15.el9_8                    baseos                115 k
 libpng-devel                            x86_64             2:1.6.37-15.el9_8                    appstream             289 k
 libselinux-devel                        x86_64             3.6-3.el9                            appstream             113 k
 libsepol-devel                          x86_64             3.6-3.el9                            appstream              39 k
 libtiff                                 x86_64             4.4.0-18.el9_8                       appstream             196 k
 libtiff-devel                           x86_64             4.4.0-18.el9_8                       appstream             527 k
 libwebp                                 x86_64             1.2.0-8.el9_3                        appstream             276 k
 libwebp-devel                           x86_64             1.2.0-8.el9_3                        appstream              32 k
 libxcb                                  x86_64             1.13.1-9.el9                         appstream             225 k
 libxcb-devel                            x86_64             1.13.1-9.el9                         appstream             1.0 M
 libxml2-devel                           x86_64             2.9.13-14.el9_7                      appstream             828 k
 libxslt                                 x86_64             1.1.34-14.el9_7.1                    appstream             240 k
 pcre-cpp                                x86_64             8.44-4.el9                           appstream              24 k
 pcre-utf16                              x86_64             8.44-4.el9                           appstream             183 k
 pcre-utf32                              x86_64             8.44-4.el9                           appstream             173 k
 pcre2-devel                             x86_64             10.40-6.el9                          appstream             471 k
 pcre2-utf16                             x86_64             10.40-6.el9                          appstream             213 k
 pcre2-utf32                             x86_64             10.40-6.el9                          appstream             202 k
 perl-AutoSplit                          noarch             5.74-481.1.el9_6                     appstream              20 k
 perl-Benchmark                          noarch             1.23-481.1.el9_6                     appstream              25 k
 perl-CPAN-Meta-Requirements             noarch             2.140-461.el9                        appstream              31 k
 perl-CPAN-Meta-YAML                     noarch             0.018-461.el9                        appstream              26 k
 perl-Devel-PPPort                       x86_64             3.62-4.el9                           appstream             211 k
 perl-ExtUtils-Command                   noarch             2:7.60-3.el9                         appstream              14 k
 perl-ExtUtils-Constant                  noarch             0.25-481.1.el9_6                     appstream              45 k
 perl-ExtUtils-Install                   noarch             2.20-4.el9                           appstream              44 k
 perl-ExtUtils-MakeMaker                 noarch             2:7.60-3.el9                         appstream             289 k
 perl-ExtUtils-Manifest                  noarch             1:1.73-4.el9                         appstream              34 k
 perl-ExtUtils-ParseXS                   noarch             1:3.40-460.el9                       appstream             181 k
 perl-Fedora-VSP                         noarch             0.001-23.el9                         appstream              23 k
 perl-File-Compare                       noarch             1.100.600-481.1.el9_6                appstream              12 k
 perl-File-Copy                          noarch             2.34-481.1.el9_6                     appstream              19 k
 perl-File-Find                          noarch             1.37-481.1.el9_6                     appstream              24 k
 perl-I18N-Langinfo                      x86_64             0.19-481.1.el9_6                     appstream              21 k
 perl-JSON-PP                            noarch             1:4.06-4.el9                         appstream              65 k
 perl-Math-BigInt                        noarch             1:1.9998.18-460.el9                  appstream             188 k
 perl-Math-Complex                       noarch             1.59-481.1.el9_6                     appstream              45 k
 perl-Test-Harness                       noarch             1:3.42-461.el9                       appstream             267 k
 perl-locale                             noarch             1.09-481.1.el9_6                     appstream              12 k
 perl-macros                             noarch             4:5.32.1-481.1.el9_6                 appstream             9.2 k
 perl-version                            x86_64             7:0.99.28-4.el9                      appstream              62 k
 pixman                                  x86_64             0.40.0-6.el9_3                       appstream             268 k
 python3-pyparsing                       noarch             2.4.7-9.el9                          baseos                149 k
 sysprof-capture-devel                   x86_64             3.40.1-3.el9                         appstream              59 k
 systemtap-sdt-devel                     x86_64             5.4-4.el9                            appstream              68 k
 systemtap-sdt-dtrace                    x86_64             5.4-4.el9                            appstream              69 k
 xml-common                              noarch             0.6.3-58.el9                         appstream              31 k
 xorg-x11-proto-devel                    noarch             2024.1-1.el9                         appstream             265 k
 xz-devel                                x86_64             5.2.5-8.el9_0                        appstream              52 k
Installing weak dependencies:
 perl-CPAN-Meta                          noarch             2.150010-460.el9                     appstream             176 k
 perl-Encode-Locale                      noarch             1.05-21.el9                          appstream              19 k
 perl-Time-HiRes                         x86_64             4:1.9764-462.el9                     appstream              57 k
 perl-doc                                noarch             5.32.1-481.1.el9_6                   appstream             4.5 M

Transaction Summary
=============================================================================================================================
Install  100 Packages
Upgrade   14 Packages

Total download size: 40 M
Is this ok [y/N]: y
Downloading Packages:
(1/114): brotli-devel-1.0.9-9.el9_7.x86_64.rpm                                               442 kB/s |  30 kB     00:00
(2/114): brotli-1.0.9-9.el9_7.x86_64.rpm                                                     3.4 MB/s | 311 kB     00:00
(3/114): bzip2-devel-1.0.8-11.el9.x86_64.rpm                                                 2.2 MB/s | 213 kB     00:00
(4/114): fontconfig-devel-2.14.0-2.el9_1.x86_64.rpm                                          3.6 MB/s | 127 kB     00:00
(5/114): fontconfig-2.14.0-2.el9_1.x86_64.rpm                                                2.9 MB/s | 274 kB     00:00
(6/114): gd-2.3.2-3.el9.x86_64.rpm                                                           4.8 MB/s | 131 kB     00:00
(7/114): cairo-1.17.4-7.el9.x86_64.rpm                                                       4.0 MB/s | 659 kB     00:00
(8/114): gd-devel-2.3.2-3.el9.x86_64.rpm                                                     1.3 MB/s |  37 kB     00:00
(9/114): graphite2-devel-1.3.14-9.el9.x86_64.rpm                                             1.3 MB/s |  21 kB     00:00
(10/114): glib2-devel-2.68.4-19.el9_8.1.x86_64.rpm                                            14 MB/s | 469 kB     00:00
(11/114): harfbuzz-icu-2.7.4-10.el9.x86_64.rpm                                               434 kB/s |  13 kB     00:00
(12/114): jbigkit-libs-2.1-23.el9.x86_64.rpm                                                 4.1 MB/s |  52 kB     00:00
(13/114): harfbuzz-devel-2.7.4-10.el9.x86_64.rpm                                             4.7 MB/s | 304 kB     00:00
(14/114): langpacks-core-font-en-3.0-16.el9.noarch.rpm                                       738 kB/s | 9.4 kB     00:00
(15/114): freetype-devel-2.10.4-10.el9_5.x86_64.rpm                                          5.0 MB/s | 1.1 MB     00:00
(16/114): libICE-1.0.10-8.el9.x86_64.rpm                                                     2.2 MB/s |  70 kB     00:00
(17/114): libSM-1.2.3-10.el9.x86_64.rpm                                                      661 kB/s |  41 kB     00:00
(18/114): libX11-common-1.8.12-1.el9.noarch.rpm                                              3.0 MB/s | 144 kB     00:00
(19/114): libX11-xcb-1.8.12-1.el9.x86_64.rpm                                                 620 kB/s |  10 kB     00:00
(20/114): libX11-1.8.12-1.el9.x86_64.rpm                                                     6.2 MB/s | 647 kB     00:00
(21/114): libXau-1.0.9-8.el9.x86_64.rpm                                                      808 kB/s |  30 kB     00:00
(22/114): libXau-devel-1.0.9-8.el9.x86_64.rpm                                                506 kB/s |  13 kB     00:00
(23/114): libXext-1.3.4-8.el9.x86_64.rpm                                                     1.1 MB/s |  39 kB     00:00
(24/114): libXpm-3.5.13-10.el9.x86_64.rpm                                                    1.4 MB/s |  57 kB     00:00
(25/114): libXpm-devel-3.5.13-10.el9.x86_64.rpm                                              676 kB/s |  34 kB     00:00
(26/114): libX11-devel-1.8.12-1.el9.x86_64.rpm                                               5.5 MB/s | 910 kB     00:00
(27/114): libXrender-0.9.10-16.el9.x86_64.rpm                                                1.0 MB/s |  27 kB     00:00
(28/114): libblkid-devel-2.37.4-25.el9.x86_64.rpm                                            800 kB/s |  15 kB     00:00
(29/114): libffi-devel-3.4.2-8.el9.x86_64.rpm                                                1.2 MB/s |  28 kB     00:00
(30/114): libXt-1.2.0-6.el9.x86_64.rpm                                                       2.7 MB/s | 179 kB     00:00
(31/114): libjpeg-turbo-2.0.90-7.el9.x86_64.rpm                                              3.2 MB/s | 174 kB     00:00
(32/114): libgpg-error-devel-1.42-5.el9.x86_64.rpm                                           565 kB/s |  65 kB     00:00
(33/114): libjpeg-turbo-devel-2.0.90-7.el9.x86_64.rpm                                        2.5 MB/s |  98 kB     00:00
(34/114): libmount-devel-2.37.4-25.el9.x86_64.rpm                                            198 kB/s |  16 kB     00:00
(35/114): libpng-devel-1.6.37-15.el9_8.x86_64.rpm                                            3.8 MB/s | 289 kB     00:00
(36/114): libicu-devel-67.1-10.el9_6.x86_64.rpm                                              3.8 MB/s | 829 kB     00:00
(37/114): libselinux-devel-3.6-3.el9.x86_64.rpm                                              1.9 MB/s | 113 kB     00:00
(38/114): libtiff-4.4.0-18.el9_8.x86_64.rpm                                                  3.8 MB/s | 196 kB     00:00
(39/114): libsepol-devel-3.6-3.el9.x86_64.rpm                                                542 kB/s |  39 kB     00:00
(40/114): libtiff-devel-4.4.0-18.el9_8.x86_64.rpm                                            9.5 MB/s | 527 kB     00:00
(41/114): libwebp-1.2.0-8.el9_3.x86_64.rpm                                                   5.4 MB/s | 276 kB     00:00
(42/114): libxcb-1.13.1-9.el9.x86_64.rpm                                                     6.2 MB/s | 225 kB     00:00
(43/114): libwebp-devel-1.2.0-8.el9_3.x86_64.rpm                                             336 kB/s |  32 kB     00:00
(44/114): libxslt-1.1.34-14.el9_7.1.x86_64.rpm                                               4.1 MB/s | 240 kB     00:00
(45/114): libxcb-devel-1.13.1-9.el9.x86_64.rpm                                               5.3 MB/s | 1.0 MB     00:00
(46/114): libxslt-devel-1.1.34-14.el9_7.1.x86_64.rpm                                         3.6 MB/s | 287 kB     00:00
(47/114): libxml2-devel-2.9.13-14.el9_7.x86_64.rpm                                           4.6 MB/s | 828 kB     00:00
(48/114): pcre-cpp-8.44-4.el9.x86_64.rpm                                                     592 kB/s |  24 kB     00:00
(49/114): pcre-devel-8.44-4.el9.x86_64.rpm                                                   8.3 MB/s | 469 kB     00:00
(50/114): pcre-utf32-8.44-4.el9.x86_64.rpm                                                   3.9 MB/s | 173 kB     00:00
(51/114): pcre-utf16-8.44-4.el9.x86_64.rpm                                                   2.2 MB/s | 183 kB     00:00
(52/114): pcre2-devel-10.40-6.el9.x86_64.rpm                                                 7.6 MB/s | 471 kB     00:00
(53/114): pcre2-utf16-10.40-6.el9.x86_64.rpm                                                 2.7 MB/s | 213 kB     00:00
(54/114): perl-AutoSplit-5.74-481.1.el9_6.noarch.rpm                                         800 kB/s |  20 kB     00:00
(55/114): pcre2-utf32-10.40-6.el9.x86_64.rpm                                                 2.9 MB/s | 202 kB     00:00
(56/114): perl-CPAN-Meta-2.150010-460.el9.noarch.rpm                                         2.9 MB/s | 176 kB     00:00
(57/114): perl-Benchmark-1.23-481.1.el9_6.noarch.rpm                                         356 kB/s |  25 kB     00:00
(58/114): perl-CPAN-Meta-YAML-0.018-461.el9.noarch.rpm                                       1.3 MB/s |  26 kB     00:00
(59/114): perl-CPAN-Meta-Requirements-2.140-461.el9.noarch.rpm                               882 kB/s |  31 kB     00:00
(60/114): perl-Encode-Locale-1.05-21.el9.noarch.rpm                                          1.5 MB/s |  19 kB     00:00
(61/114): perl-ExtUtils-Command-7.60-3.el9.noarch.rpm                                        1.1 MB/s |  14 kB     00:00
(62/114): perl-ExtUtils-Constant-0.25-481.1.el9_6.noarch.rpm                                 3.7 MB/s |  45 kB     00:00
(63/114): perl-ExtUtils-Embed-1.35-481.1.el9_6.noarch.rpm                                    1.0 MB/s |  16 kB     00:00
(64/114): perl-Devel-PPPort-3.62-4.el9.x86_64.rpm                                            2.8 MB/s | 211 kB     00:00
(65/114): openssl-devel-3.5.5-2.el9_8.x86_64.rpm                                             8.0 MB/s | 3.4 MB     00:00
(66/114): perl-ExtUtils-Install-2.20-4.el9.noarch.rpm                                        765 kB/s |  44 kB     00:00
(67/114): perl-ExtUtils-Manifest-1.73-4.el9.noarch.rpm                                       1.7 MB/s |  34 kB     00:00
(68/114): perl-Fedora-VSP-0.001-23.el9.noarch.rpm                                            1.4 MB/s |  23 kB     00:00
(69/114): perl-ExtUtils-ParseXS-3.40-460.el9.noarch.rpm                                      7.7 MB/s | 181 kB     00:00
(70/114): perl-File-Copy-2.34-481.1.el9_6.noarch.rpm                                         1.7 MB/s |  19 kB     00:00
(71/114): perl-File-Compare-1.100.600-481.1.el9_6.noarch.rpm                                 427 kB/s |  12 kB     00:00
(72/114): perl-I18N-Langinfo-0.19-481.1.el9_6.x86_64.rpm                                     1.1 MB/s |  21 kB     00:00
(73/114): perl-File-Find-1.37-481.1.el9_6.noarch.rpm                                         537 kB/s |  24 kB     00:00
(74/114): perl-Math-BigInt-1.9998.18-460.el9.noarch.rpm                                      5.4 MB/s | 188 kB     00:00
(75/114): perl-JSON-PP-4.06-4.el9.noarch.rpm                                                 1.1 MB/s |  65 kB     00:00
(76/114): perl-Math-Complex-1.59-481.1.el9_6.noarch.rpm                                      2.8 MB/s |  45 kB     00:00
(77/114): perl-ExtUtils-MakeMaker-7.60-3.el9.noarch.rpm                                      1.4 MB/s | 289 kB     00:00
(78/114): perl-Time-HiRes-1.9764-462.el9.x86_64.rpm                                          1.5 MB/s |  57 kB     00:00
(79/114): perl-Test-Harness-3.42-461.el9.noarch.rpm                                          4.8 MB/s | 267 kB     00:00
(80/114): perl-generators-1.13-1.el9.noarch.rpm                                              481 kB/s |  15 kB     00:00
(81/114): perl-locale-1.09-481.1.el9_6.noarch.rpm                                            401 kB/s |  12 kB     00:00
(82/114): perl-macros-5.32.1-481.1.el9_6.noarch.rpm                                          628 kB/s | 9.2 kB     00:00
(83/114): perl-version-0.99.28-4.el9.x86_64.rpm                                              3.2 MB/s |  62 kB     00:00
(84/114): pixman-0.40.0-6.el9_3.x86_64.rpm                                                   5.8 MB/s | 268 kB     00:00
(85/114): perl-devel-5.32.1-481.1.el9_6.x86_64.rpm                                           3.3 MB/s | 659 kB     00:00
(86/114): sysprof-capture-devel-3.40.1-3.el9.x86_64.rpm                                      2.3 MB/s |  59 kB     00:00
(87/114): systemtap-sdt-devel-5.4-4.el9.x86_64.rpm                                           2.3 MB/s |  68 kB     00:00
(88/114): systemtap-sdt-dtrace-5.4-4.el9.x86_64.rpm                                          2.2 MB/s |  69 kB     00:00
(89/114): xml-common-0.6.3-58.el9.noarch.rpm                                                 2.0 MB/s |  31 kB     00:00
(90/114): xz-devel-5.2.5-8.el9_0.x86_64.rpm                                                  1.4 MB/s |  52 kB     00:00
(91/114): xorg-x11-proto-devel-2024.1-1.el9.noarch.rpm                                       4.5 MB/s | 265 kB     00:00
(92/114): zlib-devel-1.2.11-40.el9.x86_64.rpm                                                1.0 MB/s |  44 kB     00:00
(93/114): fonts-filesystem-2.0.5-7.el9.1.noarch.rpm                                          443 kB/s | 9.0 kB     00:00
(94/114): perl-doc-5.32.1-481.1.el9_6.noarch.rpm                                             9.2 MB/s | 4.5 MB     00:00
(95/114): freetype-2.10.4-10.el9_5.x86_64.rpm                                                2.1 MB/s | 385 kB     00:00
(96/114): dejavu-sans-fonts-2.37-18.el9.noarch.rpm                                           5.6 MB/s | 1.3 MB     00:00
(97/114): graphite2-1.3.14-9.el9.x86_64.rpm                                                  2.2 MB/s |  94 kB     00:00
(98/114): libpng-1.6.37-15.el9_8.x86_64.rpm                                                  2.2 MB/s | 115 kB     00:00
(99/114): python3-pyparsing-2.4.7-9.el9.noarch.rpm                                           5.4 MB/s | 149 kB     00:00
(100/114): harfbuzz-2.7.4-10.el9.x86_64.rpm                                                  4.3 MB/s | 623 kB     00:00
(101/114): libblkid-2.37.4-25.el9.x86_64.rpm                                                 3.2 MB/s | 105 kB     00:00
(102/114): libbrotli-1.0.9-9.el9_7.x86_64.rpm                                                3.4 MB/s | 311 kB     00:00
(103/114): libfdisk-2.37.4-25.el9.x86_64.rpm                                                 2.7 MB/s | 152 kB     00:00
(104/114): libmount-2.37.4-25.el9.x86_64.rpm                                                 2.7 MB/s | 133 kB     00:00
(105/114): glib2-2.68.4-19.el9_8.1.x86_64.rpm                                                7.3 MB/s | 2.6 MB     00:00
(106/114): libsmartcols-2.37.4-25.el9.x86_64.rpm                                             1.4 MB/s |  61 kB     00:00
(107/114): gnupg2-2.3.3-5.el9_7.x86_64.rpm                                                   6.9 MB/s | 2.5 MB     00:00
(108/114): libuuid-2.37.4-25.el9.x86_64.rpm                                                  730 kB/s |  26 kB     00:00
(109/114): libxml2-2.9.13-14.el9_7.x86_64.rpm                                                4.9 MB/s | 746 kB     00:00
(110/114): openssl-3.5.5-2.el9_8.x86_64.rpm                                                  9.6 MB/s | 1.4 MB     00:00
(111/114): openssl-fips-provider-3.5.5-2.el9_8.x86_64.rpm                                    4.1 MB/s | 814 kB     00:00
(112/114): util-linux-core-2.37.4-25.el9.x86_64.rpm                                          4.5 MB/s | 430 kB     00:00
(113/114): openssl-libs-3.5.5-2.el9_8.x86_64.rpm                                             8.7 MB/s | 2.3 MB     00:00
(114/114): util-linux-2.37.4-25.el9.x86_64.rpm                                               7.3 MB/s | 2.2 MB     00:00
-----------------------------------------------------------------------------------------------------------------------------
Total                                                                                        9.5 MB/s |  40 MB     00:04
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                     1/1
  Upgrading        : libuuid-2.37.4-25.el9.x86_64                                                                      1/128
  Upgrading        : libblkid-2.37.4-25.el9.x86_64                                                                     2/128
  Running scriptlet: libblkid-2.37.4-25.el9.x86_64                                                                     2/128
  Installing       : zlib-devel-1.2.11-40.el9.x86_64                                                                   3/128
  Installing       : perl-File-Find-1.37-481.1.el9_6.noarch                                                            4/128
  Upgrading        : libmount-2.37.4-25.el9.x86_64                                                                     5/128
  Upgrading        : libxml2-2.9.13-14.el9_7.x86_64                                                                    6/128
  Installing       : libpng-2:1.6.37-15.el9_8.x86_64                                                                   7/128
  Upgrading        : glib2-2.68.4-19.el9_8.1.x86_64                                                                    8/128
  Upgrading        : libbrotli-1.0.9-9.el9_7.x86_64                                                                    9/128
  Installing       : perl-File-Copy-2.34-481.1.el9_6.noarch                                                           10/128
  Installing       : libwebp-1.2.0-8.el9_3.x86_64                                                                     11/128
  Installing       : libjpeg-turbo-2.0.90-7.el9.x86_64                                                                12/128
  Installing       : libpng-devel-2:1.6.37-15.el9_8.x86_64                                                            13/128
  Upgrading        : libsmartcols-2.37.4-25.el9.x86_64                                                                14/128
  Installing       : graphite2-1.3.14-9.el9.x86_64                                                                    15/128
  Installing       : harfbuzz-2.7.4-10.el9.x86_64                                                                     16/128
  Installing       : freetype-2.10.4-10.el9_5.x86_64                                                                  17/128
  Installing       : fonts-filesystem-1:2.0.5-7.el9.1.noarch                                                          18/128
  Installing       : dejavu-sans-fonts-2.37-18.el9.noarch                                                             19/128
  Installing       : xorg-x11-proto-devel-2024.1-1.el9.noarch                                                         20/128
  Installing       : perl-Time-HiRes-4:1.9764-462.el9.x86_64                                                          21/128
  Installing       : perl-File-Compare-1.100.600-481.1.el9_6.noarch                                                   22/128
  Installing       : perl-ExtUtils-ParseXS-1:3.40-460.el9.noarch                                                      23/128
  Installing       : libXau-1.0.9-8.el9.x86_64                                                                        24/128
  Installing       : libxcb-1.13.1-9.el9.x86_64                                                                       25/128
  Installing       : libICE-1.0.10-8.el9.x86_64                                                                       26/128
  Installing       : libSM-1.2.3-10.el9.x86_64                                                                        27/128
  Installing       : libXau-devel-1.0.9-8.el9.x86_64                                                                  28/128
  Installing       : libxcb-devel-1.13.1-9.el9.x86_64                                                                 29/128
  Installing       : langpacks-core-font-en-3.0-16.el9.noarch                                                         30/128
  Installing       : harfbuzz-icu-2.7.4-10.el9.x86_64                                                                 31/128
  Installing       : graphite2-devel-1.3.14-9.el9.x86_64                                                              32/128
  Upgrading        : util-linux-core-2.37.4-25.el9.x86_64                                                             33/128
  Running scriptlet: util-linux-core-2.37.4-25.el9.x86_64                                                             33/128
  Installing       : libjpeg-turbo-devel-2.0.90-7.el9.x86_64                                                          34/128
  Installing       : libwebp-devel-1.2.0-8.el9_3.x86_64                                                               35/128
  Installing       : perl-ExtUtils-Command-2:7.60-3.el9.noarch                                                        36/128
  Installing       : perl-ExtUtils-Manifest-1:1.73-4.el9.noarch                                                       37/128
  Installing       : brotli-1.0.9-9.el9_7.x86_64                                                                      38/128
  Installing       : brotli-devel-1.0.9-9.el9_7.x86_64                                                                39/128
  Installing       : libxslt-1.1.34-14.el9_7.1.x86_64                                                                 40/128
  Running scriptlet: xml-common-0.6.3-58.el9.noarch                                                                   41/128
  Installing       : xml-common-0.6.3-58.el9.noarch                                                                   41/128
  Installing       : fontconfig-2.14.0-2.el9_1.x86_64                                                                 42/128
  Running scriptlet: fontconfig-2.14.0-2.el9_1.x86_64                                                                 42/128
  Installing       : libblkid-devel-2.37.4-25.el9.x86_64                                                              43/128
  Upgrading        : libfdisk-2.37.4-25.el9.x86_64                                                                    44/128
  Upgrading        : openssl-libs-1:3.5.5-2.el9_8.x86_64                                                              45/128
  Upgrading        : openssl-fips-provider-1:3.5.5-2.el9_8.x86_64                                                     46/128
  Installing       : python3-pyparsing-2.4.7-9.el9.noarch                                                             47/128
  Installing       : systemtap-sdt-dtrace-5.4-4.el9.x86_64                                                            48/128
  Installing       : systemtap-sdt-devel-5.4-4.el9.x86_64                                                             49/128
  Installing       : xz-devel-5.2.5-8.el9_0.x86_64                                                                    50/128
  Installing       : libxml2-devel-2.9.13-14.el9_7.x86_64                                                             51/128
  Installing       : sysprof-capture-devel-3.40.1-3.el9.x86_64                                                        52/128
  Installing       : pixman-0.40.0-6.el9_3.x86_64                                                                     53/128
  Installing       : perl-macros-4:5.32.1-481.1.el9_6.noarch                                                          54/128
  Installing       : perl-locale-1.09-481.1.el9_6.noarch                                                              55/128
  Installing       : perl-version-7:0.99.28-4.el9.x86_64                                                              56/128
  Installing       : perl-CPAN-Meta-Requirements-2.140-461.el9.noarch                                                 57/128
  Installing       : perl-doc-5.32.1-481.1.el9_6.noarch                                                               58/128
  Installing       : perl-Math-Complex-1.59-481.1.el9_6.noarch                                                        59/128
  Installing       : perl-Math-BigInt-1:1.9998.18-460.el9.noarch                                                      60/128
  Installing       : perl-JSON-PP-1:4.06-4.el9.noarch                                                                 61/128
  Installing       : perl-I18N-Langinfo-0.19-481.1.el9_6.x86_64                                                       62/128
  Installing       : perl-Encode-Locale-1.05-21.el9.noarch                                                            63/128
  Installing       : perl-Fedora-VSP-0.001-23.el9.noarch                                                              64/128
  Installing       : perl-ExtUtils-Constant-0.25-481.1.el9_6.noarch                                                   65/128
  Installing       : perl-Devel-PPPort-3.62-4.el9.x86_64                                                              66/128
  Installing       : perl-CPAN-Meta-YAML-0.018-461.el9.noarch                                                         67/128
  Installing       : perl-CPAN-Meta-2.150010-460.el9.noarch                                                           68/128
  Installing       : perl-Benchmark-1.23-481.1.el9_6.noarch                                                           69/128
  Installing       : perl-Test-Harness-1:3.42-461.el9.noarch                                                          70/128
  Installing       : perl-AutoSplit-5.74-481.1.el9_6.noarch                                                           71/128
  Installing       : perl-ExtUtils-MakeMaker-2:7.60-3.el9.noarch                                                      72/128
  Installing       : perl-ExtUtils-Install-2.20-4.el9.noarch                                                          73/128
  Installing       : perl-devel-4:5.32.1-481.1.el9_6.x86_64                                                           74/128
  Installing       : pcre2-utf32-10.40-6.el9.x86_64                                                                   75/128
  Installing       : pcre2-utf16-10.40-6.el9.x86_64                                                                   76/128
  Installing       : pcre2-devel-10.40-6.el9.x86_64                                                                   77/128
  Installing       : pcre-utf32-8.44-4.el9.x86_64                                                                     78/128
  Installing       : pcre-utf16-8.44-4.el9.x86_64                                                                     79/128
  Installing       : pcre-cpp-8.44-4.el9.x86_64                                                                       80/128
  Installing       : pcre-devel-8.44-4.el9.x86_64                                                                     81/128
  Installing       : libsepol-devel-3.6-3.el9.x86_64                                                                  82/128
  Installing       : libselinux-devel-3.6-3.el9.x86_64                                                                83/128
  Installing       : libmount-devel-2.37.4-25.el9.x86_64                                                              84/128
  Installing       : libicu-devel-67.1-10.el9_6.x86_64                                                                85/128
  Installing       : libgpg-error-devel-1.42-5.el9.x86_64                                                             86/128
  Installing       : libffi-devel-3.4.2-8.el9.x86_64                                                                  87/128
  Installing       : glib2-devel-2.68.4-19.el9_8.1.x86_64                                                             88/128
  Installing       : libX11-xcb-1.8.12-1.el9.x86_64                                                                   89/128
  Installing       : libX11-common-1.8.12-1.el9.noarch                                                                90/128
  Installing       : libX11-1.8.12-1.el9.x86_64                                                                       91/128
  Installing       : libX11-devel-1.8.12-1.el9.x86_64                                                                 92/128
  Installing       : libXext-1.3.4-8.el9.x86_64                                                                       93/128
  Installing       : libXpm-3.5.13-10.el9.x86_64                                                                      94/128
  Installing       : libXrender-0.9.10-16.el9.x86_64                                                                  95/128
  Installing       : cairo-1.17.4-7.el9.x86_64                                                                        96/128
  Installing       : libXt-1.2.0-6.el9.x86_64                                                                         97/128
  Installing       : libXpm-devel-3.5.13-10.el9.x86_64                                                                98/128
  Installing       : jbigkit-libs-2.1-23.el9.x86_64                                                                   99/128
  Installing       : libtiff-4.4.0-18.el9_8.x86_64                                                                   100/128
  Installing       : gd-2.3.2-3.el9.x86_64                                                                           101/128
  Installing       : libtiff-devel-4.4.0-18.el9_8.x86_64                                                             102/128
  Installing       : bzip2-devel-1.0.8-11.el9.x86_64                                                                 103/128
  Installing       : harfbuzz-devel-2.7.4-10.el9.x86_64                                                              104/128
  Installing       : freetype-devel-2.10.4-10.el9_5.x86_64                                                           105/128
  Installing       : fontconfig-devel-2.14.0-2.el9_1.x86_64                                                          106/128
  Installing       : gd-devel-2.3.2-3.el9.x86_64                                                                     107/128
  Installing       : libxslt-devel-1.1.34-14.el9_7.1.x86_64                                                          108/128
  Installing       : perl-ExtUtils-Embed-1.35-481.1.el9_6.noarch                                                     109/128
  Installing       : perl-generators-1.13-1.el9.noarch                                                               110/128
  Installing       : openssl-devel-1:3.5.5-2.el9_8.x86_64                                                            111/128
  Upgrading        : openssl-1:3.5.5-2.el9_8.x86_64                                                                  112/128
  Upgrading        : util-linux-2.37.4-25.el9.x86_64                                                                 113/128
  Upgrading        : gnupg2-2.3.3-5.el9_7.x86_64                                                                     114/128
  Cleanup          : util-linux-2.37.4-21.el9.x86_64                                                                 115/128
  Cleanup          : util-linux-core-2.37.4-21.el9.x86_64                                                            116/128
  Cleanup          : openssl-1:3.5.1-3.el9.x86_64                                                                    117/128
  Cleanup          : libfdisk-2.37.4-21.el9.x86_64                                                                   118/128
  Cleanup          : glib2-2.68.4-18.el9_7.x86_64                                                                    119/128
  Cleanup          : libmount-2.37.4-21.el9.x86_64                                                                   120/128
  Cleanup          : libblkid-2.37.4-21.el9.x86_64                                                                   121/128
  Cleanup          : openssl-fips-provider-1:3.5.1-3.el9.x86_64                                                      122/128
  Cleanup          : openssl-libs-1:3.5.1-3.el9.x86_64                                                               123/128
  Cleanup          : libuuid-2.37.4-21.el9.x86_64                                                                    124/128
  Cleanup          : libsmartcols-2.37.4-21.el9.x86_64                                                               125/128
  Cleanup          : libxml2-2.9.13-12.el9_6.x86_64                                                                  126/128
  Cleanup          : libbrotli-1.0.9-7.el9_5.x86_64                                                                  127/128
  Cleanup          : gnupg2-2.3.3-4.el9.x86_64                                                                       128/128
  Running scriptlet: fontconfig-2.14.0-2.el9_1.x86_64                                                                128/128
  Running scriptlet: gnupg2-2.3.3-4.el9.x86_64                                                                       128/128
  Verifying        : brotli-1.0.9-9.el9_7.x86_64                                                                       1/128
  Verifying        : brotli-devel-1.0.9-9.el9_7.x86_64                                                                 2/128
  Verifying        : bzip2-devel-1.0.8-11.el9.x86_64                                                                   3/128
  Verifying        : cairo-1.17.4-7.el9.x86_64                                                                         4/128
  Verifying        : fontconfig-2.14.0-2.el9_1.x86_64                                                                  5/128
  Verifying        : fontconfig-devel-2.14.0-2.el9_1.x86_64                                                            6/128
  Verifying        : freetype-devel-2.10.4-10.el9_5.x86_64                                                             7/128
  Verifying        : gd-2.3.2-3.el9.x86_64                                                                             8/128
  Verifying        : gd-devel-2.3.2-3.el9.x86_64                                                                       9/128
  Verifying        : glib2-devel-2.68.4-19.el9_8.1.x86_64                                                             10/128
  Verifying        : graphite2-devel-1.3.14-9.el9.x86_64                                                              11/128
  Verifying        : harfbuzz-devel-2.7.4-10.el9.x86_64                                                               12/128
  Verifying        : harfbuzz-icu-2.7.4-10.el9.x86_64                                                                 13/128
  Verifying        : jbigkit-libs-2.1-23.el9.x86_64                                                                   14/128
  Verifying        : langpacks-core-font-en-3.0-16.el9.noarch                                                         15/128
  Verifying        : libICE-1.0.10-8.el9.x86_64                                                                       16/128
  Verifying        : libSM-1.2.3-10.el9.x86_64                                                                        17/128
  Verifying        : libX11-1.8.12-1.el9.x86_64                                                                       18/128
  Verifying        : libX11-common-1.8.12-1.el9.noarch                                                                19/128
  Verifying        : libX11-devel-1.8.12-1.el9.x86_64                                                                 20/128
  Verifying        : libX11-xcb-1.8.12-1.el9.x86_64                                                                   21/128
  Verifying        : libXau-1.0.9-8.el9.x86_64                                                                        22/128
  Verifying        : libXau-devel-1.0.9-8.el9.x86_64                                                                  23/128
  Verifying        : libXext-1.3.4-8.el9.x86_64                                                                       24/128
  Verifying        : libXpm-3.5.13-10.el9.x86_64                                                                      25/128
  Verifying        : libXpm-devel-3.5.13-10.el9.x86_64                                                                26/128
  Verifying        : libXrender-0.9.10-16.el9.x86_64                                                                  27/128
  Verifying        : libXt-1.2.0-6.el9.x86_64                                                                         28/128
  Verifying        : libblkid-devel-2.37.4-25.el9.x86_64                                                              29/128
  Verifying        : libffi-devel-3.4.2-8.el9.x86_64                                                                  30/128
  Verifying        : libgpg-error-devel-1.42-5.el9.x86_64                                                             31/128
  Verifying        : libicu-devel-67.1-10.el9_6.x86_64                                                                32/128
  Verifying        : libjpeg-turbo-2.0.90-7.el9.x86_64                                                                33/128
  Verifying        : libjpeg-turbo-devel-2.0.90-7.el9.x86_64                                                          34/128
  Verifying        : libmount-devel-2.37.4-25.el9.x86_64                                                              35/128
  Verifying        : libpng-devel-2:1.6.37-15.el9_8.x86_64                                                            36/128
  Verifying        : libselinux-devel-3.6-3.el9.x86_64                                                                37/128
  Verifying        : libsepol-devel-3.6-3.el9.x86_64                                                                  38/128
  Verifying        : libtiff-4.4.0-18.el9_8.x86_64                                                                    39/128
  Verifying        : libtiff-devel-4.4.0-18.el9_8.x86_64                                                              40/128
  Verifying        : libwebp-1.2.0-8.el9_3.x86_64                                                                     41/128
  Verifying        : libwebp-devel-1.2.0-8.el9_3.x86_64                                                               42/128
  Verifying        : libxcb-1.13.1-9.el9.x86_64                                                                       43/128
  Verifying        : libxcb-devel-1.13.1-9.el9.x86_64                                                                 44/128
  Verifying        : libxml2-devel-2.9.13-14.el9_7.x86_64                                                             45/128
  Verifying        : libxslt-1.1.34-14.el9_7.1.x86_64                                                                 46/128
  Verifying        : libxslt-devel-1.1.34-14.el9_7.1.x86_64                                                           47/128
  Verifying        : openssl-devel-1:3.5.5-2.el9_8.x86_64                                                             48/128
  Verifying        : pcre-cpp-8.44-4.el9.x86_64                                                                       49/128
  Verifying        : pcre-devel-8.44-4.el9.x86_64                                                                     50/128
  Verifying        : pcre-utf16-8.44-4.el9.x86_64                                                                     51/128
  Verifying        : pcre-utf32-8.44-4.el9.x86_64                                                                     52/128
  Verifying        : pcre2-devel-10.40-6.el9.x86_64                                                                   53/128
  Verifying        : pcre2-utf16-10.40-6.el9.x86_64                                                                   54/128
  Verifying        : pcre2-utf32-10.40-6.el9.x86_64                                                                   55/128
  Verifying        : perl-AutoSplit-5.74-481.1.el9_6.noarch                                                           56/128
  Verifying        : perl-Benchmark-1.23-481.1.el9_6.noarch                                                           57/128
  Verifying        : perl-CPAN-Meta-2.150010-460.el9.noarch                                                           58/128
  Verifying        : perl-CPAN-Meta-Requirements-2.140-461.el9.noarch                                                 59/128
  Verifying        : perl-CPAN-Meta-YAML-0.018-461.el9.noarch                                                         60/128
  Verifying        : perl-Devel-PPPort-3.62-4.el9.x86_64                                                              61/128
  Verifying        : perl-Encode-Locale-1.05-21.el9.noarch                                                            62/128
  Verifying        : perl-ExtUtils-Command-2:7.60-3.el9.noarch                                                        63/128
  Verifying        : perl-ExtUtils-Constant-0.25-481.1.el9_6.noarch                                                   64/128
  Verifying        : perl-ExtUtils-Embed-1.35-481.1.el9_6.noarch                                                      65/128
  Verifying        : perl-ExtUtils-Install-2.20-4.el9.noarch                                                          66/128
  Verifying        : perl-ExtUtils-MakeMaker-2:7.60-3.el9.noarch                                                      67/128
  Verifying        : perl-ExtUtils-Manifest-1:1.73-4.el9.noarch                                                       68/128
  Verifying        : perl-ExtUtils-ParseXS-1:3.40-460.el9.noarch                                                      69/128
  Verifying        : perl-Fedora-VSP-0.001-23.el9.noarch                                                              70/128
  Verifying        : perl-File-Compare-1.100.600-481.1.el9_6.noarch                                                   71/128
  Verifying        : perl-File-Copy-2.34-481.1.el9_6.noarch                                                           72/128
  Verifying        : perl-File-Find-1.37-481.1.el9_6.noarch                                                           73/128
  Verifying        : perl-I18N-Langinfo-0.19-481.1.el9_6.x86_64                                                       74/128
  Verifying        : perl-JSON-PP-1:4.06-4.el9.noarch                                                                 75/128
  Verifying        : perl-Math-BigInt-1:1.9998.18-460.el9.noarch                                                      76/128
  Verifying        : perl-Math-Complex-1.59-481.1.el9_6.noarch                                                        77/128
  Verifying        : perl-Test-Harness-1:3.42-461.el9.noarch                                                          78/128
  Verifying        : perl-Time-HiRes-4:1.9764-462.el9.x86_64                                                          79/128
  Verifying        : perl-devel-4:5.32.1-481.1.el9_6.x86_64                                                           80/128
  Verifying        : perl-doc-5.32.1-481.1.el9_6.noarch                                                               81/128
  Verifying        : perl-generators-1.13-1.el9.noarch                                                                82/128
  Verifying        : perl-locale-1.09-481.1.el9_6.noarch                                                              83/128
  Verifying        : perl-macros-4:5.32.1-481.1.el9_6.noarch                                                          84/128
  Verifying        : perl-version-7:0.99.28-4.el9.x86_64                                                              85/128
  Verifying        : pixman-0.40.0-6.el9_3.x86_64                                                                     86/128
  Verifying        : sysprof-capture-devel-3.40.1-3.el9.x86_64                                                        87/128
  Verifying        : systemtap-sdt-devel-5.4-4.el9.x86_64                                                             88/128
  Verifying        : systemtap-sdt-dtrace-5.4-4.el9.x86_64                                                            89/128
  Verifying        : xml-common-0.6.3-58.el9.noarch                                                                   90/128
  Verifying        : xorg-x11-proto-devel-2024.1-1.el9.noarch                                                         91/128
  Verifying        : xz-devel-5.2.5-8.el9_0.x86_64                                                                    92/128
  Verifying        : zlib-devel-1.2.11-40.el9.x86_64                                                                  93/128
  Verifying        : dejavu-sans-fonts-2.37-18.el9.noarch                                                             94/128
  Verifying        : fonts-filesystem-1:2.0.5-7.el9.1.noarch                                                          95/128
  Verifying        : freetype-2.10.4-10.el9_5.x86_64                                                                  96/128
  Verifying        : graphite2-1.3.14-9.el9.x86_64                                                                    97/128
  Verifying        : harfbuzz-2.7.4-10.el9.x86_64                                                                     98/128
  Verifying        : libpng-2:1.6.37-15.el9_8.x86_64                                                                  99/128
  Verifying        : python3-pyparsing-2.4.7-9.el9.noarch                                                            100/128
  Verifying        : glib2-2.68.4-19.el9_8.1.x86_64                                                                  101/128
  Verifying        : glib2-2.68.4-18.el9_7.x86_64                                                                    102/128
  Verifying        : gnupg2-2.3.3-5.el9_7.x86_64                                                                     103/128
  Verifying        : gnupg2-2.3.3-4.el9.x86_64                                                                       104/128
  Verifying        : libblkid-2.37.4-25.el9.x86_64                                                                   105/128
  Verifying        : libblkid-2.37.4-21.el9.x86_64                                                                   106/128
  Verifying        : libbrotli-1.0.9-9.el9_7.x86_64                                                                  107/128
  Verifying        : libbrotli-1.0.9-7.el9_5.x86_64                                                                  108/128
  Verifying        : libfdisk-2.37.4-25.el9.x86_64                                                                   109/128
  Verifying        : libfdisk-2.37.4-21.el9.x86_64                                                                   110/128
  Verifying        : libmount-2.37.4-25.el9.x86_64                                                                   111/128
  Verifying        : libmount-2.37.4-21.el9.x86_64                                                                   112/128
  Verifying        : libsmartcols-2.37.4-25.el9.x86_64                                                               113/128
  Verifying        : libsmartcols-2.37.4-21.el9.x86_64                                                               114/128
  Verifying        : libuuid-2.37.4-25.el9.x86_64                                                                    115/128
  Verifying        : libuuid-2.37.4-21.el9.x86_64                                                                    116/128
  Verifying        : libxml2-2.9.13-14.el9_7.x86_64                                                                  117/128
  Verifying        : libxml2-2.9.13-12.el9_6.x86_64                                                                  118/128
  Verifying        : openssl-1:3.5.5-2.el9_8.x86_64                                                                  119/128
  Verifying        : openssl-1:3.5.1-3.el9.x86_64                                                                    120/128
  Verifying        : openssl-fips-provider-1:3.5.5-2.el9_8.x86_64                                                    121/128
  Verifying        : openssl-fips-provider-1:3.5.1-3.el9.x86_64                                                      122/128
  Verifying        : openssl-libs-1:3.5.5-2.el9_8.x86_64                                                             123/128
  Verifying        : openssl-libs-1:3.5.1-3.el9.x86_64                                                               124/128
  Verifying        : util-linux-2.37.4-25.el9.x86_64                                                                 125/128
  Verifying        : util-linux-2.37.4-21.el9.x86_64                                                                 126/128
  Verifying        : util-linux-core-2.37.4-25.el9.x86_64                                                            127/128
  Verifying        : util-linux-core-2.37.4-21.el9.x86_64                                                            128/128

Upgraded:
  glib2-2.68.4-19.el9_8.1.x86_64       gnupg2-2.3.3-5.el9_7.x86_64                     libblkid-2.37.4-25.el9.x86_64
  libbrotli-1.0.9-9.el9_7.x86_64       libfdisk-2.37.4-25.el9.x86_64                   libmount-2.37.4-25.el9.x86_64
  libsmartcols-2.37.4-25.el9.x86_64    libuuid-2.37.4-25.el9.x86_64                    libxml2-2.9.13-14.el9_7.x86_64
  openssl-1:3.5.5-2.el9_8.x86_64       openssl-fips-provider-1:3.5.5-2.el9_8.x86_64    openssl-libs-1:3.5.5-2.el9_8.x86_64
  util-linux-2.37.4-25.el9.x86_64      util-linux-core-2.37.4-25.el9.x86_64
Installed:
  brotli-1.0.9-9.el9_7.x86_64                                    brotli-devel-1.0.9-9.el9_7.x86_64
  bzip2-devel-1.0.8-11.el9.x86_64                                cairo-1.17.4-7.el9.x86_64
  dejavu-sans-fonts-2.37-18.el9.noarch                           fontconfig-2.14.0-2.el9_1.x86_64
  fontconfig-devel-2.14.0-2.el9_1.x86_64                         fonts-filesystem-1:2.0.5-7.el9.1.noarch
  freetype-2.10.4-10.el9_5.x86_64                                freetype-devel-2.10.4-10.el9_5.x86_64
  gd-2.3.2-3.el9.x86_64                                          gd-devel-2.3.2-3.el9.x86_64
  glib2-devel-2.68.4-19.el9_8.1.x86_64                           graphite2-1.3.14-9.el9.x86_64
  graphite2-devel-1.3.14-9.el9.x86_64                            harfbuzz-2.7.4-10.el9.x86_64
  harfbuzz-devel-2.7.4-10.el9.x86_64                             harfbuzz-icu-2.7.4-10.el9.x86_64
  jbigkit-libs-2.1-23.el9.x86_64                                 langpacks-core-font-en-3.0-16.el9.noarch
  libICE-1.0.10-8.el9.x86_64                                     libSM-1.2.3-10.el9.x86_64
  libX11-1.8.12-1.el9.x86_64                                     libX11-common-1.8.12-1.el9.noarch
  libX11-devel-1.8.12-1.el9.x86_64                               libX11-xcb-1.8.12-1.el9.x86_64
  libXau-1.0.9-8.el9.x86_64                                      libXau-devel-1.0.9-8.el9.x86_64
  libXext-1.3.4-8.el9.x86_64                                     libXpm-3.5.13-10.el9.x86_64
  libXpm-devel-3.5.13-10.el9.x86_64                              libXrender-0.9.10-16.el9.x86_64
  libXt-1.2.0-6.el9.x86_64                                       libblkid-devel-2.37.4-25.el9.x86_64
  libffi-devel-3.4.2-8.el9.x86_64                                libgpg-error-devel-1.42-5.el9.x86_64
  libicu-devel-67.1-10.el9_6.x86_64                              libjpeg-turbo-2.0.90-7.el9.x86_64
  libjpeg-turbo-devel-2.0.90-7.el9.x86_64                        libmount-devel-2.37.4-25.el9.x86_64
  libpng-2:1.6.37-15.el9_8.x86_64                                libpng-devel-2:1.6.37-15.el9_8.x86_64
  libselinux-devel-3.6-3.el9.x86_64                              libsepol-devel-3.6-3.el9.x86_64
  libtiff-4.4.0-18.el9_8.x86_64                                  libtiff-devel-4.4.0-18.el9_8.x86_64
  libwebp-1.2.0-8.el9_3.x86_64                                   libwebp-devel-1.2.0-8.el9_3.x86_64
  libxcb-1.13.1-9.el9.x86_64                                     libxcb-devel-1.13.1-9.el9.x86_64
  libxml2-devel-2.9.13-14.el9_7.x86_64                           libxslt-1.1.34-14.el9_7.1.x86_64
  libxslt-devel-1.1.34-14.el9_7.1.x86_64                         openssl-devel-1:3.5.5-2.el9_8.x86_64
  pcre-cpp-8.44-4.el9.x86_64                                     pcre-devel-8.44-4.el9.x86_64
  pcre-utf16-8.44-4.el9.x86_64                                   pcre-utf32-8.44-4.el9.x86_64
  pcre2-devel-10.40-6.el9.x86_64                                 pcre2-utf16-10.40-6.el9.x86_64
  pcre2-utf32-10.40-6.el9.x86_64                                 perl-AutoSplit-5.74-481.1.el9_6.noarch
  perl-Benchmark-1.23-481.1.el9_6.noarch                         perl-CPAN-Meta-2.150010-460.el9.noarch
  perl-CPAN-Meta-Requirements-2.140-461.el9.noarch               perl-CPAN-Meta-YAML-0.018-461.el9.noarch
  perl-Devel-PPPort-3.62-4.el9.x86_64                            perl-Encode-Locale-1.05-21.el9.noarch
  perl-ExtUtils-Command-2:7.60-3.el9.noarch                      perl-ExtUtils-Constant-0.25-481.1.el9_6.noarch
  perl-ExtUtils-Embed-1.35-481.1.el9_6.noarch                    perl-ExtUtils-Install-2.20-4.el9.noarch
  perl-ExtUtils-MakeMaker-2:7.60-3.el9.noarch                    perl-ExtUtils-Manifest-1:1.73-4.el9.noarch
  perl-ExtUtils-ParseXS-1:3.40-460.el9.noarch                    perl-Fedora-VSP-0.001-23.el9.noarch
  perl-File-Compare-1.100.600-481.1.el9_6.noarch                 perl-File-Copy-2.34-481.1.el9_6.noarch
  perl-File-Find-1.37-481.1.el9_6.noarch                         perl-I18N-Langinfo-0.19-481.1.el9_6.x86_64
  perl-JSON-PP-1:4.06-4.el9.noarch                               perl-Math-BigInt-1:1.9998.18-460.el9.noarch
  perl-Math-Complex-1.59-481.1.el9_6.noarch                      perl-Test-Harness-1:3.42-461.el9.noarch
  perl-Time-HiRes-4:1.9764-462.el9.x86_64                        perl-devel-4:5.32.1-481.1.el9_6.x86_64
  perl-doc-5.32.1-481.1.el9_6.noarch                             perl-generators-1.13-1.el9.noarch
  perl-locale-1.09-481.1.el9_6.noarch                            perl-macros-4:5.32.1-481.1.el9_6.noarch
  perl-version-7:0.99.28-4.el9.x86_64                            pixman-0.40.0-6.el9_3.x86_64
  python3-pyparsing-2.4.7-9.el9.noarch                           sysprof-capture-devel-3.40.1-3.el9.x86_64
  systemtap-sdt-devel-5.4-4.el9.x86_64                           systemtap-sdt-dtrace-5.4-4.el9.x86_64
  xml-common-0.6.3-58.el9.noarch                                 xorg-x11-proto-devel-2024.1-1.el9.noarch
  xz-devel-5.2.5-8.el9_0.x86_64                                  zlib-devel-1.2.11-40.el9.x86_64

Complete!

```

Также скачаем исходный код модуля ngx_brotli. Он потребуется при сборке:

```
[root@localhost rpm]# cd /root
[root@localhost ~]# git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
Cloning into 'ngx_brotli'...
remote: Enumerating objects: 237, done.
remote: Counting objects: 100% (72/72), done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 237 (delta 55), reused 50 (delta 50), pack-reused 165 (from 1)
Receiving objects: 100% (237/237), 78.03 KiB | 1.42 MiB/s, done.
Resolving deltas: 100% (116/116), done.
Submodule 'deps/brotli' (https://github.com/google/brotli.git) registered for path 'deps/brotli'
Cloning into '/root/ngx_brotli/deps/brotli'...
remote: Enumerating objects: 9377, done.
remote: Counting objects: 100% (136/136), done.
remote: Compressing objects: 100% (78/78), done.
remote: Total 9377 (delta 87), reused 58 (delta 58), pack-reused 9241 (from 3)
Receiving objects: 100% (9377/9377), 41.80 MiB | 14.90 MiB/s, done.
Resolving deltas: 100% (6073/6073), done.
Submodule path 'deps/brotli': checked out 'ed738e842d2fbdf2d6459e39267a633c4a9b2f5d'
[root@localhost ~]#
[root@localhost ~]# cd ngx_brotli/deps/brotli/ && mkdir out && cd out
[root@localhost out]#
```

Собираем модуль ngx_brotli:

```
[root@localhost out]# cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..
-- The C compiler identification is GNU 11.5.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Build type is 'Release'
-- Performing Test BROTLI_EMSCRIPTEN
-- Performing Test BROTLI_EMSCRIPTEN - Failed
-- Compiler is not EMSCRIPTEN
-- Looking for log2
-- Looking for log2 - not found
-- Looking for log2
-- Looking for log2 - found
-- Configuring done (0.4s)
-- Generating done (0.0s)
-- Build files have been written to: /root/ngx_brotli/deps/brotli/out
[root@localhost out]# ll
total 140
-rw-r--r--.  1 root root 18095 May 26 22:34 CMakeCache.txt
drwxr-xr-x. 37 root root  4096 May 26 22:34 CMakeFiles
-rw-r--r--.  1 root root 39592 May 26 22:34 CTestTestfile.cmake
-rw-r--r--.  1 root root  2520 May 26 22:34 DartConfiguration.tcl
-rw-r--r--.  1 root root 52445 May 26 22:34 Makefile
drwxr-xr-x.  3 root root    23 May 26 22:34 Testing
-rw-r--r--.  1 root root  4478 May 26 22:34 cmake_install.cmake
-rw-r--r--.  1 root root   336 May 26 22:34 libbrotlicommon.pc
-rw-r--r--.  1 root root   363 May 26 22:34 libbrotlidec.pc
-rw-r--r--.  1 root root   363 May 26 22:34 libbrotlienc.pc

[root@localhost out]# cmake --build . --config Release -j 2 --target brotlienc
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/constants.c.o
[  6%] Building C object CMakeFiles/brotlicommon.dir/c/common/context.c.o
[ 10%] Building C object CMakeFiles/brotlicommon.dir/c/common/dictionary.c.o
[ 13%] Building C object CMakeFiles/brotlicommon.dir/c/common/platform.c.o
[ 17%] Building C object CMakeFiles/brotlicommon.dir/c/common/shared_dictionary.c.o
[ 20%] Building C object CMakeFiles/brotlicommon.dir/c/common/transform.c.o
[ 24%] Linking C static library libbrotlicommon.a
[ 24%] Built target brotlicommon
[ 31%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references.c.o
[ 31%] Building C object CMakeFiles/brotlienc.dir/c/enc/backward_references_hq.c.o
[ 34%] Building C object CMakeFiles/brotlienc.dir/c/enc/bit_cost.c.o
[ 37%] Building C object CMakeFiles/brotlienc.dir/c/enc/block_splitter.c.o
[ 41%] Building C object CMakeFiles/brotlienc.dir/c/enc/brotli_bit_stream.c.o
[ 44%] Building C object CMakeFiles/brotlienc.dir/c/enc/cluster.c.o
[ 48%] Building C object CMakeFiles/brotlienc.dir/c/enc/command.c.o
[ 51%] Building C object CMakeFiles/brotlienc.dir/c/enc/compound_dictionary.c.o
[ 55%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment.c.o
[ 58%] Building C object CMakeFiles/brotlienc.dir/c/enc/compress_fragment_two_pass.c.o
[ 62%] Building C object CMakeFiles/brotlienc.dir/c/enc/dictionary_hash.c.o
[ 65%] Building C object CMakeFiles/brotlienc.dir/c/enc/encode.c.o
[ 68%] Building C object CMakeFiles/brotlienc.dir/c/enc/encoder_dict.c.o
[ 72%] Building C object CMakeFiles/brotlienc.dir/c/enc/entropy_encode.c.o
[ 75%] Building C object CMakeFiles/brotlienc.dir/c/enc/fast_log.c.o
[ 79%] Building C object CMakeFiles/brotlienc.dir/c/enc/histogram.c.o
[ 82%] Building C object CMakeFiles/brotlienc.dir/c/enc/literal_cost.c.o
[ 86%] Building C object CMakeFiles/brotlienc.dir/c/enc/memory.c.o
[ 89%] Building C object CMakeFiles/brotlienc.dir/c/enc/metablock.c.o
[ 93%] Building C object CMakeFiles/brotlienc.dir/c/enc/static_dict.c.o
[ 96%] Building C object CMakeFiles/brotlienc.dir/c/enc/utf8_util.c.o
[100%] Linking C static library libbrotlienc.a
[100%] Built target brotlienc

[root@localhost out]# cd ../../../..
```

Правим .spec файл, чтобы Nginx собирался с необходимыми нам опциями: находим секцию с параметрами configure (до условий if) и добавляем 
указание на модуль: 

```
[root@localhost ~]# cd ./rpmbuild/SPECS/
[root@localhost SPECS]# nano nginx.spec

%build
# nginx does not utilize a standard configure script.  It has its own
# and the standard configure options cause the nginx configure script
# to error out.  This is is also the reason for the DESTDIR environment
# variable.
export DESTDIR=%{buildroot}
# So the perl module finds its symbols:
nginx_ldopts="$RPM_LD_FLAGS -Wl,-E"
if ! ./configure \
    --prefix=%{_datadir}/nginx \
    --sbin-path=%{_sbindir}/nginx \
    --modules-path=%{nginx_moduledir} \
    --conf-path=%{_sysconfdir}/nginx/nginx.conf \
    --error-log-path=%{_localstatedir}/log/nginx/error.log \
    --http-log-path=%{_localstatedir}/log/nginx/access.log \
    --http-client-body-temp-path=%{_localstatedir}/lib/nginx/tmp/client_body \
    --http-proxy-temp-path=%{_localstatedir}/lib/nginx/tmp/proxy \
    --http-fastcgi-temp-path=%{_localstatedir}/lib/nginx/tmp/fastcgi \
    --http-uwsgi-temp-path=%{_localstatedir}/lib/nginx/tmp/uwsgi \
    --http-scgi-temp-path=%{_localstatedir}/lib/nginx/tmp/scgi \
    --pid-path=/run/nginx.pid \
    --lock-path=/run/lock/subsys/nginx \
    --user=%{nginx_user} \
    --group=%{nginx_user} \
    --with-compat \
    --with-debug \
    --add-module=/root/ngx_brotli \
%if 0%{?with_aio}

```

Приступим к сборке RPM пакета:
```
[root@localhost SPECS]# rpmbuild -ba nginx.spec -D 'debug_package %{nil}'
setting SOURCE_DATE_EPOCH=1779235200
Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.vkJcnV
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cat /root/rpmbuild/SOURCES/maxim.key /root/rpmbuild/SOURCES/mdounin.key /root/rpmbuild/SOURCES/sb.key
+ /usr/lib/rpm/redhat/gpgverify --keyring=/root/rpmbuild/BUILD/nginx.gpg --signature=/root/rpmbuild/SOURCES/nginx-1.20.1.tar.gz.asc --data=/root/rpmbuild/SOURCES/nginx-1.20.1.tar.gz
gpgv: Signature made Tue May 25 15:42:56 2021 MSK
gpgv:                using RSA key 520A9993A1C052F8
gpgv: Good signature from "Maxim Dounin <mdounin@mdounin.ru>"
+ cd /root/rpmbuild/BUILD
+ rm -rf nginx-1.20.1
+ /usr/bin/gzip -dc /root/rpmbuild/SOURCES/nginx-1.20.1.tar.gz
+ /usr/bin/tar -xof -
+ STATUS=0
+ '[' 0 -ne 0 ']'
+ cd nginx-1.20.1


...


Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.m0PvE4
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd nginx-1.20.1
+ /usr/bin/rm -rf /root/rpmbuild/BUILDROOT/nginx-1.20.1-28.el9.2.alma.1.x86_64
+ RPM_EC=0
++ jobs -p
+ exit 0
```

 Убедимся, что пакеты создались:

```
[root@localhost SPECS]# cd ../../
[root@localhost ~]# ll rpmbuild/RPMS/x86_64/
total 2012
-rw-r--r--. 1 root root   37986 May 26 22:45 nginx-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root 1030483 May 26 22:45 nginx-core-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root  762249 May 26 22:45 nginx-mod-devel-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   21073 May 26 22:45 nginx-mod-http-image-filter-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   32601 May 26 22:45 nginx-mod-http-perl-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   19879 May 26 22:45 nginx-mod-http-xslt-filter-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   55488 May 26 22:45 nginx-mod-mail-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   82028 May 26 22:45 nginx-mod-stream-1.20.1-28.el9.2.alma.1.x86_64.rpm

```

Копируем пакеты в общий каталог: 

```
[root@localhost ~]#  cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64/
[root@localhost ~]# cd ~/rpmbuild/RPMS/x86_64
```

Установим пакет и убедимся, что nginx запускается:

```
[root@localhost x86_64]# yum localinstall *.rpm
Last metadata expiration check: 0:32:51 ago on Tue May 26 22:20:56 2026.
Dependencies resolved.
=============================================================================================================================
 Package                                Architecture      Version                              Repository               Size
=============================================================================================================================
Installing:
 nginx                                  x86_64            2:1.20.1-28.el9.2.alma.1             @commandline             37 k
 nginx-all-modules                      noarch            2:1.20.1-28.el9.2.alma.1             @commandline            8.9 k
 nginx-core                             x86_64            2:1.20.1-28.el9.2.alma.1             @commandline            1.0 M
 nginx-filesystem                       noarch            2:1.20.1-28.el9.2.alma.1             @commandline             10 k
 nginx-mod-devel                        x86_64            2:1.20.1-28.el9.2.alma.1             @commandline            744 k
 nginx-mod-http-image-filter            x86_64            2:1.20.1-28.el9.2.alma.1             @commandline             21 k
 nginx-mod-http-perl                    x86_64            2:1.20.1-28.el9.2.alma.1             @commandline             32 k
 nginx-mod-http-xslt-filter             x86_64            2:1.20.1-28.el9.2.alma.1             @commandline             19 k
 nginx-mod-mail                         x86_64            2:1.20.1-28.el9.2.alma.1             @commandline             54 k
 nginx-mod-stream                       x86_64            2:1.20.1-28.el9.2.alma.1             @commandline             80 k
Installing dependencies:
 almalinux-logos-httpd                  noarch            90.7-1.el9                           appstream                18 k

Transaction Summary
=============================================================================================================================
Install  11 Packages

Total size: 2.0 M
Total download size: 18 k
Installed size: 9.5 M
Is this ok [y/N]: y
Downloading Packages:
almalinux-logos-httpd-90.7-1.el9.noarch.rpm                                                  321 kB/s |  18 kB     00:00
-----------------------------------------------------------------------------------------------------------------------------
Total                                                                                         26 kB/s |  18 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                     1/1
  Running scriptlet: nginx-filesystem-2:1.20.1-28.el9.2.alma.1.noarch                                                   1/11
  Installing       : nginx-filesystem-2:1.20.1-28.el9.2.alma.1.noarch                                                   1/11
  Installing       : nginx-core-2:1.20.1-28.el9.2.alma.1.x86_64                                                         2/11
  Installing       : almalinux-logos-httpd-90.7-1.el9.noarch                                                            3/11
  Installing       : nginx-2:1.20.1-28.el9.2.alma.1.x86_64                                                              4/11
  Running scriptlet: nginx-2:1.20.1-28.el9.2.alma.1.x86_64                                                              4/11
  Installing       : nginx-mod-http-image-filter-2:1.20.1-28.el9.2.alma.1.x86_64                                        5/11
  Running scriptlet: nginx-mod-http-image-filter-2:1.20.1-28.el9.2.alma.1.x86_64                                        5/11
  Installing       : nginx-mod-http-perl-2:1.20.1-28.el9.2.alma.1.x86_64                                                6/11
  Running scriptlet: nginx-mod-http-perl-2:1.20.1-28.el9.2.alma.1.x86_64                                                6/11
  Installing       : nginx-mod-http-xslt-filter-2:1.20.1-28.el9.2.alma.1.x86_64                                         7/11
  Running scriptlet: nginx-mod-http-xslt-filter-2:1.20.1-28.el9.2.alma.1.x86_64                                         7/11
  Installing       : nginx-mod-mail-2:1.20.1-28.el9.2.alma.1.x86_64                                                     8/11
  Running scriptlet: nginx-mod-mail-2:1.20.1-28.el9.2.alma.1.x86_64                                                     8/11
  Installing       : nginx-mod-stream-2:1.20.1-28.el9.2.alma.1.x86_64                                                   9/11
  Running scriptlet: nginx-mod-stream-2:1.20.1-28.el9.2.alma.1.x86_64                                                   9/11
  Installing       : nginx-all-modules-2:1.20.1-28.el9.2.alma.1.noarch                                                 10/11
  Installing       : nginx-mod-devel-2:1.20.1-28.el9.2.alma.1.x86_64                                                   11/11
  Running scriptlet: nginx-mod-devel-2:1.20.1-28.el9.2.alma.1.x86_64                                                   11/11
  Verifying        : almalinux-logos-httpd-90.7-1.el9.noarch                                                            1/11
  Verifying        : nginx-2:1.20.1-28.el9.2.alma.1.x86_64                                                              2/11
  Verifying        : nginx-all-modules-2:1.20.1-28.el9.2.alma.1.noarch                                                  3/11
  Verifying        : nginx-core-2:1.20.1-28.el9.2.alma.1.x86_64                                                         4/11
  Verifying        : nginx-filesystem-2:1.20.1-28.el9.2.alma.1.noarch                                                   5/11
  Verifying        : nginx-mod-devel-2:1.20.1-28.el9.2.alma.1.x86_64                                                    6/11
  Verifying        : nginx-mod-http-image-filter-2:1.20.1-28.el9.2.alma.1.x86_64                                        7/11
  Verifying        : nginx-mod-http-perl-2:1.20.1-28.el9.2.alma.1.x86_64                                                8/11
  Verifying        : nginx-mod-http-xslt-filter-2:1.20.1-28.el9.2.alma.1.x86_64                                         9/11
  Verifying        : nginx-mod-mail-2:1.20.1-28.el9.2.alma.1.x86_64                                                    10/11
  Verifying        : nginx-mod-stream-2:1.20.1-28.el9.2.alma.1.x86_64                                                  11/11

Installed:
  almalinux-logos-httpd-90.7-1.el9.noarch                           nginx-2:1.20.1-28.el9.2.alma.1.x86_64
  nginx-all-modules-2:1.20.1-28.el9.2.alma.1.noarch                 nginx-core-2:1.20.1-28.el9.2.alma.1.x86_64
  nginx-filesystem-2:1.20.1-28.el9.2.alma.1.noarch                  nginx-mod-devel-2:1.20.1-28.el9.2.alma.1.x86_64
  nginx-mod-http-image-filter-2:1.20.1-28.el9.2.alma.1.x86_64       nginx-mod-http-perl-2:1.20.1-28.el9.2.alma.1.x86_64
  nginx-mod-http-xslt-filter-2:1.20.1-28.el9.2.alma.1.x86_64        nginx-mod-mail-2:1.20.1-28.el9.2.alma.1.x86_64
  nginx-mod-stream-2:1.20.1-28.el9.2.alma.1.x86_64

Complete!


[root@localhost x86_64]# systemctl start nginx
[root@localhost x86_64]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-05-26 22:54:17 MSK; 9s ago
    Process: 50888 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 50889 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 50890 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 50891 (nginx)
      Tasks: 3 (limit: 10429)
     Memory: 5.1M (peak: 5.4M)
        CPU: 40ms
     CGroup: /system.slice/nginx.service
             ├─50891 "nginx: master process /usr/sbin/nginx"
             ├─50892 "nginx: worker process"
             └─50893 "nginx: worker process"

May 26 22:54:17 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
May 26 22:54:17 localhost.localdomain nginx[50889]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
May 26 22:54:17 localhost.localdomain nginx[50889]: nginx: configuration file /etc/nginx/nginx.conf test is successful
May 26 22:54:17 localhost.localdomain systemd[1]: Started The nginx HTTP and reverse proxy server.
```

## 2) Создание своего репозитория и размещение там ранее собранных пакетов RPM

Теперь приступим к созданию своего репозитория. Директория для статики у Nginx 
по умолчанию /usr/share/nginx/html. Создадим там каталог repo и скопируем туда наши собранные RPM-пакеты:

```
[root@localhost user]# mkdir /usr/share/nginx/html/repo
[root@localhost user]# cp ~/rpmbuild/RPMS/x86_64/*.rpm /usr/share/nginx/html/repo/
[root@localhost user]# ll /usr/share/nginx/html/repo
total 2036
-rw-r--r--. 1 root root   37986 May 27 22:16 nginx-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root    9089 May 27 22:16 nginx-all-modules-1.20.1-28.el9.2.alma.1.noarch.rpm
-rw-r--r--. 1 root root 1030483 May 27 22:16 nginx-core-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   10688 May 27 22:16 nginx-filesystem-1.20.1-28.el9.2.alma.1.noarch.rpm
-rw-r--r--. 1 root root  762249 May 27 22:16 nginx-mod-devel-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   21073 May 27 22:16 nginx-mod-http-image-filter-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   32601 May 27 22:16 nginx-mod-http-perl-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   19879 May 27 22:16 nginx-mod-http-xslt-filter-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   55488 May 27 22:16 nginx-mod-mail-1.20.1-28.el9.2.alma.1.x86_64.rpm
-rw-r--r--. 1 root root   82028 May 27 22:16 nginx-mod-stream-1.20.1-28.el9.2.alma.1.x86_64.rpm
```

Инициализируем репозиторий командой: 

```
[root@localhost user]# createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 10 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
```

Для прозрачности настроим в NGINX доступ к листингу каталога. В файле 
/etc/nginx/nginx.conf в блоке server добавим следующие директивы: 

```
[root@localhost user]# nano /etc/nginx/nginx.conf
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;
        index index.html index.htm;
        autoindex on;
```

Проверяем синтаксис и перезапускаем NGINX: 

```
[root@localhost user]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@localhost user]# nginx -s reload
[root@localhost user]#
```

Проверим содержимое репозитория с помощью curl: 

```
[root@localhost user]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          27-May-2026 19:19                   -
<a href="nginx-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-1.20.1-28.el9.2.alma.1.x86_64.rpm</a>            27-May-2026 19:16               37986
<a href="nginx-all-modules-1.20.1-28.el9.2.alma.1.noarch.rpm">nginx-all-modules-1.20.1-28.el9.2.alma.1.noarch..&gt;</a> 27-May-2026 19:16                9089
<a href="nginx-core-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-core-1.20.1-28.el9.2.alma.1.x86_64.rpm</a>       27-May-2026 19:16             1030483
<a href="nginx-filesystem-1.20.1-28.el9.2.alma.1.noarch.rpm">nginx-filesystem-1.20.1-28.el9.2.alma.1.noarch.rpm</a> 27-May-2026 19:16               10688
<a href="nginx-mod-devel-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-mod-devel-1.20.1-28.el9.2.alma.1.x86_64.rpm</a>  27-May-2026 19:16              762249
<a href="nginx-mod-http-image-filter-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-mod-http-image-filter-1.20.1-28.el9.2.alm..&gt;</a> 27-May-2026 19:16               21073
<a href="nginx-mod-http-perl-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-mod-http-perl-1.20.1-28.el9.2.alma.1.x86_..&gt;</a> 27-May-2026 19:16               32601
<a href="nginx-mod-http-xslt-filter-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-mod-http-xslt-filter-1.20.1-28.el9.2.alma..&gt;</a> 27-May-2026 19:16               19879
<a href="nginx-mod-mail-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-mod-mail-1.20.1-28.el9.2.alma.1.x86_64.rpm</a>   27-May-2026 19:16               55488
<a href="nginx-mod-stream-1.20.1-28.el9.2.alma.1.x86_64.rpm">nginx-mod-stream-1.20.1-28.el9.2.alma.1.x86_64.rpm</a> 27-May-2026 19:16               82028
</pre><hr></body>
</html>
```

 Все готово для того, чтобы протестировать репозиторий. Добавим его в /etc/yum.repos.d:

 ```
[root@localhost user]# cat >> /etc/yum.repos.d/otus.repo << EOF
[otus]
name=otus-linux
baseurl=http://localhost/repo
gpgcheck=0
enabled=1
EOF
```

Убедимся, что репозиторий подключился и посмотрим, что в нем есть: 

```
[root@localhost user]# yum repolist enabled | grep otus
otus                otus-linux
```

Добавим пакет в наш репозиторий:

```
[root@localhost user]# cd /usr/share/nginx/html/repo/
[root@localhost repo]# wget https://repo.percona.com/yum/percona-release-latest.noarch.rpm
--2026-05-27 22:38:47--  https://repo.percona.com/yum/percona-release-latest.noarch.rpm
Resolving repo.percona.com (repo.percona.com)... 49.12.125.205, 2a01:4f8:242:5792::2
Connecting to repo.percona.com (repo.percona.com)|49.12.125.205|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 28152 (27K) [application/x-redhat-package-manager]
Saving to: ‘percona-release-latest.noarch.rpm’

percona-release-latest.noarch.r 100%[====================================================>]  27.49K  --.-KB/s    in 0s

2026-05-27 22:38:47 (83.3 MB/s) - ‘percona-release-latest.noarch.rpm’ saved [28152/28152]
```

Обновим список пакетов в репозитории:

```
[root@localhost repo]#  createrepo /usr/share/nginx/html/repo/
Directory walk started
Directory walk done - 11 packages
Temporary output repo path: /usr/share/nginx/html/repo/.repodata/
Preparing sqlite DBs
Pool started (with 5 workers)
Pool finished
[root@localhost repo]#  yum makecache
AlmaLinux 9 - AppStream                                                                      6.7 kB/s | 4.2 kB     00:00
AlmaLinux 9 - BaseOS                                                                         6.0 kB/s | 3.8 kB     00:00
AlmaLinux 9 - Extras                                                                         5.9 kB/s | 3.8 kB     00:00
Extra Packages for Enterprise Linux 9 - x86_64                                                36 kB/s |  36 kB     00:00
Extra Packages for Enterprise Linux 9 openh264 (From Cisco) - x86_64                         4.0 kB/s | 993  B     00:00
otus-linux                                                                                   565 kB/s | 7.2 kB     00:00
Metadata cache created.
[root@localhost repo]# yum list | grep otus
percona-release.noarch                                                                   1.0-33                                otus
```
Так как Nginx уже установлен, установим репозиторий percona-release:

```
[root@localhost repo]# yum install -y percona-release.noarch
Last metadata expiration check: 0:02:35 ago on Wed May 27 22:40:35 2026.
Dependencies resolved.
=============================================================================================================================
 Package                             Architecture               Version                       Repository                Size
=============================================================================================================================
Installing:
 percona-release                     noarch                     1.0-33                        otus                      27 k

Transaction Summary
=============================================================================================================================
Install  1 Package

Total download size: 27 k
Installed size: 49 k
Downloading Packages:
percona-release-latest.noarch.rpm                                                             18 MB/s |  27 kB     00:00
-----------------------------------------------------------------------------------------------------------------------------
Total                                                                                        2.7 MB/s |  27 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                     1/1
  Installing       : percona-release-1.0-33.noarch                                                                       1/1
  Running scriptlet: percona-release-1.0-33.noarch                                                                       1/1
* Enabling the Percona Release repository
<*> All done!
* Enabling the Percona Telemetry repository
<*> All done!
The percona-release package now contains a percona-release script that can enable additional repositories for our newer products.

Note: currently there are no repositories that contain Percona products or distributions enabled. We recommend you to enable Percona Distribution repositories instead of individual product repositories, because with the Distribution you will get not only the database itself but also a set of other componets that will help you work with your database.

For example, to enable the Percona Distribution for MySQL 8.0 repository use:

  percona-release setup pdps8.0

Note: To avoid conflicts with older product versions, the percona-release setup command may disable our original repository for some products.

For more information, please visit:
  https://docs.percona.com/percona-software-repositories/percona-release.html


  Verifying        : percona-release-1.0-33.noarch                                                                       1/1

Installed:
  percona-release-1.0-33.noarch

Complete!
``

Установка прошла успешно.





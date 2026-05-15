# Занятие 7. Управление пакетами. Дистрибьюция софта

## Задание
1) Создать свой RPM пакет (можно взять свое приложение, либо собрать, например,
Apache с определенными опциями).
2) Создать свой репозиторий и разместить там ранее собранный RPM.

## Стенд
Виртуальная машина: Alma Linux 10.1, 2Gb RAM, 2 CPU.
Гипервизор: Virtual Box 7.2.8 r173730 (Qt6.8.0 on cocoa)

```
user@alma10:~$ hostnamectl
     Static hostname: alma10
           Icon name: computer
          Machine ID: 7d59c0f25614449b8502a4712ad922dc
             Boot ID: ee6d8e2472824a75b67a05561ddf6bf6
    Operating System: AlmaLinux 10.1 (Heliotrope Lion)    
         CPE OS Name: cpe:/o:almalinux:almalinux:10.1
      OS Support End: Fri 2035-06-01
OS Support Remaining: 9y 2w 2d                            
              Kernel: Linux 6.12.0-124.55.3.el10_1.aarch64
        Architecture: arm64
```

Установим пакеты необходимые для выполнения задания:

```
root@alma10:/home/user# yum install -y rpmdevtools rpm-build createrepo_c yum-utils cmake gcc git
Last metadata expiration check: 0:07:11 ago on Fri 15 May 2026 11:37:13 AM MSK.
Dependencies resolved.
====================================================================================================
 Package                         Architecture  Version                        Repository       Size
====================================================================================================
Installing:
 cmake                           aarch64       3.30.5-3.el10_0                appstream       7.9 M
 createrepo_c                    aarch64       1.1.2-4.el10                   appstream        74 k
 gcc                             aarch64       14.3.1-2.1.el10.alma.1         appstream        34 M
 git                             aarch64       2.47.3-1.el10_0                appstream        50 k
 rpm-build                       aarch64       4.19.1.1-20.el10.alma.1        appstream        67 k
 rpmdevtools                     noarch        9.6-9.el10                     appstream        88 k
 yum-utils                       noarch        4.7.0-9.el10                   baseos           39 k
Installing dependencies:
 annobin-docs                    noarch        12.99-1.el10                   appstream        88 k
 annobin-plugin-gcc              aarch64       12.99-1.el10                   appstream       996 k
 cmake-data                      noarch        3.30.5-3.el10_0                appstream       1.8 M
 cmake-filesystem                aarch64       3.30.5-3.el10_0                appstream        15 k
 cmake-rpm-macros                noarch        3.30.5-3.el10_0                appstream        15 k
 cpp                             aarch64       14.3.1-2.1.el10.alma.1         appstream        11 M
 createrepo_c-libs               aarch64       1.1.2-4.el10                   appstream       104 k
 debugedit                       aarch64       5.1-8.el10                     appstream        79 k
 dwz                             aarch64       0.16-1.el10                    appstream       137 k
 efi-srpm-macros                 noarch        6-6.el10.alma.1                appstream        23 k
 elfutils                        aarch64       0.193-1.el10                   baseos          540 k
 fonts-srpm-macros               noarch        1:2.0.5-18.el10                appstream        26 k
 forge-srpm-macros               noarch        0.4.0-6.el10                   appstream        20 k
 gcc-plugin-annobin              aarch64       14.3.1-2.1.el10.alma.1         appstream        67 k
 gdb-minimal                     aarch64       16.3-2.el10                    appstream       4.1 M
 git-core                        aarch64       2.47.3-1.el10_0                appstream       4.9 M
 git-core-doc                    noarch        2.47.3-1.el10_0                appstream       2.8 M
 glibc-devel                     aarch64       2.39-58.el10_1.7.alma.1        appstream       462 k
 go-srpm-macros                  noarch        3.6.0-8.el10_1                 appstream        27 k
 kernel-headers                  aarch64       6.12.0-124.55.3.el10_1         appstream       3.0 M
 kernel-srpm-macros              noarch        1.0-25.el10                    appstream       9.7 k
 libasan                         aarch64       14.3.1-2.1.el10.alma.1         appstream       529 k
 libubsan                        aarch64       14.3.1-2.1.el10.alma.1         appstream       236 k
 libxcrypt-devel                 aarch64       4.4.36-10.el10                 appstream        33 k
 lua-srpm-macros                 noarch        1-15.el10                      appstream       8.7 k
 make                            aarch64       1:4.4.1-9.el10                 baseos          588 k
 ocaml-srpm-macros               noarch        10-4.el10                      appstream       9.0 k
 openblas-srpm-macros            noarch        2-19.el10                      appstream       7.6 k
 package-notes-srpm-macros       noarch        0.5-13.el10                    appstream       9.1 k
 patch                           aarch64       2.7.6-26.el10                  appstream       126 k
 perl-Error                      noarch        1:0.17029-18.el10              appstream        45 k
 perl-Git                        noarch        2.47.3-1.el10_0                appstream        37 k
 perl-TermReadKey                aarch64       2.38-24.el10                   appstream        35 k
 perl-lib                        aarch64       0.65-512.2.el10_0              appstream        15 k
 perl-srpm-macros                noarch        1-57.el10                      appstream       8.4 k
 pyproject-srpm-macros           noarch        1.16.2-1.el10                  appstream        14 k
 python-srpm-macros              noarch        3.12-10.el10                   appstream        23 k
 qt6-srpm-macros                 noarch        6.9.1-1.el10                   appstream       9.5 k
 redhat-rpm-config               noarch        293-1.el10.alma.2              appstream        68 k
 rust-toolset-srpm-macros        noarch        1.88.0-1.el10.alma.1           appstream        12 k
 systemd-rpm-macros              noarch        257-13.el10_1.3.alma.1         baseos           29 k
 zstd                            aarch64       1.5.5-9.el10                   baseos          454 k

Transaction Summary
====================================================================================================
Install  49 Packages

Total download size: 76 M
Installed size: 244 M
Downloading Packages:
(1/49): annobin-docs-12.99-1.el10.noarch.rpm                        1.1 MB/s |  88 kB     00:00    
(2/49): annobin-plugin-gcc-12.99-1.el10.aarch64.rpm                 2.6 MB/s | 996 kB     00:00    
(3/49): cmake-data-3.30.5-3.el10_0.noarch.rpm                       5.9 MB/s | 1.8 MB     00:00    
(4/49): cmake-filesystem-3.30.5-3.el10_0.aarch64.rpm                351 kB/s |  15 kB     00:00    
(5/49): cmake-rpm-macros-3.30.5-3.el10_0.noarch.rpm                 389 kB/s |  15 kB     00:00    
(6/49): createrepo_c-1.1.2-4.el10.aarch64.rpm                       608 kB/s |  74 kB     00:00    
(7/49): createrepo_c-libs-1.1.2-4.el10.aarch64.rpm                  1.3 MB/s | 104 kB     00:00    
(8/49): debugedit-5.1-8.el10.aarch64.rpm                            704 kB/s |  79 kB     00:00    
(9/49): dwz-0.16-1.el10.aarch64.rpm                                 817 kB/s | 137 kB     00:00    
(10/49): efi-srpm-macros-6-6.el10.alma.1.noarch.rpm                 972 kB/s |  23 kB     00:00    
(11/49): fonts-srpm-macros-2.0.5-18.el10.noarch.rpm                 1.3 MB/s |  26 kB     00:00    
(12/49): forge-srpm-macros-0.4.0-6.el10.noarch.rpm                  317 kB/s |  20 kB     00:00    
(13/49): cmake-3.30.5-3.el10_0.aarch64.rpm                          3.1 MB/s | 7.9 MB     00:02    
(14/49): gcc-plugin-annobin-14.3.1-2.1.el10.alma.1.aarch64.rpm      1.1 MB/s |  67 kB     00:00    
(15/49): cpp-14.3.1-2.1.el10.alma.1.aarch64.rpm                     4.3 MB/s |  11 MB     00:02    
(16/49): git-2.47.3-1.el10_0.aarch64.rpm                            916 kB/s |  50 kB     00:00    
(17/49): gdb-minimal-16.3-2.el10.aarch64.rpm                        3.4 MB/s | 4.1 MB     00:01    
(18/49): git-core-2.47.3-1.el10_0.aarch64.rpm                       4.1 MB/s | 4.9 MB     00:01    
(19/49): glibc-devel-2.39-58.el10_1.7.alma.1.aarch64.rpm            3.5 MB/s | 462 kB     00:00    
(20/49): go-srpm-macros-3.6.0-8.el10_1.noarch.rpm                   1.0 MB/s |  27 kB     00:00    
(21/49): kernel-headers-6.12.0-124.55.3.el10_1.aarch64.rpm          6.1 MB/s | 3.0 MB     00:00    
(22/49): kernel-srpm-macros-1.0-25.el10.noarch.rpm                  404 kB/s | 9.7 kB     00:00    
(23/49): libasan-14.3.1-2.1.el10.alma.1.aarch64.rpm                 3.7 MB/s | 529 kB     00:00    
(24/49): libubsan-14.3.1-2.1.el10.alma.1.aarch64.rpm                2.4 MB/s | 236 kB     00:00    
(25/49): libxcrypt-devel-4.4.36-10.el10.aarch64.rpm                 547 kB/s |  33 kB     00:00    
(26/49): lua-srpm-macros-1-15.el10.noarch.rpm                       386 kB/s | 8.7 kB     00:00    
(27/49): ocaml-srpm-macros-10-4.el10.noarch.rpm                     469 kB/s | 9.0 kB     00:00    
(28/49): openblas-srpm-macros-2-19.el10.noarch.rpm                  378 kB/s | 7.6 kB     00:00    
(29/49): package-notes-srpm-macros-0.5-13.el10.noarch.rpm           513 kB/s | 9.1 kB     00:00    
(30/49): git-core-doc-2.47.3-1.el10_0.noarch.rpm                    1.8 MB/s | 2.8 MB     00:01    
(31/49): perl-Error-0.17029-18.el10.noarch.rpm                      1.7 MB/s |  45 kB     00:00    
(32/49): perl-Git-2.47.3-1.el10_0.noarch.rpm                        1.5 MB/s |  37 kB     00:00    
(33/49): patch-2.7.6-26.el10.aarch64.rpm                            1.9 MB/s | 126 kB     00:00    
(34/49): perl-TermReadKey-2.38-24.el10.aarch64.rpm                  847 kB/s |  35 kB     00:00    
(35/49): perl-srpm-macros-1-57.el10.noarch.rpm                      474 kB/s | 8.4 kB     00:00    
(36/49): perl-lib-0.65-512.2.el10_0.aarch64.rpm                     204 kB/s |  15 kB     00:00    
(37/49): pyproject-srpm-macros-1.16.2-1.el10.noarch.rpm             617 kB/s |  14 kB     00:00    
(38/49): python-srpm-macros-3.12-10.el10.noarch.rpm                 1.1 MB/s |  23 kB     00:00    
(39/49): qt6-srpm-macros-6.9.1-1.el10.noarch.rpm                    542 kB/s | 9.5 kB     00:00    
(40/49): redhat-rpm-config-293-1.el10.alma.2.noarch.rpm             2.7 MB/s |  68 kB     00:00    
(41/49): rpm-build-4.19.1.1-20.el10.alma.1.aarch64.rpm              1.1 MB/s |  67 kB     00:00    
(42/49): rust-toolset-srpm-macros-1.88.0-1.el10.alma.1.noarch.rpm   701 kB/s |  12 kB     00:00    
(43/49): rpmdevtools-9.6-9.el10.noarch.rpm                          562 kB/s |  88 kB     00:00    
(44/49): elfutils-0.193-1.el10.aarch64.rpm                          2.8 MB/s | 540 kB     00:00    
(45/49): systemd-rpm-macros-257-13.el10_1.3.alma.1.noarch.rpm       1.2 MB/s |  29 kB     00:00    
(46/49): yum-utils-4.7.0-9.el10.noarch.rpm                          1.7 MB/s |  39 kB     00:00    
(47/49): make-4.4.1-9.el10.aarch64.rpm                              3.2 MB/s | 588 kB     00:00    
(48/49): zstd-1.5.5-9.el10.aarch64.rpm                              2.8 MB/s | 454 kB     00:00    
(49/49): gcc-14.3.1-2.1.el10.alma.1.aarch64.rpm                     4.9 MB/s |  34 MB     00:07    
----------------------------------------------------------------------------------------------------
Total                                                               8.4 MB/s |  76 MB     00:08     
AlmaLinux 10 - AppStream                                            1.6 MB/s | 1.6 kB     00:00    
Importing GPG key 0xC2A1E572:
 Userid     : "AlmaLinux OS 10 <packager@almalinux.org>"
 Fingerprint: EE6D B7B9 8F5B F5ED D9DA 0DE5 DEE5 C11C C2A1 E572
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-AlmaLinux-10
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                            1/1 
  Installing       : make-1:4.4.1-9.el10.aarch64                                               1/49 
  Installing       : elfutils-0.193-1.el10.aarch64                                             2/49 
  Installing       : git-core-2.47.3-1.el10_0.aarch64                                          3/49 
  Installing       : gdb-minimal-16.3-2.el10.aarch64                                           4/49 
  Installing       : dwz-0.16-1.el10.aarch64                                                   5/49 
  Installing       : cmake-rpm-macros-3.30.5-3.el10_0.noarch                                   6/49 
  Installing       : cmake-filesystem-3.30.5-3.el10_0.aarch64                                  7/49 
  Installing       : cmake-data-3.30.5-3.el10_0.noarch                                         8/49 
  Installing       : cmake-3.30.5-3.el10_0.aarch64                                             9/49 
  Installing       : debugedit-5.1-8.el10.aarch64                                             10/49 
  Installing       : git-core-doc-2.47.3-1.el10_0.noarch                                      11/49 
  Installing       : zstd-1.5.5-9.el10.aarch64                                                12/49 
  Installing       : rust-toolset-srpm-macros-1.88.0-1.el10.alma.1.noarch                     13/49 
  Installing       : qt6-srpm-macros-6.9.1-1.el10.noarch                                      14/49 
  Installing       : perl-srpm-macros-1-57.el10.noarch                                        15/49 
  Installing       : perl-lib-0.65-512.2.el10_0.aarch64                                       16/49 
  Installing       : perl-TermReadKey-2.38-24.el10.aarch64                                    17/49 
  Installing       : perl-Error-1:0.17029-18.el10.noarch                                      18/49 
  Installing       : perl-Git-2.47.3-1.el10_0.noarch                                          19/49 
  Installing       : git-2.47.3-1.el10_0.aarch64                                              20/49 
  Installing       : patch-2.7.6-26.el10.aarch64                                              21/49 
  Installing       : package-notes-srpm-macros-0.5-13.el10.noarch                             22/49 
  Installing       : openblas-srpm-macros-2-19.el10.noarch                                    23/49 
  Installing       : ocaml-srpm-macros-10-4.el10.noarch                                       24/49 
  Installing       : lua-srpm-macros-1-15.el10.noarch                                         25/49 
  Installing       : libubsan-14.3.1-2.1.el10.alma.1.aarch64                                  26/49 
  Installing       : libasan-14.3.1-2.1.el10.alma.1.aarch64                                   27/49 
  Installing       : kernel-srpm-macros-1.0-25.el10.noarch                                    28/49 
  Installing       : kernel-headers-6.12.0-124.55.3.el10_1.aarch64                            29/49 
  Installing       : libxcrypt-devel-4.4.36-10.el10.aarch64                                   30/49 
  Installing       : glibc-devel-2.39-58.el10_1.7.alma.1.aarch64                              31/49 
  Installing       : efi-srpm-macros-6-6.el10.alma.1.noarch                                   32/49 
  Installing       : createrepo_c-libs-1.1.2-4.el10.aarch64                                   33/49 
  Installing       : cpp-14.3.1-2.1.el10.alma.1.aarch64                                       34/49 
  Installing       : gcc-14.3.1-2.1.el10.alma.1.aarch64                                       35/49 
  Installing       : gcc-plugin-annobin-14.3.1-2.1.el10.alma.1.aarch64                        36/49 
  Installing       : annobin-docs-12.99-1.el10.noarch                                         37/49 
  Installing       : annobin-plugin-gcc-12.99-1.el10.aarch64                                  38/49 
  Installing       : fonts-srpm-macros-1:2.0.5-18.el10.noarch                                 39/49 
  Installing       : forge-srpm-macros-0.4.0-6.el10.noarch                                    40/49 
  Installing       : go-srpm-macros-3.6.0-8.el10_1.noarch                                     41/49 
  Installing       : python-srpm-macros-3.12-10.el10.noarch                                   42/49 
  Installing       : pyproject-srpm-macros-1.16.2-1.el10.noarch                               43/49 
  Installing       : redhat-rpm-config-293-1.el10.alma.2.noarch                               44/49 
  Running scriptlet: redhat-rpm-config-293-1.el10.alma.2.noarch                               44/49 
  Installing       : rpm-build-4.19.1.1-20.el10.alma.1.aarch64                                45/49 
  Installing       : rpmdevtools-9.6-9.el10.noarch                                            46/49 
  Installing       : createrepo_c-1.1.2-4.el10.aarch64                                        47/49 
  Installing       : yum-utils-4.7.0-9.el10.noarch                                            48/49 
  Installing       : systemd-rpm-macros-257-13.el10_1.3.alma.1.noarch                         49/49 
  Running scriptlet: systemd-rpm-macros-257-13.el10_1.3.alma.1.noarch                         49/49 

Installed:
  annobin-docs-12.99-1.el10.noarch                                                                  
  annobin-plugin-gcc-12.99-1.el10.aarch64                                                           
  cmake-3.30.5-3.el10_0.aarch64                                                                     
  cmake-data-3.30.5-3.el10_0.noarch                                                                 
  cmake-filesystem-3.30.5-3.el10_0.aarch64                                                          
  cmake-rpm-macros-3.30.5-3.el10_0.noarch                                                           
  cpp-14.3.1-2.1.el10.alma.1.aarch64                                                                
  createrepo_c-1.1.2-4.el10.aarch64                                                                 
  createrepo_c-libs-1.1.2-4.el10.aarch64                                                            
  debugedit-5.1-8.el10.aarch64                                                                      
  dwz-0.16-1.el10.aarch64                                                                           
  efi-srpm-macros-6-6.el10.alma.1.noarch                                                            
  elfutils-0.193-1.el10.aarch64                                                                     
  fonts-srpm-macros-1:2.0.5-18.el10.noarch                                                          
  forge-srpm-macros-0.4.0-6.el10.noarch                                                             
  gcc-14.3.1-2.1.el10.alma.1.aarch64                                                                
  gcc-plugin-annobin-14.3.1-2.1.el10.alma.1.aarch64                                                 
  gdb-minimal-16.3-2.el10.aarch64                                                                   
  git-2.47.3-1.el10_0.aarch64                                                                       
  git-core-2.47.3-1.el10_0.aarch64                                                                  
  git-core-doc-2.47.3-1.el10_0.noarch                                                               
  glibc-devel-2.39-58.el10_1.7.alma.1.aarch64                                                       
  go-srpm-macros-3.6.0-8.el10_1.noarch                                                              
  kernel-headers-6.12.0-124.55.3.el10_1.aarch64                                                     
  kernel-srpm-macros-1.0-25.el10.noarch                                                             
  libasan-14.3.1-2.1.el10.alma.1.aarch64                                                            
  libubsan-14.3.1-2.1.el10.alma.1.aarch64                                                           
  libxcrypt-devel-4.4.36-10.el10.aarch64                                                            
  lua-srpm-macros-1-15.el10.noarch                                                                  
  make-1:4.4.1-9.el10.aarch64                                                                       
  ocaml-srpm-macros-10-4.el10.noarch                                                                
  openblas-srpm-macros-2-19.el10.noarch                                                             
  package-notes-srpm-macros-0.5-13.el10.noarch                                                      
  patch-2.7.6-26.el10.aarch64                                                                       
  perl-Error-1:0.17029-18.el10.noarch                                                               
  perl-Git-2.47.3-1.el10_0.noarch                                                                   
  perl-TermReadKey-2.38-24.el10.aarch64                                                             
  perl-lib-0.65-512.2.el10_0.aarch64                                                                
  perl-srpm-macros-1-57.el10.noarch                                                                 
  pyproject-srpm-macros-1.16.2-1.el10.noarch                                                        
  python-srpm-macros-3.12-10.el10.noarch                                                            
  qt6-srpm-macros-6.9.1-1.el10.noarch                                                               
  redhat-rpm-config-293-1.el10.alma.2.noarch                                                        
  rpm-build-4.19.1.1-20.el10.alma.1.aarch64                                                         
  rpmdevtools-9.6-9.el10.noarch                                                                     
  rust-toolset-srpm-macros-1.88.0-1.el10.alma.1.noarch                                              
  systemd-rpm-macros-257-13.el10_1.3.alma.1.noarch                                                  
  yum-utils-4.7.0-9.el10.noarch                                                                     
  zstd-1.5.5-9.el10.aarch64                                                                         

Complete!
root@alma10:/home/user# rpm -qi git
Name        : git
Version     : 2.47.3
Release     : 1.el10_0
Architecture: aarch64
Install Date: Fri 15 May 2026 11:44:36 AM MSK
Group       : Unspecified
Size        : 87295
License     : BSD-3-Clause AND GPL-2.0-only AND GPL-2.0-or-later AND LGPL-2.1-or-later AND MIT
Signature   :
              RSA/SHA256, Wed 23 Jul 2025 10:34:03 PM MSK, Key ID dee5c11cc2a1e572
              RSA/SHA256, Wed 23 Jul 2025 10:34:04 PM MSK, Key ID dee5c11cc2a1e572
Source RPM  : git-2.47.3-1.el10_0.src.rpm
Build Date  : Tue 22 Jul 2025 09:00:41 PM MSK
Build Host  : arm64-builder01.almalinux.org
Packager    : AlmaLinux Packaging Team <packager@almalinux.org>
Vendor      : AlmaLinux
URL         : https://git-scm.com/
Summary     : Fast Version Control System
Description :
Git is a fast, scalable, distributed revision control system with an
unusually rich command set that provides both high-level operations
and full access to internals.

The git rpm installs common set of tools which are usually using with
small amount of dependencies. To install all git packages, including
tools for integrating with other SCMs, install the git-all meta-package.
root@alma10:/home/user# rpm -qi gcc
Name        : gcc
Version     : 14.3.1
Release     : 2.1.el10.alma.1
Architecture: aarch64
Install Date: Fri 15 May 2026 11:44:37 AM MSK
Group       : Unspecified
Size        : 104317042
License     : GPL-3.0-or-later AND LGPL-3.0-or-later AND (GPL-3.0-or-later WITH GCC-exception-3.1) AND (GPL-3.0-or-later WITH Texinfo-exception) AND (LGPL-2.1-or-later WITH GCC-exception-2.0) AND (GPL-2.0-or-later WITH GCC-exception-2.0) AND (GPL-2.0-or-later WITH GNU-compiler-exception) AND BSL-1.0 AND GFDL-1.3-or-later AND Linux-man-pages-copyleft-2-para AND SunPro AND BSD-1-Clause AND BSD-2-Clause AND BSD-2-Clause-Views AND BSD-3-Clause AND BSD-4-Clause AND BSD-Source-Code AND Zlib AND MIT AND Apache-2.0 AND (Apache-2.0 WITH LLVM-Exception) AND ZPL-2.1 AND ISC AND LicenseRef-Fedora-Public-Domain AND HP-1986 AND curl AND Martin-Birgmeier AND HPND-Markus-Kuhn AND dtoa AND SMLNJ AND AMD-newlib AND OAR AND HPND-merchantability-variant AND HPND-Intel
Signature   :
              RSA/SHA256, Wed 02 Jul 2025 11:11:42 AM MSK, Key ID dee5c11cc2a1e572
              RSA/SHA256, Wed 02 Jul 2025 11:11:43 AM MSK, Key ID dee5c11cc2a1e572
Source RPM  : gcc-14.3.1-2.1.el10.alma.1.src.rpm
Build Date  : Tue 01 Jul 2025 11:02:56 AM MSK
Build Host  : arm-builder03-almalinux-org.novalocal
Packager    : AlmaLinux Packaging Team <packager@almalinux.org>
Vendor      : AlmaLinux
URL         : http://gcc.gnu.org
Summary     : Various compilers (C, C++, Objective-C, ...)
Description :
The gcc package contains the GNU Compiler Collection version 14.
You'll need this package in order to compile C code.
root@alma10:/home/user# rpm -qi cmake
Name        : cmake
Version     : 3.30.5
Release     : 3.el10_0
Architecture: aarch64
Install Date: Fri 15 May 2026 11:44:36 AM MSK
Group       : Unspecified
Size        : 30279573
License     : BSD-3-Clause AND MIT-open-group AND Zlib AND Apache-2.0
Signature   :
              RSA/SHA256, Wed 25 Jun 2025 09:02:44 AM MSK, Key ID dee5c11cc2a1e572
              RSA/SHA256, Wed 25 Jun 2025 09:02:45 AM MSK, Key ID dee5c11cc2a1e572
Source RPM  : cmake-3.30.5-3.el10_0.src.rpm
Build Date  : Tue 24 Jun 2025 01:00:24 PM MSK
Build Host  : arm64-builder01.almalinux.org
Packager    : AlmaLinux Packaging Team <packager@almalinux.org>
Vendor      : AlmaLinux
URL         : http://www.cmake.org
Summary     : Cross-platform make system
Description :
CMake is used to control the software compilation process using simple
platform and compiler independent configuration files. CMake generates
native makefiles and workspaces that can be used in the compiler
environment of your choice. CMake is quite sophisticated: it is possible
to support complex environments requiring system configuration, preprocessor
generation, code generation, and template instantiation.
root@alma10:/home/user# rpm -qi yum-utils
Name        : yum-utils
Version     : 4.7.0
Release     : 9.el10
Architecture: noarch
Install Date: Fri 15 May 2026 11:44:37 AM MSK
Group       : Unspecified
Size        : 21891
License     : GPL-2.0-or-later
Signature   :
              RSA/SHA256, Wed 16 Apr 2025 10:52:17 AM MSK, Key ID dee5c11cc2a1e572
              RSA/SHA256, Wed 16 Apr 2025 10:52:18 AM MSK, Key ID dee5c11cc2a1e572
Source RPM  : dnf-plugins-core-4.7.0-9.el10.src.rpm
Build Date  : Tue 15 Apr 2025 10:27:06 PM MSK
Build Host  : s390x-builder01.almalinux.org
Packager    : AlmaLinux Packaging Team <packager@almalinux.org>
Vendor      : AlmaLinux
URL         : https://github.com/rpm-software-management/dnf-plugins-core
Summary     : Yum-utils CLI compatibility layer
Description :
As a Yum-utils CLI compatibility layer, supplies in CLI shims for
debuginfo-install, repograph, package-cleanup, repoclosure, repomanage,
repoquery, reposync, repotrack, repodiff, builddep, config-manager,
download and yum-groups-manager that use new implementations using DNF.
root@alma10:/home/user# rpm -qi createrepo
package createrepo is not installed
root@alma10:/home/user# rpm -qi rpm-build
Name        : rpm-build
Version     : 4.19.1.1
Release     : 20.el10.alma.1
Architecture: aarch64
Install Date: Fri 15 May 2026 11:44:37 AM MSK
Group       : Unspecified
Size        : 467333
License     : GPL-2.0-or-later
Signature   :
              RSA/SHA256, Sat 06 Sep 2025 02:40:23 PM MSK, Key ID dee5c11cc2a1e572
              RSA/SHA256, Sat 06 Sep 2025 02:40:24 PM MSK, Key ID dee5c11cc2a1e572
Source RPM  : rpm-4.19.1.1-20.el10.alma.1.src.rpm
Build Date  : Sat 06 Sep 2025 06:12:34 AM MSK
Build Host  : arm64-builder01.almalinux.org
Packager    : AlmaLinux Packaging Team <packager@almalinux.org>
Vendor      : AlmaLinux
URL         : http://www.rpm.org/
Summary     : Scripts and executable programs used to build packages
Description :
The rpm-build package contains the scripts and executable programs
that are used to build packages using the RPM Package Manager.
root@alma10:/home/user# rpm -qi rpmdevtools
Name        : rpmdevtools
Version     : 9.6
Release     : 9.el10
Architecture: noarch
Install Date: Fri 15 May 2026 11:44:37 AM MSK
Group       : Unspecified
Size        : 202812
License     : GPL-2.0-or-later AND GPL-2.0-only
Signature   :
              RSA/SHA256, Sun 15 Dec 2024 04:07:42 PM MSK, Key ID dee5c11cc2a1e572
              RSA/SHA256, Sun 15 Dec 2024 04:07:43 PM MSK, Key ID dee5c11cc2a1e572
Source RPM  : rpmdevtools-9.6-9.el10.src.rpm
Build Date  : Sun 15 Dec 2024 06:41:59 AM MSK
Build Host  : s390x-builder02.almalinux.org
Packager    : AlmaLinux Packaging Team <packager@almalinux.org>
Vendor      : AlmaLinux
URL         : https://pagure.io/rpmdevtools
Summary     : RPM Development Tools
Description :
This package contains scripts and Emacs support files to aid in
development of RPM packages.
rpmdev-setuptree    Create RPM build tree within user's home directory
rpmdev-diff         Diff contents of two archives
rpmdev-newspec      Creates new .spec from template
rpmdev-rmdevelrpms  Find (and optionally remove) "development" RPMs
rpmdev-checksig     Check package signatures using alternate RPM keyring
rpminfo             Print information about executables and libraries
rpmdev-md5/sha*     Display checksums of all files in an archive file
rpmdev-vercmp       RPM version comparison checker
rpmdev-spectool     Expand and download sources and patches in specfiles
rpmdev-wipetree     Erase all files within dirs created by rpmdev-setuptree
rpmdev-extract      Extract various archives, "tar xvf" style
rpmdev-bumpspec     Bump revision in specfile
...and many more.
```

В качестве примера соберем пакет Nginx с дополнительным модулем ngx_brotli.
Создадим отдельную директорию и загрузим SRPM пакет Nginx для дальнейшей работы над ним:

```
root@alma10:/home/user# mkdir rpm && cd rpm
root@alma10:/home/user/rpm# yumdownloader --source nginx
enabling appstream-source repository
enabling baseos-source repository
enabling crb-source repository
enabling extras-source repository
AlmaLinux 10 - AppStream - Source                                   435 kB/s | 527 kB     00:01    
AlmaLinux 10 - BaseOS - Source                                      152 kB/s | 184 kB     00:01    
AlmaLinux 10 - CRB - Source                                         100 kB/s |  88 kB     00:00    
AlmaLinux 10 - Extras - Source                                      5.1 kB/s | 4.1 kB     00:00    
nginx-1.26.3-2.el10_1.1.alma.1.src.rpm                              1.8 MB/s | 1.3 MB     00:00    
root@alma10:/home/user/rpm# ll
total 1348
-rw-r--r--. 1 root root 1380042 May 15 11:57 nginx-1.26.3-2.el10_1.1.alma.1.src.rpm
```

Установим пакет Nginx и зависимости необходимые для сборки:

```
root@alma10:/home/user/rpm# rpm -Uvh nginx-1.26.3-2.el10_1.1.alma.1.src.rpm
Updating / installing...
   1:nginx-2:1.26.3-2.el10_1.1.alma.1 ################################# [100%]

root@alma10:/home/user/rpm# yum-builddep nginx
enabling appstream-source repository
enabling baseos-source repository
enabling crb-source repository
enabling extras-source repository
Last metadata expiration check: 0:13:10 ago on Fri 15 May 2026 11:57:02 AM MSK.
Package make-1:4.4.1-9.el10.aarch64 is already installed.
Package gcc-14.3.1-2.1.el10.alma.1.aarch64 is already installed.
Package systemd-rpm-macros-257-13.el10_1.3.alma.1.noarch is already installed.
Package systemd-257-13.el10_1.3.alma.1.aarch64 is already installed.
Package gnupg2-2.4.5-4.el10_1.aarch64 is already installed.
Dependencies resolved.
====================================================================================================
 Package                          Architecture Version                         Repository      Size
====================================================================================================
Installing:
 gd-devel                         aarch64      2.3.3-20.el10_0                 appstream       40 k
 libxslt-devel                    aarch64      1.1.39-8.el10_0                 appstream      124 k
 openssl-devel                    aarch64      1:3.5.1-7.el10_1.alma.1         appstream      2.9 M
 pcre2-devel                      aarch64      10.44-1.el10.3                  appstream      535 k
 perl-ExtUtils-Embed              noarch       1.35-512.2.el10_0               appstream       17 k
 perl-devel                       aarch64      4:5.40.2-512.2.el10_0           appstream      750 k
 perl-generators                  noarch       1.16-7.el10                     appstream       16 k
 zlib-ng-compat-devel             aarch64      2.2.3-3.el10_1                  appstream       36 k
Installing dependencies:
 brotli                           aarch64      1.1.0-7.el10_1                  appstream       19 k
 brotli-devel                     aarch64      1.1.0-7.el10_1                  appstream       34 k
 bzip2-devel                      aarch64      1.0.8-25.el10                   appstream      214 k
 cairo-devel                      aarch64      1.18.2-2.el10                   appstream      199 k
 fontconfig-devel                 aarch64      2.15.0-7.el10                   appstream      179 k
 freetype-devel                   aarch64      2.13.2-8.el10                   appstream      966 k
 glib2-devel                      aarch64      2.80.4-10.el10_1.12             appstream      1.4 M
 graphite2-devel                  aarch64      1.3.14-17.el10                  appstream       21 k
 harfbuzz-cairo                   aarch64      8.4.0-6.el10                    appstream       28 k
 harfbuzz-devel                   aarch64      8.4.0-6.el10                    appstream      460 k
 harfbuzz-icu                     aarch64      8.4.0-6.el10                    appstream       15 k
 libX11-devel                     aarch64      1.8.10-1.el10                   appstream      1.1 M
 libXau-devel                     aarch64      1.0.11-8.el10                   appstream       14 k
 libXext-devel                    aarch64      1.3.6-3.el10                    appstream       91 k
 libXpm-devel                     aarch64      3.5.17-5.el10                   appstream       69 k
 libXrender-devel                 aarch64      0.9.11-8.el10                   appstream       19 k
 libXt                            aarch64      1.3.0-5.el10                    appstream      175 k
 libblkid-devel                   aarch64      2.40.2-16.el10_1                appstream       25 k
 libffi-devel                     aarch64      3.4.4-10.el10                   appstream       27 k
 libgpg-error-devel               aarch64      1.50-2.el10                     appstream       70 k
 libicu-devel                     aarch64      74.2-5.el10_0                   appstream      855 k
 libjpeg-turbo-devel              aarch64      3.0.2-4.el10                    appstream      102 k
 libmount-devel                   aarch64      2.40.2-16.el10_1                appstream       26 k
 libpng-devel                     aarch64      2:1.6.40-8.el10_1.3             appstream      291 k
 libselinux-devel                 aarch64      3.9-1.el10                      appstream      116 k
 libsepol-devel                   aarch64      3.9-1.el10                      appstream       40 k
 libtiff-devel                    aarch64      4.6.0-6.el10_1.3                appstream      246 k
 libwebp-devel                    aarch64      1.3.2-8.el10                    appstream       43 k
 libxcb-devel                     aarch64      1.17.0-3.el10                   appstream      1.6 M
 libxml2-devel                    aarch64      2.12.5-9.el10_0                 appstream      495 k
 libzstd-devel                    aarch64      1.5.5-9.el10                    appstream       51 k
 pcre2-utf16                      aarch64      10.44-1.el10.3                  appstream      208 k
 perl-AutoSplit                   noarch       5.74-512.2.el10_0               appstream       21 k
 perl-Benchmark                   noarch       1.25-512.2.el10_0               appstream       26 k
 perl-CPAN-Meta-Requirements      noarch       2.143-11.el10                   appstream       35 k
 perl-CPAN-Meta-YAML              noarch       0.018-512.el10                  appstream       26 k
 perl-Devel-PPPort                aarch64      3.72-512.el10                   appstream      216 k
 perl-ExtUtils-Command            noarch       2:7.70-513.el10                 appstream       14 k
 perl-ExtUtils-Constant           noarch       0.25-512.2.el10_0               appstream       43 k
 perl-ExtUtils-Install            noarch       2.22-511.el10                   appstream       43 k
 perl-ExtUtils-MakeMaker          noarch       2:7.70-513.el10                 appstream      296 k
 perl-ExtUtils-Manifest           noarch       1:1.75-511.el10                 appstream       34 k
 perl-ExtUtils-ParseXS            noarch       1:3.51-512.el10                 appstream      189 k
 perl-Fedora-VSP                  noarch       0.001-36.el10                   appstream       23 k
 perl-File-Compare                noarch       1.100.800-512.2.el10_0          appstream       13 k
 perl-File-Copy                   noarch       2.41-512.2.el10_0               appstream       20 k
 perl-I18N-Langinfo               aarch64      0.24-512.2.el10_0               appstream       25 k
 perl-JSON-PP                     noarch       1:4.16-512.el10                 appstream       66 k
 perl-Test-Harness                noarch       1:3.48-512.el10                 appstream      283 k
 perl-macros                      noarch       4:5.40.2-512.2.el10_0           appstream       12 k
 perl-version                     aarch64      8:0.99.32-4.el10                appstream       62 k
 pixman-devel                     aarch64      0.43.4-2.el10                   appstream       17 k
 python3-pyparsing                noarch       3.1.1-7.el10                    baseos         271 k
 sysprof-capture-devel            aarch64      47.2-1.el10                     appstream       62 k
 systemtap-sdt-devel              aarch64      5.3-3b.el10                     appstream       69 k
 systemtap-sdt-dtrace             aarch64      5.3-3b.el10                     appstream       70 k
 xorg-x11-proto-devel             noarch       2024.1-3.el10                   appstream      314 k
 xz-devel                         aarch64      1:5.6.2-4.el10_0                appstream       67 k
Installing weak dependencies:
 perl-CPAN-Meta                   noarch       2.150010-511.el10               appstream      198 k
 perl-Encode-Locale               noarch       1.05-31.el10                    appstream       18 k
 perl-Time-HiRes                  aarch64      4:1.9777-511.el10               appstream       57 k
 perl-doc                         noarch       5.40.2-512.2.el10_0             appstream      4.8 M

Transaction Summary
====================================================================================================
Install  70 Packages

Total download size: 21 M
Installed size: 83 M
Is this ok [y/N]: y
Downloading Packages:
(1/70): brotli-1.1.0-7.el10_1.aarch64.rpm                           221 kB/s |  19 kB     00:00    
(2/70): bzip2-devel-1.0.8-25.el10.aarch64.rpm                       1.6 MB/s | 214 kB     00:00    
(3/70): brotli-devel-1.1.0-7.el10_1.aarch64.rpm                     213 kB/s |  34 kB     00:00    
(4/70): cairo-devel-1.18.2-2.el10.aarch64.rpm                       1.9 MB/s | 199 kB     00:00    
(5/70): gd-devel-2.3.3-20.el10_0.aarch64.rpm                        595 kB/s |  40 kB     00:00    
(6/70): fontconfig-devel-2.15.0-7.el10.aarch64.rpm                  1.0 MB/s | 179 kB     00:00    
(7/70): freetype-devel-2.13.2-8.el10.aarch64.rpm                    5.1 MB/s | 966 kB     00:00    
(8/70): graphite2-devel-1.3.14-17.el10.aarch64.rpm                  507 kB/s |  21 kB     00:00    
(9/70): harfbuzz-cairo-8.4.0-6.el10.aarch64.rpm                     939 kB/s |  28 kB     00:00    
(10/70): harfbuzz-icu-8.4.0-6.el10.aarch64.rpm                      408 kB/s |  15 kB     00:00    
(11/70): harfbuzz-devel-8.4.0-6.el10.aarch64.rpm                    1.8 MB/s | 460 kB     00:00    
(12/70): libX11-devel-1.8.10-1.el10.aarch64.rpm                     4.8 MB/s | 1.1 MB     00:00    
(13/70): libXau-devel-1.0.11-8.el10.aarch64.rpm                     308 kB/s |  14 kB     00:00    
(14/70): libXext-devel-1.3.6-3.el10.aarch64.rpm                     1.8 MB/s |  91 kB     00:00    
(15/70): libXpm-devel-3.5.17-5.el10.aarch64.rpm                     1.8 MB/s |  69 kB     00:00    
(16/70): libXrender-devel-0.9.11-8.el10.aarch64.rpm                 479 kB/s |  19 kB     00:00    
(17/70): libblkid-devel-2.40.2-16.el10_1.aarch64.rpm                808 kB/s |  25 kB     00:00    
(18/70): libXt-1.3.0-5.el10.aarch64.rpm                             2.4 MB/s | 175 kB     00:00    
(19/70): libgpg-error-devel-1.50-2.el10.aarch64.rpm                 1.6 MB/s |  70 kB     00:00    
(20/70): libffi-devel-3.4.4-10.el10.aarch64.rpm                     417 kB/s |  27 kB     00:00    
(21/70): libjpeg-turbo-devel-3.0.2-4.el10.aarch64.rpm               474 kB/s | 102 kB     00:00    
(22/70): libicu-devel-74.2-5.el10_0.aarch64.rpm                     3.4 MB/s | 855 kB     00:00    
(23/70): libmount-devel-2.40.2-16.el10_1.aarch64.rpm                829 kB/s |  26 kB     00:00    
(24/70): libselinux-devel-3.9-1.el10.aarch64.rpm                    1.7 MB/s | 116 kB     00:00    
(25/70): libpng-devel-1.6.40-8.el10_1.3.aarch64.rpm                 2.4 MB/s | 291 kB     00:00    
(26/70): libsepol-devel-3.9-1.el10.aarch64.rpm                      776 kB/s |  40 kB     00:00    
(27/70): glib2-devel-2.80.4-10.el10_1.12.aarch64.rpm                1.5 MB/s | 1.4 MB     00:00    
(28/70): libwebp-devel-1.3.2-8.el10.aarch64.rpm                     1.0 MB/s |  43 kB     00:00    
(29/70): libtiff-devel-4.6.0-6.el10_1.3.aarch64.rpm                 2.7 MB/s | 246 kB     00:00    
(30/70): libxslt-devel-1.1.39-8.el10_0.aarch64.rpm                  1.7 MB/s | 124 kB     00:00    
(31/70): libzstd-devel-1.5.5-9.el10.aarch64.rpm                     1.6 MB/s |  51 kB     00:00    
(32/70): libxml2-devel-2.12.5-9.el10_0.aarch64.rpm                  3.0 MB/s | 495 kB     00:00    
(33/70): pcre2-devel-10.44-1.el10.3.aarch64.rpm                     2.7 MB/s | 535 kB     00:00    
(34/70): libxcb-devel-1.17.0-3.el10.aarch64.rpm                     3.6 MB/s | 1.6 MB     00:00    
(35/70): perl-AutoSplit-5.74-512.2.el10_0.noarch.rpm                651 kB/s |  21 kB     00:00    
(36/70): perl-Benchmark-1.25-512.2.el10_0.noarch.rpm                658 kB/s |  26 kB     00:00    
(37/70): pcre2-utf16-10.44-1.el10.3.aarch64.rpm                     1.4 MB/s | 208 kB     00:00    
(38/70): perl-CPAN-Meta-Requirements-2.143-11.el10.noarch.rpm       1.2 MB/s |  35 kB     00:00    
(39/70): perl-CPAN-Meta-YAML-0.018-512.el10.noarch.rpm              762 kB/s |  26 kB     00:00    
(40/70): perl-CPAN-Meta-2.150010-511.el10.noarch.rpm                2.1 MB/s | 198 kB     00:00    
(41/70): perl-Encode-Locale-1.05-31.el10.noarch.rpm                 480 kB/s |  18 kB     00:00    
(42/70): perl-ExtUtils-Command-7.70-513.el10.noarch.rpm             449 kB/s |  14 kB     00:00    
(43/70): perl-ExtUtils-Constant-0.25-512.2.el10_0.noarch.rpm        1.2 MB/s |  43 kB     00:00    
(44/70): perl-Devel-PPPort-3.72-512.el10.aarch64.rpm                1.6 MB/s | 216 kB     00:00    
(45/70): perl-ExtUtils-Install-2.22-511.el10.noarch.rpm             1.4 MB/s |  43 kB     00:00    
(46/70): perl-ExtUtils-Embed-1.35-512.2.el10_0.noarch.rpm           450 kB/s |  17 kB     00:00    
(47/70): perl-ExtUtils-Manifest-1.75-511.el10.noarch.rpm            878 kB/s |  34 kB     00:00    
(48/70): perl-ExtUtils-ParseXS-3.51-512.el10.noarch.rpm             2.7 MB/s | 189 kB     00:00    
(49/70): openssl-devel-3.5.1-7.el10_1.alma.1.aarch64.rpm            3.9 MB/s | 2.9 MB     00:00    
(50/70): perl-Fedora-VSP-0.001-36.el10.noarch.rpm                   1.0 MB/s |  23 kB     00:00    
(51/70): perl-File-Compare-1.100.800-512.2.el10_0.noarch.rpm        645 kB/s |  13 kB     00:00    
(52/70): perl-ExtUtils-MakeMaker-7.70-513.el10.noarch.rpm           1.7 MB/s | 296 kB     00:00    
(53/70): perl-I18N-Langinfo-0.24-512.2.el10_0.aarch64.rpm           939 kB/s |  25 kB     00:00    
(54/70): perl-File-Copy-2.41-512.2.el10_0.noarch.rpm                197 kB/s |  20 kB     00:00    
(55/70): perl-JSON-PP-4.16-512.el10.noarch.rpm                      935 kB/s |  66 kB     00:00    
(56/70): perl-Test-Harness-3.48-512.el10.noarch.rpm                 2.5 MB/s | 283 kB     00:00    
(57/70): perl-Time-HiRes-1.9777-511.el10.aarch64.rpm                1.1 MB/s |  57 kB     00:00    
(58/70): perl-generators-1.16-7.el10.noarch.rpm                     758 kB/s |  16 kB     00:00    
(59/70): perl-macros-5.40.2-512.2.el10_0.noarch.rpm                 596 kB/s |  12 kB     00:00    
(60/70): perl-version-0.99.32-4.el10.aarch64.rpm                    1.5 MB/s |  62 kB     00:00    
(61/70): pixman-devel-0.43.4-2.el10.aarch64.rpm                     446 kB/s |  17 kB     00:00    
(62/70): sysprof-capture-devel-47.2-1.el10.aarch64.rpm              1.0 MB/s |  62 kB     00:00    
(63/70): systemtap-sdt-devel-5.3-3b.el10.aarch64.rpm                1.4 MB/s |  69 kB     00:00    
(64/70): systemtap-sdt-dtrace-5.3-3b.el10.aarch64.rpm               1.4 MB/s |  70 kB     00:00    
(65/70): xorg-x11-proto-devel-2024.1-3.el10.noarch.rpm              2.4 MB/s | 314 kB     00:00    
(66/70): perl-devel-5.40.2-512.2.el10_0.aarch64.rpm                 1.6 MB/s | 750 kB     00:00    
(67/70): zlib-ng-compat-devel-2.2.3-3.el10_1.aarch64.rpm            1.1 MB/s |  36 kB     00:00    
(68/70): xz-devel-5.6.2-4.el10_0.aarch64.rpm                        673 kB/s |  67 kB     00:00    
(69/70): python3-pyparsing-3.1.1-7.el10.noarch.rpm                  1.7 MB/s | 271 kB     00:00    
(70/70): perl-doc-5.40.2-512.2.el10_0.noarch.rpm                    5.6 MB/s | 4.8 MB     00:00    
----------------------------------------------------------------------------------------------------
Total                                                               1.0 MB/s |  21 MB     00:20     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                            1/1 
  Installing       : zlib-ng-compat-devel-2.2.3-3.el10_1.aarch64                               1/70 
  Installing       : xorg-x11-proto-devel-2024.1-3.el10.noarch                                 2/70 
  Installing       : perl-version-8:0.99.32-4.el10.aarch64                                     3/70 
  Installing       : libpng-devel-2:1.6.40-8.el10_1.3.aarch64                                  4/70 
  Installing       : perl-File-Copy-2.41-512.2.el10_0.noarch                                   5/70 
  Installing       : perl-CPAN-Meta-Requirements-2.143-11.el10.noarch                          6/70 
  Installing       : perl-Time-HiRes-4:1.9777-511.el10.aarch64                                 7/70 
  Installing       : perl-JSON-PP-1:4.16-512.el10.noarch                                       8/70 
  Installing       : perl-File-Compare-1.100.800-512.2.el10_0.noarch                           9/70 
  Installing       : perl-ExtUtils-ParseXS-1:3.51-512.el10.noarch                             10/70 
  Installing       : libwebp-devel-1.3.2-8.el10.aarch64                                       11/70 
  Installing       : libjpeg-turbo-devel-3.0.2-4.el10.aarch64                                 12/70 
  Installing       : perl-ExtUtils-Command-2:7.70-513.el10.noarch                             13/70 
  Installing       : perl-ExtUtils-Manifest-1:1.75-511.el10.noarch                            14/70 
  Installing       : libXau-devel-1.0.11-8.el10.aarch64                                       15/70 
  Installing       : libxcb-devel-1.17.0-3.el10.aarch64                                       16/70 
  Installing       : libX11-devel-1.8.10-1.el10.aarch64                                       17/70 
  Installing       : libXext-devel-1.3.6-3.el10.aarch64                                       18/70 
  Installing       : libXrender-devel-0.9.11-8.el10.aarch64                                   19/70 
  Installing       : python3-pyparsing-3.1.1-7.el10.noarch                                    20/70 
  Installing       : systemtap-sdt-dtrace-5.3-3b.el10.aarch64                                 21/70 
  Installing       : systemtap-sdt-devel-5.3-3b.el10.aarch64                                  22/70 
  Installing       : xz-devel-1:5.6.2-4.el10_0.aarch64                                        23/70 
  Installing       : libxml2-devel-2.12.5-9.el10_0.aarch64                                    24/70 
  Installing       : sysprof-capture-devel-47.2-1.el10.aarch64                                25/70 
  Installing       : pixman-devel-0.43.4-2.el10.aarch64                                       26/70 
  Installing       : perl-macros-4:5.40.2-512.2.el10_0.noarch                                 27/70 
  Installing       : perl-doc-5.40.2-512.2.el10_0.noarch                                      28/70 
  Installing       : perl-I18N-Langinfo-0.24-512.2.el10_0.aarch64                             29/70 
  Installing       : perl-Encode-Locale-1.05-31.el10.noarch                                   30/70 
  Installing       : perl-Fedora-VSP-0.001-36.el10.noarch                                     31/70 
  Installing       : perl-ExtUtils-Constant-0.25-512.2.el10_0.noarch                          32/70 
  Installing       : perl-Devel-PPPort-3.72-512.el10.aarch64                                  33/70 
  Installing       : perl-CPAN-Meta-YAML-0.018-512.el10.noarch                                34/70 
  Installing       : perl-CPAN-Meta-2.150010-511.el10.noarch                                  35/70 
  Installing       : perl-Benchmark-1.25-512.2.el10_0.noarch                                  36/70 
  Installing       : perl-Test-Harness-1:3.48-512.el10.noarch                                 37/70 
  Installing       : perl-AutoSplit-5.74-512.2.el10_0.noarch                                  38/70 
  Installing       : perl-ExtUtils-MakeMaker-2:7.70-513.el10.noarch                           39/70 
  Installing       : perl-ExtUtils-Install-2.22-511.el10.noarch                               40/70 
  Installing       : perl-devel-4:5.40.2-512.2.el10_0.aarch64                                 41/70 
  Installing       : pcre2-utf16-10.44-1.el10.3.aarch64                                       42/70 
  Installing       : pcre2-devel-10.44-1.el10.3.aarch64                                       43/70 
  Installing       : libzstd-devel-1.5.5-9.el10.aarch64                                       44/70 
  Installing       : libtiff-devel-4.6.0-6.el10_1.3.aarch64                                   45/70 
  Installing       : libsepol-devel-3.9-1.el10.aarch64                                        46/70 
  Installing       : libselinux-devel-3.9-1.el10.aarch64                                      47/70 
  Installing       : libicu-devel-74.2-5.el10_0.aarch64                                       48/70 
  Installing       : libgpg-error-devel-1.50-2.el10.aarch64                                   49/70 
  Installing       : libffi-devel-3.4.4-10.el10.aarch64                                       50/70 
  Installing       : libblkid-devel-2.40.2-16.el10_1.aarch64                                  51/70 
  Installing       : libmount-devel-2.40.2-16.el10_1.aarch64                                  52/70 
  Installing       : glib2-devel-2.80.4-10.el10_1.12.aarch64                                  53/70 
  Installing       : libXt-1.3.0-5.el10.aarch64                                               54/70 
  Installing       : libXpm-devel-3.5.17-5.el10.aarch64                                       55/70 
  Installing       : harfbuzz-icu-8.4.0-6.el10.aarch64                                        56/70 
  Installing       : harfbuzz-cairo-8.4.0-6.el10.aarch64                                      57/70 
  Installing       : graphite2-devel-1.3.14-17.el10.aarch64                                   58/70 
  Installing       : bzip2-devel-1.0.8-25.el10.aarch64                                        59/70 
  Installing       : brotli-1.1.0-7.el10_1.aarch64                                            60/70 
  Installing       : brotli-devel-1.1.0-7.el10_1.aarch64                                      61/70 
  Installing       : cairo-devel-1.18.2-2.el10.aarch64                                        62/70 
  Installing       : fontconfig-devel-2.15.0-7.el10.aarch64                                   63/70 
  Installing       : harfbuzz-devel-8.4.0-6.el10.aarch64                                      64/70 
  Installing       : freetype-devel-2.13.2-8.el10.aarch64                                     65/70 
  Installing       : gd-devel-2.3.3-20.el10_0.aarch64                                         66/70 
  Installing       : libxslt-devel-1.1.39-8.el10_0.aarch64                                    67/70 
  Installing       : perl-ExtUtils-Embed-1.35-512.2.el10_0.noarch                             68/70 
  Installing       : perl-generators-1.16-7.el10.noarch                                       69/70 
  Installing       : openssl-devel-1:3.5.1-7.el10_1.alma.1.aarch64                            70/70 
  Running scriptlet: openssl-devel-1:3.5.1-7.el10_1.alma.1.aarch64                            70/70 

Installed:
  brotli-1.1.0-7.el10_1.aarch64                    brotli-devel-1.1.0-7.el10_1.aarch64              
  bzip2-devel-1.0.8-25.el10.aarch64                cairo-devel-1.18.2-2.el10.aarch64                
  fontconfig-devel-2.15.0-7.el10.aarch64           freetype-devel-2.13.2-8.el10.aarch64             
  gd-devel-2.3.3-20.el10_0.aarch64                 glib2-devel-2.80.4-10.el10_1.12.aarch64          
  graphite2-devel-1.3.14-17.el10.aarch64           harfbuzz-cairo-8.4.0-6.el10.aarch64              
  harfbuzz-devel-8.4.0-6.el10.aarch64              harfbuzz-icu-8.4.0-6.el10.aarch64                
  libX11-devel-1.8.10-1.el10.aarch64               libXau-devel-1.0.11-8.el10.aarch64               
  libXext-devel-1.3.6-3.el10.aarch64               libXpm-devel-3.5.17-5.el10.aarch64               
  libXrender-devel-0.9.11-8.el10.aarch64           libXt-1.3.0-5.el10.aarch64                       
  libblkid-devel-2.40.2-16.el10_1.aarch64          libffi-devel-3.4.4-10.el10.aarch64               
  libgpg-error-devel-1.50-2.el10.aarch64           libicu-devel-74.2-5.el10_0.aarch64               
  libjpeg-turbo-devel-3.0.2-4.el10.aarch64         libmount-devel-2.40.2-16.el10_1.aarch64          
  libpng-devel-2:1.6.40-8.el10_1.3.aarch64         libselinux-devel-3.9-1.el10.aarch64              
  libsepol-devel-3.9-1.el10.aarch64                libtiff-devel-4.6.0-6.el10_1.3.aarch64           
  libwebp-devel-1.3.2-8.el10.aarch64               libxcb-devel-1.17.0-3.el10.aarch64               
  libxml2-devel-2.12.5-9.el10_0.aarch64            libxslt-devel-1.1.39-8.el10_0.aarch64            
  libzstd-devel-1.5.5-9.el10.aarch64               openssl-devel-1:3.5.1-7.el10_1.alma.1.aarch64    
  pcre2-devel-10.44-1.el10.3.aarch64               pcre2-utf16-10.44-1.el10.3.aarch64               
  perl-AutoSplit-5.74-512.2.el10_0.noarch          perl-Benchmark-1.25-512.2.el10_0.noarch          
  perl-CPAN-Meta-2.150010-511.el10.noarch          perl-CPAN-Meta-Requirements-2.143-11.el10.noarch 
  perl-CPAN-Meta-YAML-0.018-512.el10.noarch        perl-Devel-PPPort-3.72-512.el10.aarch64          
  perl-Encode-Locale-1.05-31.el10.noarch           perl-ExtUtils-Command-2:7.70-513.el10.noarch     
  perl-ExtUtils-Constant-0.25-512.2.el10_0.noarch  perl-ExtUtils-Embed-1.35-512.2.el10_0.noarch     
  perl-ExtUtils-Install-2.22-511.el10.noarch       perl-ExtUtils-MakeMaker-2:7.70-513.el10.noarch   
  perl-ExtUtils-Manifest-1:1.75-511.el10.noarch    perl-ExtUtils-ParseXS-1:3.51-512.el10.noarch     
  perl-Fedora-VSP-0.001-36.el10.noarch             perl-File-Compare-1.100.800-512.2.el10_0.noarch  
  perl-File-Copy-2.41-512.2.el10_0.noarch          perl-I18N-Langinfo-0.24-512.2.el10_0.aarch64     
  perl-JSON-PP-1:4.16-512.el10.noarch              perl-Test-Harness-1:3.48-512.el10.noarch         
  perl-Time-HiRes-4:1.9777-511.el10.aarch64        perl-devel-4:5.40.2-512.2.el10_0.aarch64         
  perl-doc-5.40.2-512.2.el10_0.noarch              perl-generators-1.16-7.el10.noarch               
  perl-macros-4:5.40.2-512.2.el10_0.noarch         perl-version-8:0.99.32-4.el10.aarch64            
  pixman-devel-0.43.4-2.el10.aarch64               python3-pyparsing-3.1.1-7.el10.noarch            
  sysprof-capture-devel-47.2-1.el10.aarch64        systemtap-sdt-devel-5.3-3b.el10.aarch64          
  systemtap-sdt-dtrace-5.3-3b.el10.aarch64         xorg-x11-proto-devel-2024.1-3.el10.noarch        
  xz-devel-1:5.6.2-4.el10_0.aarch64                zlib-ng-compat-devel-2.2.3-3.el10_1.aarch64      

Complete!
```

Также скачаем исходный код модуля ngx_brotli. Он потребуется при сборке:

```
root@alma10:~/rpmbuild/SOURCES# cd /root

root@alma10:~# git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli
Cloning into 'ngx_brotli'...
remote: Enumerating objects: 237, done.
remote: Counting objects: 100% (72/72), done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 237 (delta 55), reused 50 (delta 50), pack-reused 165 (from 1)
Receiving objects: 100% (237/237), 78.03 KiB | 1.20 MiB/s, done.
Resolving deltas: 100% (116/116), done.
Submodule 'deps/brotli' (https://github.com/google/brotli.git) registered for path 'deps/brotli'
Cloning into '/root/ngx_brotli/deps/brotli'...
remote: Enumerating objects: 9377, done.        
remote: Counting objects: 100% (123/123), done.        
remote: Compressing objects: 100% (64/64), done.        
remote: Total 9377 (delta 86), reused 59 (delta 59), pack-reused 9254 (from 3)        
Receiving objects: 100% (9377/9377), 41.79 MiB | 5.69 MiB/s, done.
Resolving deltas: 100% (6077/6077), done.
Submodule path 'deps/brotli': checked out 'ed738e842d2fbdf2d6459e39267a633c4a9b2f5d'

root@alma10:~# cd ngx_brotli/deps/brotli/ && mkdir out && cd out
root@alma10:~/ngx_brotli/deps/brotli/out# 
```

Собираем модуль ngx_brotli:

```

```

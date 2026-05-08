# Setting up LINUX hosts for SPOL/LROSE

## Install CENTOS 7 or 8

### Partitions

```
  /boot 2GB
  /     100GB
  swap  64 GB
  /home 100 GB
  /data rest
```

### Packages

```
  KDE plasma desktop
  All related packages
```

### Users

```
  root (shell bash)
  spol (shell csh)
```

### Timezone

```
  UTC
```

### Network

```
  subnet:     192.168.4
  gateway:    192.168.4.251
  DNS:        192.168.4.251
```

### Hosts

```
  control1:   192.168.4.2
  control2:   192.168.4.10

  mgen1:      192.168.4.3
  mgen2:      192.168.4.22
  pgen1:      192.168.4.23
  pgen2:      192.168.4.25

  sci1:       192.168.4.19
  sci2:       192.168.4.20
  sci3:       192.168.4.34
  eng:        192.168.4.24
```

## Mounting usb drives for archival

Make the following directories:

```
  mkdir /mnt/usb1
  mkdir /mnt/usb2
```

Add the following lines to ```/etc/fstab```:

```
/dev/sdc1  /mnt/usb1   ext4    users,noauto 0 0
/dev/sdd1  /mnt/usb2   ext4    users,noauto 0 0
```

You can then mount the usb drives manually as follows:

```
  mount /mnt/usb1
  mount /mnt/usb2
```

This is done automatically in the script:

```
  projDir/archive/scripts/doArchive.py
```

Add spol to disk group using:

```
  usermod -a -G disk spol
```

This is required so that the spol user can run the e2label command.


## Crossmounts

Set up for manual crossmounts from mgen and pgen machines onto control1.

On mgen1, mgen2, pgen1, pgen2, set up the ```/etc/exports``` file as follows:

### mgen1:

```
/data/spol/data.mgen1 192.168.4.2(ro,nohide,insecure,sync,no_subtree_check,root_squash) 
```

### mgen2:

```
/data/spol/data.mgen2 192.168.4.2(ro,nohide,insecure,sync,no_subtree_check,root_squash) 
```

### pgen1, pgen2:

```
/data/spol/data.pgen 192.168.4.2(ro,nohide,insecure,sync,no_subtree_check,root_squash) 
```

### control1, control2:

On control1, and the backup control2, you can then mount the data disks
from the other hosts using the following scripts:

```
  projDir/system/scripts/mount-data
  projDir/system/scripts/mount-mgen1
  projDir/system/scripts/mount-mgen2
  projDir/system/scripts/mount-pgen1
  projDir/system/scripts/mount-pgen2

```

## SHELL - csh

The SPOL runtime configuration expects the login shell to be ```csh``` or ```tcsh```.

## Install packages

Install as follows:

```

yum -y update

yum install -y epel-release

yum install -y \
    tcsh wget git ftp tkcvs emacs m4 \
    tkcvs emacs rsync perl perl-Env python \
    m4 make scons libtool autoconf automake \
    gcc gcc-c++ gcc-gfortran glibc-devel \
    libX11-devel libXext-devel \
    libpng-devel libtiff-devel zlib-devel \
    expat-devel libcurl-devel jasper-devel \
    flex-devel fftw3-devel \
    bzip2-devel qt5-qtbase-devel qt5-qtdeclarative-devel \
    hdf5-devel netcdf-devel \
    rpm-build redhat-rpm-config \
    rpm-devel rpmdevtools \
    gnuplot ImageMagick-devel ImageMagick-c++-devel log4cpp-devel \
    qwt-devel glut-devel boost-devel \
    xorg-x11-xauth xrdb xorg-x11-apps Xvfb \
    xorg-x11-fonts-misc xorg-x11-fonts-75dpi xorg-x11-fonts-100dpi \
    check-mk-agent xinetd

#    compat-libstdc++-296.i686 \
#    compat-libstdc++-33.i686 \

cd /usr/bin
sudo ln -s qmake-qt5 qmake

```

For CIDD, add 32-bit support:

```
yum install -y tcsh perl perl-Env ftp git svn cvs tkcvs emacs \
  gcc gcc-c++ gcc-gfortran \
  glibc-devel.i686 libX11-devel.i686 libXext-devel.i686 \
  libtiff-devel.i686 libpng-devel.i686 libcurl-devel.i686 \
  libstdc++-devel.i686 libtiff-devel.i686 \
  zlib-devel.i686 expat-devel.i686 flex-devel.i686 \
  fftw-devel.i686 bzip2-devel.i686 xrdb Xvfb \
  gnuplot ImageMagick-devel ImageMagick-c++-devel \
  xorg-x11-fonts-100dpi xorg-x11-fonts-ISO8859-1-100dpi \
  xorg-x11-fonts-75dpi xorg-x11-fonts-ISO8859-1-75dpi \
  xorg-x11-fonts-misc
```

## Install google-chrome

Create repo file, with google-chrome details:

```
  touch /etc/yum.repos.d/google-chrome.repo
```

Edit the file, to contain the following contents:

```
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```

Do the install:

```
  yum install -y google-chrome-stable
```

## Install NVIDIA driver if display has NVIDIA card

See:
```
  https://www.dedoimedo.com/computers/centos-7-nvidia-second.html
  http://elrepo.org/tiki/tiki-index.php
```

```
  rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
  yum install -y http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
  yum install -y nvidia-detect.x86_64
  yum install -y kmod-nvidia-340xx.x86_64 nvidia-x11-drv-340xx.x86_64
```

## Disable the firewall

```
  systemctl stop firewalld
  systemctl stop iptables
  systemctl disable ip6tables
```

## disable SELINUX

In

```
  /etc/sysconfig/selinux
```

set

```
  SELINUX=disabled
```

## set up chronyd

Edit:

```
  /etc/chrony.conf
```

and at the top set:

```
  server ntp iburst
  server 0.centos.pool.ntp.org iburst
  server 1.centos.pool.ntp.org iburst
  etc ...
```

## Start up xinetd

Edit:

```
  systemctl enable xinetd
  systemctl start xinetd
```


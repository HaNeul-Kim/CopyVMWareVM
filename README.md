# CopyVMWareVM

* 원본 VM 으로부터 복사 후 파일명들을 바꾸고 내용을 알맞게 바꿉니다.
* IP, HOSTNAME 을 자동으로 설정합니다.

# Build

```shell
git clone https://github.com/HaNeul-Kim/CopyVMWareVM  
cd CopyVMWareVM  
mvn clean install -DskipTests dependency:sources
java -jar target/CopyVMWare-0.5.jar --sourcePath /path/to/source/vm --sourceVMName centos_6.7_template --targetPath /path/to/target/vm --targetVMNames test1,test2 --encoding UTF-8
```

# Preparation

## VM Tools Dependency 설치

아래 방법들 중 한가지 이상 수행해야 함

### Kernel Source 설치

kernel version 에 맞는 source 가 설치되는지 확인 필요

```shell
uname -a
yum install -y kernel-devel
```

```text
[root@template79:~]# uname -a
Linux template79.sky.local 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
[root@template79:~]#
```

설치하고 나면 `/usr/src/kernels/` 에 설치됨

```text
[root@template79:~]# ll /usr/src/kernels/
total 4
drwxr-xr-x.  3 root root   41 May 16 15:37 ./
drwxr-xr-x.  4 root root   34 May 16 15:31 ../
drwxr-xr-x. 22 root root 4096 May 16 15:37 3.10.0-1160.25.1.el7.x86_64/
```

### policycoreutils-python 설치

```shell
yum install -y policycoreutils-python
```

## VM Tools 설치

```shell
yum install -y perl
mkdir /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
cd /tmp
rm -rf vmware-tools-distrib
tar zxpf /mnt/cdrom/VMwareTools-x.x.x-yyyy.tar.gz
umount /dev/cdrom
cd vmware-tools-distrib
./vmware-install.pl
# 물음에 답하면 됨
```

```text
[root@template76:/tmp/vmware-tools-distrib]# ./vmware-install.pl
The installer has detected an existing installation of open-vm-tools packages
on this system and will not attempt to remove and replace these user-space
applications. It is recommended to use the open-vm-tools packages provided by
the operating system. If you do not want to use the existing installation of
open-vm-tools packages and use VMware Tools, you must uninstall the
open-vm-tools packages and re-run this installer.
The packages that need to be removed are:
open-vm-tools
The installer will next check if there are any missing kernel drivers. Type yes
if you want to do this, otherwise type no [yes]

INPUT: [yes]  default

Creating a new VMware Tools installer database using the tar4 format.

Installing VMware Tools.

In which directory do you want to install the binary files?
[/usr/bin]

INPUT: [/usr/bin]  default

What is the directory that contains the init directories (rc0.d/ to rc6.d/)?
[/etc/rc.d]

INPUT: [/etc/rc.d]  default

What is the directory that contains the init scripts?
[/etc/rc.d/init.d]

INPUT: [/etc/rc.d/init.d]  default

In which directory do you want to install the daemon files?
[/usr/sbin]

INPUT: [/usr/sbin]  default

In which directory do you want to install the library files?
[/usr/lib/vmware-tools]

INPUT: [/usr/lib/vmware-tools]  default

The path "/usr/lib/vmware-tools" does not exist currently. This program is
going to create it, including needed parent directories. Is this what you want?
[yes]

INPUT: [yes]  default

In which directory do you want to install the documentation files?
[/usr/share/doc/vmware-tools]

INPUT: [/usr/share/doc/vmware-tools]  default

The path "/usr/share/doc/vmware-tools" does not exist currently. This program
is going to create it, including needed parent directories. Is this what you
want? [yes]

INPUT: [yes]  default

The installation of VMware Tools 10.3.22 build-15902021 for Linux completed
successfully. You can decide to remove this software from your system at any
time by invoking the following command: "/usr/bin/vmware-uninstall-tools.pl".

Before running VMware Tools for the first time, you need to configure it by
invoking the following command: "/usr/bin/vmware-config-tools.pl". Do you want
this program to invoke the command for you now? [yes]

INPUT: [yes]  default


You have chosen to install VMware Tools on top of an open-vm-tools package.
You will now be given the option to replace some commands provided by
open-vm-tools.  Please note that if you replace any commands at this time and
later remove VMware Tools, it may be necessary to re-install the open-vm-tools.

The file /usr/bin/vmware-hgfsclient that this program was about to install
already exists.  Overwrite? [no] yes

INPUT: [yes]

The file /usr/bin/vmhgfs-fuse that this program was about to install already
exists.  Overwrite? [no] yes

INPUT: [yes]

Initializing...


Making sure services for VMware Tools are stopped.

Stopping vmware-tools (via systemctl):  [  OK  ]


The module vmci has already been installed on this system by another installer
or package and will not be modified by this installer.

The module vsock has already been installed on this system by another installer
or package and will not be modified by this installer.

The module vmxnet3 has already been installed on this system by another
installer or package and will not be modified by this installer.

The module pvscsi has already been installed on this system by another
installer or package and will not be modified by this installer.

The module vmmemctl has already been installed on this system by another
installer or package and will not be modified by this installer.

The VMware Host-Guest Filesystem allows for shared folders between the host OS
and the guest OS in a Fusion or Workstation virtual environment.  Do you wish
to enable this feature? [yes]

INPUT: [yes]  default

The vmxnet driver is no longer supported on kernels 3.3 and greater. Please
upgrade to a newer virtual NIC. (e.g., vmxnet3 or e1000e)


Skipping configuring automatic kernel modules as no drivers were installed by
this installer.


Skipping rebuilding initrd boot image for kernel as no drivers to be included
in boot image were installed by this installer.

The configuration of VMware Tools 10.3.22 build-15902021 for Linux for this
running kernel completed successfully.

Enjoy,

--the VMware team

[root@template76:/tmp/vmware-tools-distrib]#
```

```shell
systemctl status vmware-tools
```

`/mnt/hgfs/` 에 공유한 directory 가 나옴

그래도 안나온다면

아래 두줄 중 한줄을 `/etc/fstab` 에 추가함

```text
vmhgfs-fuse /mnt/hgfs fuse             defaults,allow_other 0 0
```

```text
.host:/     /mnt/hgfs fuse.vmhgfs-fuse defaults,allow_other 0 0
```

## epel-release baseurl 수정

epel-release baseurl 을 kakao 로 수정

```shell
yum install -y epel-release

mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.template
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.template

cat > /etc/yum.repos.d/epel.repo << EOF
[epel]
name=Extra Packages for Enterprise Linux 7 - \$basearch
#baseurl=http://download.fedoraproject.org/pub/epel/7/\$basearch
baseurl=http://mirror.kakao.com/epel/7/\$basearch
#metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=\$basearch
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - \$basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/7/\$basearch/debug
baseurl=http://mirror.kakao.com/epel/7/\$basearch/debug
#metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-debug-7&arch=\$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-source]
name=Extra Packages for Enterprise Linux 7 - \$basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/7/SRPMS
baseurl=http://mirror.kakao.com/epel/7/SRPMS
#metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=\$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
EOF

cat > /etc/yum.repos.d/epel-testing.repo << EOF
[epel-testing]
name=Extra Packages for Enterprise Linux 7 - Testing - \$basearch
#baseurl=http://download.fedoraproject.org/pub/epel/testing/7/\$basearch
baseurl=http://mirror.kakao.com/epel/testing/7/\$basearch
#metalink=https://mirrors.fedoraproject.org/metalink?repo=testing-epel7&arch=\$basearch
failovermethod=priority
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-testing-debuginfo]
name=Extra Packages for Enterprise Linux 7 - Testing - \$basearch - Debug
#baseurl=http://download.fedoraproject.org/pub/epel/testing/7/\$basearch/debug
baseurl=http://mirror.kakao.com/epel/testing/7/\$basearch/debug
#metalink=https://mirrors.fedoraproject.org/metalink?repo=testing-debug-epel7&arch=\$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1

[epel-testing-source]
name=Extra Packages for Enterprise Linux 7 - Testing - \$basearch - Source
#baseurl=http://download.fedoraproject.org/pub/epel/testing/7/SRPMS
baseurl=http://mirror.kakao.com/epel/testing/7/SRPMS
#metalink=https://mirrors.fedoraproject.org/metalink?repo=testing-source-epel7&arch=\$basearch
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=1
EOF
```

## Template VM

host 의 특정 directory 를 vm 에 공유 설정함

1. VM 우클릭 > Settings > Options 탭
1. Shared Folders 선택 > Always enabled 선택
1. Add...
1. 공유할 folder 선택
1. 이후 작업은 본인 환경에 맞도록

## VM Workstation

vm 시작시 shared folder disabled is default 라면서 shared folder 가 disable 되지 않기 위해

1. Edit > Preferences
1. Enable all shared folders by default 체크

## Template VM Power On

```shell
echo "~/set_auto.sh" >> /etc/rc.d/rc.local
chmod 755 /etc/rc.d/rc.local
```

```shell
vim ~/set_auto.sh
```

```shell
#! /bin/bash
AUTO_CONF_DIR=/mnt/hgfs/conf
if [ $# -gt 0 ] ; then
  AUTO_CONF_DIR=$1
fi
if [ ! -d ${AUTO_CONF_DIR} ] ; then
  echo "dost not exists ${AUTO_CONF_DIR}"
  exit 1
fi

AUTO_IP_FILE=${AUTO_CONF_DIR}/IPADDR
AUTO_HOSTNAME_FILE=${AUTO_CONF_DIR}/HOSTNAME
if [ ! -f ${AUTO_IP_FILE} ] ; then
  echo "dost not exists ${AUTO_IP_FILE}"
  exit 1
fi
if [ ! -f ${AUTO_HOSTNAME_FILE} ] ; then
  echo "dost not exists ${AUTO_HOSTNAME_FILE}"
  exit 1
fi

NEW_IP=$(cat ${AUTO_IP_FILE})
NEW_HOSTNAME=$(cat ${AUTO_HOSTNAME_FILE})

echo "AUTO_CONF_DIR=${AUTO_CONF_DIR}"
echo "NEW_IP=${NEW_IP}"
echo "NEW_HOSTNAME=${NEW_HOSTNAME}"

sed -i "s/192.168.181.128/${NEW_IP}/g" /etc/sysconfig/network-scripts/ifcfg-ens33
hostnamectl set-hostname ${NEW_HOSTNAME}
systemctl restart network

systemctl disable rc-local
systemctl stop    rc-local
sed -i 's/~\/set_auto.sh//g' /etc/rc.d/rc.local
chmod 644 /etc/rc.d/rc.local

ping -c 1 google.com
cat /etc/resolv.conf
```

# Usage

Windows 의 경우 다음 명령어 실행

```shell
java -jar E:\vm\CopyVMWare-0.5.jar `
 --autoConfPath E:\vm\conf `
 --sourcePath   E:\vm\linux `
 --sourceVMName template76 `
 --targetPath   F:\vm\linux `
 --targetVMNames nfsserver,nfsclient `
 --targetVMIPs   192.168.181.211,192.168.181.212 `
 --targetDomain  sky.local
```

*nix 의 경우 다음 명령어 실행

```shell
java -jar /path/to/CopyVMWare-0.5.jar \
 --autoConfPath /path/to/conf \
 --sourcePath   /path/to/linux \
 --sourceVMName template76 \
 --targetPath   /path/to/linux \
 --targetVMNames nfsserver,nfsclient \
 --targetVMIPs   192.168.181.211,192.168.181.212 \
 --targetDomain  sky.local
```

VM On

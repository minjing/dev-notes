## Basic OS and hardware requirement

Vertica require below:
* Ext3 or Ext4
* Use Standard Partition (Do use LVM)
* The swap size must be more than 2GB
* 1GB of RAM per logic processor

## Disable CPU frequency scaling
Vertica requires CPU frequency scaling policy is performance, for most cases you need disable in BIOS settings, somehow if you can't find such setting in your BIOS, you can disable it in linux :

```shell
echo performance | tee /sys/devices/system/cpu/cpu?/cpufreq/scaling_governor >& /dev/null
```

Note that the **cpu?** shoud be replace by your cpu number like cpu0, cpu1...

## Make rc.local executable
```shell
chmod +x /etc/rc.d/rc.local
```

## Set Readahead
Vertica requires that Disk Readahead be set to at least 2048.
Check the setting first:

```shell
/sbin/blockdev --getra /dev/sda
```

Set Disk Readahead:

```shell
/sbin/blockdev --setra /dev/sda
```

Enable setting during system lauching:

```shell
echo '/sbin/blockdev --setra 2048 /dev/sda' >> /etc/rc.local
```

## Set I/O scheduler
```shell
echo deadline > /sys/block/sda/queue/scheduler
```

Enable setting during system lauching:

```shell
echo 'echo deadline > /sys/block/sda/queue/scheduler' >> /etc/rc.local
```

## Enabling Transparent Hugepages and disable Defrag
Note: for non Red Hat 7/CentOS 7, the Transparent Hugepates needs to disable or set them to madvise.
```shell
echo always &gt; /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

Enable setting during system launching:
Add below script to /etc/rc.local:
```shell
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
    echo always > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag`
fi
```

## Install NTP service
```shell
yum -y install ntp ntpdate
systemctl start ntpd
systemctl enable ntpd
```

## Disable SELinux
```shell
setenforce 0
```

Enable setting during system launching:
Edit /etc/selinux/config and set follong:
```shell
SELINUX=disabled
```

## Disable firewall
```shell
systemctl mask firewalld
systemctl disable firewalld
systemctl stop firewalld
```

## Language setting
Edit /etc/profile, modify/add below:
```shell
export LANG=en_US.utf-8
```

## Timezone setting
Update time zone
```shell
yum update tzdata
```

edit /etc/profile, add/modify below:
```shell
export TZ="Asia/Shanghai"
```

## Install support tools
```shell
yum install gdb mcelog sysstat dialog
```

## Install Vertica Server Package
Copy vertica server rpm package to server, then run:
```shell
rpm -Uvh [vertica server package rpm]
```

## Install Vertica
```shell
/opt/vertica/sbin/install_vertica --hosts [host list] --rpm [vertica server package rpm]
```

The host list is separated by space.
During the installation the installation script will create dbadmin account for you and prompt you to input password for dbadmin user.

## Launch Vertica Admin Tools
Login system by dbadmin, and input:
```shell
/opt/vertica/bin/admintools
```

The admin tools dialog will be shown.
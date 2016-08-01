# debian-docker
Build a Debian OS installation to host Docker

First case: Jessie

[https://github.com/eschen42/debian-docker/jessie](https://github.com/eschen42/debian-docker/jessie)

# See also:

## Very similar to this project
[https://github.com/petrkle/debian](https://github.com/petrkle/debian) - Build a Debian system using preseed files from https://github.com/petrkle/debian to support subsequent configuration by 'salt' configuration manager.  Shows how to include .ssh/authorized_keys in the built system using wget at build time.

[https://github.com/ajclark/preseed](https://github.com/ajclark/preseed) - Build a Debian 6 system within VirtualBox using preseed. UK/GB oriented, but very readable.

[https://github.com/mharj/preseed](https://github.com/mharj/preseed) - Build a Debian systemusing preseed files; this project assumes the system is built for Finland; but it's goal appears very similar to that of this project.

[https://github.com/jedipunkz/preseed/blob/master/preseed-debian-default.cfg] (https://github.com/jedipunkz/preseed/blob/master/preseed-debian-default.cfg) - This shows why the preseed file should be named with the '.cfg' extension, i.e., so that you can easily see which parts are not commented out when viewing it on GitHub.

### Copying files from installation media and running post-installation script
[https://help.ubuntu.com/community/InstallCDCustomization/PreseedExamples](https://help.ubuntu.com/community/InstallCDCustomization/PreseedExamples) - Demonstrates "copy post-installation script from source media to target and run one of them at first boot" by subverting the "exit 0" at the end of /etc/rc.local.  Here are the relevant parts:

#### preseed.cfg

You may want to study preseed.cfg in its entirety at the link!
```
# Copy the install script from the CD to the HDD,
#   and set it in /etc/rc.local so it runs on first boot.
d-i     preseed/late_command string \
          cp -a /cdrom/preseed/install.sh /target/root;  \
          sed -i 's_exit 0_sh /root/install.sh_' /target/etc/rc.local;
```

#### install.sh

```
INSTALL="apt-get install -y"
REMOVE="apt-get remove -y"

apt-get update

# email
$INSTALL exim4-daemon-heavy mailx mailgraph

# etc - loads cut

sed -i 's_sh /root/install.sh_exit 0_' /etc/rc.local

echo "Installation complete."
```

### Copying files from web and running late-within-installation script to install docker
[https://jrm.cc/blog/home-cluster-part-ii-machine-setup-pixies/](https://jrm.cc/blog/home-cluster-part-ii-machine-setup-pixies/) - Demonstrates 'd-i preseed/run string path/to/script' where the script includes installation of docker-engine. Here are the relevant parts:

#### preseed.cfg

```
# This is a critical step -- the preseed can only do steps from what the
# installer offers with this one exception, the "late_command" option can run
# any arbitrary command. However, you can only specify a single instance of
# the late_command option, but you can make it do multiple things. To keep this
# file sane, I have it fetch a bash script, chmod it, and execute it in the
# machine which is being built.
d-i preseed/late_command string \
 in-target wget ftp://192.168.1.1/sdc1/tftp/ubuntu/16.04/late.sh -O /tmp/late_command.sh ;\
 in-target chmod +x /tmp/late_command.sh ;\
 in-target /tmp/late_command.sh
```

#### late.sh

```
#!/bin/bash

# Update to proper sources -- as noted in preseed, the sources setup for
# install are local and static. This gets us back to normal sources AND adds docker
cat <<'EOF' > /etc/apt/sources.list
# Ubuntu Main Repos
deb     http://us.archive.ubuntu.com/ubuntu/ xenial main restricted universe
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial main restricted universe

# Ubuntu Update Repos
deb     http://us.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe
deb     http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe
deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe

# Docker
deb     https://apt.dockerproject.org/repo ubuntu-xenial main
EOF

# Add Docker Key
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

# Update apt, upgrade everything
apt-get update
apt-get upgrade -y

# Install docker
apt-get install -y docker-engine
``` 

## Distantly related to this project
[https://github.com/cdown/rebuild-debian-iso](https://github.com/cdown/rebuild-debian-iso) - Rebuild a Debian ISO with preseed/custom files

[http://serverfault.com/questions/528582/debian-ubuntu-set-preseed-mirror-variable-via-early-run-command](http://serverfault.com/questions/528582/debian-ubuntu-set-preseed-mirror-variable-via-early-run-command) - Demonstrates 'd-i preseed/run string path/to/script'

[https://gist.github.com/coderanger/664608] (https://gist.github.com/coderanger/664608) - Demonstrates 'd-i preseed/include string preseed.cfg', i.e., innsert one file of preseed cfg lines midstream within another (like C preprocessor #include) 


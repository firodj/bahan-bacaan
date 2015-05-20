Build Wine on Linux:
====================
mkdir build_dbg
cd build_dbg
../configure CFLAGS="-ggdb3 -Og"
make -j 2

Build i386 on lxc (BiArch)
--------------------------
[http://wiki.winehq.org/BuildingBiarchWineOnUbuntu]

The basic approach is:

1. Build 64-bit wine
2. Build 32-bit tools in lxc
3. Build 32-bit wine in lxc, referring to the 64-bit wine and 32-bit tools built in the previous steps
4. Install 32-bit wine
5. Install 64-bit wine

sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ubuntu-wine/ppa
sudo apt-get update
sudo apt-get build-dep wine

mkdir wine64
../configure --enable-win64
make -j 2
cd ..

sudo apt-get install lxc lxc-templates debootstrap
sudo lxc-create -t ubuntu -n my32bitbox -- --bindhome $LOGNAME -a i386
sudo cp -R /etc/apt /var/lib/lxc/my32bitbox/rootfs/etc

sudo lxc-start -n my32bitbox
sudo lxc-attach -n my32bitbox
login yourusername+password

lxc$ sudo apt-get update
lxc$ sudo apt-get install python-software-properties git-core
lxc$ sudo apt-get build-dep wine
lxc$ mkdir wine32-tools
lxc$ wine32-tools
lxc$ ../configure
lxc$ make -j2
lxc$ cd..

lxc$ mkdir wine32
lxc$ cd wine32
lxc$ ../configure --with-wine64=../wine64 --with-wine-tools=../wine32-tools
lxc$ make -j2
lxc$ cd..

lxc$ shutdown -h now

cd wine32
sudo make install
cd ../wine64
sudo make install

Chek Wine support Valgrind
--------------------------
$ ~/wine/build/configure CFLAGS="-g -O0"

$ ~/wine/build/grep VALGRIND include/config.h
#define HAVE_VALGRIND_MEMCHECK_H 1
#define HAVE_VALGRIND_VALGRIND_H 1


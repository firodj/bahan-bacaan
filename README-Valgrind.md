Build Linux
-----------
sudo apt-get install libc6
sudo apt-get install libc6-dbg

cd valgrind
./autogen.sh

mkdir build
cd build
make distclean
../configure
make -j 2
make install

Build Linux-x86_64
------------------
sudo apt-get install libc6:i386
sudo apt-get install libc6-dbg:i386
sudo apt-get install gcc-multilib g++-multilib
sudo apt-get install libc6-i386 libc6-dev-i386
sudo apt-get install libc6-x32 libc6-dev-x32

mkdir build-32
cd build-32
make distclean
../configure --enable-only32bit
make -j 2
make install

Running Wine under Valgrind
--------------------------
$ wineconsole &

$ export WINELOADERNOEXEC=1

$ valgrind --trace-children=yes \
  --vex-iropt-register-updates=allregs-at-mem-access \
  --workaround-gcc296-bugs=yes 
  wine foo.exe

Controlling Memory Leak logging
-------------------------------
--leak-check=full
  more info memory leaks

--show-possibly-lost=no
  ignore less-certain memory leaks

--track-origins=yes
  stack backtrace of the point where the memory block was allocated

--vgdb=yes --vgdb-error=0
  enable remote debugging, in another shell type:
  $ gdb <app>
  (gdb) target remote | vgdb

Build on Linux:
===============

For DrMemory:
------------
git tag
git checkout release_1.8.0
git submodule update --init
mkdir build
cd build
cmake ..

for DynamoRIO only:
----------------------------
git pull --rebase
git checkout master
drmemory\dynamorio\
mkdir build_dbg
cd build_dbg
cmake -DDEBUG=ON ..

Build on Windows:
=================
Open Command Prompt for Visual Studio 12 > VS2013 x64 Cross Tools Command Prompt

is same, but for SAFESEH problem, do
change in `Linker` options to `No`

# Ubuntu 15.04 Build and Install Instructions
The following dependencies were necessary for a clean install of Ubuntu 15.04:

    sudo apt-get install gcc-4.9 g++-4.9 cmake make libbz2-dev libdb++-dev libdb-dev libssl-dev openssl libreadline-dev autoconf libtool git

## Build Boost 1.57.0 

The Boost which ships with Ubuntu 15.04 is too old.  You need to download the Boost tarball for Boost 1.57.0
(Note, 1.58.0 requires C++14 and will not build on Ubuntu LTS; this requirement was an accident, see ).  

    sudo apt-get update
    sudo apt-get install autotools-dev build-essential g++ libbz2-dev libicu-dev python-dev
    wget -c 'http://sourceforge.net/projects/boost/files/boost/1.57.0/boost_1_57_0.tar.bz2/download'
    tar -xf download
    rm download
    cd boost_1_57_0/
    ./bootstrap.sh --prefix=/usr/local/
    sudo ./b2 install

## Build Graphene 

    cd ..
    git clone https://github.com/cryptonomex/graphene.git
    cd graphene
    git submodule update --init --recursive
    cmake .
    make 


## Ubuntu 15.04
Ubuntu 15.04 uses gcc 5, which has the c++11 ABI as default, but the boost libraries were compiled with the cxx11 ABI (this is an issue in many distros). If you get build failures due to abi incompatibilities, just use gcc 4.9

   CC=gcc-4.9 CXX=g++-4.9 cmake .

The following dependencies were necessary for a clean install of Ubuntu 15.04:

    sudo apt-get install cmake libdb++-dev libdb-dev libssl-dev openssl libreadline-dev autoconf libtool

The Boost which ships with Ubuntu 14.04 LTS is too old.  You need to download the Boost tarball for Boost 1.57.0
(Note, 1.58.0 requires C++14 and will not build on Ubuntu LTS; this requirement was an accident, see ).  Build Boost as follows:

    sudo apt-get update
    sudo apt-get install autotools-dev build-essential g++ libbz2-dev libicu-dev python-dev
    wget -c 'http://sourceforge.net/projects/boost/files/boost/1.57.0/boost_1_57_0.tar.bz2/download'
    tar -xf download
    cd boost_1_57_0/
    ./bootstrap.sh --prefix=/usr/local/
    sudo ./b2 install

This should install boost exactly to where the installation script is looking for it. If not, then
we need to tell `cmake` to use the Boost we just built, instead of using the system-wide Boost:

    cd ~/src/graphene
    [ -e ./docs/build-ubuntu.md ] && sh -c 'cmake -DBOOST_ROOT="$BOOST_ROOT" -DCMAKE_BUILD_TYPE=Debug .'

If all goes well, you should see the correct Boost version in the output messages to the above command.

## Ubuntu 15.04
Ubuntu 15.04 uses gcc 5, which has the c++11 ABI as default, but the boost libraries were compiled with the cxx11 ABI (this is an issue in many distros). If you get build failures due to abi incompatibilities, just use gcc 4.9

    CC=gcc-4.9 CXX=g++-4.9 cmake -DCMAKE_BUILD_TYPE=Debug .
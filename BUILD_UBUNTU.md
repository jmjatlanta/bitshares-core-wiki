# Ubuntu 16.04 LTS (64-bit) Build and Install Instructions
The following dependencies were necessary for a clean install on Ubuntu 16.04 LTS (64-bit):

    sudo apt-get update
    sudo apt-get install autoconf cmake make automake libtool git libboost-all-dev libssl-dev g++ libcurl4-openssl-dev

## Build BitShares Core

    git clone https://github.com/bitshares/bitshares-core.git
    cd bitshares-core
    git submodule update --init --recursive
    cmake -DCMAKE_BUILD_TYPE=Release .
    make 

## Build Support Boost Version
NOTE: BitShares requires a Boost version in the range [1.57 - 1.65.1]. Versions earlier than 1.57 or newer than 1.65.1 are NOT supported. If your system's Boost version is newer, then you will need to manually build an older version of Boost and specify it to CMake using DBOOST_ROOT.

The Boost which ships with Ubuntu 14.04 LTS (64-bit) is too old.  You need to download the Boost tarball for Boost 1.57.0
(Note, 1.58.0 requires C++14 and will not build on Ubuntu 14.04 LTS (64-bit); this requirement was an accident, see [this mailing list post](http://boost.2283326.n4.nabble.com/1-58-1-bugfix-release-necessary-td4674686.html)).

    BOOST_ROOT=$HOME/opt/boost_1_57_0
    sudo apt-get update
    sudo apt-get install autotools-dev build-essential libbz2-dev libicu-dev python-dev
    wget -c 'http://sourceforge.net/projects/boost/files/boost/1.57.0/boost_1_57_0.tar.bz2/download' -O boost_1_57_0.tar.bz2
    [ $( sha256sum boost_1_57_0.tar.bz2 | cut -d ' ' -f 1 ) == "910c8c022a33ccec7f088bd65d4f14b466588dda94ba2124e78b8c57db264967" ] || ( echo 'Corrupt download' ; exit 1 )
    tar xjf boost_1_57_0.tar.bz2
    cd boost_1_57_0/
    ./bootstrap.sh "--prefix=$BOOST_ROOT"
    ./b2 install

Build with specific Boost version:

    cmake -DBOOST_ROOT="$BOOST_ROOT" -DCMAKE_BUILD_TYPE=Release .
    make

## Error `{"message":"Timer Expired"}` in Ubuntu 16.04 LTS (64-bit) 
 
If error `{"message":"Timer Expired"}` dropped then it could be issue with websocketpp in linux kernel > 4.4.

Details [here](https://github.com/DECENTfoundation/DECENT-Network/issues/194).
 
Steps to fix:

    cd ~/bitshares-core/libraries/fc/vendor/websocketpp
    git remote set-url origin https://github.com/DECENTfoundation/websocketpp.git
    git fetch
    git checkout 

And then build BitShares Core.

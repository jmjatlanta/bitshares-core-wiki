The following instructions covers the setup of a witness in Ubuntu 14.04. The instructions are specific for the Test 1 release, but should be easily ported to other releases.

Note: Nodes might gone down as everyone is testing.

##Installation of dependencies
Extracted from instructions here: https://github.com/cryptonomex/graphene/wiki/build-ubuntu, but limited only for witness / cli build.

If you already have done your installation move to next step

    sudo apt-get update

Installation of development tools

    sudo apt-get install gcc-4.9 g++-4.9 cmake make libbz2-dev libdb++-dev libdb-dev libssl-dev openssl libreadline-dev autoconf libtool git

If you cannot install gcc-4.9, you will need to add this repository before hand and try again.

    sudo add-apt-repository ppa:ubuntu-toolchain-r/test

###Installation of BOOST

    BOOST_ROOT=$HOME/opt/boost_1_57_0
    sudo apt-get update
    sudo apt-get install autotools-dev build-essential g++ libbz2-dev libicu-dev python-dev
    wget -c 'http://sourceforge.net/projects/boost/files/boost/1.57.0/boost_1_57_0.tar.bz2/download' -O boost_1_57_0.tar.bz2
    [ $( sha256sum boost_1_57_0.tar.bz2 | cut -d ' ' -f 1 ) == "910c8c022a33ccec7f088bd65d4f14b466588dda94ba2124e78b8c57db264967" ] || ( echo 'Corrupt download' ; exit 1 )
    tar xjf boost_1_57_0.tar.bz2
    cd boost_1_57_0/
    ./bootstrap.sh "--prefix=$BOOST_ROOT"
    ./b2 install [/code]

##Git  checkout and build
Ensure your boost path is correct

    BOOST_ROOT=$HOME/opt/boost_1_57_0

Check out and build

    cd ~
    git clone https://github.com/cryptonomex/graphene.git
    cd graphene

**Check out test 1 (specific to this chain)**

    git checkout test1

Update submodules and build make file
 
    git submodule update --init --recursive
    cmake -DBOOST_ROOT="$BOOST_ROOT" -DCMAKE_BUILD_TYPE=Debug .
    make 


##Setup witness / import balances

_Navigate to the witness directory_

    cd ~/graphene/programs/witness_node

_Download genesis_ 
**Path specific for Test 1**

     wget https://github.com/cryptonomex/graphene/releases/download/test1/aug-20-test-genesis.json

_Start a new terminal screen_

     screen

_Run the witness_
**Current nodes for test 1 (replace for other tests)**

    ./witness_node --rpc-endpoint "127.0.0.1:8090"  --genesis-json aug-20-test-genesis.json -d test_net_1 -s  "104.236.51.238:1776" -s "176.221.43.130:33323" -s "45.55.6.216:1776" -s "114.92.254.159:62015"

Note: 
-d parameter is for the directory you want the witness data to be stored
-s is the node you want to connect

If you have problems, you might need to put the whole path for the genesis

    ./witness_node --rpc-endpoint "127.0.0.1:8090"  --genesis-json ~/graphene/programs/witness-node/aug-20-test-genesis.json -d test_net_1 -s "104.236.51.238:1776" -s "176.221.43.130:33323" -s "45.55.6.216:1776" -s "114.92.254.159:62015"

_Detach from screen_

     Ctrl A Ctrl D

_Extract your wif keys for user and balances as per xeroc's instructions_

     [Howto-become-an-active-witness-in-BitShares-2.0](https://github.com/cryptonomex/graphene/wiki/Howto-become-an-active-witness-in-BitShares-2.0)

_Navigate to cli_wallet_

    cd ~/graphene/programs/cli_wallet

_Run cli_  
**Current chain id for test 1**

     ./cli_wallet -w test_wallet_puppies  --chain-id d011922587473757011118587f93afcc314fbaea094fc1055574721b27975083

Note:
-w is your directory wallet

_Setup witness as per xerocs instructions_ 

      [Howto-become-an-active-witness-in-BitShares-2.0](https://github.com/cryptonomex/graphene/wiki/Howto-become-an-active-witness-in-BitShares-2.0)


Remember to copy your keys, witness id
Note you need to wait for a maintenance period to be voted in

_Exit_

_Go back to your witness screen_

  screen -r 

_Exit your witness_

    ctrl c

_Restart with parameters to start block producing (block producing needs your witness id and private keys)_ **Current nodes for test 1 (replace for other tests)**

    ./witness_node --rpc-endpoint "127.0.0.1:8090"  --genesis-json aug-20-test-genesis.json -d test_net_1 -s "104.236.51.238:1776" -s "176.221.43.130:33323" -s "45.55.6.216:1776" -s "114.92.254.159:62015"  --witness-id '"1.6.5156"' --private-key '["GPH6JhL..your.signing.key..bc5mWyCvERV3coy","5K..your.secret..a"]'

_See your witness producing blocks and 
you can Ctrl A Ctrl D to detach from screen._





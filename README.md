# Bitcoin-Simulator, capable of simulating any re-parametrization of Bitcoin
Bitcoin Simulator is built on ns3, the popular discrete-event simulator. We also made use of rapidjson to facilitate the communication process among the nodes. The purpose of this project is to study how consensus parameteres, network characteristics and protocol modifications affect the scalability, security and efficiency of Proof of Work powered blockchains.

Our goal is to make the simulator as realistic as possible. So, we collected real network statistics and incorporated them in the simulator. Specifically, we crawled popular explorers, like blockchain.info to estimate the block generation and block size distribution, and used the bitcoin crawler to find out the average number of nodes in the network and their geographic distribution. Futhermore, we used the data provided by coinscope regarding the connectivity of nodes.

Clone the Bitcoin Simulator.
The Bitcoin Simulator is built on ns3 and it has been tested with versions 3.24 and 3.25. So, go to the official site and grab a suitable release. Detailed installation instructions for various platforms can be found here.
Untar it with the following command:
tar xvfj ns-allinone-3.25.tar.bz2
For exhanging block messages, rapidjson is used. Download it from here.
Create a new directory named rapidjson under ns-allinone-3.xx/ns-3.xx and copy the contents of the directory include/rapidjson (from the downloaded rapidjson project in step 4) to the newly created rapidjson folder (ns-allinone-3.xx/ns-3.xx/rapidjson). You also have to copy all the files from Bitcoin-Simulation to the respective folders under ns-allinone-3.xx/ns-3.xx/. All the above can be done automatically by the provided copy-to-ns3.sh script.
Update the ns-allinone-3.xx/ns-3.xx/src/applications/wscript.
Add the following lines in module.source:
```
'model/bitcoin.cc',
'model/bitcoin-node.cc',
'model/bitcoin-miner.cc',
'model/bitcoin-simple-attacker.cc',
'model/bitcoin-selfish-miner.cc',
'model/bitcoin-selfish-miner-trials.cc',
'helper/bitcoin-topology-helper.cc',
'helper/bitcoin-node-helper.cc',
'helper/bitcoin-miner-helper.cc',			
```

Add the following lines in headers.source:
```
'model/bitcoin.h',
'model/bitcoin-node.h',
'model/bitcoin-miner.h',
'model/bitcoin-simple-attacker.h',
'model/bitcoin-selfish-miner.h',
'model/bitcoin-selfish-miner-trials.h',
'helper/bitcoin-topology-helper.h',
'helper/bitcoin-node-helper.h',
'helper/bitcoin-miner-helper.h',					
```

Update the ns-allinone-3.xx/ns-3.xx/src/internet/wscript.
```
Add the following line in obj.source:
'helper/ipv4-address-helper-custom.cc',
Add the following line in headers.source:
'helper/ipv4-address-helper-custom.h',
```

Configure ns3 with the follow command to ensure compatibility and maximum performance.
```

CXXFLAGS="-std=c++11" ./waf configure --build-profile=optimized --out=build/optimized --with-pybindgen=/home/bill/Desktop/workspace/ns-allinone-3.24/pybindgen-0.17.0.post41+ngd10fa60 --enable-mpi --enable-static
```

Build ns3 using the ./waf command.



BITCOIN SIMULATOR
The Bitcoin Simulator can be easily configured via command line parameters. Below is presented a table of all the input parameters along with their default values.

INPUT PARAMETERS
Parameter	Description	Default Value
blockSize	The fixed block size (Bytes). If the default value is used then the blockSize follows the real bitcoin block size distribution as estimated by collecting stats from blockchain.info	-1
noBlocks	The number of generated blocks	100
nodes	The total number of nodes in the network. The number of nodes should always be greater of equal to the number of miners. The number of miners, their hash rates and their locations can be changed by modifying the variables bitcoinMinersHash/litecoinMinersHash/dogecoinMinersHash and bitcoinMinersRegions/litecoinMinersRegions/dogecoinMinersRegions.	16
minConnections	The minConnectionsPerNode of the grid.	-1
maxConnections	The maxConnectionsPerNode of the grid. If minConnections <= 0 or maxConnections <=0, the nodes follow the connections distribution as described in here.	-1
blockIntervalMinutes	The average block generation interval in minutes.	10
invTimeoutMins	The inv block timeout. If the default value is used, the timeouts are twice as long as blockIntervalMinutes.	-1
unsolicited	Change the miners block broadcast type to UNSOLICITED. Each newly mined block is broadcast immediately to all the peers of the miner who mined it.	false
relayNetwork	Change the miners block broadcast type to RELAY_NETWORK. The miners use a relay network for communicating among themselves and send compressed blocks.	false
unsolicitedRelayNetwork	Change the miners block broadcast type to UNSOLICITED_RELAY_NETWORK. The miners use a relay network among themselves and send compressed blocks and each newly mined block is broadcast immediately to all the peers of the miner who mined it.	false
sendheaders	Change the protocol to sendheaders.	false
litecoin	Imitate the litecoin network behaviour. It sets the --nodes=1000, --blockIntervalMinutes=2.5, follows the same connection distribution as bitcoin, but the litecoin distribution for the block generation interval and block size.	false
dogecoin	Imitate the dogecoin network behaviour. It sets the --nodes=650, --blockIntervalMinutes=1, follows the same connection distribution as bitcoin, but the dogecoin distribution for the block generation interval and block size.	false
blockTorrent	Enable the BlockTorrent protocol.	false
chunkSize	The chunksize of the blockTorrent in Bytes. Used only in conjuction with --blockTorrent.	-1
spv	Enable the spv mechanism in blockTorrent.Used only in conjuction with --blockTorrent. The nodes are able to advertise chunks of blocks which are not yet validated.	false
OPTIMAL ADVERSARIAL ATTACKER
The optimal adversarial attacker can also be configured via command line parameters. Below is presented a table of all the input parameters along with their default values.

INPUT PARAMETERS
Parameter	Description	Default Value
blockIntervalMinutes	The average block generation interval in minutes.	10
noBlocks	The number of generated blocks	100
ud	The transaction value which is double-spent. If ud = 0, the attacker follows the optimal strategy for selfish-mining. Otherwise, the attacker follows the optimal strategy for performing a double-spending attack of a transaction with value equals to ud.	0
r	The stale block rate	0
unsolicited	Change the miners block broadcast type to UNSOLICITED. Each newly mined block is broadcast immediately to all the peers of the miner who mined it.	false
relayNetwork	Change the miners block broadcast type to RELAY_NETWORK. The miners use a relay network for communicating among themselves and send compressed blocks.	false
unsolicitedRelayNetwork	Change the miners block broadcast type to UNSOLICITED_RELAY_NETWORK. The miners use a relay network among themselves and send compressed blocks and each newly mined block is broadcast immediately to all the peers of the miner who mined it.	falseNotes
The number of miners, their hash rates and their locations can be changed by modifying the arrays minersHash and minersRegions. The attacker's hash rate is always the last value in minersHash.
To modify the optimal strategy, the m_decisionMatrix in bitcoin-selfish-miner.h must be updated accordingly.
TUTORIAL
Here we demonstrate some basic examples of how you can use the simulator:
If you want to run a simple Bitcoin simulation for 100 blocks and 6000 nodes, you can just enter the following command:
./waf --run "bitcoin-test --noBlocks=100 --nodes=6000"
If you want to start the simulation for 1000 blocks in LITECOIN mode and with a block size of 2MB you have to enter the following command:
./waf --run "bitcoin-test --noBlocks=1000 --litecoin --blockSize=2000000"
You can also use mpi to speed up the simulation:
mpirun -n 2 ./waf --run "bitcoin-test --noBlocks=1000 --litecoin --blockSize=2000000"

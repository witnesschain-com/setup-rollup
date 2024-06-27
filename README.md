Welcome to this guide on setting up rollup nodes (optimism and base) for 
watchtower clients.

A watchtower client needs a rollup node to locally and independently 
verify transactions on an optimistic blockchain network. In this guide, 
we will be using the docker compose utility to setup watchtowers.

## Step 1: Clone the setup repository

First, lets clone the 
[setup-rollup](https://github.com/kaleidoscope-blockchain/setup-rollup) 
repository that contains docker compose YAML file.

**Note**: The disk must have at least 2 TB of free disk space. 

```
git clone https://github.com/kaleidoscope-blockchain/setup-rollup.git
```

## Step 2: Configure the network parameters

There are two main files (`geth.yml` and `node.yml`) that need to be 
configure with the rollup network parameters.


In the `geth.yml`, make the following changes:

1. Change the `--network=<value>` with one of the following values: 
   `op-mainnet` for optimism, and `base-mainnet` for base.  A full list 
   of network options is available at [Optimism 
   Docs](https://docs.optimism.io/builders/node-operators/management/configuration#network).

In the `node.yml`, file configure the following three parameters:

1. Change the `--network=<value>` to the rollup that you configured in 
   the previous step, i.e. `op-mainnet` or `base-mainnet`.

2. Change the `--l1=<value>` to your L1 (Ethereum Mainnet) node address. 
   Watchtowers listens for L2 block proposals on L1 chain.

3. Change the `--l1.beacon=<value>` to your Ethereum Mainnet beacon 
   node. Watchtowers fetch blobs of [EIP-4844: Shard Blob 
   Transactions](https://eips.ethereum.org/EIPS/eip-4844). This 
   transaction type is by rollup's sequencer to reduce transaction costs 
   when posting transactions to mainnet chain.

## Step 3: Generate the secret authentication secret

Finally, generate a secret authentication token which is used by geth 
and node to authenticate each other.
```
openssl rand -hex 32 | tr -d "\n" | tee jwt.txt
```

## Step 4: Start the geth and node containers

Finally, start the geth and node containers with the following command:-
```
docker compose up
```

The optimistic rollup nodes sync using [snap 
sync](https://docs.optimism.io/builders/node-operators/management/snap-sync), 
which takes couple of hours depending on your network bandwidth and 
peers available for the rollup that you are syncing. 

The sync consists of three stages. First, the beacon headers are 
downloaded, then chain blocks, and finally chain state is downloaded. 
During the sync process you will see the following logs.

Initially, you will get the following logs:
```
setup-rollup-geth-1  | INFO [04-24|11:45:14.098] Looking for peers                        peercount=0 tried=111 static=0
```

Once, you are connected to some peers you will get following logs:
```
setup-rollup-geth-1  | INFO [04-24|11:46:44.530] Looking for peers                        peercount=1 tried=106 static=0
```

After, you are connected to at least one peer, the sync process starts, 
and you will see the following log messages syncing beacon headers:
```
setup-rollup-geth-1  | INFO [04-24|11:51:45.841] Forkchoice requested sync to new head    number=119,180,364 hash=b65f5a..50e4de
setup-rollup-geth-1  | INFO [04-24|11:51:47.681] Forkchoice requested sync to new head    number=119,180,365 hash=be3cf3..f38a4d
setup-rollup-geth-1  | INFO [04-24|11:51:49.728] Forkchoice requested sync to new head    number=119,180,366 hash=bd6747..87e8ef
setup-rollup-geth-1  | INFO [04-24|11:51:51.734] Forkchoice requested sync to new head    number=119,180,367 hash=6c73be..c084f2
setup-rollup-geth-1  | INFO [04-24|11:49:04.521] Syncing beacon headers                   downloaded=37376 left=118,010,123 eta=1h8m3.072s
```

Log messages during chain download:
```
INFO [04-19|13:35:15.948] Syncing: chain download in progress      synced=3.51% chain=219.92MiB headers=501,760@120.75MiB bodies=469,010@90.74MiB receipts=469,010@8.44MiB eta=1h3m19.777s
```

Log messages during state download:-
```
INFO [04-19|13:35:22.871] Syncing: state download in progress      synced=5.70% state=2.51GiB   accounts=3,857,895@950.66MiB slots=7,098,356@1.46GiB    codes=21063@128.88MiB eta=39m22.074s
```

When the rollup node is fully synced, you will see following messages:
```
setup-rollup-geth-1 | INFO [05-24|11:49:04.521] Imported new potential chain segment     number=13,585,173 hash=a796a3..9d10a9 blocks=1          txs=65          mgas=7.603   elapsed=119.637ms    mgasps=63.552  snapdiffs=3.95MiB    triedirty=0.00B
INFO [04-24|11:54:55.297] Chain head was updated                   number=13,585,173 hash=a796a3..9d10a9 root=899b9c..84fda0 elapsed=2.472614ms
```

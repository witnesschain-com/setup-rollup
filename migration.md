### Migration of rollup node from archive to full node

Rollup node operators can reduce storage cost by changing the type of 
rollup node. An `archive` node stores the full state history for each 
block since genesis block.  In contrast, a full node stores the state 
only for the recent 128 blocks.  Therefore, moving from `archive` node 
to `full` node will significantly reduces the storage needs for running 
rollup nodes.

To migrate the rollup node from `archive` node to `full` node, take the 
following steps:

1. Stop the current rollup node. Change current working directory to the 
   root of `setup-rollup` repository.

```
cd setup-rollup
docker compose down
```

2. In `geth.yml`, change `--gcmode` from `archive` to `full` node.
```
--gcmode=full
```

3. Resume the rollup node
```
docker compose up -d
```

Your rollup node should be up now. It will garbage collect the states 
older than 128 blocks.

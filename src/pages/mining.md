---
title: Mine Stacks tokens
description: Set up and run a miner on the Stacks 2.0 testnet
icon: TestnetIcon
experience: beginners
duration: 10 minutes
tags:
  - tutorial
images:
  large: /images/pages/mining.svg
  sm: /images/pages/mining-sm.svg
---

## Introduction

Make sure you've followed the [Running testnet node](/stacks-blockchain/running-testnet-node) tutorial. Once completed it's only a few more steps to run a proof-of-burn miner on the testnet.

[@page-reference | inline]
| /stacks-blockchain/running-testnet-node

## Running a miner

First, we need to generate a keychain. With this keychain, we'll get some testnet BTC from a faucet, and then use that BTC to start mining.

To get a keychain, the simplest way is to use the `blockstack-cli`. We'll use the `make_keychain` command, and pass `-t` to indicate that we want a testnet keychain.

```bash
npx blockstack-cli@1.1.0-beta.1 make_keychain -t
```

After this runs, you'll probably see some installation logs, and at the end you should see some JSON that looks like this:

```json
{
  "mnemonic": "exhaust spin topic distance hole december impulse gate century absent breeze ostrich armed clerk oak peace want scrap auction sniff cradle siren blur blur",
  "keyInfo": {
    "privateKey": "2033269b55026ff2eddaf06d2e56938f7fd8e9d697af8fe0f857bb5962894d5801",
    "address": "STTX57EGWW058FZ6WG3WS2YRBQ8HDFGBKEFBNXTF",
    "btcAddress": "mkRYR7KkPB1wjxNjVz3HByqAvVz8c4B6ND",
    "index": 0
  }
}
```

We need to get some testnet BTC to that address. Grab the `btcAddress` field, and call the BTC faucet:

```bash
# replace <btc_address> with `btcAddress` property from your keychain
curl -XPOST "https://stacks-node-api.blockstack.org/extended/v1/faucets/btc?address=<btc_address>" | json_pp
```

You'll be sent 0.5 testnet BTC to that address. **Don't lose this information** - we'll need to use the `privateKey` field later on.

Now, we need to configure out node to use this Bitcoin keychain. In the `stacks-blockchain` folder, create a new file called `testnet/stacks-node/conf/testnet-miner-conf.toml`.

Paste in the following configuration:

```toml
[node]
rpc_bind = "0.0.0.0:20443"
p2p_bind = "0.0.0.0:20444"
bootstrap_node = "048dd4f26101715853533dee005f0915375854fd5be73405f679c1917a5d4d16aaaf3c4c0d7a9c132a36b8c5fe1287f07dad8c910174d789eb24bdfb5ae26f5f27@testnet-miner.blockstack.org:20444"
# Enter your private key here!
seed = "replace-with-your-private-key"
miner = true

[burnchain]
chain = "bitcoin"
mode = "krypton"
peer_host = "bitcoind.blockstack.org"
rpc_port = 18443
peer_port = 18444

[[mstx_balance]]
address = "STB44HYPYAT2BB2QE513NSP81HTMYWBJP02HPGK6"
amount = 10000000000000000
[[mstx_balance]]
address = "ST11NJTTKGVT6D1HY4NJRVQWMQM7TVAR091EJ8P2Y"
amount = 10000000000000000
[[mstx_balance]]
address = "ST1HB1T8WRNBYB0Y3T7WXZS38NKKPTBR3EG9EPJKR"
amount = 10000000000000000
[[mstx_balance]]
address = "STRYYQQ9M8KAF4NS7WNZQYY59X93XEKR31JP64CP"
amount = 10000000000000000
```

Now, grab your `privateKey` from earlier, when you ran the `make_keychain` command. Replace the `seed` field with your private key. Save and close this configuration file.

To run your miner, run this in the command line:

```bash
stacks-node start --config=./testnet/stacks-node/conf/testnet-miner-conf.toml
```

Your node should start. It will take some time to sync, and then your miner will be running!

### Creating an optimized binary

The steps above are great for trying to run a node temporarily. If you want to host a node on a server somewhere, you might want to generate an optimized binary. To do so, use the same configuration as above, but run:

```bash
cd testnet/stacks-node
cargo build --release --bin stacks-node
```

The above code will compile an optimized binary. To use it, run:

```bash
cd ../..
./target/release/stacks-node start --config=./testnet/conf/krypton-follower-conf.toml
```

### Enable debug logging

In case you are running into issues or would like to see verbose logging, you can run your node with debug logging enabled. In the command line, run:

```bash
BLOCKSTACK_DEBUG=1 stacks-node krypton
```

## Optional: Running with Docker

Alternatively, you can run the testnet node with Docker.

-> Ensure you have [Docker](https://docs.docker.com/get-docker/) installed on your machine.

### Generate keychain and get testnet tokens

Generate a keychain:

```bash
docker run -i node:alpine npx blockstack-cli@1.1.0-beta.1 make_keychain -t
```

Request BTC from the faucet:

```bash
# replace <btc_address> with `btcAddress` property from your keychain
curl -XPOST "https://stacks-node-api.blockstack.org/extended/v1/faucets/btc?address=<btc_address>" | json_pp
```

### Create a config file directory

You need a dedicated directory to keep the config file(s):

```bash
mkdir -p $HOME/stacks
```

### Create configuration file

Inside the new `$HOME/stacks` folder, you should create a new miner config `Config.toml`:

```toml
[node]
working_dir = "/root/stacks-node/current"
rpc_bind = "0.0.0.0:20443"
p2p_bind = "0.0.0.0:20444"
# Enter your private key here!
seed = "replace-with-your-privateKey-from-generate-keychain-step"
miner = true

[burnchain]
chain = "bitcoin"
mode = "krypton"
peer_host = "bitcoind.krypton.blockstack.org"
#process_exit_at_block_height = 5340
#burnchain_op_tx_fee = 5500
#commit_anchor_block_within = 10000
rpc_port = 18443
peer_port = 18444

[[mstx_balance]]
address = "STB44HYPYAT2BB2QE513NSP81HTMYWBJP02HPGK6"
amount = 10000000000000000
[[mstx_balance]]
address = "ST11NJTTKGVT6D1HY4NJRVQWMQM7TVAR091EJ8P2Y"
amount = 10000000000000000
[[mstx_balance]]
address = "ST1HB1T8WRNBYB0Y3T7WXZS38NKKPTBR3EG9EPJKR"
amount = 10000000000000000
[[mstx_balance]]
address = "STRYYQQ9M8KAF4NS7WNZQYY59X93XEKR31JP64CP"
amount = 10000000000000000
```

-> Notice that this configuration differs from the one used to run the miner locally

### Run the miner

-> The ENV VARS `RUST_BACKTRACE` and `BLOCKSTACK_DEBUG` are optional. If removed, debug logs will be disabled

```bash
docker run -d \
  --name stacks_miner \
  --rm \
  -e RUST_BACKTRACE="full" \
  -e BLOCKSTACK_DEBUG="1" \
  -v "$HOME/stacks/Config.toml:/src/stacks-node/Config.toml" \
  -p 20443:20443 \
  -p 20444:20444 \
  blockstack/stacks-blockchain:latest \
/bin/stacks-node start --config /src/stacks-node/Config.toml
```

You can review the node logs with this command:

```bash
docker logs -f stacks_miner
```

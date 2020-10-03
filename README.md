## Lightning Channel

The objective of this practical work is to open a lightning channel with the professor.

Here are the node info.

![alt nodeInfo](./assets/professor-node.png 'node info')

We need the following data

    Node public key: 03838257f4b8b001f7ea43ba7bb383e908e5ee32745f0a3d04d690dd28c8b39b57
    Node onion address: hjyuqfmpg3ns7f4ohqqpa2akl3yi3dlv3kk66xiwpm6uytr5ad62r3yd.onion
    Port: 9735

### Setup

The first step is to set up a bitcoin node and then a lightning node on top of it. The setup process is described **[here](https://github.com/krouspy/monnaies-numeriques/tree/master/td1#bitcoin-core)** for the bitcoin node and **[here](https://github.com/krouspy/monnaies-numeriques/tree/master/td1#lightning-node)** for the lightning node.

#### [Tor](https://github.com/krouspy/monnaies-numeriques/tree/master/td1#tor)

We can see the professor node is routed via `tor` (.onion) so we should also route our node via `tor`. The installation process is done and described [here](https://github.com/krouspy/monnaies-numeriques/tree/master/td1#tor).

The step that has not be done is telling `lnd` to route via `tor`. To do so, we can specify it with `flags` when launching `lnd` like `--tor.active=true` or update the `lnd.conf`

```bash
# ~/.lnd/lnd.conf

[Application Options]
alias=Zeus
debuglevel=info
maxpendingchannels=5
listen=0.0.0.0:9735
restlisten=0.0.0.0:8080
rpclisten=0.0.0.0:10009
tlsextraip=0.0.0.0

[Bitcoin]
bitcoin.active=1
bitcoin.testnet=1
bitcoin.node=bitcoind

[Bitcoind]
bitcoind.rpcuser=bitcoin
bitcoind.rpcpass=420
bitcoind.zmqpubrawblock=tcp://127.0.0.1:28332
bitcoind.zmqpubrawtx=tcp://127.0.0.1:28333

[autopilot]
autopilot.active=1
autopilot.maxchannels=5
autopilot.allocation=0.6

[tor]
tor.active=true
tor.v3=true
tor.streamisolation=true
```

Also we should update the `bitcoin.conf` by adding `proxy` and `listenonion`.

```bash
# ~/.bitcoin/bitcoin.conf

# Daemon

daemon=1
# Run on testnet
testnet=1
# Listen to JSON-RPC commands
server=1
# Maintain full tx indexing
txindex=1

# Network
listen=1
proxy=127.0.0.1:9050
listenonion=1

# Connection settings
rpcuser=<user>
rpcpassword=<password>
rpcallowip=127.0.0.1
zmqpubrawblock=tcp://127.0.0.1:28332
zmqpubrawtx=tcp://127.0.0.1:28333
whitelist=127.0.0.1
```

Then restart services.

```bash
$ sudo systemctl restart bitcoin.service
$ sudo supervisorctl restart lnd
```

#### LND

Regarding the previous work done, we now need to unlock our lnd wallet.

```bash
$ lncli unlock
> lnd successfully unlocked!
$ lncli --network=testnet walletbalance
> {
    "total_balance": "38498843",
    "confirmed_balance": "38498843",
    "unconfirmed_balance": "0"
  }
```

Check if our node is routed via tor.

```bash
$ lncli --network=testnet getinfo | grep identity_pubkey
> 02d0e67ad94d503cd8bb5bbdaf7f3abeb16519b3108e02cb80be43ff4a59d284d8

$ lncli --network=testnet getnodeinfo 02d0e67ad94d503cd8bb5bbdaf7f3abeb16519b3108e02cb80be43ff4a59d284d8

> {
    "node": {
        "last_update": 1601652311,
        "pub_key": "02d0e67ad94d503cd8bb5bbdaf7f3abeb16519b3108e02cb80be43ff4a59d284d8",
        "alias": "Zeus",
        "addresses": [
            {
                "network": "tcp",
                "addr": "evex3d4aqodntelfgs374l3c6ajphwtxb3h4xswc2slohf6vadfzvead.onion:9735"
            }
        ],
        "color": "#3399ff",
        "features": {
            ...
        }
    },
    "num_channels": 5,
    "total_capacity": "46797038",
    "channels": [
    ]
}
```

We can see the `addr` field has a `.onion`

> evex3d4aqodntelfgs374l3c6ajphwtxb3h4xswc2slohf6vadfzvead.onion: 9735

So we can now open a channel with the professor's node.

```bash
$ lncli --network=testnet connect 03838257f4b8b001f7ea43ba7bb383e908e5ee32745f0a3d04d690dd28c8b39b57@hjyuqfmpg3ns7f4ohqqpa2akl3yi3dlv3kk66xiwpm6uytr5ad62r3yd.onion:9735

$ lncli --network=testnet openchannel --node_key 03838257f4b8b001f7ea43ba7bb383e908e5ee32745f0a3d04d690dd28c8b39b57 --connect hjyuqfmpg3ns7f4ohqqpa2akl3yi3dlv3kk66xiwpm6uytr5ad62r3yd.onion:9735 --local_amt 200000
> {
	"funding_txid": "e474fb7f5e360bc3cfdd0a9f458f66163d2648ad61122b03bfa9968f42ac1c1c"
  }
```

The transaction is viewable [here](https://live.blockcypher.com/btc-testnet/tx/e474fb7f5e360bc3cfdd0a9f458f66163d2648ad61122b03bfa9968f42ac1c1c/).

![alt proof](./assets/channel-proof.png 'proof')

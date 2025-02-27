[![Build Status](https://github.com/hukkin/cosmospy/workflows/Tests/badge.svg?branch=master)](https://github.com/hukkin/cosmospy/actions?query=workflow%3ATests+branch%3Amaster+event%3Apush)
[![codecov.io](https://codecov.io/gh/hukkin/cosmospy/branch/master/graph/badge.svg)](https://codecov.io/gh/hukkin/cosmospy)
[![PyPI version](https://img.shields.io/pypi/v/cosmospy)](https://pypi.org/project/cosmospy)

# cosmospy

<!--- Don't edit the version line below manually. Let bump2version do it for you. -->

> Version 6.0.0

> Tools for Cosmos wallet management and offline transaction signing

**Table of Contents**  *generated with [mdformat-toc](https://github.com/hukkin/mdformat-toc)*

<!-- mdformat-toc start --slug=github --maxlevel=6 --minlevel=2 -->

- [Installing](#installing)
- [Usage](#usage)
  - [Generating a wallet](#generating-a-wallet)
  - [Converter functions](#converter-functions)
    - [Mnemonic seed to private key](#mnemonic-seed-to-private-key)
    - [Private key to public key](#private-key-to-public-key)
    - [Public key to address](#public-key-to-address)
    - [Private key to address](#private-key-to-address)
  - [Signing transactions](#signing-transactions)

<!-- mdformat-toc end -->

## Installing<a name="installing"></a>

Installing from PyPI repository (https://pypi.org/project/cosmospy):

```bash
pip install cosmospy
```

## Usage<a name="usage"></a>

### Generating a wallet<a name="generating-a-wallet"></a>

```python
from cosmospy import generate_wallet

wallet = generate_wallet()
```

The value assigned to `wallet` will be a dictionary just like:

```python
{
    "seed": "arch skill acquire abuse frown reject front second album pizza hill slogan guess random wonder benefit industry custom green ill moral daring glow elevator",
    "derivation_path": "m/44'/118'/0'/0/0",
    "private_key": b"\xbb\xec^\xf6\xdcg\xe6\xb5\x89\xed\x8cG\x05\x03\xdf0:\xc9\x8b \x85\x8a\x14\x12\xd7\xa6a\x01\xcd\xf8\x88\x93",
    "public_key": b"\x03h\x1d\xae\xa7\x9eO\x8e\xc5\xff\xa3sAw\xe6\xdd\xc9\xb8b\x06\x0eo\xc5a%z\xe3\xff\x1e\xd2\x8e5\xe7",
    "address": "cosmos1uuhna3psjqfxnw4msrfzsr0g08yuyfxeht0qfh",
}
```

### Converter functions<a name="converter-functions"></a>

#### Mnemonic seed to private key<a name="mnemonic-seed-to-private-key"></a>

```python
from cosmospy import BIP32DerivationError, seed_to_privkey

seed = (
    "teach there dream chase fatigue abandon lava super senior artefact close upgrade"
)
try:
    privkey = seed_to_privkey(seed, path="m/44'/118'/0'/0/0")
except BIP32DerivationError:
    print("No valid private key in this derivation path!")
```

#### Private key to public key<a name="private-key-to-public-key"></a>

```python
from cosmospy import privkey_to_pubkey

privkey = bytes.fromhex(
    "6dcd05d7ac71e09d3cf7da666709ebd59362486ff9e99db0e8bc663570515afa"
)
pubkey = privkey_to_pubkey(privkey)
```

#### Public key to address<a name="public-key-to-address"></a>

```python
from cosmospy import pubkey_to_address

pubkey = bytes.fromhex(
    "03e8005aad74da5a053602f86e3151d4f3214937863a11299c960c28d3609c4775"
)
addr = pubkey_to_address(pubkey)
```

#### Private key to address<a name="private-key-to-address"></a>

```python
from cosmospy import privkey_to_address

privkey = bytes.fromhex(
    "6dcd05d7ac71e09d3cf7da666709ebd59362486ff9e99db0e8bc663570515afa"
)
addr = privkey_to_address(privkey)
```

### Signing transactions<a name="signing-transactions"></a>

```python
from cosmospy import Transaction

tx = Transaction(
    privkey=bytes.fromhex(
        "26d167d549a4b2b66f766b0d3f2bdbe1cd92708818c338ff453abde316a2bd59"
    ),
    account_num=11335,
    sequence=0,
    fee=1000,
    gas=70000,
    memo="",
    chain_id="cosmoshub-4",
)
tx.add_transfer(
    recipient="cosmos103l758ps7403sd9c0y8j6hrfw4xyl70j4mmwkf", amount=387000
)
tx.add_transfer(recipient="cosmos1lzumfk6xvwf9k9rk72mqtztv867xyem393um48", amount=123)


import httpx
import json

tx_bytes = tx.get_tx_bytes()


# Submit the transaction through the Tendermint RPC
rpc_url = "https://rpc.cosmos.network/"
pushable_tx = json.dumps(
              {
                "jsonrpc": "2.0",
                "id": 1,
                "method": "broadcast_tx_sync", # Available methods: broadcast_tx_sync, broadcast_tx_async, broadcast_tx_commit
                "params": {
                    "tx": tx_bytes
                }
              }
            )
r = httpx.post(rpc_url, data=pushable_tx)

# Submit the transaction through the Cosmos REST API
rpc_api = "https://api.cosmos.network/cosmos/tx/v1beta1/txs"
pushable_tx = json.dumps(
                {
                  "tx_bytes": tx_bytes, 
                  "mode": "BROADCAST_MODE_SYNC" # Available modes: BROADCAST_MODE_SYNC, BROADCAST_MODE_ASYNC, BROADCAST_MODE_BLOCK
                }
              )
r = httpx.post(rpc_api, data=pushable_tx)
```

One or more token transfers can be added to a transaction by calling the `add_transfer` method.

#### Note
* The account number and sequence can be queried through following api endpoint: ``https://api.cosmos.network/cosmos/auth/v1beta1/accounts/{address}``


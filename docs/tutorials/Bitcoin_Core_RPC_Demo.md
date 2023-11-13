---
title: "BDK wallet with Bitcoin core RPC "
description: "Tutorial showing usage of Bitcoin core backend with BDK wallet"
authors:
    - Rajarshi Maitra
    - ValuedMammal
date: "2023-11-15"
tags: ["tutorial", "BDK", "Bitcoin Core", "RPC", "Wallet"]
hidden: true
draft: false
---

## Introduction
BDK exists among a family of libraries that can be used to deploy wallets supporting a variety of blockchain backends like [electrum](https://github.com/romanz/electrs), [esplora](https://github.com/Blockstream/esplora), and [compact block filters](https://github.com/bitcoin/bips/blob/master/bip-0157.mediawiki). Since v0.10.0, BDK also supports Bitcoin Core as a blockchain backend by making use of rust-bitcoin's [bitcoincore-rpc](https://github.com/rust-bitcoin/rust-bitcoincore-rpc) crate. The combined tooling is elegantly designed to allow developers to get up to speed when it comes to prototyping and deploying custom wallets.

In this tutorial we will see how to write a simple wallet that will connect to a Bitcoin Core node, send and receive transactions, and update its balance on the fly. What makes it unique is that rather than use `bdk-cli`, we'll accomplish a number of tasks by coding a project in rust that uses the BDK library.

## You will need
- A Bitcoin Core node running on regtest. Download an official release from [bitcoincore.org](https://bitcoincore.org/bin/bitcoin-core-0.25.0/) or build it from [source](https://github.com/bitcoin/bitcoin). For this tutorial we're using Bitcoin 25.0.
- Rust toolchain. It's recommended to install from [rust-lang.org](https://www.rust-lang.org/tools/install).
- Time to complete: 30min


## Set Up
Configure `bitcoind` to run in regtest by creating a `bitcoin.conf` that looks like this.
```txt
server=1
regtest=1
rpcuser=admin
rpcpassword=password
txindex=1
fallbackfee=0.0001
```

Then create a new cargo project called `bdk-example` in whatever folder you like to write code, in this case `~/tutorial`.
```shell
cd ~
mkdir tutorial
cd tutorial
cargo new bdk-example
cd bdk-example
```

We created a new binary executable with the following directory structure. The two files to notice are `src/main.rs` and `Cargo.toml`.
```shell
$ tree -L 3 .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

The file `main.rs` is the main entry point of our program and looks like this:

``` rust
fn main() {
    println!("Hello, world!");
}
```

Open `Cargo.toml` which has also been populated with a simple skeleton.
```toml
[package]
name = "bdk-example"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

Try executing `cargo run` in a terminal. If the line "Hello, world!" is printed, then great, we're ready to go. If not, make sure you have properly installed rust for your system, then come back and repeat the steps above.


## Setting dependencies
We must specify the dependencies to be used in our project.

The `bdk` crate provides most of what we need for building a wallet. For this tutorial, we're interested in interfacing with Bitcoin Core, and for that we'll use another crate in the BDK ecosystem called `bdk_bitcoind_rpc`. The final dependency we're using is `anyhow` which provides additional ergonomic error handling.

Add these to `Cargo.toml` under the dependencies section like so:

```toml
[package]
name = "bdk-example"
version = "0.1.0"
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
anyhow = "1.0.75"
bdk = { version = "1.0", features = ["all-keys"] }
bdk_bitcoind_rpc = "0.1.0"
```

We enabled BDK's **all-keys** feature, which let's us derive keys as specified by [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki).

Now that we have the required dependencies, we can import everything we'll use at the top of `main.rs`.

```rust
use anyhow::Result;

use bdk::bitcoin::bip32::DerivationPath;
use bdk::bitcoin::secp256k1::Secp256k1;
use bdk::bitcoin::Amount;
use bdk::bitcoin::Network;

use bdk::descriptor;
use bdk::descriptor::IntoWalletDescriptor;
use bdk_bitcoind_rpc::bitcoincore_rpc;
use bdk_bitcoind_rpc::bitcoincore_rpc::Auth;
use bdk_bitcoind_rpc::bitcoincore_rpc::Client;
use bdk_bitcoind_rpc::bitcoincore_rpc::RpcApi;
use bdk_bitcoind_rpc::Emitter;

use bdk::keys::bip39::Language;
use bdk::keys::bip39::Mnemonic;
use bdk::keys::bip39::WordCount;
use bdk::keys::GeneratableKey;
use bdk::keys::GeneratedKey;

use bdk::miniscript::miniscript::Tap;
use bdk::wallet::AddressIndex;
use bdk::SignOptions;
use bdk::Wallet;

use std::str::FromStr;
```


## Descriptors
In BDK, [descriptors](https://bitcoindevkit.org/descriptors/) are a first-class citizen. Descriptors are a powerful and compact way of representing keychains capable of generating all the addresses a wallet may need, and can be used to encode more elaborate spending policies compatible with [miniscript](https://bitcoin.sipa.be/miniscript/). Bitcoin Core provides a good [primer on descriptors](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md).

Therefore, to define the keys our wallet is interested in, we must inform BDK of them in the form of a descriptor, which for example may look something like this:

`tr([04d89365/86'/1'/0']tpubDDmjFF6aY9qfiAQLWktoBZCBwVRWRqQeaxoxU3Fcxd3LxeRpSk2pU4WSsvXZbdXiMP1GKXSDUj5i4ExMmnehH9kxmL4gEWQGhmAYufv5hQh/0/*)#l8dq6la9`

To break it down, this is a Taproot descriptor with master fingerprint `04d89365` derived using the path `m/86'/1'/0'/0`, which along with `tpub...` signifies that we're using regtest (or testnet). Notice the `'*'` that lets us generate a range of child keys and addresses. We indicate the script type is pay-to-taproot by wrapping it all in `tr()`, and finally the bit at the end `#l8dq6la9` is a checksum commonly used by software to check data integrity. BDK can be used to generate a new keypair and BIP39 mnemonic, and from there create its own wallet descriptors, or you can bring your own.

Let's start coding.

We will use a function `create_descriptors` to create a pair of descriptors, one for the external (receive) addresses and the other for internal (change) addresses.

You'll notice two methods are shown for generating keys, only one of which is strictly necessary. The first is to generate a random mnemonic and the other is to provide our own. As a reminder, **the mnemonic is your private key. Don't use any of the secrets generated in this example with real money.**

Add the function `create_descriptors` right below `main()` as shown.
```rust
fn main() {
    ...
}

/// Creates wallet `Descriptor`s from a master secret
fn create_descriptors() -> Result<(String, String)> {
    // Create new secp context
    let secp = Secp256k1::new();

    // Generate a mnemonic from random entropy
    let mnemonic: GeneratedKey<_, Tap> =
       Mnemonic::generate((WordCount::Words12, Language::English)).expect("generate secret");
    println!("Mnemonic: {}", *mnemonic);

    // Provide our own mnemonic
    // let mnemonic = Mnemonic::from_str(
    //     "print region fury craft unique forest humble famous river cargo job egg",
    // )?;

    // Set an optional passphrase for the mnemonic
    let passphrase: Option<String> = None;
    let mnemonic_with_passphrase = (mnemonic, passphrase);

    // Define external and internal derivation paths
    let external_path = DerivationPath::from_str("m/86h/1h/0h/0")?;
    let internal_path = DerivationPath::from_str("m/86h/1h/0h/1")?;

    // Generate external and internal `Descriptors` from mnemonic
    let (external_descriptor, ext_keymap) =
        descriptor!(tr((mnemonic_with_passphrase.clone(), external_path)))?
            .into_wallet_descriptor(&secp, Network::Regtest)?;

    let (internal_descriptor, int_keymap) =
        descriptor!(tr((mnemonic_with_passphrase, internal_path)))?
            .into_wallet_descriptor(&secp, Network::Regtest)?;

    // Display public descriptor strings
    println!("External: {}", external_descriptor.to_string());
    println!("Internal: {}", internal_descriptor.to_string());

    Ok((
        external_descriptor.to_string_with_secret(&ext_keymap),
        internal_descriptor.to_string_with_secret(&int_keymap),
    ))
}
```

To check it's working as expected, go ahead and call the function from `main`. If you haven't already, remove the `println!("Hello, world!");` and do also add a `Result<()>` as the return type for `main`, since many of our functions require error handling.
``` rust
fn main() -> Result<()> {
    // Make descriptors
    let (recv_desc, chng_desc) = create_descriptors()?;
    
    Ok(())
}
```

```
$ cargo run

Mnemonic: print region fury craft unique forest humble famous river cargo job egg
External: "tr([fd985ed4/86'/1'/0']tpubDDAigjCsRaR5FM7nCwdC5NoJh2sAN4rRYA13zgVg9323Zf4Z1SCtCWfU55ttNXrxmbnGC2ZdCQFLqxhUUNGvNgbMVEocRJfSJtr7UKzBDQd/0/*)#n8e7s0pc"
Internal: "tr([fd985ed4/86'/1'/0']tpubDDAigjCsRaR5FM7nCwdC5NoJh2sAN4rRYA13zgVg9323Zf4Z1SCtCWfU55ttNXrxmbnGC2ZdCQFLqxhUUNGvNgbMVEocRJfSJtr7UKzBDQd/1/*)#znuld63q"

```

These are our *public* descriptors. Note however, we used `to_string_with_secret` in the last expression of `create_descriptors`, because to sign transactions, the wallet must have knowledge of the private keys, and so in this case we initialized a wallet using descriptors with the secrets included. Try adding another `println!` statement somewhere to see what those descriptors look like.


## Setup the BDK wallet
Now let's create a new BDK wallet. There are a few ways to persist wallet data to a file store, but for simplicity we'll use a transient, in-memory database provided by the `Wallet::new_no_persist` API.

```rust
fn main() -> Result<()> {
    // Make descriptors
    let (recv_desc, chng_desc) = create_descriptors()?;

    // Create Bdk wallet
    let mut wallet = Wallet::new_no_persist(&recv_desc, Some(&chng_desc), Network::Regtest)?;

    let balance = wallet.get_balance().total();
    println!("Wallet balance before sync: {balance} sat");

    let bdk_address = wallet.get_address(AddressIndex::New).address;
    println!("Wallet new address: {bdk_address}");

    Ok(())
}
```

Try getting a fresh address ready to be funded.
```
$ cargo run

...

Wallet balance before sync: 0 sat
Wallet new address: bcrt1phjt3lxr5rs0t9d2fvtmlp0vvhn4ru6w6zcveu45zkv0usrpyhfnset0658
```


## Talking to Bitcoin Core Programmatically
We'll be using two wallets to send coins back and forth: a Bitcoin Core wallet running `bitcoind`, and the BDK wallet we just created. Instead of using `bitcoin-cli` for wallet operations, we'll take advantage of the `bdk_bitcoind_rpc` library to talk to Core listening at `127.0.0.1:18443` from rust. This will allow us to write code in one place that handles both wallets simultaneously.

Start `bitcoind`.

We'll now configure the Core RPC client and test that we're able to make calls to `bitcoind` over the json-rpc interface.
```rust
    ...

    // Configure Core Rpc interface
    let auth = Auth::UserPass("admin".to_string(), "password".to_string());

    let client = bitcoincore_rpc::Client::new("http://127.0.0.1:18443/wallet/test", auth)?;

    // Test connection
    println!("Best block hash: {:?}", client.get_best_block_hash()?);

    Ok(())
}
```

We have provided the RPC `Auth` credentials (same as in `bitcoin.conf`), and we provided the URL to our local `bitcoind` with the path to a wallet file named `test`. We then asked the client for the current best block hash. That's all that's needed to programmatically talk to our Core node.

`cargo run` again and you should see the following:
```
$ cargo run

...

Best block hash: 008c1c54bfec053e788a7051a8bfaab69d3a529228887de04ca9bff682610217
```

A list of available RPC methods like `get_best_block_hash` can be found in the documentation for the [bitcoincore_rpc](https://docs.rs/bitcoincore-rpc/latest/bitcoincore_rpc/trait.RpcApi.html) crate. You may already be familiar with some of them.


## Mine regtest coins
We're going to need funds to play with. Create Bitcoin Core's wallet now, get a new address, and generate enough blocks for our balance to become spendable.
```rust
    ...

    // Create the test wallet 
    client.create_wallet("test", None, None, None, None)?;
    
    // Get new address
    let core_address = client.get_new_address(None, None)?.assume_checked();
    
    // Generate blocks
    println!("Generating blocks...");
    client.generate_to_address(101, &core_address)?;
    
    // Get Core balance
    let core_balance = client.get_balance(None, None)?.to_btc();
    println!("Core balance: {core_balance} BTC");

    Ok(())
}
```

```
$ cargo run

...

Core balance: 50 BTC       
```

Let's recap what we've done so far.

- Made a function `create_descriptors` to create the descriptors which define two new keychains.
- Created a temporary `Wallet` for BDK initialized with our descriptors and derived our first `Address`.
- Configured the `Client` from a `url` and `Auth` that's capable of speaking to Core over json-rpc, and practiced calling some common rpc methods.
- Created Bitcoin Core's wallet and mined 101 blocks on regtest.


## Sending Sats Around
If you made it this far, now it's time for the fun part. How do we go about syncing the state of BDK's wallet with transaction data from the blockchain? For this we introduce a new type `Emitter` who will serve two primary functions: 1) polling `bitcoind` for new blocks, and 2) querying the node's mempool for any unconfirmed transactions we're interested in. `Emitter` will emit all the relevant transaction data it finds to the wallet so that it may update its view of the best chain. This allows the BDK wallet to discover when it has recieved coins so it can proceed to create transactions as well.

The general flow will be:

- Send 2M satoshis from Core to BDK
- Send back 1M satoshis from BDK to Core
- Display the end balances

First we need to initialize `Emitter` with the rpc `Client` and a given `start_height`, which we can assume is zero, provided the node starts mining from genesis. Now is also a good time to set the payment amount to some constant value and place it before the start of `main`.
```rust
const AMOUNT: u64 = 2_000_000;

fn main() -> Result<()> {
    ...

    // Create chain source emitter
    let start_height = 0u32;
    let mut emitter = Emitter::new(&client, wallet.latest_checkpoint(), start_height);

    // Sync Bdk wallet
    sync(&mut wallet, &mut emitter)?;

    Ok(())
}
```

We then do an initial `sync` of the wallet so it can catch up to the chain tip. For that to work, we need another dedicated function which serves merely as a convenience, since it will be called two more times before we're finished. Let's do that now.

Create a new function called `sync` that accepts as parameters a mutable reference to `Wallet` and a mutable reference to `Emitter` and returns `Result<()>`. It can go at the bottom of the file for now.

```rust
/// Calls `Emitter::next_block`, applying transactions we care about to the wallet's
/// transaction graph until all updates have been consumed.
fn sync(wallet: &mut Wallet, emitter: &mut Emitter<Client>) -> Result<()> {
    print!("Syncing... ");
    while let Some((height, block)) = emitter.next_block()? {
        wallet.apply_block_relevant(block, height)?;
    }
    wallet.commit()?;

    println!("Ok");
    Ok(())
}
```

Basically, we're calling `Emitter::next_block` repeatedly until it returns `None`, signifying we reached the tip. On each round, `apply_block_relevant` scans a block for transactions sending to or from the wallet, staging changes to the local chain and transaction graph to be committed by the wallet.

Now that we're in sync with the chain, let's carry on. We left `main` right after calling `sync`.

```rust
    ...

    // Fund Bdk wallet (tx 1)
    println!("Sending 2M sat Core -> Bdk");
    client.send_to_address(&bdk_address, Amount::from_sat(AMOUNT), None, None, None, None, None, None)?;

    // Query mempool
    let unconfirmed = emitter.mempool()?;
    wallet.batch_insert_relevant_unconfirmed(unconfirmed.iter().map(|(tx, time)| (tx, *time)));
    wallet.commit()?;

    // Show pending balance
    let balance = wallet.get_balance().untrusted_pending;
    println!("Wallet pending: {balance} sat");

    // Confirm tx by generating blocks
    client.generate_to_address(1, &core_addr)?;

    // Sync (2)
    sync(&mut wallet, &mut emitter)?;

    // Show confirmed
    let balance = wallet.get_balance().confirmed;
    assert_eq!(balance, AMOUNT);
    println!("Confirmed! {balance} sat");
    println!();

    // Build tx with Bdk wallet (tx 2)
    println!("Sending 1M sat Bdk -> Core");
    let mut tx_builder = wallet.build_tx();
    let amount = AMOUNT / 2;
    tx_builder.add_recipient(core_address.script_pubkey(), amount);

    // Finish building tx and get a new Psbt
    let mut psbt = tx_builder.finish()?;

    // Sign it
    wallet.sign(&mut psbt, SignOptions::default())?;

    // Extract raw tx from signed psbt and broadcast
    let tx = psbt.extract_tx();
    let fee = wallet.calculate_fee(&tx).expect("calculate fee");
    let txid = client.send_raw_transaction(&tx)?;
    println!("Txid: {txid}");

    // Confirm tx
    client.generate_to_address(1, &core_address)?;

    // Sync (3)
    sync(&mut wallet, &mut emitter)?;

    // Display balances
    let core_balance = client.get_balance(None, None)?.to_btc();
    let bdk_balance = wallet.get_balance().total();
    assert!(core_balance < 150.0);
    assert!(bdk_balance < amount);
    println!("Core balance: {core_balance} BTC");
    println!("Wallet balance: {bdk_balance} sat ");
    println!("Fee: {fee} sat");

    Ok(())
}
```

That's it for the project code. Note what occurred in the last steps:

- Sent tx 1 from Core to BDK
- Used emitter to find the unconfirmed tx
- Mined a block to confirm tx 1
- Sync wallet
- Constructed tx 2 sending from BDK to Core and got a new PSBT
- Signed tx 2 and passed it back to Core to be broadcast
- Mined a block to confirm tx 2
- Sync wallet
- Assert balances are expected

Here's the expected output. If anything went wrong, check that you did the following:

- Properly configured `bitcoind` for regtest and started with a clean data directory
- Used the correct RPC `Auth` credentials when creating the `bitcoincore_rpc::Client`
- Included all dependencies including `bdk` v1.0
- Checked for syntax errors

```
$ cargo run
    Compiling bdk-example v0.1.0 (/home/satoshi/tutorial/bdk-example)
    Finished dev [unoptimized + debuginfo] target(s) in 2.26s
    Running `/target/debug/bdk-example`
Mnemonic: print region fury craft unique forest humble famous river cargo job egg
External: tr([fd985ed4/86'/1'/0']tpubDDAigjCsRaR5FM7nCwdC5NoJh2sAN4rRYA13zgVg9323Zf4Z1SCtCWfU55ttNXrxmbnGC2ZdCQFLqxhUUNGvNgbMVEocRJfSJtr7UKzBDQd/0/*)#n8e7s0pc
Internal: tr([fd985ed4/86'/1'/0']tpubDDAigjCsRaR5FM7nCwdC5NoJh2sAN4rRYA13zgVg9323Zf4Z1SCtCWfU55ttNXrxmbnGC2ZdCQFLqxhUUNGvNgbMVEocRJfSJtr7UKzBDQd/1/*)#znuld63q
Wallet balance before sync: 0 sat
Wallet new address: bcrt1phjt3lxr5rs0t9d2fvtmlp0vvhn4ru6w6zcveu45zkv0usrpyhfnset0658
Best block hash: 0x0f9188f13cb7b2c71f2a335e3a4fc328bf5beb436012afca590b1a11466e2206
Generating blocks...
Core balance: 50 BTC

Syncing... Ok
Sending 2M sat Core -> Bdk
Wallet pending: 2000000 sat
Syncing... Ok
Confirmed! 2000000 sat

Sending 1M sat Bdk -> Core
Txid: a996ec1e384bdd693220121d6919c0cbafb3b3bd7f5e7164cacfc84f1e4d9c86
Syncing... Ok
Core balance: 149.9899835 BTC
Wallet bal: 999857 sat 
Fee: 143 sat
```


## Conclusion
Hopefully you have a better idea of how BDK can fetch updates by polling `bitcoind` for block data. You should feel more confident now implementing something similar in your own apps. Next steps might include doing a deep dive on crafting transactions with [`TxBuilder`](https://docs.rs/bdk/latest/bdk/wallet/tx_builder/struct.TxBuilder.html) or exploring spending policies with miniscript descriptors.

As you can imagine, this is just tip of the iceberg. BDK aims for maximum flexibility in every dimension of bitcoin wallets, from keys and signers, to getting blocks, and ensuring data stays consistent. With this power in hand, we were able to implement a trustless, non-custodial, private wallet backed by a full node in under 200 lines of code. It's made possible because BDK handles the fine details under the hood, and it remains a goal and obsession to offer a convenient, high-level API so wallet devs can focus on writing the app logic they care about.

To discover more of the magic behind BDK and how it might fit your development needs refer the resources below. *Please note the BDK software is under rapid development, and features are rolled out frequently. If at any point you get stuck or believe this tutorial is out of date, don't hesitate to pop into one of the communication channels to share your thoughts.

Here's the final code.

```rust
use anyhow::Result;

use bdk::bitcoin::bip32::DerivationPath;
use bdk::bitcoin::secp256k1::Secp256k1;
use bdk::bitcoin::Amount;
use bdk::bitcoin::Network;

use bdk::descriptor;
use bdk::descriptor::IntoWalletDescriptor;
use bdk_bitcoind_rpc::bitcoincore_rpc;
use bdk_bitcoind_rpc::bitcoincore_rpc::Auth;
use bdk_bitcoind_rpc::bitcoincore_rpc::Client;
use bdk_bitcoind_rpc::bitcoincore_rpc::RpcApi;
use bdk_bitcoind_rpc::Emitter;

use bdk::keys::bip39::Language;
use bdk::keys::bip39::Mnemonic;
use bdk::keys::bip39::WordCount;
use bdk::keys::GeneratableKey;
use bdk::keys::GeneratedKey;

use bdk::miniscript::miniscript::Tap;
use bdk::wallet::AddressIndex;
use bdk::SignOptions;
use bdk::Wallet;

use std::str::FromStr;

const AMOUNT: u64 = 2_000_000;

fn main() -> Result<()> {
    // Make descriptors
    let (recv_desc, chng_desc) = create_descriptors()?;

    // Create Bdk wallet
    let mut wallet = Wallet::new_no_persist(&recv_desc, Some(&chng_desc), Network::Regtest)?;

    let balance = wallet.get_balance().total();
    println!("Wallet balance before sync: {balance} sat");

    let bdk_address = wallet.get_address(AddressIndex::New).address;
    println!("Wallet new address: {bdk_address}");

    // Configure Core Rpc interface
    let auth = Auth::UserPass("admin".to_string(), "password".to_string());

    let client = bitcoincore_rpc::Client::new("http://127.0.0.1:18443/wallet/test", auth)?;

    // Test connection
    println!("Best block hash: {:?}", client.get_best_block_hash()?);

    // Create Core wallet
    client.create_wallet("test", None, None, None, None)?;

    // Get new Core address
    let core_address = client.get_new_address(None, None)?.assume_checked();

    // Generate blocks
    println!("Generating blocks...");
    client.generate_to_address(101, &core_address)?;

    // Get Core balance
    let core_balance = client.get_balance(None, None)?.to_btc();
    println!("Core balance: {core_balance} BTC");
    println!();

    // Configure chain source emitter
    let start_height = 0u32;
    let mut emitter = Emitter::new(&client, wallet.latest_checkpoint(), start_height);

    // Sync Bdk wallet (1)
    sync(&mut wallet, &mut emitter)?;

    // Fund Bdk wallet (tx 1)
    println!("Sending 2M sat Core -> Bdk");
    client.send_to_address(&bdk_address, Amount::from_sat(AMOUNT), None, None, None, None, None, None)?;

    // Query mempool tx
    let unconfirmed = emitter.mempool()?;
    wallet.batch_insert_relevant_unconfirmed(unconfirmed.iter().map(|(tx, time)| (tx, *time)));
    wallet.commit()?;

    // Show pending balance
    let balance = wallet.get_balance().untrusted_pending;
    println!("Wallet pending: {balance} sat");

    // Confirm tx by generating blocks
    client.generate_to_address(1, &core_address)?;

    // Sync (2)
    sync(&mut wallet, &mut emitter)?;

    // Show confirmed
    let balance = wallet.get_balance().confirmed;
    assert_eq!(balance, AMOUNT);
    println!("Confirmed! {balance} sat");
    println!();

    // Build tx with Bdk wallet (tx 2)
    println!("Sending 1M sat Bdk -> Core");
    let mut tx_builder = wallet.build_tx();
    let amount = AMOUNT / 2;
    tx_builder.add_recipient(core_address.script_pubkey(), amount);

    // Finish building tx and get a new Psbt
    let mut psbt = tx_builder.finish()?;

    // Sign it
    wallet.sign(&mut psbt, SignOptions::default())?;

    // Extract raw tx from signed psbt and broadcast
    let tx = psbt.extract_tx();
    let fee = wallet.calculate_fee(&tx).expect("calculate fee"); // TODO: propagate the error
    let txid = client.send_raw_transaction(&tx)?;
    println!("Txid: {txid}");

    // Confirm tx
    client.generate_to_address(1, &core_address)?;

    // Sync (3)
    sync(&mut wallet, &mut emitter)?;

    // Display balances
    let core_balance = client.get_balance(None, None)?.to_btc();
    let bdk_balance = wallet.get_balance().total();
    assert!(core_balance < 150.0);
    assert!(bdk_balance < amount);
    println!("Core balance: {core_balance} BTC");
    println!("Wallet balance: {bdk_balance} sat ");
    println!("Fee: {fee} sat");

    Ok(())
}

/// Creates wallet `Descriptor`s from a master secret
fn create_descriptors() -> Result<(String, String)> {
    // Create new secp context
    let secp = Secp256k1::new();

    // Generate mnemonic
    let mnemonic: GeneratedKey<_, Tap> =
        Mnemonic::generate((WordCount::Words12, Language::English)).expect("generate secret");
    println!("Mnemonic: {}", *mnemonic);

    // Provide our own mnemonic
    // let mnemonic = Mnemonic::from_str(
    //     "print region fury craft unique forest humble famous river cargo job egg",
    // )?;
    // println!("Mnemonic: {}", mnemonic);

    // Set an optional passphrase to unlock the mnemonic
    let passphrase: Option<String> = None;
    let mnemonic_with_passphrase = (mnemonic, passphrase);

    // define external and internal derivation key path
    let external_path = DerivationPath::from_str("m/86h/1h/0h/0")?;
    let internal_path = DerivationPath::from_str("m/86h/1h/0h/1")?;

    // generate external and internal descriptor from mnemonic
    let (external_descriptor, ext_keymap) =
        descriptor!(tr((mnemonic_with_passphrase.clone(), external_path)))?
            .into_wallet_descriptor(&secp, Network::Regtest)?;

    let (internal_descriptor, int_keymap) =
        descriptor!(tr((mnemonic_with_passphrase, internal_path)))?
            .into_wallet_descriptor(&secp, Network::Regtest)?;

    println!("External: {}", external_descriptor.to_string());
    println!("Internal: {}", internal_descriptor.to_string());

    Ok((
        external_descriptor.to_string_with_secret(&ext_keymap),
        internal_descriptor.to_string_with_secret(&int_keymap),
    ))
}

/// Calls `Emitter::next_block`, applying transactions we care about to the wallet's
/// transaction graph until all updates have been consumed.
fn sync(wallet: &mut Wallet, emitter: &mut Emitter<Client>) -> Result<()> {
    print!("Syncing... ");
    while let Some((height, block)) = emitter.next_block()? {
        wallet.apply_block_relevant(block, height)?;
    }
    wallet.commit()?;

    println!("Ok");
    Ok(())
}
```

 - [source code](https://github.com/bitcoindevkit/bdk)
 - [dev docs](https://docs.rs/bdk/latest/bdk/)
 - [community](https://discord.com/invite/d7NkDKm)


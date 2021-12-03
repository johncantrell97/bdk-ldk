## BDK-LDK

The `bdk-ldk` library hopes to bridge the gaps needed to implement a ldk-based lightning client using a bdk::Wallet.

* Implements `lightning::chain::chaininterface::FeeEstimator` so LDK can estimate fees when constructing transactions.
* Implements `lightning::chain::chaininterface::BroadcasterInterface` so LDK can broadcast transactions to the network.
* Implements `lightning::chain::Filter` so we know what transactions and scripts to watch for during sync.
* Provides chain data to LDK using the `lightning::chain::Confirm` interface.

## Note(s)

This repo currently uses my fork of `bdk` until `IndexedChain` trait (or similar) gets merged upstream.  This will work with any `bdk::Blockchain` that implements `IndexedChain`. My fork currently implements `IndexedChain` for `bdk::ElectrumBlockchain` and `bdk::EsploraBlockchain`.

Hopefully the functionality can be extended to the rest of the bdk `Blockchain`s.  It will likely mean implementing a sync using ldk's `Listen` interface where you must provide full blocks instead of transaction information.

This cannot be published to crates.io with the git dependency, use this from source or as a reference for now.

## Example Node

There will be a full bdk-ldk example node that uses this library published in a separate repository soon.

## Usage

```rust
use std::sync::Arc;
use bdk::{Wallet, database::MemoryDatabase, blockchain::EsploraBlockchain};
use lightning::chain::Confirm;
use bdk_ldk::LightningWallet;


fn main() {
    let bdk_wallet = Wallet::new(
        "wpkh([c258d2e4/84h/1h/0h]tpubDDYkZojQFQjht8Tm4jsS3iuEmKjTiEGjG6KnuFNKKJb5A6ZUCUZKdvLdSDWofKi4ToRCwb9poe1XdqfUnP4jaJjCB2Zwv11ZLgSbnZSNecE/0/*)",
        Some("wpkh([c258d2e4/84h/1h/0h]tpubDDYkZojQFQjht8Tm4jsS3iuEmKjTiEGjG6KnuFNKKJb5A6ZUCUZKdvLdSDWofKi4ToRCwb9poe1XdqfUnP4jaJjCB2Zwv11ZLgSbnZSNecE/1/*)"),
        bitcoin::Network::Testnet,
        MemoryDatabase::default(),
        EsploraBlockchain::new("https://blockstream.info/testnet/api", 20)
    );

    let ldk_wallet = Arc::new(LightningWallet::new(bdk_wallet));
    let fee_estimator = ldk_wallet.clone();
    let filter = ldk_wallet.clone();
    let broadcaster = ldk_wallet.clone();

    ...

    let mut channel_manager = ...;
    let mut chain_monitor = ...;

    let confirmables = vec![
		&*channel_manager as &dyn Confirm, 
		&*chain_monitor as &dyn Confirm
	];

    ldk_wallet.sync(confirmables);
}
```

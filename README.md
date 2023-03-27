
**WARNING** : This repo deos not contain the full implementation but is intended to act as a source of inspiration for anyone who wants to implement the PSP34 complaint smart contracts XCM.


# PSP34 compliant XCM 

In order to transfer a PSP34 compliant NFT, we have to implement `TransactAsset` trait which deals with how to withdraw and deposit an Asset which in our case is a NFT.

We have implemented a struct `PSP34MutateAdapter` which implements the TransactAsset trait for PSP34complaint smart contracts. 
```
pub type PSP34Transactor = PSP34MutateAdapter<
    // Use this non-fungibles implementation:
    Contracts,
    // Use this currency when it is a non-fungible asset matching the given location or name:
    ConvertedConcreteAssetId<ClassId, InstanceId, AstarPSP34ClassIdConverter, AstarPSP34InstanceIdConverter>,
    // Convert an XCM MultiLocation into a local account id:
    LocationToAccountId,
    // Our chain's account ID type (we can't get away without mentioning it explicitly):
    AccountId,
    // We don't track any teleports of `Assets`.
    Nothing,
    // We don't track any teleports of `Assets`.
    CheckingAccount,
>;
```
Trait definition for PSP34MutateAdapter can be found here-> https://github.com/gitofdeepanshu/polkadot/pull/1

## Trait Bounds for Pallet Contracts
Structure of our new Adapter

```
impl<
		Assets: nonfungibles::MutatePSP34<AccountId>,
		Matcher: MatchesNonFungibles<Assets::CollectionId, Assets::ItemId>,
		AccountIdConverter: Convert<MultiLocation, AccountId>,
		AccountId: Clone + Eq + From<[u8;32]>, // can't get away without it since Currency is generic over it.
		CheckAsset: AssetChecking<Assets::CollectionId>,
		CheckingAccount: Get<Option<AccountId>>,
	>
	PSP34MutateAdapter<
		Assets,
		Matcher,
		AccountIdConverter,
		AccountId,
		CheckAsset,
		CheckingAccount,
	>
```

In order to make pallet contracts compatible with our new adapter, we have to implement trait `nonfungibles::MutatePSP34<AccountId>` for pallet_contracts.
Here is the PR for new trait definitions for pallet_contracts: https://github.com/gitofdeepanshu/substrate/pull/1

and here is the actual implentation of traits : https://github.com/paritytech/substrate/commit/ab6646e57c8a340d9365833e198f766fefc80ede

### MultiAsset Scheme for PSP34 
To reference a NFT, we need :

a. Smart Contract Address

b. Item Id for a specific NFTin collection

```
MultiAsset {
id: Concrete( MultiLocation { parents: 0, interior: Junctions::X1(AccountId32 {network: None, id: [u8;32] } ) } ),
fun: Fungibility::NonFungible::Index(u128)
```

We get smart contract address from `id` and item id from `fun`

## Analysis

### What are the potential problems with this approach?

Problems can be:
1. No Gas Limit set for XCM call execution.
2. Pallet contracts require caller_account even when reading data from contract storage which makes traits complex.
### What are the benefits compared to using `pallet-uniques` in the backend to hold NFTs?
Benefits:
1. No need for chain extension (to interact with pallet_uniques) while using this implementation. Also bypass the use of pallet_unique and hence NFT can be teleported to a chain which doesn't implement pallet_uniques.
2. Smart Contracts are user composable but Runtime is more rigid and would require more resources to edit (considering security implications etc)
### What would be the next step with this implementation?
Next steps for the implementation can be found in TODO section.

## TODO
1. Connecting all the pieces together.
2. Implementing the non-implemented functions.
3. Writing tests.

# Configuration Traits

Rather than _directly_ depend on the Balances pallet from within our pallet, we can instead define a configurable trait!

## Define Currency Type

```rust
pub trait Trait: system::Trait {
	//--note------------------^^
	type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
	/// The type for our blockchain currency
	type Currency: ReservableCurrency<Self::AccountId>;
}
```

## Use Currency Type

You then call the functions using the `Currency` type instead!

```rust
T::Currency::reserve(&sender, 1000.into())?;
//--snip--
T::Currency::unreserve(&sender, 1000.into());
```

<!-- slide:break -->

## Implement Currency Type

Then you can implement this trait _using_ the balances pallet, but outside of your own pallet.

**runtime/src/lib.rs**

```rust
impl template::Trait for Runtime {
	type Event = Event;
	type Currency = Balances;
}
```

<!-- tabs:start -->

#### ** Hide Hints **

Click the other tabs to view hints.

#### ** Final Code **

```rust
use frame_support::{decl_module, decl_storage, decl_event, ensure};
use sp_std::vec::Vec;
use system::ensure_signed;
use frame_support::traits::ReservableCurrency;

/// The pallet's configuration trait.
pub trait Trait: system::Trait {
	/// The overarching event type.
	type Event: From<Event<Self>> + Into<<Self as system::Trait>::Event>;
	/// The type for our blockchain currency
	type Currency: ReservableCurrency<Self::AccountId>;
}

// This pallet's events.
decl_event! {
	pub enum Event<T> where AccountId = <T as system::Trait>::AccountId {
		/// Event emitted when a proof has been claimed.
		ClaimCreated(AccountId, Vec<u8>),
		/// Event emitted when a claim is revoked by the owner.
		ClaimRevoked(AccountId, Vec<u8>),
	}
}

// This pallet's storage items.
decl_storage! {
	trait Store for Module<T: Trait> as PoeStorage {
		/// The storage item for our proofs.
		/// It maps a proof to the user who made the claim and when they made it.
		Proofs: map Vec<u8> => (T::AccountId, T::BlockNumber);
	}
}

// The pallet's dispatchable functions.
decl_module! {
	/// The module declaration.
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {
		// A default function for depositing events
		fn deposit_event() = default;

		/// Allow a user to claim ownership of an unclaimed proof
		fn create_claim(origin, proof: Vec<u8>) {
			// Verify that the incoming transaction is signed and store who the
			// caller of this function is.
			let sender = ensure_signed(origin)?;

			// Verify that the specified proof has not been claimed yet or error with the message
			ensure!(!Proofs::<T>::exists(&proof), "This proof has already been claimed.");

			// Try to reserve the deposit from the user
			T::Currency::reserve(&sender, 1000.into())?;

			// Call the `system` pallet to get the current block number
			let current_block = <system::Module<T>>::block_number();

			// Store the proof with the sender and the current block number
			Proofs::<T>::insert(&proof, (sender.clone(), current_block));

			// Emit an event that the claim was created
			Self::deposit_event(RawEvent::ClaimCreated(sender, proof));
		}

		/// Allow the owner to revoke their claim
		fn revoke_claim(origin, proof: Vec<u8>) {
			// Determine who is calling the function
			let sender = ensure_signed(origin)?;

			// Verify that the specified proof has been claimed
			ensure!(Proofs::<T>::exists(&proof), "This proof has not been stored yet.");

			// Get owner of the claim
			let (owner, _) = Proofs::<T>::get(&proof);

			// Verify that sender of the current call is the claim owner
			ensure!(sender == owner, "You must own this claim to revoke it.");

			// Unreserve the deposit from the user
			T::Currency::unreserve(&sender, 1000.into());

			// Remove claim from storage
			Proofs::<T>::remove(&proof);

			// Emit an event that the claim was erased
			Self::deposit_event(RawEvent::ClaimRevoked(sender, proof));
		}
	}
}
```
<!-- tabs:end -->

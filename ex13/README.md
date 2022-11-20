## 1. What is this exercise?

It teaches you about:
Privacy level in the blockchain

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    expected_value: felt
) {
    // Reading caller address
    let (sender_address) = get_caller_address();
    // Checking that the user got a slot assigned
    let (user_slot) = user_slots_storage.read(sender_address);

    with_attr error_message("User slot not assigned. Call assign_user_slot") {
        assert_not_zero(user_slot);
    }

    with_attr error_message("Input value is not the expected secret value") {
        // Checking that the value provided by the user is the one we expect
        let (value) = values_mapped_secret_storage.read(user_slot);
        assert value = expected_value;
    }

    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass a `expected_value`. It checks if:

- user got a slot assigned (just call `assign_user_slot()`)
- the value of `expected_value` is the same as the value linked to the user address in the `values_mapped_secret_storage` mapping

I found this exercisa a bit tough, there is no view function to see that mapping value or clear indicative about that, but reading the code we can have a clue how to get it.

There is a view function called `user_slots()`, when you call it passing your account it returns a number, you will need that later.

On the top of the code, there is that comment:

```
// - Use past data from transactions sent to the contract to find a value that is supposed to be "secret"
// you might need this endpoint https://alpha4.starknet.io/feeder_gateway/get_transaction?transactionHash=
```

https://alpha4.starknet.io/feeder_gateway/get_transaction?transactionHash= is an important link, as said in the comment you should pass a hash of a transaction to that link, so just to go to the blockchain explorer and get the hash there.

You will see it returns an object, there is a `constructor_calldata` with some elements, within that you can see a lot of numbers in hexadecimal. That previous number that you got with `user_slots()` indicates the secret value position that you should find on the array of hexadecimal numbers, just be aware that that was stored backawards.

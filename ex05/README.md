## 1. What is this exercise?

It teaches you about:
About public and private variables

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

    with_attr error_message("User slot not assigned. Call assign_user_slot") {
        // Checking that the user got a slot assigned
        let (user_slot) = user_slots_storage.read(sender_address);
        assert_not_zero(user_slot);
    }

    // Checking that the value provided by the user is the one we expect
    // Still sneaky.
    let (value) = values_mapped_secret_storage.read(user_slot);
    with_attr error_message("Input value is not the expected secret value") {
        assert value = expected_value + 23;
    }

    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass an `expected_value`, it will evaluate:

- if the user has got a slot assigned (just need call `assign_user_slot()` external function, for that).
- if the value passed by the user is equal to the `value` variable.

The `value` variable is storing a mapping from `values_mapped_secret_storage`, which links a value to a user slot:

```
@storage_var
func values_mapped_secret_storage(slot: felt) -> (values_mapped_secret_storage: felt) {
}
```

Since that `values_mapped_secret_storage` is a private variable, it can't be called externally, but fortunately there is the external function `copy_secret_value_to_readable_mapping()`:

```
@external
func copy_secret_value_to_readable_mapping{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}() {
    // Reading caller address
    let (sender_address) = get_caller_address();

    with_attr error_message("User slot not assigned. Call assign_user_slot") {
        // Checking that the user got a slot assigned
        let (user_slot) = user_slots_storage.read(sender_address);
        assert_not_zero(user_slot);
    }
    // Reading user secret value
    let (secret_value) = values_mapped_secret_storage.read(user_slot);

    // Copying the value from non accessible values_mapped_secret_storage to
    user_values_public_storage.write(sender_address, secret_value - 23);
    return ();
}

```

`copy_secret_value_to_readable_mapping()` reads internally the `values_mapped_secret_storage()` mapping and store its value into the public mapping `user_values_public_storage()`, just be aware that it does subtracting - 20. Also you should note that `claim_points()` does a sum when checking the value.

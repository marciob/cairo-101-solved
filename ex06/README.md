## 1. What is this exercise?

It teaches you about:
About internal and external functions

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

    // Checking that the value provided by the user is the one we expect
    // Still sneaky.
    // Or not. Is this psyops?
    let (value) = values_mapped_secret_storage.read(user_slot);
    with_attr error_message("random values already initialized") {
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

`claim_points()` is asking to pass an `expected_value`, it will evaluate:

- if the user has got a slot assigned (just need call `assign_user_slot()` external function, for that).
- if the value passed by the user is equal to the `value` variable.

The `value` variable is storing a mapping from `values_mapped_secret_storage`, which links a value to a user slot:

```
@storage_var
func values_mapped_secret_storage(slot: felt) -> (values_mapped_secret_storage: felt) {
}
```

Since that `values_mapped_secret_storage` is a private variable, it can't be called externally, just like the previous exercise there is the function `copy_secret_value_to_readable_mapping()`:

```
func copy_secret_value_to_readable_mapping{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}(sender_address: felt) {
    with_attr error_message("User slot not assigned. Call assign_user_slot") {
        // Checking that the user got a slot assigned
        let (user_slot) = user_slots_storage.read(sender_address);
        assert_not_zero(user_slot);
    }

    // Reading user secret value
    let (secret_value) = values_mapped_secret_storage.read(user_slot);

    // Copying the value from non accessible values_mapped_secret_storage to
    user_values_public_storage.write(sender_address, secret_value);
    return ();
}

```

But now `copy_secret_value_to_readable_mapping()` is a private function, fortunately there is the external function `external_handler_for_internal_function()`:

```
@external
func external_handler_for_internal_function{
    syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr
}(a_value: felt) {
    // Reading caller address
    let (sender_address) = get_caller_address();
    with_attr error_message("Value provided is not 0") {
        // Just for fun
        assert a_value = 0;
    }
    // Calling internal function
    copy_secret_value_to_readable_mapping(sender_address);
    return ();
}
```

`external_handler_for_internal_function()` calls `copy_secret_value_to_readable_mapping()` which stores the secret value in a `user_values_public_storage()` public mapping. Doing that you are able to know the secret value and pass in the test.

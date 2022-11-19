## 1. What is this exercise?

It teaches you about:
Reading a mapping

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
    // Yes, I'm sneaky
    let (value) = values_mapped_storage.read(user_slot);
    with_attr error_message("Input value is not the expected secret value") {
        assert value = expected_value + 32;
    }
    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass an `expected_value`, it will evaluate if the value passed is equal to the `value` variable.

The `value` variable is a mapping related to `user_slot`.

`user_slot` is set at the time that the user calls `assign_user_slot`. There is a `user_slots()` view function to check the slot related to an account.

After you know the slot related your account, you can call `values_mapped()` view function to check the value that is linked to that slot in the mapping.

With that value, you can pass into `claim_points()` function, but, there is a small detail, the function is evaluating not the simple number, but the number + 32, so be aware.

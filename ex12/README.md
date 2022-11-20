## 1. What is this exercise?

It teaches you about:
Events

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
    with_attr error_message("Input value is not the expected secret value") {
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
- the value of `expected_value` is the same as the value linked to the user slot in the `values_mapped_secret_storage` mapping

There is no view function to see that mapping value, but in the `assign_user_slot()` you can see that it triggers an event with that value:

```
@external
func assign_user_slot{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    // Reading caller address
    let (sender_address) = get_caller_address();
    let (next_slot_temp) = next_slot.read();
    let (next_value) = values_mapped_secret_storage.read(next_slot_temp + 1);
    if (next_value == 0) {
        user_slots_storage.write(sender_address, 1);
        next_slot.write(0);
    } else {
        user_slots_storage.write(sender_address, next_slot_temp + 1);
        next_slot.write(next_slot_temp + 1);
    }
    let (user_slot) = user_slots_storage.read(sender_address);
    let (secret_value) = values_mapped_secret_storage.read(user_slot);
    // Emit an event with secret value
    assign_user_slot_called.emit(sender_address, secret_value + 32);
    return ();
}
```

The event is being emited with a summing, so just do the proper calculation to get the real value and pass in the test.

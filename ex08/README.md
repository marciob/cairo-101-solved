## 1. What is this exercise?

It teaches you about:
The basics of Recursions

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    // Reading caller address
    let (sender_address) = get_caller_address();

    // Checking the value of user_values_storage for the user, at slot 10
    let (user_value_at_slot_ten) = user_values_storage.read(sender_address, 10);

    with_attr error_message("User value should be 10 (you can set it with set_user_values)") {
        // This value should be equal to 10
        assert user_value_at_slot_ten = 10;
    }

    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` isn't asking to pass any value, what it does is checking the state of `user_values_storage` mapping to see the value of the slot 10 is equal to 10.

To pass in the exercise the user needs to set the number 10 to the slot 10 in the `user_values_storage` mapping.

`user_values_storage` is a private variable, but there is the `user_values` view function that checks its value:

```
@view
func user_values{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    account: felt, slot: felt
) -> (value: felt) {
    let (value) = user_values_storage.read(account, slot);
    return (value,);
}
```

The `user_values_storage` mapping is being set at the `set_user_values_internal()` internal function:

```
func set_user_values_internal{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    account: felt, length: felt, array: felt*
) {
    // This function is used recursively to set all the user values
    // Recursively, we first go through the length of the array
    // Once at the end of the array (length = 0), we start summing
    if (length == 0) {
        // Start with sum=0.
        return ();
    }

    // If length is NOT zero, then the function calls itself again, moving forward one slot
    set_user_values_internal(account=account, length=length - 1, array=array + 1);

    // This part of the function is first reached when length=0.
    user_values_storage.write(account, length - 1, [array]);
    return ();
}
```

Reading the function you see that it pass as value a length (but if you have attention you see that it's also subracting -1 there, take into in account when chosing which value to pass) and an array.

At this point you know that we can't call that internal function directly to set our desired value, but you may imagine that there is an external function to call that, there is:

```
@external
func set_user_values{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    account: felt, array_len: felt, array: felt*
) {
    set_user_values_internal(account, array_len, array);
    return ();
}
```

Nice, now you just need to pass your desired values. One very important detail that you need to notice is that the `set_user_values_internal()` sets the value in a back foward way, take it into account when passing the values.

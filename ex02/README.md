# 1. What is this exercise?

It teaches you about:

- understanding asserts.

# 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(my_value: felt) {
    // Reading caller address
    let (sender_address) = get_caller_address();
    // Reading stored value from storage
    let (my_secret_value) = my_secret_value_storage.read();
    // Checking that the value sent is correct
    // Using assert this way is similar to using "require" in Solidity
    with_attr error_message("Wrong secret value") {
        assert my_value = my_secret_value;
    }
    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

# 3. How to call claim_points() successfully?

You pass if you are able to call `claim_points()` without an error.

`claim_points()` asks to pass a value for `my_value`, the function evaluates if that value is the same as `my_secret_value`. So you need to know what is the value of `my_secret_value`.

To know that, there is a view function that returns that value, called `my_secret_value()`, just call it and pass the data into claim_points().

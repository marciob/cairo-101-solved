## 1. What is this exercise?

It teaches you about:
using contract functions to manipulate contract variables

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() {
    // Reading caller address
    let (sender_address) = get_caller_address();
    // Checking that user's counter is equal to 7
    let (current_counter_value) = user_counters_storage.read(sender_address);
    with_attr error_message("Counter is not equal to 7") {
        assert current_counter_value = 7;
    }
    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` isn’t asking to pass any value, but the function evaluates if `user_counters_storage` for the user account address is equal to 7.

You can get the value of `user_counters_storage` using the view function `user_counters`, it starts as 0, the default value for felts.

How can you change it to 7? In this contract there are two functions to move foward and back the value, `increment_counter` and `decrement_counter`. Just take into account that `increment_counter` increase the number by 2. Doing the current process you will be able set the number to 7 and pass.

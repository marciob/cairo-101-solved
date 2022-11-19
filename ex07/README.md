## 1. What is this exercise?

It teaches you about:
Understanding functions to compare values

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    value_a: felt, value_b: felt
) {
    // Reading caller address
    let (sender_address) = get_caller_address();
    // Checking that the passed values are correct
    with_attr error_message("value_a shouldn't be 0") {
        assert_not_zero(value_a);
    }

    with_attr error_message("value_b shouldn't be negative") {
        assert_nn(value_b);
    }

    with_attr error_message("value_a and value_b should be different") {
        assert_not_equal(value_a, value_b);
    }

    with_attr error_message("value_a should be <= 75") {
        assert_le(value_a, 75);
    }

    with_attr error_message("value_a should be between 40 and 69") {
        assert_in_range(value_a, 40, 70);
    }

    with_attr error_message("value_b should be < 1") {
        assert_lt(value_b, 1);
    }
    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass `value_a` and `value_b`, but you can not pass aleatory numbers, the function is doing a lot of comparisons with the number.

Check all the comparisons and make sure that both number pass in all of them, so you will be able to pass in `claim_points()`.

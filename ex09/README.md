## 1. What is this exercise?

It teaches you about:
Advanced Recursions lessons

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```

@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    array_len: felt, array: felt*
) {
    // Checking that the array is at least of length 4
    with_attr error_message("Expected an array of at least 4 elements but got {array_len}") {
        assert_le(4, array_len);
    }
    // Calculating the sum of the array sent by the user
    let (array_sum) = get_sum_internal(array_len, array);

    with_attr error_message("Expected the sum of the array to be at least 50") {
        // The sum should be higher than 50
        assert_le(50, array_sum);
    }
    // Reading caller address
    let (sender_address) = get_caller_address();
    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass an `array_len` and `array` values, with those values it checks:

- if the `array_len` is at least of length 4 (interesting to notice that it uses the imported `assert_le()` function to check that)
- if the sum of the array is higher than 50

To evaluate the sum of the array, it uses the `get_sum_internal()` to check that, but you need to have attention, this function also have other requirements:

```
func get_sum_internal{range_check_ptr}(length: felt, array: felt*) -> (sum: felt) {
    // This function is used recursively to calculate the sum of all the values in an array
    // Recursively, we first go through the length of the array
    // Once at the end of the array (length = 0), we start summing
    if (length == 0) {
        // Start with sum=0.
        return (sum=0);
    }

    // If length is NOT zero, then the function calls itself again, by moving forward one slot
    let (current_sum) = get_sum_internal(length=length - 1, array=array + 1);

    // This part of the function is first reached when length=0.
    // Checking that the first value in the array ([array]) is not 0
    with_attr error_message("First value of the array should not be 0") {
        assert_not_zero([array]);
    }
    // The sum begins
    let sum = [array] + current_sum;

    with_attr error_message(
            "value at index i should be at least the sum of values of index strictly higher than i") {
        assert_le(current_sum * 2, sum);
    }
    // The return function targets the body of this function
    return (sum,);
}

```

`get_sum_internal()` requires:

- length is not 0
- first value of the array is not 0
- value at index i should be at least the sum of the values of the index strictly higher than i

One very important detail that you need to note is that the `get_sum_internal()` only start to evaluate when the recursion fuction reaches 0, it implies in the order that the arguments are passed, take it into account.

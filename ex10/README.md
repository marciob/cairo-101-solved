## 1. What is this exercise?

It teaches you about:
Composability

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    secret_value_i_guess: felt, next_secret_value_i_chose: felt
) {
    // Reading caller address
    let (sender_address) = get_caller_address();

    // Retrieve secret value by READING
    let (ex10b_address) = ex10b_address_storage.read();
    let (secret_value) = Iex10b.secret_value(contract_address=ex10b_address);
    with_attr error_message("Input value is not the expected secret value") {
        assert secret_value = secret_value_i_guess;
    }

    // choosing next secret_value for contract 10b. We don't want 0, it's not funny
    with_attr error_message("Next secret value shouldn't be 0") {
        assert_not_zero(next_secret_value_i_chose);
    }

    with_attr error_message("Contract 10b error") {
        Iex10b.change_secret_value(
            contract_address=ex10b_address, new_secret_value=next_secret_value_i_chose
        );
    }

    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}
```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass an `secret_value_i_guess` and `next_secret_value_i_chose` values. It checks if:

- `next_secret_value_i_chose` is not 0
- `next_secret_value_i_chose` is the same as the secret_value returned from `Iex10b` contract

Then we need to know which value `Iex10b` contract returns. To do that we need to get the address of that contract to check there. It's possible by calling `ex10b_address()` view function.

Once checking the new contract on the blockchain explorer, you can see there how to find the value and how to set a new secret value, to pass both into the `claim_points()`.

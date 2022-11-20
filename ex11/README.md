## 1. What is this exercise?

It teaches you about:
Importing functions

## 2. How to pass?

You pass if you are able to call claim_points() without an error.

`claim_points()` entire code is:

```
@external
func claim_points{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    secret_value_i_guess: felt, next_secret_value_i_chose: felt
) {
    alloc_locals;
    // Reading caller address and setting it into a local to avoid revoked references
    let (local sender_address) = get_caller_address();
    // Check if your answer is correct
    validate_answers(sender_address, secret_value_i_guess, next_secret_value_i_chose);
    // Checking if the user has validated the exercise before
    validate_exercise(sender_address);
    // Sending points to the address specified as parameter
    distribute_points(sender_address, 2);
    return ();
}

```

## 3. How to call claim_points() successfully?

`claim_points()` is asking to pass an `secret_value_i_guess` and `next_secret_value_i_chose` values. It checks if:

- `secret_value_i_guess` is a valid answer by passing it into `validate_answers()` function

`validate_answers()` is an imported function from `ex11_base` contract:

```
from contracts.utils.ex11_base import (
    tderc20_address,
    has_validated_exercise,
    distribute_points,
    validate_exercise,
    ex_initializer,
    validate_answers,
    ex11_secret_value,
    secret_value
)
```

<!-- That `ex_initializer()` function from the imported contract, is being called at the moment of the deployment, in the constructor:

```
@constructor
func constructor{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    _tderc20_address: felt, _players_registry: felt, _workshop_id: felt, _exercise_id: felt
) {
    ex_initializer(_tderc20_address, _players_registry, _workshop_id, _exercise_id);
    return ();
}
``` -->

Checking in the `ex11_base` contract file (it's at the same repo, within utils folder), we can see the code for `validate_answers()` function:

```
func validate_answers{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}(
    sender_address: felt, secret_value_i_guess: felt, next_secret_value_i_chose: felt
) {
    // CAREFUL THERE IS A TRAP FOR PEOPLE WHO WON'T READ THE CODE
    // This exercise looks like the previous one, but actually the view secret_value returns a different value than secret_value
    // Sending the wrong execution result will remove some of your points, then validate the exercise. You won't be able to get those points back later on!
    alloc_locals;
    let (secret_value) = ex11_secret_value.read();
    local diff = secret_value_i_guess - secret_value;
    // Laying our trap here
    if (diff == 42069) {
        // Converting felt to uint256. We assume it's a small number
        // We also add the required number of decimals
        let points_to_remove: Uint256 = Uint256(2 * 1000000000000000000, 0);
        // # Retrieving contract address from storage
        let (contract_address) = tderc20_address_storage.read();
        // # Calling the ERC20 contract to distribute points
        ITDERC20.remove_points(
            contract_address=contract_address, to=sender_address, amount=points_to_remove
        );
        // This is necessary because of revoked references. Don't be scared, they won't stay around for too long...
        return ();
    } else {
        // If secret value is correct, set new secret value
        if (diff == 0) {
            with_attr error_message("Chosen value can't be 0") {
                assert_not_zero(next_secret_value_i_chose);
            }
            ex11_secret_value.write(next_secret_value_i_chose);
            // This is necessary because of revoked references. Don't be scared, they won't stay around for too long...
            return ();
        } else {
            assert 1 = 0;
            return ();
        }
    }
}
```

It's a private function, in this case called by our ex11 contract.

You need to pass a correct value that will make `diff` variable equal to 42069. There are some catches there, also read the warning about the possibility of get some points removed if you send a wrong transaction, so be aware and be cautious.

You can check the current `secret_value` by calling that function:

```
@view
func secret_value{syscall_ptr: felt*, pedersen_ptr: HashBuiltin*, range_check_ptr}() -> (
    secret_value: felt
) {
    let (secret_value) = ex11_secret_value.read();
    // There is a trick here
    return (secret_value + 42069,);
}
```

As you can see, the returned value isn't the pure secret_value, so do the correct calculation to identify it. Once you find the secret value, the `next_secret_value_i_chose` can be anything excepts 0, because of that assert in the `validate_answers()` function.

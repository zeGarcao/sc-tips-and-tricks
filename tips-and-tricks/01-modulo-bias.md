# Tips & Tricks - Modulo Bias

Did you know that the modulo operation can sometimes be biased? It’s important to consider this when using it to upper bound a random number, as it can produce a biased distribution. Let's break it down.

Imagine we are generating a pseudo-random number using the following formula:

```solidity
uint256 PRN = randomNumber % upperBound;
```

To simplify things, let's assume the following:

- `randomNumber` is an unsigned integer ranging from 0 to 9.
- `upperBound` is 3, so the resulting PRN will range from 0 to 2.

Based on the above assumptions, let's look at the resulting distribution:

- `PRN = 0 % 3 = 0`
- `PRN = 1 % 3 = 1`
- `PRN = 2 % 3 = 2`
- `PRN = 3 % 3 = 0`
- `PRN = 4 % 3 = 1`
- `PRN = 5 % 3 = 2`
- `PRN = 6 % 3 = 0`
- `PRN = 7 % 3 = 1`
- `PRN = 8 % 3 = 2`
- `PRN = 9 % 3 = 0`

By observing the results, we can see that `PRN = 0` appears one more time than `PRN = 1` and `PRN = 2`. This means that `PRN` is more likely to be `0` than `1` or `2`. Therefore, we can conclude that the modulo operation is biased in this case.

To remove the bias from the modulo operation, we can use the [UniformRandomNumber.sol](https://github.com/GenerationSoftware/uniform-random-number/blob/main/src/UniformRandomNumber.sol) library. Essentially, this library ensures uniform probabilities by discarding certain combinations and setting a minimum value for `randomNumber`. If `randomNumber` is below this minimum value, it computes a new hash and casts the value to `uint256` until a valid number is generated. Only then does it perform the modulo operation. In our example, the minimum value for `randomNumber` would be `1`, ensuring equal probabilities for `0`, `1`, and `2`.
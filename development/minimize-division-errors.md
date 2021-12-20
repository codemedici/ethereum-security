# Minimize Division Errors

{% hint style="success" %}
To minimize division errors, refactor your arithmetic to perform division last.
{% endhint %}

## Description

The output of an integer division operation is truncated to remove any non-zero remainder. Any arithmetic operations (other than addition and subtraction) that occur _after_ a division operation will exacerbate this truncation error.

To minimize this kind of error, refactor your arithmetic operations to perform division operations last. At a minimum, do not perform any multiplication or exponentiation after a division that may have non-zero truncation errors. Note however that ordering division last may increase the possibility of overflow.

## Example

In this example we compute `(21/5)*7` in two different ways. Note that the true value is `29.4`.

```solidity
import "@openzeppelin/contracts/math/SafeMath.sol";

contract Test {
    using SafeMath for uint256;

    function bigError() public pure returns (uint256) {
        return uint256(21).div(5).mul(7); // Returns 28. Total error: 1.4.
    }

    function smallError() public pure returns (uint256) {
        return uint256(21).mul(7).div(5); // Returns 29. Total error: 0.4.
    }

}
```

The `bigError()` function performs the division first. The resulting intermediate value of `uint256(21).div(5)` is `4`, which has a truncation error of `0.2` (because the real value of `21/5` is `4.2`). This error increases 7-fold with the proceeding multiplication, yielding a total error of `1.4`.

The `smallError()` function performs the division last, and results in a smaller error. This is the recommended approach.

## Resources

* [https://defender.openzeppelin.com/#/advisor/docs/minimize-division-errors?](https://defender.openzeppelin.com/#/advisor/docs/minimize-division-errors?)

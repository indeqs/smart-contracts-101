## Integer Overflow and Underflow
In solidity, integer types have maximum and minimum values. 

An integer overflow occurs when an integer exceeds the maximum value that can be stored in that integer type. 

Similarly, an integer underflow occurs when an integer goes below the minimum value for that integer type. 

Example: The maximum value `uint8` can store is `255`. When you store `256` in `uint8` it will overflow and the value will reset to 0. When you store `257`, the value will be `1`, `2` for `258` and so on.

Similarly, if you try to store `-1` in `uint8` the value becomes `255`, and so on as it will underflow.

Integer types and their min/max values:
| Type   |      Max      |  Min |
|----------|-------------|------|
| uint8 |  255 | 0 |
| uint16 | 65535 |   0 |
| uint24 | 16777215 | 0 |
| uint256 | 2^256 - 1 |  0 |

Since smaller integer types like: `uint8`, `uint16`, etc have smaller maximum values, it can be easier to cause an overflow, thus they should be used with greater caution.

To prevent over/underflows, [Safe Math Library](https://github.com/ConsenSysMesh/openzeppelin-solidity/blob/master/contracts/math/SafeMath.sol) is often used by contracts with older versions of Solidity.

Solidity >=v0.8 protects against integer overflows and underflows by default through the use of built-in safe math functions.

### Typecasting
The most common way in which integer over/underflow occurs is when you convert an integer of a larger type to a smaller type, e.g

```solidity
uint256 public a = 258;
uint8 public b = uint8(a); // typecasting uint256 to uint8
```

The above code snippet will overflow and `2` will end up being stored in the variable `b` due to the fact that maximum value of `uint8`  is `255`. So, it overflows and resets to `0` without reverting.
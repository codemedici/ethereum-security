# Use of var to declare variables

## Use of var to declare variables

## Use of var to declare variables

**`var`** is deprecated **since version** **`0.4.20`**, but is still be available in versions below 0.5.0 for compatibility. The issue with var is that the type of the variable is assumed from it's first assigned value.

This can cause a number of issues, as assigning 0 or any low value using var will result in an uint8, which will overflow if it exceeds 255. This causes an infinite loop in the case below:

 `for (var i = 0; i < 500; i++) {  
 // do something  
 }`

The issue would be resolved with `uint i`.


---
description: >-
  Test cases that check for error conditions, such as failed require statements,
  must not only assert that the action failed (typically a transaction revert),
  but also that the cause of failure is the e
---

# Assert Revert Reasons

## Description

A comprehensive test suite will check for expected error conditions, which might include arithmetic overflow protection or access control mechanisms. The outcome of these conditions being triggered will usually be a reverted transaction. Given how opaque and hard to debug the EVM can be, it is common for these tests to only check for generic errors that indicate a failed transaction.

This is a dangerous practice: merely asserting the existence of generic errors, such as runtime exceptions, can hide bugs in the test case that cause execution to fail \(and the test to pass\) for reasons unrelated to the conditions under test. A specialized assertion library like the [OpenZeppelin Test Helpers](https://docs.openzeppelin.com/test-helpers) should be used when writing tests of this kind.

This type of testing bugs can be avoided altogether by using the [`expectRevert`](https://docs.openzeppelin.com/test-helpers/api#expect-revert) helper function when testing `require` or `revert` statements, along with unique revert reasons. See the ['Include Revert Reasons'](https://defender.openzeppelin.com/development/include-revert-reasons.md) article for more details on why these are important.

## Example

The following test only checks for generic errors and will pass despite having an error: `transferFrom` takes 3 arguments instead of 2.

```text
// Inadequate code - do not use

it('transferFrom reverts when sender has insufficient balance', async () => {
  expect(() => await token.transferFrom(sender, recipient)).to.throw();
});
```

This can be improved by using [`expectRevert`](https://docs.openzeppelin.com/test-helpers/api#expect-revert) from the OpenZeppelin Test Helpers. However, the test still emits a false positive due to not checking the revert reason. The actual cause for revert is that the \(unspecified\) transaction sender does not have token allowance.

```text
// Inadequate code - do not use

const { expectRevert } = require('@openzeppelin/test-helpers');

it('transferFrom reverts when sender has insufficient balance', async () => {
  await expectRevert.unspecified(token.transferFrom(sender, recipient, amount));
});
```

Including the expected revert reason in the assertion eliminates all false positives, and ensures the test will only pass under the expected conditions.

```text
const { expectRevert } = require('@openzeppelin/test-helpers');

it('transferFrom reverts when sender has insufficient balance', async () => {
  await expectRevert(
    token.transferFrom(sender, recipient, amount, { from: approved }),
    'ERC20: insufficient balance'
  );
});
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/assert-revert-reasons?](https://defender.openzeppelin.com/#/advisor/docs/assert-revert-reasons?)


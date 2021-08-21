# Use the Offer-Accept Pattern for Transferring Admin Role

When transferring critical resources or rights, split the operation in two steps to prevent transfers to unintended accounts:

1. Offer: The grantor records which grantees are entitled to which resources.
2. Accept: The grantee/s claim the resources.

### Description

This pattern protects resources or rights transfer operations from accidental errors in grantee addresses, which may result \(for example\) in contracts that are no longer manageable because admin rights are transferred to the zero address or an address whose owner is not the intended one.

The basic idea behind this pattern is to request grantees of a permission to confirm that they want to receive the rights before confirming the transfer. The confirmation step is a proof that the grantee knows about the transfer and will be able to exercise the received rights.

Note this strategy _does not_ protect operations against an unintended but malicious grantee.

For bigger projects with more complex requirements around resource management, check out [OpenZeppelin's AccessControl contract](https://docs.openzeppelin.com/contracts/3.x/access-control#role-based-access-control), which provides a more comprehensive solution with much more granularity.

### Examples

The code below transfers ownership of a contract to any address, provided it is called by the current owner of the contract. Note that calling this function with the zero address or an address with a typo will result in ownership of the contract being locked forever.

```text
address private _owner;
event OwnershipTransferred(address indexed newOwner);

function transferOwnership(address newOwner) public virtual onlyOwner {
    emit OwnershipGranted(newOwner);
    _owner = newOwner;
}
```

Compare with the example below. This implementation makes it impossible to transfer ownership to the zero address because there is no way to confirm the transfer by calling `claimOwnership`. It also protects the operation from transfers to unowned addresses or accounts whose owners are unaware or uninterested in this contract. Remember, though, that this still allows an unintended grantee to claim the granted rights.

```text
address private _grantedOwner;
address private _owner;

event OwnershipGranted(address indexed grantedOwner);
event OwnershipTransferred(address indexed newOwner);

function grantOwnership(address newOwner) public virtual onlyOwner {
    emit OwnershipGranted(newOwner);
    _grantedOwner = newOwner;
}

function claimOwnership() public virtual {
    require(_grantedOwner == msg.sender, "Ownable: caller is not the granted owner");
    emit OwnershipTransferred(_owner, _grantedOwner);
    _owner = _grantedOwner;
    _grantedOwner = address(0);
}
```

## Resources

* [https://defender.openzeppelin.com/\#/advisor/docs/use-the-offer-accept-pattern-for-transferring-admin-role?](https://defender.openzeppelin.com/#/advisor/docs/use-the-offer-accept-pattern-for-transferring-admin-role?)


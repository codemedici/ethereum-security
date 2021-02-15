# Prevent contracts from executing

## Prevent contracts from executing

## Prevent contracts from executing

In some applications, there's a business requirement that other contracts aren't allowed to interact with your own contract and calls should be limited to externally owned accounts \(EOA\). When that happens there's a widely used function to check whether or not the msg.sender is an address:

function isContract\(address account\) internal view returns \(bool\) {  
    bytes32 codehash;  
    bytes32 accountHash = 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470;  
    assembly { codehash := extcodehash\(account\) }  
    return \(codehash != 0x0 && codehash != accountHash\);  
}

It's a mistake to rely on these kinds of checks, because the caller might be a constructor, in which case the function will return false, since code is stored on chain only at the end of the execution. Another aspect is that there's no guarantee that the passed address won't become a contract in the future.


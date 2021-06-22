# Uninitialized Storage pointers

### Relationships

[CWE-824: Access of Uninitialized Pointer](https://cwe.mitre.org/data/definitions/824.html)

### Description

Uninitialized local storage variables can point to unexpected storage locations in the contract, which can lead to intentional or unintentional vulnerabilities.

Local variables within functions default to storage or memory depending on their type. Uninitialized local storage variables may contain the value of other storage variables in the contract; this fact can cause unintentional vulnerabilities, or be exploited deliberately.

Solidity stores variables to storage or memory, depending on the type. Uninitialized storage pointers will, by default, point to the initial storage position \(0\) and can be used to alter the stored value.  
The EVM stores data either as storage or as memory. Understanding exactly how this is done and the default types for local variables of functions is highly recommended when developing contracts. This is because it is possible to produce vulnerable contracts by inappropriately intializing variables.  
To read more about storage and memory in the EVM, see the Solidity documentation on [data location](http://bit.ly/2OdUU0l), [layout of state variables in storage](https://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage), and [layout in memory](https://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-in-memory).  
Note: This section is based on an excellent [post by Stefan Beyer](http://bit.ly/2ERI0pb). Further reading on this topic, inspired by Stefan, can be found in this [Reddit thread](http://bit.ly/2OgxPtG).

## Real-world examples

Real-World Examples: OpenAddressLottery and CryptoRoulette Honey Pots

A honey pot named [OpenAddressLottery](http://bit.ly/2AAVnWD) was deployed that used this uninitialized storage variable quirk to collect ether from some would-be hackers. The contract is rather involved, so we will leave the analysis to the [Reddit thread](http://bit.ly/2OgxPtG) where the attack is quite clearly explained.

Another honey pot, [CryptoRoulette](http://bit.ly/2OfNGJ2), also utilized this trick to try and collect some ether. If you can’t figure out how the attack works, see “[An Analysis of a Couple Ethereum Honeypot Contracts](http://bit.ly/2OVkSL4)” for an overview of this contract and others.

### CryptoRoulette

#### crypto\_roulette.sol ^0.4.19

```text
/*
 * @source: https://github.com/thec00n/smart-contract-honeypots/blob/master/CryptoRoulette.sol
 */
pragma solidity ^0.4.19;

// CryptoRoulette
//
// Guess the number secretly stored in the blockchain and win the whole contract balance!
// A new number is randomly chosen after each try.
//
// To play, call the play() method with the guessed number (1-20).  Bet price: 0.1 ether

contract CryptoRoulette {

    uint256 private secretNumber;
    uint256 public lastPlayed;
    uint256 public betPrice = 0.1 ether;
    address public ownerAddr;

    struct Game {
        address player;
        uint256 number;
    }
    Game[] public gamesPlayed;

    function CryptoRoulette() public {
        ownerAddr = msg.sender;
        shuffle();
    }

    function shuffle() internal {
        // randomly set secretNumber with a value between 1 and 20
        secretNumber = uint8(sha3(now, block.blockhash(block.number-1))) % 20 + 1;
    }

    function play(uint256 number) payable public {
        require(msg.value >= betPrice && number <= 10);

        Game game;
        game.player = msg.sender;
        game.number = number;
        gamesPlayed.push(game);

        if (number == secretNumber) {
            // win!
            msg.sender.transfer(this.balance);
        }

        shuffle();
        lastPlayed = now;
    }

    function kill() public {
        if (msg.sender == ownerAddr && now > lastPlayed + 1 days) {
            suicide(msg.sender);
        }
    }

    function() public payable { }
}
```

#### crypto\_roulette.sol ^0.4.23

All variables declared in a function default to a storage pointer when uninitialized, and can be used to manipulate variables in an obscure way. Storage pointers are often used in honeypot contracts like this one:

```text
pragma solidity ^0.4.23;
// CryptoRoulette
//
// Guess the number secretly stored in the blockchain and win the whole contract balance!
// A new number is randomly chosen after each try.
//
// To play, call the play() method with the guessed number (1-16). Bet price: 0.2 ether
contract CryptoRoulette {
    uint256 public secretNumber;
    uint256 public lastPlayed;...
    uint256 public betPrice = 0.001 ether;
    address public ownerAddr;
    struct Game {
        address player;
        uint256 number;
    }
   Game[] public gamesPlayed;
Above the variables from the contract which suggest a game. Look at secretNumber, and memorize for a bit that it is the first variable being declared.
    constructor() public {
        ownerAddr = msg.sender;
        shuffle();
    }
    function shuffle() internal {
        // randomly set secretNumber with a value between 1 and 10
        secretNumber = 6;
    }
```

Here things start to get interesting. The game's shuffle function seems to have been mistakenly hardwired to return 6. This will catch the attention of a reader, and makes this contract seem easily exploitable. A contract like this with a large balance is tempting, the very definition of a honeypot.    

```text
function play(uint256 number) payable public {
    require(msg.value >= betPrice && number <= 10);
    Game game;
    game.player = msg.sender;
    game.number = number;
    gamesPlayed.push(game);
    if (number == secretNumber) {
        // win!
        msg.sender.transfer(this.balance);
    }
    shuffle();
    lastPlayed = now;
}
```

Now, before you continue, take another look at the section above. This is where the exploit resides. It exploits in a way that will not be clear to casual readers, by design. The game storage pointer \(right after require\) is not initialized. In solidity, uninitialized storage pointers will, by default point to the first storage slot. As you probably guessed by now, the secretNumber variable is there.  
In Solidity, the variables are laid out in the storage slots in the order they are declared, so msg.sender will be written there causing a casual reader to lose what they thought was a certain bet.  
Then, `shuffle()` is called again, resetting the secret number for the next victim.    

```text
function kill() public {
        if (msg.sender == ownerAddr && now > lastPlayed + 6 hours) {
            selfdestruct(msg.sender);
        }
    }
    function() public payable { }
}
```

And as if to put a cherry on top, there is a function to allow the owner to withdraw their revenue.  
This issue is hard to spot, and often used in underhanded code. They can \(intentionally or not\) cause variables used for other functionality in the contract to be altered, in a way that is not obvious for the casual reviewer.

This issue is no longer applicable in solidity `0.5.0` and above, but can happen with previous compilers.

#### crypto\_roulette\_fixed.sol

```text
12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061/*
 * @source: https://github.com/thec00n/smart-contract-honeypots/blob/master/CryptoRoulette.sol
 */
pragma solidity ^0.4.19;

// CryptoRoulette
//
// Guess the number secretly stored in the blockchain and win the whole contract balance!
// A new number is randomly chosen after each try.
//
// To play, call the play() method with the guessed number (1-20).  Bet price: 0.1 ether

contract CryptoRoulette {

    uint256 private secretNumber;
    uint256 public lastPlayed;
    uint256 public betPrice = 0.1 ether;
    address public ownerAddr;

    struct Game {
        address player;
        uint256 number;
    }
    Game[] public gamesPlayed;

    function CryptoRoulette() public {
        ownerAddr = msg.sender;
        shuffle();
    }

    function shuffle() internal {
        // randomly set secretNumber with a value between 1 and 20
        secretNumber = uint8(sha3(now, block.blockhash(block.number-1))) % 20 + 1;
    }

    function play(uint256 number) payable public {
        require(msg.value >= betPrice && number <= 10);

        Game memory game;
        game.player = msg.sender;
        game.number = number;
        gamesPlayed.push(game);

        if (number == secretNumber) {
            // win!
            msg.sender.transfer(this.balance);
        }

        shuffle();
        lastPlayed = now;
    }

    function kill() public {
        if (msg.sender == ownerAddr && now > lastPlayed + 1 days) {
            suicide(msg.sender);
        }
    }

    function() public payable { }
}

```

### NameRegistrar

[https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc\#nameregistrar\_security](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#nameregistrar_security)

Let’s consider the relatively simple name registrar contract in [NameRegistrar.sol](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#nameregistrar_security).

```text
// A locked name registrar
contract NameRegistrar {
    bool public unlocked = false;  // registrar locked, no name updates
    struct NameRecord { // map hashes to addresses
        bytes32 name;
        address mappedAddress;
    }
    // records who registered names
    mapping(address => NameRecord) public registeredNameRecord;
    // resolves hashes to addresses
    mapping(bytes32 => address) public resolve;
    function register(bytes32 _name, address _mappedAddress) public {
        // set up the new NameRecord
        NameRecord newRecord;
        newRecord.name = _name;
        newRecord.mappedAddress = _mappedAddress;
        resolve[_name] = _mappedAddress;
        registeredNameRecord[msg.sender] = newRecord;
        require(unlocked); // only allow registrations if contract is unlocked
    }
}
```

This simple name registrar has only one function. When the contract is unlocked, it allows anyone to register a name \(as a bytes32 hash\) and map that name to an address. The registrar is initially locked, and the require on line 25 prevents register from adding name records. It seems that the contract is unusable, as there is no way to unlock the registry! There is, however, a vulnerability that allows name registration regardless of the unlocked variable.

To discuss this vulnerability, first we need to understand how storage works in Solidity. As a high-level overview \(without any proper technical detail—we suggest reading the Solidity docs for a proper review\), state variables are stored sequentially in slots as they appear in the contract \(they can be grouped together but aren’t in this example, so we won’t worry about that\). Thus, unlocked exists in slot\[0\], registeredNameRecord in slot\[1\], and resolve in slot\[2\], etc. Each of these slots is 32 bytes in size \(there are added complexities with mappings, which we’ll ignore for now\). The Boolean unlocked will look like 0x000…​0 \(64 0s, excluding the 0x\) for false or 0x000…​1 \(63 0s\) for true. As you can see, there is a significant waste of storage in this particular example.

The next piece of the puzzle is that Solidity by default puts complex data types, such as structs, in storage when initializing them as local variables. Therefore, newRecord on line 18 defaults to storage. The vulnerability is caused by the fact that `newRecord` is not initialized. Because it defaults to storage, it is mapped to storage slot\[0\], which currently contains a pointer to unlocked. Notice that on lines 19 and 20 we then set newRecord.name to \_name and newRecord.mappedAddress to \_mappedAddress; this updates the storage locations of slot\[0\] and slot\[1\], which modifies both unlocked and the storage slot associated with registeredNameRecord.

This means that unlocked can be directly modified, simply by the bytes32 \_name parameter of the register function. Therefore, if the last byte of \_name is nonzero, it will modify the last byte of storage slot\[0\] and directly change unlocked to true. Such \_name values will cause the require call on line 25 to succeed, as we have set unlocked to true. Try this in Remix. Note the function will pass if you use a \_name of the form:  
`0x000000000000000000000000000000000000000000000000000000000000000`

### **OpenAddressLottery**

**TODO**

## Preventative Techniques

The Solidity compiler shows a warning for uninitialized storage variables; developers should pay careful attention to these warnings when building smart contracts. The current version of Mist \(0.10\) doesn’t allow these contracts to be compiled. It is often good practice to explicitly use the memory or storage specifiers when dealing with complex types, to ensure they behave as expected.

Check if the contract requires a storage object as in many situations this is actually not the case. If a local variable is sufficient, mark the storage location of the variable explicitly with the `memory` attribute. If a storage variable is needed then initialize it upon declaration and additionally specify the storage location `storage`.

**Note**: As of compiler version 0.5.0 and higher this issue has been systematically resolved as contracts with uninitialized storage pointers do no longer compile.

## Resources

* [SigmaPrime - Uninitialised Storage Pointers](https://github.com/sigp/solidity-security-blog#unintialised-storage-pointers-1)
* [https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc\#uninitialized-storage-pointers](https://github.com/ethereumbook/ethereumbook/blob/develop/09smart-contracts-security.asciidoc#uninitialized-storage-pointers)
* [https://swcregistry.io/docs/SWC-109](https://swcregistry.io/docs/SWC-109)
* [https://solidity.readthedocs.io/en/latest/miscellaneous.html\#layout-of-state-variables-in-storage](https://solidity.readthedocs.io/en/latest/miscellaneous.html#layout-of-state-variables-in-storage) [https://blog.b9lab.com/storage-pointers-in-solidity-7dcfaa536089](https://blog.b9lab.com/storage-pointers-in-solidity-7dcfaa536089)
* Consensys Smart Contract Best Practices
* Solidity Documentation security considerations
* EthereumMagicians security ring
* Open Zeppelin's security forum
* Martin Swende Blog
* Storage Pointers in Solidity


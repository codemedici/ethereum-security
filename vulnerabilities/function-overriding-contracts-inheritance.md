# Function Overriding \(Contracts Inheritance\)

## Contracts Inheritance

Contracts can inherit from other contracts. This is a widely used functionality, and although it helps organize code, it may help to hide problems with variables and functions. Built-in functions can be overridden, as the example below shows.

contract Shadow {  
    function require\(bool parameter\) {  
        // never reverts  
    }  
}

contract Safe is Shadow {  
    address public owner;

    function Safe\(\) public payable {  
        owner = msg.sender;  
    }

    function \(\) public payable {}

    function withdraw\(\) public {  
        require\(owner == msg.sender\);  
    }  
}

The inheritance graph can also mislead developers. If you have multiple inheritance, Solidity will inherit one by one starting from the right. Best practice is to linearize the structure to prevent unwanted behavior.


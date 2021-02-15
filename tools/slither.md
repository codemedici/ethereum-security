# Slither

Slither  
Slither, by Trail of Bits, is a static analyzer that performs amazingly. It's blazingly fast, static analyzers usually are, and will get all the low-hanging fruits for you in a second. Let's try it:

pip3 install slither-analyzer  
slither UselessEthereumToken.sol  
No time for coffee this time. You will notice it didn't bring the alert Mythril did, but it did bring lots of alerts, for older compiler versions, functions that could be external, coding style, and more.

Some of these issues are false positives, such as the strict equality one - it's checking for an uninitialized variable, strictly - so you have to check one by one, do not report an issue based on tool reports only.

Although screening through false positives can get tedious, tools are an effective way of harvesting all low-hanging fruit in the audit. They will help you not need to worry about reporting an older version of Solidity, and focus on business logic errors, and bugs that these tools simply can't find. Though both tools gave different results \(complementing each other nicely\) the more tools you add to your toolbox, the less value you get \(against the time you will spend\), so using a balanced set is advisable.

Use the tools you know and trust, but try a new one every now and then.

You can see the contracts and resulting analyses in this shared repository.


---
description: Step-by-step instructions for a setting up a React/Redux DApp
---

# DApp Starter Kit

### Requirements

Node.js ^9.10.0 % node -v v9.10.0

Python ^2.7.0 % python -V Python 2.7.16

Ganache

Truffle@5.0.5 npm i -g truffle@5.0.5

node-gyp@3.6.2 % npm i -g node-gyp@3.6.2

* node-gyp@3.6.2

create-react-app@2.1.3 % npm i -g create-react-app@2.1.3

xcode tools On Mac: % xcode-select --install

Git

Tools

• Metamask • Infura • Redux DevTools addon • Heroku

Create the Project

% create-react-app dapp-starter-kit

Push to github % git remote add origin [https://github.com/codemedici/dapp-starter-kit.git](https://github.com/codemedici/dapp-starter-kit.git) % git push -u origin master

Merge with package.json

```text
"dependencies": {
    "babel-polyfill": "6.26.0",
    "babel-preset-env": "1.7.0",
    "babel-preset-es2015": "6.24.1",
    "babel-preset-stage-2": "6.24.1",
    "babel-preset-stage-3": "6.24.1",
    "babel-register": "6.26.0",
    "bootstrap": "4.3.1",
    "dotenv": "6.2.0",
    "chai": "4.2.0",
    "chai-as-promised": "7.1.1",
    "chai-bignumber": "3.0.0",
    "react": "16.8.4",
    "react-bootstrap": "1.0.0-beta.5",
    "react-dom": "16.8.4",
    "react-scripts": "2.1.3",
    "truffle": "5.0.5",
    "web3": "1.0.0-beta.55"
  },
```

Install modules % npm i

Initialize truffle  
% truffle init

change truffle-config.js

```text
require('babel-register');
require('babel-polyfill');

module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*" // Match any network id
    },
  },
  contracts_directory: './src/contracts/',
  contracts_build_directory: './src/abis/',
  compilers: {
    solc: {
      optimizer: {
        enabled: true,
        runs: 200
      }
    }
  }
}
```

create .babelrc

```text
{
    "presets": ["es2015","stage-2","stage-3"]
}
```

create .env \(empty for now\)

add .env to .gitignore

import .env into truffle-config.js : require\('dotenv'\).config\(\);

Move contracts directory \(create react app doesn't read outside the src directory, i.e. abis need to be inside src\): contracts\_directory: './src/contracts' dapp-starter-kit % mv contracts src

Create 2\_deploy\_contracts.js in migrations/ Example:

```text
const Token = artifacts.require("Token");
const Exchange = artifacts.require("Exchange");

module.exports = async function(deployer) {
  const accounts = await web3.eth.getAccounts()

  await deployer.deploy(Token);

  const feeAccount = accounts[0]
  const feePercent = 10

  await deployer.deploy(Exchange, feeAccount, feePercent)
};
```


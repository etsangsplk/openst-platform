## OpenST Platform development installation steps

Document has steps to configure OpenST platform for development and test environments. 
For production environment, following steps are not recommended to avoid single point 
failures and scalability issues caused because of single machine.

You can test platform as standalone system or with Platform RESTful APIs. We have publish 
this document for both kind of setups.

## Installation prerequisite 

* Install node version >= 7
* Install geth version >= 1.7.2 

## Choose the platform flavour
     
#### * Platform as standalone system

* Checkout platform code from repository

```bash
  > git clone git@github.com:OpenSTFoundation/openst-platform.git
  > cd openst-platform
  > export OPENST_PLATFORM_PATH=$(pwd)
  > echo "export OPENST_PLATFORM_PATH=$(pwd)" >> ~/.bash_profile
```

#### * Platform with Platform RESTful APIs

* Checkout RESTful APIs code from repository

```bash
  > git clone git@github.com:OpenSTFoundation/openst-platform-apis.git
  > cd openst-platform-apis
  > export OPENST_PLATFORM_PATH=$(pwd)/node_modules/@openstfoundation/openst-platform
  > echo "export OPENST_PLATFORM_PATH=$(pwd)/node_modules/@openstfoundation/openst-platform" >> ~/.bash_profile
```

## Start OpenST Platform Setup

* <b>Install all required node modules</b>

```bash
  > npm install
```

* <b>Start the openST platform setup</b>

```bash
  > node $OPENST_PLATFORM_PATH/tools/setup/index.js
```

Setup will create "openst-setup" folder in your HOME folder with following folders and files:

1. openst-geth-value - Acts as ethereum MainNet for development/test environment using POW consensus algorithm.
2. openst-geth-utility - Acts as openST side chain network for development/test environment using POA consensus algorithm.  
3. bin - Contain multiple executables for ethereum chains and platform services
4. logs - Contain logs generated by executables in bin folder
5. openst_env_vars.sh - Platform related environment variables 

```bash
  > ls -alt $HOME/openst-setup/
```

## (Optional) Configure [Cache](https://github.com/OpenSTFoundation/openst-cache) and [Notification](https://github.com/OpenSTFoundation/openst-notification) layers in setup
The default Platform setup is done with "in-process" caching and EventEmitter notifications. To use different caching and notifications implementation, edit $HOME/openst-setup/openst_env_vars.sh file

```bash
  > cat $HOME/openst-setup/openst_env_vars.sh
```

## Register and Mint branded tokens on Platform

### [x] On Terminal 1 - Start all the required services
* <b>Load platform environment variables</b>

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
```

* <b>Start all platform services in background</b>   
```bash
  > node $OPENST_PLATFORM_PATH/tools/setup/start_services.js
```
Important Note: Wait until all service are up and running. A success message will be displayed when everything is good to go.
Let this script be running while branded tokens are registered and minted.

Note: Script also monitor these services and alert if any required service terminates.

* <b>Optional steps on separate Terminals</b>
  - Listen notifications published from platform over RabbitMQ
  ```bash
    > source $HOME/openst-setup/openst_env_vars.sh
    > node $OPENST_PLATFORM_PATH/executables/notification_subscribe.js
  ```
  
  - All logs created by different services are present in logs folder
  
  ```bash
    > ls -alt $HOME/openst-setup/logs/
  ```

### [x] On Terminal 2 - Once all required services are up and running, let's onboard our first branded token

* <b>Load platform environment variables</b>  

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
```

* <b>Onboard/Register Branded Token</b> - Registration requires three input parameters:
1. Name - branded token name (example: "ACME Coin")
2. Symbol - branded token symbol (example: "ACME")
3. Conversion Factor - branded token to OST conversion factor, 1 OST = x BT (example: 10).
   - This is a number and has a precision of <b>5</b>. 
   - This cannot be <b>0</b>
   - Valid examples: 
     ``` 1.0 ``` , ``` 0.222 ``` , ``` .3 ``` , ``` 1000 ``` , ``` 15.001 ``` 
   - Invalid examples:
      ``` 2.002222 ``` , ``` 0 ``` ,  ``` xyz ``` 

```bash
  > node $OPENST_PLATFORM_PATH/tools/setup/branded_token/register.js "ACME Coin" "ACME" 10
```

NOTE: Upon successful registration, branded token details will be published in the branded token configuration file.

```bash
  > cat $HOME/openst-setup/branded_tokens.json
  
  {
    "0x9b8f63ed597ca654262e21647d59f5ef495d173909d7816982d367b85f5ebc76": {
      "Name": "ACME Coin",
      "Symbol": "ACME",
      "ConversionFactor": 10,
      "Reserve": "0xEB05083DE29860b912151d93DB24C55b7beB6936", // Branded Token owner address on utility chain
      "ReservePassphrase": "acmeOnopenST",
      "UUID": "0x9b8f63ed597ca654262e21647d59f5ef495d173909d7816982d367b85f5ebc76",
      "ERC20": "0x3B662406CCab34fd2Ce81Bf7154987DDCE82F6EF" // Branded Token EIP20 contract address
    }
  }
```

* <b>Mint branded tokens and get ST' (gas) on utility chain by staking OST</b> - Minting requires 2 input arguments:
1. Symbol - branded token symbol (example: "ACME")
2. Amount - The OST amount in Weis to stake, where 1 OST = (1 X 10^18) OST Wei (example: 500 OST = 500000000000000000000 OST Wei)

```bash
  > node $OPENST_PLATFORM_PATH/tools/setup/branded_token/mint.js "ACME" 500000000000000000000
```

NOTE: Upon successful minting, Branded Token reserve address will receive branded tokens, of worth 
90% of staked OST, and ST' (gas for OpenST utility chain), of worth 10% of staked OST.  

Example: For 500 OST, reserve address will get: 

- 4500 ACME tokens ((90% of 500 staked OST) * (10 as conversion factor))
- 50 ST' ((10% of 500 staked OST) * (1 as conversion factor))
     

### [x] Back on Terminal 1 - Stop start_services.js script, if you don't want to register and mint more branded tokens on utility chain.

## Start using branded tokens on utility chain

### For Platform RESTful APIs

* <b>Start utility chain in new terminal</b>

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
  > sh $HOME/openst-setup/bin/run-utility.sh
```

* <b>Start value chain in new terminal</b>

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
  > sh $HOME/openst-setup/bin/run-value.sh
```

* <b>Start application server in new terminal</b>

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
  > node app.js
```
* <b>Use PostMan files for Platform RESTful Apis testing and reference: https://github.com/OpenSTFoundation/openst-platform-apis/tree/master/postman</b>

### For Standalone System

* <b>Start utility chain in new terminal</b>

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
  > sh $HOME/openst-setup/bin/run-utility.sh
```

* <b>Start value chain in new terminal</b>

```bash
  > source $HOME/openst-setup/openst_env_vars.sh
  > sh $HOME/openst-setup/bin/run-value.sh
```

* <b>Open node console in new terminal</b>
```bash
  > source $HOME/openst-setup/openst_env_vars.sh
  > node
```

* Generate new address on utility chain
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.utils.generateAddress({passphrase: 'my-secret-pass', chain: 'utility'});
serviceObj.perform().then(function(response) { 
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Get branded token details
```bash
var os = require('os');
var brandedTokenConfig = require(os.homedir() + "/openst-setup/branded_tokens.json");
var uuid = Object.keys(brandedTokenConfig)[0];
var platformServices = require('./index');
var serviceObj = new platformServices.services.utils.getBrandedTokenDetails({uuid: uuid});
serviceObj.perform().then(function(response) { 
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Transfer branded token on utility chain
```bash
var os = require('os');
var brandedTokenConfig = require(os.homedir() + "/openst-setup/branded_tokens.json");
var brandedTokenDetails = brandedTokenConfig[Object.keys(brandedTokenConfig)[0]];
var data = {
  erc20_address: brandedTokenDetails['ERC20'],
  sender_address: brandedTokenDetails['Reserve'],
  sender_passphrase: brandedTokenDetails['ReservePassphrase'],
  recipient_name: 'foundation',
  amount_in_wei: 2,
  options: {
    returnType: 'txReceipt',
    tag: 'ILoveOST'
  }
};
var platformServices = require('./index');
var serviceObj = new platformServices.services.transaction.transfer.brandedToken(data);
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Transfer OST on value chain
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.transaction.transfer.simpleToken({sender_name: 'foundation', recipient_name: 'utilityChainOwner', amount_in_wei: 10, options: {returnType: 'txHash', tag: 'Grant'}});
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Transfer Ether on value chain
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.transaction.transfer.eth({sender_name: 'foundation', recipient_name: 'utilityChainOwner', amount_in_wei: 10000, options: {returnType: 'txHash', tag: 'GasRefill'}});
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Transfer ST' (gas) on utility chain
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.transaction.transfer.simpleTokenPrime({sender_name: 'utilityChainOwner', recipient_name: 'foundation', amount_in_wei: 10, options: {returnType: 'txHash', tag: 'GasRefill'}});
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Get transaction receipt
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.transaction.getReceipt({chain: 'utility', transaction_hash: '0xe4945b1c90d291074b74c9ed211c6fbae2702d1bd33e7b53c3f55a6b3c62c270'});
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Get branded token balance
```bash
var os = require('os');
var brandedTokenConfig = require(os.homedir() + "/openst-setup/branded_tokens.json");
var brandedTokenDetails = brandedTokenConfig[Object.keys(brandedTokenConfig)[0]];
var data = {address: brandedTokenDetails['Reserve'], erc20_address: brandedTokenDetails['ERC20']};
var platformServices = require('./index');
var serviceObj = new platformServices.services.balance.brandedToken(data);
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Get OST balance
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.balance.simpleToken({address: process.env.OST_FOUNDATION_ADDR});
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

* Get ST' (gas) balance
```bash
var platformServices = require('./index');
var serviceObj = new platformServices.services.balance.simpleTokenPrime({address: process.env.OST_UTILITY_CHAIN_OWNER_ADDR});
serviceObj.perform().then(function(response) {
  if (response.isSuccess()){
    console.log(response.data);
  } else {
    console.log(response.err)
  } 
});
```

For complete implementation details of OpenST Platform, please refer [API documentation](http://docs.openst.org/).

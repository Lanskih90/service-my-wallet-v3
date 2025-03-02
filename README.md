
# Blockchain Wallet API V2

Programmatically interface with your Blockchain.info wallet.

## Contents

  * [Getting Started](#getting-started)
  * [Upgrading](#upgrading)
  * [API Documentation](#api-documentation)
  * [Installation](#installation)
  * [Troubleshooting](#troubleshooting)
  * [Usage](#usage)
  * [Development](#development)
  * [Deployment](#deployment)

## Getting Started

To use this API, you will need to run small local service which be responsible for managing your Blockchain.info wallet. Your application interacts with this service locally via HTTP API calls.

Start by completing the following steps:

  1. Follow the [installation instructions](#installation)
  2. Start the server: `$ blockchain-wallet-service start --port 3000`
  3. Reference the [documentation](#api-documentation) and start interacting with your wallet programmatically!

Note that `blockchain-wallet-service` is designed to be run locally on the same machine as your application and therefore will only accept connections from `localhost`. If you modify this service to accept external connections, be sure to add the appropriate firewall rules to prevent unauthorized use.

## Upgrading

If you already have an application that uses [Blockchain.info's Wallet API](https://blockchain.info/api/blockchain_wallet_api), you will need to complete the steps in the Getting Started section above and then complete the following:

  1. In your application code, replace calls to `blockchain.info/merchant/...` with `localhost:<port>/merchant/...`
  2. Add an initial call, during your application's initialization, that calls [`/login`](#logging-into-a-wallet) with your wallet GUID and password. This call must be made before programmatically accessing your wallet. If your application interacts with more than one wallet, you must call the /login endpoint to switch between wallets. Subsequent calls made to any endpoint will use the last logged-in wallet.

## API Documentation

View the [original documentation](https://blockchain.info/api/blockchain_wallet_api).

All endpoints present in the API documentation above are supported in Blockchain Wallet API V2. The differences between two are:

  1. The `/login` endpoint that must be called prior to accessing a wallet
  2. The "consolidate addresses" endpoint has been omitted

All endpoints can be called with `GET` or `POST`, and can only be accessed from `localhost`.

### Logging into a Wallet

Loads a blockchain.info wallet. A wallet must be loaded via this endpoint before any other api interactions can occur.

Endpoint: `/merchant/:guid/login`

Query Parameters:

  * `password` - main wallet password (required)
  * `api_code` - blockchain.info wallet api code (required)

Get an API code [here](https://blockchain.info/api/api_create_code). **Note**: You must check the "Create Wallets" checkbox under "Permissions" when requesting an API code in order for it to be compatible with this app.

The `api_code` parameter is only required for the call to `/login`. Subsequent API calls for this wallet will not require the api code.

Note: at the moment, only one wallet can be "logged into" at a time. To make api calls to different wallets, run separate instances of this service for each wallet, or just remember to call `/login` each time you want to switch wallets.

### Make Payment

Endpoint: `/merchant/:guid/payment`

Query Parameters:

  * `to` - bitcoin address to send to (required)
  * `amount` - amount **in satoshi** to send (required)
  * `password` - main wallet password (required)
  * `second_password` - second wallet password (required, only if second password is enabled)
  * `from` - bitcoin address or account index to send from (optional)
  * `fee` - specify transaction fee **in satoshi** (optional, otherwise fee is computed)
  * `note` - public note to include with the transaction (optional, limit 255 characters)

Sample Response:

```json
{
  "to" : ["1A8JiWcwvpY7tAopUkSnGuEYHmzGYfZPiq"],
  "from": ["17p49XUC2fw4Fn53WjZqYAm4APKqhNPEkY"],
  "amounts": [200000],
  "fee": 1000,
  "txid": "f322d01ad784e5deeb25464a5781c3b20971c1863679ca506e702e3e33c18e9c",
  "success": true
}
```

### Send to Many

Endpoint: `/merchant/:guid/sendmany`

Query Parameters:

  * `recipients` - a *URI encoded* [JSON object](http://json.org/example.html), with bitcoin addresses as keys and the **satoshi** amounts as values (required, see example below)
  * `password` - main wallet password (required)
  * `second_password` - second wallet password (required, only if second password is enabled)
  * `from` - bitcoin address or account index to send from (optional)
  * `fee` - specify transaction fee **in satoshi** (optional, otherwise fee is computed)
  * `note` - public note to include with the transaction (optional, limit 255 characters)

URI Encoding a JSON object in JavaScript:

```js
var myObject = { address1: 10000, address2: 50000 };
var myJSONString = JSON.stringify(myObject);
// `encodeURIComponent` is a global function
var myURIEncodedJSONString = encodeURIComponent(myJSONString);
// use `myURIEncodedJSONString` as the `recipients` parameter
```

Sample Response:

```json
{
  "to" : ["1A8JiWcwvpY7tAopUkSnGuEYHmzGYfZPiq", "18fyqiZzndTxdVo7g9ouRogB4uFj86JJiy"],
  "from": ["17p49XUC2fw4Fn53WjZqYAm4APKqhNPEkY"],
  "amounts": [16000, 5400030],
  "fee": 2000,
  "txid": "f322d01ad784e5deeb25464a5781c3b20971c1863679ca506e702e3e33c18e9c",
  "success": true
}
```

### Fetch Wallet Balance

Endpoint: `/merchant/:guid/balance`

Query Parameters:

  * `password` - main wallet password (required)

Sample Response:

```json
{ "balance": 10000 }
```

### List Addresses

Endpoint: `/merchant/:guid/list`

Query Parameters:

  * `password` - main wallet password (required)

Sample Response:

```json
{
  "addresses": [
    {
        "balance": 79434360,
        "address": "1A8JiWcwvpY7tAopUkSnGuEYHmzGYfZPiq",
        "label": "My Wallet",
        "total_received": 453300048335
    },
    {
        "balance": 0,
        "address": "17p49XUC2fw4Fn53WjZqYAm4APKqhNPEkY",
        "total_received": 0
    }
  ]
}
```

### Fetch Address Balance

Endpoint: `/merchant/:guid/address_balance`

Query Parameters:

  * `address` - address to fetch balance for (required)
  * `password` - main wallet password (required)

Note: unlike the hosted API, there is no option of a `confirmations` parameter for specifying minimum confirmations.

Sample Response:

```json
{ "balance": 129043, "address": "19r7jAbPDtfTKQ9VJpvDzFFxCjUJFKesVZ", "total_received": 53645423 }
```

### Generate Address

Endpoint: `/merchant/:guid/new_address`

Query Parameters:

  * `password` - main wallet password (required)
  * `label` - label to give to the address (optional)

Sample Response:

```json
{ "address" : "18fyqiZzndTxdVo7g9ouRogB4uFj86JJiy" , "label":  "My New Address" }
```

### Archive Address

Endpoint: `/merchant/:guid/archive_address`

Query Parameters:

  * `address` - address to archive (required)
  * `password` - main wallet password (required)

Sample Response:

```json
{ "archived" : "18fyqiZzndTxdVo7g9ouRogB4uFj86JJiy" }
```

### Unarchive Address

Endpoint: `/merchant/:guid/unarchive_address`

Query Parameters:

  * `address` - address to unarchive (required)
  * `password` - main wallet password (required)

Sample Response:

```json
{ "active" : "18fyqiZzndTxdVo7g9ouRogB4uFj86JJiy" }
```

### Enable HD Functionality

Endpoint: `/merchant/:guid/enableHD`

This will upgrade a wallet to an HD (Hierarchical Deterministic) Wallet, which allows the use of accounts. See [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) for more information on HD wallets and accounts.

### List Active HD Accounts

Endpoint: `/merchant/:guid/accounts`

### List HD xPubs

Endpoint: `/merchant/:guid/accounts/xpubs`

### Create New HD Account

Endpoint: `/merchant/:guid/accounts/create`

Query Parameters (optional):

  * `label` - label to assign to the newly created account

### Get Single HD Account

Endpoint: `/merchant/:guid/accounts/:xpub_or_index`

### Get HD Account Receiving Address

Endpoint: `/merchant/:guid/accounts/:xpub_or_index/receiveAddress`

### Check HD Account Balance

Endpoint: `/merchant/:guid/accounts/:xpub_or_index/balance`

### Archive HD Account

Endpoint: `/merchant/:guid/accounts/:xpub_or_index/archive`

### Unarchive HD Account

Endpoint: `/merchant/:guid/accounts/:xpub_or_index/unarchive`

## Installation

[`npm`](https://npmjs.com) is required for installing this API service. Installation:

```sh
$ npm install -g blockchain-wallet-service
```

If you have issues with the installation process, see the troubleshooting section below.

## Troubleshooting

If installation fails:

  * If you are getting `EACCESS` or permissions-related errors, it might be necessary to run the install as root, using the `sudo` command.

Runtime errors:

  * If you are getting wallet decryption errors despite having correct credentials, then it's possible that you do not have Java installed, which is required by a dependency of the my-wallet-v3 module. Not having Java installed during the `npm install` process can result in the inability to decrypt wallets. Download the JDK from [here for Mac](https://support.apple.com/kb/DL1572) or by running `apt-get install default-jdk` on debian-based linux systems.

If this section did not help, please open a github issue or visit our [support center](https://blockchain.zendesk.com).

## Usage

After installing the service, the command `blockchain-wallet-service` will be available for use.

### Options

  * `-h, --help` - output usage information
  * `-V, --version` - output the version number
  * `-c, --cwd` - use the current directory as the wallet service module (development only)

### Commands

#### start

Usage: `blockchain-wallet-service start [options]`

This command will start the service, making Blockchain Wallet API V2 available on a specified port.

Command options:

  * `-h, --help` - output usage information
  * `-p, --port` - port number to run the server on (defaults to `3000`)
  * `-b, --bind` - bind to a specific ip (defaults to `127.0.0.1`, note that binding to an ip other than this can lead to security vulnerabilities)

To open the service to all incoming connections, bind to `0.0.0.0`.

### Examples

To start the Wallet API service on port 3000:

```sh
$ blockchain-wallet-service start --port 3000
```

## Development

  1. Clone this repo
  2. Run `npm install`
  3. Run `npm start`
  4. Dev server is now running on port 3000

If you are developing `blockchain-wallet-client` alongside this module, it is useful to create a symlink to `my-wallet-v3`:

```sh
$ ln -s ../path/to/my-wallet-v3 node_modules/blockchain-wallet-client
```

### Testing

```sh
$ npm test
```

### Configuration

Optional parameters can be configured in a `.env` file:

  * `PORT` - port number for running dev server (default: `3000`)
  * `BIND` - ip address to bind the service to (default: `127.0.0.1`)

## Deployment

If you want to use blockchain-wallet-service in your UNIX production server, you just have to run:

```sh
$ nohup blockchain-wallet-service start --port 3000 &
```

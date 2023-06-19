# Auto-Deployment

## Prerequisites

- Ubuntu 20.04
- docker and docker-compose
- node >= v18.12
- npm >= 8.19

## Install and start the Parachains and EVM chain locally for `O-DLT` token

We have made Docker images for these nodes, so you can launch nodes easily
```
wget https://omniversedlt.s3.amazonaws.com/docker-compose.yaml
docker-compose up -d
```

![](./assets/auto-deploy/launch%20nodes.png)

The following chains will be installed and launched locally:
- Ink! Parachain
- Swap Parachain
- Local EVM chain

If you do not change ports or other fields in the `docker-compose.yaml`, you can get node addresses shown below:

- Local EVM chain: http://127.0.0.1:10100
- Ink! Parachain: ws://127.0.0.1:10101
- Swap Parachain: ws://127.0.0.1:10102

## Auto-deploy `O-DLT` token and initialization

We use the [system test tool](https://github.com/Omniverse-Web3-Labs/omniverse-system-test/tree/milestone-2) which is shown in [Test guide](https://github.com/Omniverse-Web3-Labs/Omniverse-DLT-Introduction/blob/main/docs/test-guide/m2-test-guide.md) to deploy contracts.

### Installation

#### Clone the repository

```sh
git clone -b milestone-2 --recursive https://github.com/Omniverse-Web3-Labs/omniverse-system-test.git
```

#### Install

Enter the working directory, and execute the following commands

```sh
npm install
```

#### Install for related projects

```sh
node src/index.js -i
```

### Configure

Enter the working directory, replace `config/default.json` with `config/deploy.template.json`
```
cp config/deploy.template.json config/default.json
```

Open `config/default.json`

`tokenInfo` is the token information of the omniverse tokens you will deploy, you can change it as what you like.

```
"tokenInfo": {
        "token": [{
            "name": "SKYWALKER",
            "symbol": "SKYWALKER"
        }]
    },
```

`networks` contains the chains on which you will deploy omniverse tokens

- rpc: `http` end point of a node connected to the network
- ws: `websocket` end point of a node connected to the network
- chainType: What kind of chain you will deploy omniverse token on, currently you can choose: EVM, INK, SUBSTRATE
- omniverseChainId: The omniverse chain id is set for all public chains, namely each chain will have a unique omniverse chain id. Currently, you can set any id the this field, just keep it unique in the configuration.
- coolingDown: Cooling down time, the interval between two omniverse transactions, just let what it is if you use local nodes.
- chainId: The field is the EVM chain id, so it is only used when the chain type is `EVM`.
- chainName: The chain name you assign to the chain.

We have configured this file for this demonstration, so you do not need to change it.

### Auto-deployment and initializations

```
node src/index.js -d token [-c <NUM>]
```

`-c <NUM>` indicates how many tokens you want to deploy, the token name will be picked from the field `tokenInfo` in `config/default`, the default value is 1. If the number is larger than the array size of `tokenInfo`, token name will derived from the last token name by adding a suffix, such as `SKYWALKER1`.

This process is almost the same as [test guide](https://github.com/Omniverse-Web3-Labs/Omniverse-DLT-Introduction/blob/main/docs/test-guide/m2-test-guide.md#explaination-of-fungible-tokens-test), except that it will not run test cases.
    
The following things will be done in the deployment process
- Deployment of the related (set in the `default.json`) Ink! omniverse token 
- Deployment of the related (set in the `default.json`) EVM omniverse token 
- Deployment of the related (set in the `default.json`) Pallet omniverse token
- Initilazation of omniverse tokens
    - Set members
    - Set cooling time
    - Set Decimal
- Transfer gas tokens to testing accounts

## Launch the auto-synchronizer

You can use the `omniverse-synchronizer` in `./submodules`

### Configure

The config file `config/default.json` and secret key file `.secret` will be created automatically after you run deploy command. You do not need to change it here, you can refer [Omniverse-synchronizer](https://github.com/Omniverse-Web3-Labs/omniverse-synchronizer/blob/milestone-2/README.md) for more information.

### Launch the synchronizer

Make a directory as the working directory of the synchronizer
```
sudo mkdir -p /opt/omniverse/node/test/test/latest/
```

Download the `docker-compose.yaml` file into the synchronizer working directory
```
sudo wget https://omniversedlt.s3.amazonaws.com/synchronizer/docker-compose.yaml -O /opt/omniverse/node/test/test/latest/docker-compose.yaml
```

Copy the config directory of the synchronizer project in `omniverse-system-test/submodules` into the synchronizer working directory
```
sudo cp -r ./submodules/omniverse-synchronizer/config /opt/omniverse/node/test/test/latest/
```

Execute the following command to launch the synchronizer.
```
cd /opt/omniverse/node/test/test/latest/
docker-compose up -d
```

You can see outputs like this
![](./assets/auto-deploy/synchronizer.png)

You can check the logs of the synchronizer
```
sudo docker logs -f test-test
```

![](./assets/auto-deploy/docker%20logs.png)

You can use different working directory of the synchronizer, just change the field in the `docker-compose.yaml` as following. If you are running multiple synchronizers, you must use different working directory.



## Note(Option) 

- Launch more auto-synchronizers

    If you want to start more auto-synchronizers, repeat the [Launch the auto-synchronizer](#launch-the-auto-synchronizer)

- Create more omniverse tokens

    If you want to create more omniverse tokens, repeat the [Auto-deploy `O-DLT` token and initillization](#auto-deploy-o-dlt-token-and-initillization) and [Launch the aotu-synchronizer](#launch-the-aotu-synchronizer)

- Deploy on public EVM chains

There are only four additional steps to deploy contracts on live EVM chains

1. Prepare two accounts with tokens
2. Add a field `accounts` in `config/default.json`
```
"accounts": "./config/.secret"
```
3. Add a secret key file `.secret` in the directory `config`, and input private keys in the file
```
[
    "0x1234...",
    "0x5678..."
]
```
4. Change the field `networks` in `config/default.json`

For example, to deploy on Ethereum
```
"networks": [
        {
            "rpc": "https://eth.llamarpc.com",
            "ws": "wss://eth.llamarpc.com",
            "chainId": 1,
            "chainType": "EVM",
            "coolingDown": 10,
            "chainName": "ETHEREUM"
        }
    ],
```
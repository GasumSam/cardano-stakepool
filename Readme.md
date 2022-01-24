---------------

# <img align="left" src="https://styles.redditmedia.com/t5_4bhj33/styles/communityIcon_u389y4dxqov61.png" alt="Novapago Labs" style="height: 100px; width:130px;"/> Cardano Research
### Internship project
-------------------------

Visit @ [Novapago Cardano Stake Pool](https://pooltool.io/pool/ab16ce81507d4298dbb37532c3fb38397b42f0abd00e9a5caa9ab511/epochs) (Testnet).

-----------------------
# Running a Cardano Stake Pool

Sources:

[Cardano Foundation Docs](https://docs.cardano.org/) \
[Cardano Developers](https://developers.cardano.org/) \
[IOHK](https://iohk.zendesk.com/hc/en-us) \
[IOHK](https://github.com/input-output-hk/cardano-node) (Github) \
[Cardano Blockchain Explorer](https://explorer.cardano-testnet.iohkdev.io/en) (Testnet) \
[Cardano Blockchain Explorer](https://explorer.cardano.org) (Mainnet) \
[Faucet Cardano Testnet](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/)\
[Abradacaniel Github](https://github.com/abracadaniel/cardano-pool-docker) (Best Practise)

> **This exercise has an only educational porpouse**. Hence, all decisions about arquitecture or features are  based to learn how Cardano environment works and his blockchain network. Any use of knowledge published here with no educational porpouse will be under your own responsability.
>
> For those reasons, you need to consider several aspects:
> * The present metodology is about a Cardano Stake Pool develop onto testnet and non real ADA cryptocurrency (Faucet tADA used for educational porpouse).
> * Any key and certificate are generated and storage in the same node to ease transactions and Command Line Interface (cli) operations. It's not a safe practise, so if you are thinking to use these arquitecture onto mainnet, we recommend you create a [Air Gap system](https://en.wikipedia.org/wiki/Air_gap_(networking)) to manage your cold keys or any else offline storage and wallet.

## About Cardano

Cardano is a third generation groundbreaking Proof of Stake (PoS) Blockchain Network. Cardano aims to achieve the scalability, interoperability and sustainability needed for real-world applications. Based in openness and transparency, Cardano is a public project to underpin the economy of the future. Since it was founded at 2015 by ex-ethereum co-founder Charles Hoskinson, Cardano is considered one of the most important Blockchain Projects as several reasons such  be a more ecological alternative with his PoS protocol,  lead development activity ranking between Blockchain Technologies (https://beincrypto.com/cardano-leads-pack-with-most-developer-activity-in-2021/) or offer a seriously roadmap.

## Introduction

High costs to operate in other protocols and the promise to guarante a solid platform for SmartContracts, convert Cardano in an interest project to know and research beyond the Ethereum and Bitcoin protocols worked in classroom during the course. 

Therefore, our work lines are based in several targets and are structured by own particular roadmap: 
- Be part of Cardano Network participating in the protocol actively (`Stake Pool`)
- Simplify development improving the potential scalability (`Docker`)
- Testing Plutus Platform as alternative for DApps and SmartContracts (`SmartContract`)

## Arquitecture

The main target is deploy a Stake Pool with two nodes: a Block Producer Node responsible to close blocks that is only connected with a Relay Node for security reasons. In that case, Relay Node is main connection with the rest of Cardano Blockchain Network, protecting Block Production Node against posible security breaks.

The tutorials to raise a Cardano Stake Pool suggest deploy each node of the Stake Pool in his own server usually. In this case,  we add complexity using a only one server for all nodes used (two in our case) trought a Docker Network operating inside the server.

Finally, Block Producer Node is connected with a node-exporter (hosted inside the Docker Network), that is connected with Prometheus (hosted in the server out of the Docker Nerwork) to monitor activity in Grafanna Dashboard at the same time.


<img src="https://iili.io/YwnTJa.png" alt="Novapago Stake Pool Arquitecture" style="height: 600px; width: 820px;"/>

## Minimum Hardware Requirements

* Operative system: 64-bit Linux (i.e Ubuntu Server 20.04 LTS)
* Processor: An Intel or AMD x86 processor with two or more cores, at 2GHZ or faster
* Memory: 12Gb of RAM
* Storage: 50GB of free storage
* Internet: Broadband internet connection with speeds at least 10 Mbps

## Prerequisites

- #### Server 
You can choice install an [Ubuntu Server](https://https://ubuntu.com/tutorials/install-ubuntu-server#1-overview) or [Virtual Box](https://www.virtualbox.org/wiki/Downloads) with [Ubuntu 20.04](https://linuxhint.com/install_ubuntu_virtualbox_2004/) for each node. However we choice finally a [Linux Server on AWS](https://cardano-foundation.gitbook.io/stake-pool-course/stake-pool-guide/system-setup/aws), where we host a Docker Network with our two nodes. Please, follow referral steps for each method in links above.
- #### SSH Client
We work on Windows OS for this exercise, indeed, we use [Bitvise SSH Client](https://www.bitvise.com/) to manage or AWS Server. Bitvise allows create a SSH keys (Through _Client Key Manager_) to improve security conditions at file transfers or console access. Otherwise, if you are using an alternative OS you can use other reference as [Cyberduck](https://cyberduck.io/) or [Filezilla](https://filezilla-project.org/) and [generate your own SSH Keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
- #### Docker
As part of current project we use microservice such Docker to ease deployment tasks. So install [Docker](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html) and [Docker Compose](https://docs.docker.com/compose/install/) at your server is mandatory to complete these steps. Please follow steps in each link. 

## Docker Compose 

Docker Compose allows configure features for our services in only one commands file, easier to deploy. We guide you step by step about each service in Docker Compose file to finally deploy with a simple `docker-compose up` in your command prompt.

#### Building _docker-compose.yml_

`Version 3.5` allows networks names features as improvements, so we can use that version or higher.

```
version: '3.5'
```

Featuring parameters for a Docker Network where we will host our nodes. In this case, with `cardano_stake` as name and with a subnet within `172.20.0.0/24` range IP Address.

```
networks:
  cardano_stake:
    ipam:
      config:
        - subnet: 172.20.0.0/24
```
Services include a first node as `relay` that Docker Compose pull from `inputoutput/cardano-node`. 

Hosted at `172.20.0.23` IP Adress whitin `cardano_stake` Docker Network.

Restart policy declare `always` to keep container up in case of any problem.

For `volumes`, we need to follow the features for cardano-node Docker Image and how mount path from container folders to server folders. In this case, we create three volumes in our server that correspond with `/ipc` (for node.socket file), `/data` and `/config` (configuration files) folders.

For `command`, we declare `run`, `--config` to enroute with the _config.json_, `--topology` to enroute with _topoology.json_, `--database-path`, `--socket-path` with the path of the socket file, `--host-addr` for any ip entrace and `--port`, declaring the only port of communications available for the node (connect wih the entire Cardano Blockchain).

At `ports` details, we maps from server port to node port, so there`ll be a connection between them. In this case at port 3001.

About `environment` we specify map to socket path file and update command to _config_ and _topology_  .json files.

```
services: 

  relay_node:
    container_name: relay
    image: inputoutput/cardano-node

    networks:
      cardano_stake:
        aliases:
          - relay_node
        ipv4_address: 172.20.0.23

    restart: always

    volumes:
      - ./00cardanorelay/cardano-node-ipc:/ipc
      - ./00cardanorelay/cardano-node-data:/data
      - ./00cardanorelay/config:/config

    command: 
      - run
      - --config /config/testnet-config.json
      - --topology /config/testnet-topology.json
      - --database-path /db
      - --socket-path opt/cardano/ipc/node.socket
      - --host-addr 0.0.0.0
      - --port 3001

    ports:
      - 3001:3001

    environment:
      - CARDANO_NODE_SOCKET_PATH=opt/cardano/ipc/node.socket
      - CARDANO_UPDATE_TOPOLOGY=true
      - CARDANO_UPDATE_CONFIG=true
```

And now we continuous with the second service: Block Production Node (Core Node) with a similar configuration.

```

  core_node:
    container_name: core
    image: inputoutput/cardano-node

    networks:
      cardano_stake:
        aliases:
          - core_node
        ipv4_address: 172.20.0.24

    restart: always

    volumes:
      - ./01cardanocore/cardano-node-ipc:/ipc
      - ./01cardanocore/cardano-node-data:/data
      - ./01cardanocore/config:/config

    command: 
      - run
      - --config /config/testnet-config.json
      - --topology /config/testnet-topology.json
      - --database-path /db
      - --socket-path opt/cardano/ipc/node.socket
      - --host-addr 0.0.0.0
      - --port 3002

    ports:
      - 3002:3002

    environment:
      - CARDANO_NODE_SOCKET_PATH=opt/cardano/ipc/node.socket
      - CARDANO_UPDATE_TOPOLOGY=true
      - CARDANO_UPDATE_CONFIG=true

```

**Note:** This is a basic configuration to deploy two Cardano nodes in a Docker Network (non Stake Pool config yet). We should improve it with more services and conf details later.

## Cardano Stake Pool configuration files

We can see that both services above came from the same Docker Image (`inputoutput/cardano-node`). It means that the final behavior of the node (as `relay` or `core`) depends of the configuration files.

Now, you can download from [Cardano configurations files download page](https://hydra.iohk.io/build/7654130/download/1/index.html) for one of the specific networks (`Mainnet` or `Testnet`). The files that we needs are (in our case of testnet):

- [_testnet-config.json_](https://hydra.iohk.io/build/7654130/download/1/testnet-config.json), who read from/to different forks genesis files that need to download also:

> - [_testnet-byron-genesis.json_](https://hydra.iohk.io/build/7654130/download/1/testnet-byron-genesis.json)
> - [_testnet-shelley-genesis.json_](https://hydra.iohk.io/build/7654130/download/1/testnet-shelley-genesis.json)
> - [_testnet-alonzo-genesis.json_](https://hydra.iohk.io/build/7654130/download/1/testnet-alonzo-genesis.json)
- [_testnet-topology.json_](https://hydra.iohk.io/build/7654130/download/1/testnet-topology.json), who specify the communications with others nodes and the blockchain.

And store  in each specific config folder of each node (relay and core) at server.

#### What about eras and forks in Cardano??

The Cardano Roadmap contemplate five mains development eras:

1. **Byron**. Is the foundational era. Byron includes the development period from original idea (2015) to final Ada Cryptocurrency launch at 2017. Based on Ouroboros protocol, it's also launch Daedalus and Yoroi wallets as part of newborn ecosystem.
2. **Shelley**. Is the descentrailization era. The hard fork was at 29th July 2020, and means the transition from federate network to a user descentralized network. Add a reward system to drive stakes pools. The incentives scheme was designed to reach equilibrium with around 1,000 stakes pools. 
Is the descentrailization era. The hard fork was at 29th July 2020, and means the transition from federate network to a user descentralized network. Add a reward system to drive stakes pools. The incentives scheme was designed to reach equilibrium with around 1,000 stakes pools. In addition, Shelley era was divided in two periods: 
> - _Allegra_: With a token locking mechanism.
> - _Mary_: Where blockchain test his own capability to support Cardano Native Tokens (Similar to ERC tokens). 
3. **Goguen** (Current). Is the Smart Contracts era. The target is build decentralized applications (DApps) and deploy Smart Contracts through his own language Plutus, based on functional language Haskell. Allow the creation of fungible and non-fungible tokens also. The technical complexity of Goguen era force to recognized differents forks:

> - _Alonzo Blue_: Around 50 operators deploying Smart Contracts. Is a fix phase.
> - _Alonzo White_: Under more in a exercise boot camp to test Smart Contracts.
> - _Alonzo Purple_:  More than 1000 participants in a fully testnet. Divide in _light purple_, about simples SC, and _dark purple_, with complexes SC. The final phase for Alonzo gather to news pertiods: _purple red_ and _purple black_, as an improvements phases on Smart Contracts system.

4. **Basho**. Scaling era. To implement _Sidechains_, new blockchains interoperable with the main Cardano Blockchain that will allows experimental features.
5. **Voltaire**. Governance era. The maintenance and manage of the network will be in hands of the community.


Note: The several technical develops corresponding for each era have been simultaneous during a transition period normally (In any hard fork there's a considerable segment of the community that did not switch over the new version). Cardano use _hard fork combinator_, a system that combines two different protocols and allows nodes running multiples versions at once.

That's the reason why the genesis config files gather differents eras.

### Configuring relay node

**Note:** To ease the learn process we don't talk about specific fields in these files, we only do basic changes to deploy our nodes as relay and core (block producer). If you want understand deeply how they works, please link [here](https://cardano-foundation.gitbook.io/stake-pool-course/stake-pool-guide/getting-started/understanding-config-files).

#### _testnet-topology.json_

Tells your node to which nodes in the network it should talk to. In our case, _Relay Node_ talks with _Block Producer Node_ (hosted at 172.20.0.24 IP address for Docker Network and port 3002) and with Cardano Blockchain testnet through the port maped from node (3001).
In our case, _Relay Node_ talks with _Block Producer Node_ (hosted at 172.20.0.24 IP address for Docker Network and port 3002) and with Cardano Blockchain testnet through the port maped from node (3001).

```
{
  "Producers": [
    {
      "addr": "172.20.0.24",
      "port": 3002,
      "valency": 1
    },
    {
      "addr": "relays-new.cardano-testnet.iohkdev.io",
      "port": 3001,
      "valency": 1
    }
  ]
}
```

### Configuring core node

#### _testnet-topology.json_

In this case, for security reasons, _core node_ only talks with _relay node_.

```
{
  "Producers": [
    {
      "addr": "172.20.0.23",
      "port": 3001,
      "valency": 1
    }
  ]
}
```

#### _testnet-config.json_

This changes might be later, but we take advantage and we do it yet: Inside the code, from 65 to 70 line approx, appear details for our future connection with _Prometheus_ to measure node behavior. So we recommend you open IP address to any connection and use port 12789 (default), that we should open in Amazon server Instance configuration also.

```
...

  "hasEKG": 12788,
  "hasPrometheus": [
    "0.0.0.0",
    12789
  ],

...


```

## Start the nodes

At glance, we have our docker-compose.yml  configured and with the mandatory config files stored at server. So just we need to open a terminal at our lines server a execute:

```
docker.compose.yml up -d

```
`-d` to run everything in the background.

#### Check that everything is ok (using basic Docker commands)

List docker process running:

```
docker ps
```
List containers:
```
docker containers ls
```
Stop container:
```
docker stop <name of container> or <container id>
```
Remove container:
```
docker rm -f <name of container> or <container id>
```


## Cardano CLI (Command Line Interface)

Cardano nodes provides a command line interface as a tools for key generation, transaction construction, certification creation and other task.

Access to node bash (Cardano Command Line Interface) from Docker:
```
docker exec -it <name of container> bash
```
And we should sees something like that: 
```
bash-4.4#
```
Remember that you can use `cardano-cli --help` command to consult the different options that Cardano CLI offers you.

However, a first step with the cardano-cli might be launch a query to the testnet and check that everything is ok:

Note: `--testnet-magic 1097911063` is the --testnet parameter. 

```
cardano-cli query tip --testnet-magic 1097911063
```
At this moment, probably you receive that from cardano-cli:

```
cardano-cli: Network.Socket.connect: <socket: 11>: does not exist (No such file or directory)
```
This occurs because node unknowns the path to `node.socket`, the file that allows communications works in the node. So we need to enroute again the socket file path with follow command from cardano-cli: 
```
export CARDANO_NODE_SOCKET_PATH=opt/cardano/ipc/node.socket
```
Note: In some cases, the path to socket file can change, so ensure that `node.socket` is in the indicated path or another else (normally at `/ipc` or `/opt/cardano/ipc`). 

Execute again a query over Cardano testnet (`cardano-cli query tip --testnet-magic 1097911063`). And if everything is ok, probably we sees something like that:

```
{
    "epoch": 181,
    "hash": "6d13dd1503f005b6183617d7987e41db76686de0af16421dcc9cfa4abcc9c2cb",
    "slot": 48149654,
    "block": 3247977,
    "era": "Alonzo",
    "syncProgress": "100.00"
}
```
If you want to check that the information above is correct and your node is reporting from de testnet, you can copy the `hash` and check it in [Cardano Testnet Explorer](https://explorer.cardano-testnet.iohkdev.io/en).

## Generate payment keys and addresses

> <span style="color:red">**WARNING</span>: For ease of use, we will keep our Cardano Testnet Key files in the server. But this is `NOT SECURE`** \
> **In a real scenario (MAINNET), you need to have your keys under cold storage (offline)**.

We need to create two sets of keys and addresses. One set to control our funds (make and receive payments) and one set to control our stake (to participate in the protocol delegating our stake)

Let's produce our cryptographic keys firts, as we will need them to later create our addresses:

#### Payment key pair

Generate a payment key pair:

```
cardano-cli address key-gen \
    --verification-key-file payment.vkey \
    --signing-key-file payment.skey

```
Our payment key pair are a `payment.vkey`, as public verification key, and `payment.skey`, as private signing key.

>Note: If we need to store these keys somewhere, we can't decide the route here.
>> In our case, we can do `pwd` inside the node a check that we are in root `\`. Now, we can create a folder to store our keys; `mkdir /myKeys` and use that route to our cardano-cli commad above.
>> ```
cardano-cli address key-gen \
    --verification-key-file myKeys/payment.vkey \
    --signing-key-file myKeys/payment.skey```
> 
> Or you can create on the root and move it later.

#### Payment address

We then use these keys pair (`payment.vkey` & `payment.skey`) to create the _payment address_:

```
 cardano-cli address build \
    --payment-verification-key-file myKeys/payment.vkey \
    --out-file myAddress/payment.addr \
    --testnet-magic 1097911063

```

As you can see, this `payment.addr` is associated with our payment keys through `payment.vkey`, used to build it. You can see in details through:

```
cat /myAddress/payment.addr
> addr_test1vrxfh9larxm7uhwfks4danzgffhsjkwwtxam6ad8sy7566sp0r7fj (not real, example)

```
> Note: Remember that probably you need to set the environment variable to the socket-path specified in your node configuration:
>> ```
>>export CARDANO_NODE_SOCKET_PATH=~/opt/cardano/ipc/node.socket
>> ```
>

Now we can make sure that the node is running check in `payment.addr` UTxO and find out address balance:

```
cardano-cli query utxo \
    --address $(cat myAddress/payment.addr) \
    --testnet-magic 1097911063

```
And we should see something like this:

```
                       TxHash                                 TxIx        Lovelace
----------------------------------------------------------------------------------
```

## Faucet

To continue with our exercise, we need to have some balance to do a transaction. We can request funds to the faucet, a web service that provide `tADA` (fake ADA).

Click [here to redirect to the faucet](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/). You just need to put your `payment.addr` (i.e. `addr_test1vrxfh9larxm7uhwfks4danzgffhsjkwwtxam6ad8sy7566sp0r7fj`) to receive _1000_ tADA.

## Create a simple transaction

`cardano-cli` offers us many possibilities to build a transaction. We can specify  many parameters about _inputs_, _outputs_, _address_, _sign_ or _time_, among others. In order to simplify this learn process we'll go to create a simple transaction directly.

If you want research about parameters posibility in `cardano-cli`, use `cardano-cli transaction --help` or click [here](https://cardano-foundation.gitbook.io/stake-pool-course/stake-pool-guide/stake-pool-operations/building-and-signing-tx) ([Building and signing transaction](https://cardano-foundation.gitbook.io/stake-pool-course/stake-pool-guide/stake-pool-operations/building-and-signing-tx)).

#### Creating a new payment address

A new **payment address** is needed to send it some _tADAs_ in our simple transaction. So we need to repeat some steps above:

```
cardano-cli address key-gen \
    --verification-key-file myKeys/payment2.vkey \
    --signing-key-file myKeys/payment2.skey

```
```
cardano-cli address build \
    --payment-verification-key-file myKeys/payment2.vkey \
    --out-file myAddress/payment2.addr \
    --testnet-magic 1097911063

```

#### 1. Get protocol parameters

Get the protocol parameters and save them to `protocol.json`:

```
cardano-cli query protocol-parameters \
--testnet-magic 1097911063 \
--out-file data/protocol.json
```
To improve storage organization inside the node, we choice store `protocol.json` at `/data` folder.  But remember that you can manage that as you want. 

#### 2. Determine the TTL (Time to Live) for the transaction

We need the CURRENT TIP of the blockchain, this is, the height of the last block produced. We are looking for the value of SlotNo.

```
cardano-cli query tip --testnet-magic 1097911063

{
    "epoch": 181,
    "hash": "edf8fb518db1faa945a15ddb78df7a9b5a1a78255a55ae44e8fb604198989f8a",
    "slot": 48219923,
    "block": 3250043,
    "era": "Alonzo",
    "syncProgress": "100.00"
}

```
At this moment the tip is on `slot` `48219923`.

To build the transaction we need to specify the **TTL** (Time To Live), this is the block height limit for our transaction to be included in a block, if it is not in a block by that slot the transaction will be cancelled.

From `protocol.json` we know that we have 1 slot per second. Lets say that it will take us 10 minutes to build the transaction, and that we want to give it another 10 minutes window to be included in a block. So we need 20 minutes or 1200 slots. So we add 1200 to the current tip: 48219923 + 1200 = 370415.
```
expr 48219923 + 1200

> 48221123
```
So our TTL is `48221123`

#### 3. Draft the transaction 
We create a draft for the transaction and save it in `tx.raw`.

We need the transaction hash and index of the **UTXO** we want to spend:

```
cardano-cli query utxo \
    --address $(cat myAddress/payment.addr) \
    --testnet-magic 1097911063

>                            TxHash                                 TxIx        Lovelace
> ----------------------------------------------------------------------------------------
> 4e3a6e7fdcb0d0efa17bf79c13aed2b4cb9baf37fb1aa2e39553d5bd720c5c99     1       1000000000
```
Note that for `--tx-in` we use the following syntax: `TxId#TxIx` where `TxId` is the transaction hash and `TxIx` is the index (you found these values in the above step) they will be added like this `4e3a6e7fdcb0d0efa17bf79c13aed2b4cb9baf37fb1aa2e39553d5bd720c5c99#1`.

For `--tx-out` we use: `TxOut+Lovelace` where TxOut is the hex encoded address followed by the amount in Lovelace. `--tx-out $(cat payment.addr)+0`, `--ttl`, `--fees`, are all 0 for now. We will revisit these in a later step after we calculate fees and ttl.

```
cardano-cli transaction build-raw \
    --tx-in 4e3a6e7fdcb0d0efa17bf79c13aed2b4cb9baf37fb1aa2e39553d5bd720c5c99#1 \ 
    --tx-out $(cat myAddress/payment2.addr)+100000000 \
    --tx-out $(cat myAddress/payment.addr)+0 \
    --ttl 0 \
    --fee 0 \
    --out-file data/tx.raw

```

#### 4. Calculate the fee

The transaction needs one (1) input: a valid UTXO from `payment.addr`, and two (2) outputs: The receiving address **payment2.addr** and an address to send the change back, in this case we use **payment.addr**. You also need to include the _Draft transaction file_. `Witnesses` are the number of signing keys used to sign the transaction, in this case `1`.

```
cardano-cli transaction calculate-min-fee \
   --tx-body-file data/tx.raw \
   --tx-in-count 1 \
   --tx-out-count 2 \
   --witness-count 1 \
   --byron-witness-count 0 \
   --testnet-magic 1097911063 \
   --protocol-params-file data/protocol.json


   > 174433 Lovelace
```
So we need to pay **174433 Lovelace** fee to create this transaction.

Assuming we want to send 100 ada to **payment2.addr** spending a **UTxO** containing 1,000 ada (1,000,000,000 Lovelace), now we need to calculate how much is the change to send back to **payment.addr**.

```
expr 1000000000 - 100000000 - 174433

> 899825567
```

#### 5. Build the transaction

Now we need to repeat the _Draft the transaction_ step above to update `tx.raw` with the new known arguments.

```
cardano-cli transaction build-raw \
    --tx-in 4e3a6e7fdcb0d0efa17bf79c13aed2b4cb9baf37fb1aa2e39553d5bd720c5c99#1 \
    --tx-out $(cat myAddress/payment2.addr)+100000000 \
    --tx-out $(cat myAddress/payment.addr)+899825567 \
    --ttl 370415 \
    --fee 174433 \
    --out-file data/tx.raw
```

#### 6. Sign the transaction

Sign the transaction with the signing key **payment.skey** and save the signed transaction in **tx.signed**

```
cardano-cli transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file myKeys/payment.skey \
    --testnet-magic 1097911063 \
    --out-file data/tx.signed
```

#### 7. Submit the transaction

Make sure that your node is running and set **CARDANO_NODE_SOCKET_PATH** variable to:
```
export CARDANO_NODE_SOCKET_PATH=~/opt/cardano/ipc/node.socket
```
And submit the transaction:

```
cardano-cli transaction submit \
    --tx-file data/tx.signed \
    --testnet-magic 1097911063
```

#### 8. Check the balance

At several minutes later, maybe the transaction will get incorporated into blockchain and we will see it when we do a query.

```
cardano-cli query utxo \
    --address $(cat myAddress/payment.addr) \
    --testnet-magic 1097911063

>                            TxHash                                 TxIx        Lovelace
> ----------------------------------------------------------------------------------------
> b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee     1        899825567

cardano-cli query utxo \
    --address $(cat /myAddress/payment2.addr) \
    --testnet-magic 1097911063

>                            TxHash                                 TxIx        Lovelace
> ----------------------------------------------------------------------------------------
> b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee     0         100000000

```
>**Note:** This is the easier way to do a transaction with `cardano-cli`, a methodology that offer us the possibility to manage _Cardano Blockchain_ through his transactions and payments. However, there are some troubleshooting we need to know. In our experience with the process above, the most common interruption is about calculated fees. (i.e.) When we submit the transaction (Step 7) it throws a error who describe that and specify the real fee that Cardano Blockchain needed to create it. That's no sense aparentely, because we had calculated fees previously about `protocol.json` parameters, but we understand that conditions can change during the process and fees requirenments too.\
>In that case, just copy the fees requiered and repeat the process above from `step 5` (recalculating _outputs_).
>
>> Others posibles problems and troubleshooting are described [here](https://cardano-foundation.gitbook.io/stake-pool-course/stake-pool-guide/stake-pool-operations/diagnosing-transactions).
>
>

## Generate Stake Keys and Address

> <span style="color:red">**WARNING</span>: For ease of use, we will keep our Cardano Testnet Key files in the server. But this is `NOT SECURE`** \
> **In a real scenario (MAINNET), you need to have your keys under cold storage (offline)**.

If you are a Cardano stakeholders you can have to sets of keys and addresses:
- `Payment Keys` and `addresses`: To send and receive transactions.
- `Stake Keys` and `addresses`: To _**control protocol participation**_, _**create a stake pool**_, _**delegate**_ and _**receive rewards**_.

At this moment, we have a `payment key pair`, to _public verification_ and _private sign_, and a `payment address`, to _receive and send transactions_.

So we need to create a Stake Key pair and Stake Address to continue with the steps to rise a Stake Pool.

```
cardano-cli stake-address key-gen \
    --verification-key-file myKeys/stake.vkey \
    --signing-key-file myKeys/stake.skey

```
The step above allows us create Stake Address that **can't receive payments** but will receive the rewards from participating in the protocol.

```
cardano-cli stake-address build \
    --stake-verification-key-file /myKeys/stake.vkey \
    --out-file myAddress/stake.addr \
    --testnet-magic 109791106

```

### Regenerating payment address

With a `Stake Address` is time to regenerate a `Payment Address` who link `Payment Verification Key` and `Stake Verification Key`. 

```
cardano-cli address build \
    --payment-verification-key-file myKeys/payment.vkey \
    --stake-verification-key-file myKeys/stake.vkey \
    --out-file myAddress/paymentwithstake.addr \
    --testnet-magic 109791106
```

## Register Stake Address on the Blockchain

Stake address needs to be registered on the blockchain to be useful. Registerinr keys requires:

- Create a registration certificate.
- Submit the certificate to the blockchain with a transaction.

### Create a registration certificate

```
cardano-cli stake-address registration-certificate \
    --stake-verification-key-file myKeys/stake.vkey \
    --out-file myCerts/stake.cert
```
#### Draft transaction
```
cardano-cli transaction build-raw \
    --tx-in b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee#1 \
    --tx-out $(cat myAddress/paymentwithstake.addr)+0 \
    --invalid-hereafter 0 \
    --fee 0 \
    --out-file data/tx.draft \
    --certificate-file myCerts/stake.cert
```
#### Calculate fees

```
cardano-cli transaction calculate-min-fee \
    --tx-body-file data/tx.draft \
    --tx-in-count 1 \
    --tx-out-count 1 \
    --witness-count 2 \
    --byron-witness-count 0 \
    --testnet-magic 1097911063 \
    --protocol-params-file data/protocol.json
    
> 171485
    
```
Registering the stake address, not only pay transaction fees, but also includes a deposit (which you get back when deregister the key) as stated in the protocol parameters.

The deposit amount can be found in the `protocol.json` under `stakeAddressDeposit`:


    cat /data/protocol.json
    
    > {
        "txFeePerByte": 44,
        "minUTxOValue": 1000000,
        "decentralization": 1,
        "utxoCostPerWord": null,
        "stakePoolDeposit": 500000000,
        "poolRetireMaxEpoch": 18,
        "extraPraosEntropy": null,
        "collateralPercentage": null,
        "stakePoolTargetNum": 150,
        "maxBlockBodySize": 65536,
        "minPoolCost": 340000000,
        "maxTxSize": 16384,
        "treasuryCut": 0.2,
        "maxBlockExecutionUnits": null,
        "maxCollateralInputs": null,
        "maxValueSize": null,
        "maxBlockHeaderSize": 1100,
        "maxTxExecutionUnits": null,
        "costModels": {},
        "protocolVersion": {
            "minor": 0,
            "major": 2
        },
        "txFeeFixed": 155381,
        "stakeAddressDeposit": 2000000,
        "monetaryExpansion": 3.0e-3,
        "poolPledgeInfluence": 0.3,
        "executionUnitPrices": null
    }

In this case `"stakeAddressDeposit": 2000000,`

Query the UTXO of the address that pays for the transaction and deposit:

    cardano-cli query utxo \
        --address $(cat /myAddress/paymentwithstake.addr) \
        --testnet-magic 1097911063


    >                            TxHash                                 TxIx      Amount
    > ----------------------------------------------------------------------------------------
    > b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee     1      1000000000 lovelace


Calculate the change to send back to payment address after including the deposit


    expr 1000000000 - 171485 - 2000000
    
    > 997828515

### Submit the certificate to the blockchain with a transaction

`--invalid-hereafter` as a `--ttl` represent aproximatly upcoming slot. It should be greater than the current slot number.

Tip to the Testnet and calculate de `--invalid-hereafter`:
```
    cardano-cli query tip --testnet-magic 1097911063

    > {
        "epoch": 181,
        "hash": "1c067aa87c49e3b22892aa6c347319e48ee51bf045d2116750cb3ef0e7a2aabb",
        "slot": 48232840,
        "block": 3250410,
        "era": "Alonzo",
        "syncProgress": "100.00"
    }
```
```
    expr 48232840 + 1200
    > 48234040
```
    cardano-cli transaction build-raw \
        --tx-in b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee#1 \
        --tx-out $(cat myAddress/payment.addr)+997828515 \
        --invalid-hereafter 48234040 \
        --fee 171485 \
        --out-file data/tx.raw \
        --certificate-file myCerts/stake.cert

#### Sign it:
    
    cardano-cli transaction sign \
        --tx-body-file data/tx.raw \
        --signing-key-file myKeys/payment.skey \
        --signing-key-file myKeys/stake.skey \
        --testnet-magic 1097911063 \
        --out-file data/tx.signed
    
#### And submit it:

    cardano-cli transaction submit \
        --tx-file data/tx.signed \
        --testnet-magic 1097911063

Our Stake Key is **now registered** on the blockchain.

## Stake Pool Keys

> <span style="color:red">**WARNING</span>: For ease of use, we will keep our Cardano Testnet Key files in the server. But this is `NOT SECURE`** \
> **In a real scenario (MAINNET), you need to have your keys under cold storage (offline)**.

Now we have:

| File  | Content |
| ------------- |:-------------:|
| `payment.vkey`| payment verification key |
| `payment.skey`| payment signing key |
| `stake.vkey`| staking verification key |
| `stake.skey`| staking signing key |
| `stake.addr`| registered stake address |
| `paymentwithstake.addr`| funded address linked to `stake` |


But the **Block Producer Node** needs to operate:

- **Cold** Key pair
- **VRF** Key pair
- **KES** Key pair
- **Operational Certificate**

As first step, we are going to create a directory to store Stake Pool Keys:

    mkdir pool-key

Hereinafter, we indicate the correct path to every key from the cardano-cli.

#### Cold Key pair

Grants the right to sign block to KES Key. Should not reside on a device that has internet connectivity. Allow to generate new _KES Period_.

    cardano-cli node key-gen \
        --cold-verification-key-file pool-key/cold.vkey \
        --cold-signing-key-file pool-key/cold.skey \
        --operational-certificate-issue-counter-file pool-key/cold.counter

| File  | Content |
| ------------- |:-------------:|
| `cold.vkey`| cold verification key |
| `cold.skey`| cold signing key |
| `cold.counter`| issue counter |

#### VRF Key pair

_Verificable Random Function_. Used to find out whether a node is a slot leader in on ongoing slot.

    cardano-cli node key-gen-VRF \
        --verification-key-file pool-keys/vrf.vkey \
        --signing-key-file pool-keys/vrf.skey

| File  | Content |
| ------------- |:-------------:|
| `vrf.vkey`| VRF verification key |
| `vrf.skey`| VRF signing key |


#### KES Key pair

_Key Envolving Signature_. Using to sign a new block. Expire periodically.

> About **Key Envolving Signature Period**
>> Means that after a certain _period_, the key will _evolve_ to a new key and discard it's old version. This is useful, because it means that even if an attacker compromises the key and gets access to the signing key, he can only use that to sign blocks from now on, but not blocks dating from earlier periods, making it impossible for the attacker to rewrite history.\
>> Unfortunately, there is a catch: A KES key can only evolve for a certain number od periods and becomes useless afterwards. This means that before that number of periods hs passed, the node operator has to generate a new KES Key pair, issue a new operational node certificate with that new key pair and restart the node with the new certificate.\

In order to find out how long one period is and for how long a key can evolve, we can look into the _genesis_ file:
```
    cat config/testnet-shelley-genesis.json
    
    > "slotsPerKESPeriod": 129600,
    > ...
    > "maxKESEvolutions": 62,  
    
```
This means that KES Period evolve after each period of `129600` slots and that it can evolve `62` times before it needs to be renewed.

Before we can create an operational certificate for our node, we need to figure out the start of the KES validity period, i.e. which KES evolution period we are in.

Let's go to check the current tip of the blockchain:

    cardano-cli query tip --testnet-magic 1097911063

    {
        "epoch": 181,
        "hash": "5e954221735917cbb018d70defee7cf61f9bf165163232376b0fa9dcb59be5bf",
        "slot": 48238480,
        "block": 3250580,
        "era": "Alonzo",
        "syncProgress": "100.00"
    }

We are currently in slot `48238480`, and we know from the genesis file that one period lasts for 129600 slots. So we calculate the current period by:

     expr 48238480 / 129600
    372

Create the **KES Key pair**

    cardano-cli node key-gen-KES \
        --verification-key-file pool-keys/kes.vkey \
        --signing-key-file pool-keys/kes.skey

| File  | Content |
| ------------- |:-------------:|
| `kes.vkey`| KES verification key |
| `kes.skey`| KES signing key |

Now we are able to generate an operational certificate for our stake pool:

    cardano-cli node issue-op-cert \
        --kes-verification-key-file pool-keys/kes.vkey \
        --cold-signing-key-file pool-keys/cold.skey \
        --operational-certificate-issue-counter pool-keys/cold.counter \
        --kes-period 372 \
        --out-file pool-keys/node.cert

| File  | Content |
| ------------- |:-------------:|
| `node.cert`| Operational certificate |


## Update our core node

`KES`, `VRF` and `operational certificate` it's just we need to empower our core node (Block Production Node) to producer blocks. 

So we may back to our `docker-compose.yml` and update with some more parameteres at `core-node` services.

    command: 
      - run
      - --config /config/testnet-config.json
      - --topology /config/testnet-topology.json
      - --database-path /db
      - --socket-path opt/cardano/ipc/node.socket
      - --host-addr 0.0.0.0
      - --port 3002
      - --shelley-kes-key pool-keys/kes.skey
      - --shelley-vrf-key pool-keys/vrf.skey
      - --shelley-operational-certificate pool-keys/node.cert

As you can see, in command parameters, we add path to `kes.skey`, `vrf.skey` and `node.cert`.

Upload to the server and update `docker-compose.yml`:

    docker-compose up -d

Please, check that the nodes are running fine as Docker Containers:

    docker ps


## Register Stake Pool with Metadata

Registering your stake pool requires:

- Create JSON file with your metadata and store it in the node and in a url you maintain
- Get the hash of your JSON file
- Generate the stake pool registration certificate
- Create a delegation certificate pledge
- Submit the certificates to the blockchain

> **WARNING:** Generating the stake pool registration certificate and the delegation certificate requires the cold keys. So, when doing this on mainnet you may want to generate these certificates in your local machine taking the proper security measures to avoid exposing your cold keys to the internet.

#### Create a JSON file with your pool's metadata

    {
      "name": "Novapago ADA Pool",
      "description": "The Novapago Testnet Cardano Stake Pool",
      "ticker": "NAP",
        "homepage": "https://www.novapago.com"
    }

Store the file in a path inside de `core node`.

Store the file in a url you control. You can use a GIST in github to store the definition and git.io to make it short. Ensure that the URL is less than 65 characters long.

Our example in Gist-URL:

[https://gist.githubusercontent.com/GasumSam/19e269b9b4e6277261cb8c47978e3a05/raw/c0d8cda9b958c47a898b2800d6c9a12796c039be/gistfile1.txt](https://gist.githubusercontent.com/GasumSam/19e269b9b4e6277261cb8c47978e3a05/raw/c0d8cda9b958c47a898b2800d6c9a12796c039be/gistfile1.txt)

[https://git.io/JyihM](https://git.io/JyihM)

#### Get the hash of your metadata JSON file:

This validate that the JSON fits the required schema, if it does, you will get the hash of your file.

    cardano-cli stake-pool metadata-hash --pool-metadata-file /data/nvpADAPool.json
    
    > 46cc4b898ae045705d3940200f0a3202ac7b8260865c5608af1ee124c975ca41

> Note: We use the path to `/data` folder in the `core node`.

This action return us the hash: `46cc4b898ae045705d3940200f0a3202ac7b8260865c5608af1ee124c975ca41`


#### Generate Stake pool registration certificate

Just we have all ingredients to get our registration certificate, and this is our template:


    cardano-cli stake-pool registration-certificate \
        --cold-verification-key-file pool-keys/cold.vkey \
        --vrf-verification-key-file pool-keys/vrf.vkey \
        --pool-pledge 70000000000 \
        --pool-cost 4321000000 \
        --pool-margin 0.04 \
        --pool-reward-account-verification-key-file myKeys/stake.vkey \
        --pool-owner-stake-verification-key-file myKeys/stake.vkey \
        --testnet-magic 1097911063 \
        --single-host-pool-relay 3.68.219.45 \
        --pool-relay-port 3001 \
        --metadata-url https://git.io/JyihM \
        --metadata-hash 46cc4b898ae045705d3940200f0a3202ac7b8260865c5608af1ee124c975ca41 \
        --out-file pool-keys/pool-registration.cert


| Parameter   | Explanation |
| ------------- |:-------------:|
| `cold-verification-key-file`| verification cold key |
| `vrf-verification-key-file`| verification VRS key |
| `pool-pledge`| pledge lovelace |
| `pool-cost`| operational costs per epoch lovelace |
| `pool-margin`| operator margin |
| `pool-reward-account-verification-key-file`| verification staking key for the rewards |
| `pool-owner-staking-verification-key-file`| verification staking keys for the pool owners |
| `pool-relay-ipv4`| relay node ip address |
| `pool-relay-port`| port |
| `metadata-url`| url of your json file |
| `metadata-hash`| the hash of pools json metadata file |
| `out-file`| output file to write the certificate to |

> **Note:** We can use a different key for the rewards, and can provide more than one owner key if there were multiple owners who share the pledge.

Let's go to check our registration certificate:

    cat /pool-keys/pool-registration.cert
    
    > {
        "type": "CertificateShelley",
        "description": "Stake Pool Registration Certificate",
        "cborHex": "8a03581cab16ce81507d4298dbb37532c3fb38397b42f0abd00e9a5caa9ab51158200f3163ec3de6d4f044461d31aaab51d174cc33f67513372e1b5ac224ee1d90911b000000104c533c001b00000001018d3a40d81e82011819581de087cea81c429ca692929029df25aa9a23acf8c3aa9f9c26beaae89f2d81581c87cea81c429ca692929029df25aa9a23acf8c3aa9f9c26beaae89f2d818301190bb96b332e36382e3231392e3435827468747470733a2f2f6769742e696f2f4a7969684d582046cc4b898ae045705d3940200f0a3202ac7b8260865c5608af1ee124c975ca41"
    }

#### Generate delegation certificate pledge

To honor your pledge, create a _delegation certificate_:

    cardano-cli stake-address delegation-certificate \
        --stake-verification-key-file myKeys/stake.vkey \
        --cold-verification-key-file myKeys/cold.vkey \
        --out-file pool-keys/delegation.cert

This creates a delegation certificate which delegates funds from all stake addresses associated with key `stake.vkey` to the pool belonging to cold key `cold.vkey`. If there are many staking keys as pool owners in the first step, we need delegation certificates for all of them.

#### Submit the pool certificate and delegation certificate to the blockchain

To submit the `pool registration certificate` and the `delegation certificates` to the blockchain by including them in one or more transactions. We can use one transaction for multiple certificates, the certificates will be applied in order. 

*  **Draft the transaction**

```
cardano-cli transaction build-raw \
    --tx-in c7aa7cc7582a2771faf50993ff9c458e24b6b82e46d0dfd5850c4743d62985aa#0 \
    --tx-out $(cat myAddress/paymentwithstake.addr)+0 \
    --ttl 0 \
    --fee 0 \
    --out-file data/tx.raw \
    --certificate-file pool-keys/pool-registration.cert \
    --certificate-file pool-keys/delegation.cert
```

* **Calculate the fees**

```
cardano-cli transaction calculate-min-fee \
    --tx-body-file data/tx.raw \
    --tx-in-count 1 \
    --tx-out-count 1 \
    --testnet-magic 1097911063 \
    --witness-count 1 \
    --byron-witness-count 0 \
    --protocol-params-file data/protocol.json
    
    
> 187149

```
Registering a stake pool requires a deposit. This amount is specified in `protocol.json`. For example, for Shelley Mainnet we have:

    "poolDeposit": 500000000

*  **Calculate the change for --tx-out**

All amounts in Lovelace

    expr <UTxO BALANCE> - <poolDeposit> - <TRANSACTION FEE>

* **Build the transaction:**

```
cardano-cli transaction build-raw \
    --tx-in <TxHash>#<TxIx> \
    --tx-out $(cat payment.addr)+<CHANGE IN LOVELACE> \
    --invalid-hereafter <TTL> \
    --fee <TRANSACTION FEE> \
    --out-file tx.raw \
    --certificate-file pool-registration.cert \
    --certificate-file delegation.cert
```
In our case:

```
cardano-cli transaction build-raw \
    --tx-in c7aa7cc7582a2771faf50993ff9c458e24b6b82e46d0dfd5850c4743d62985aa#0 \
    --tx-out $(cat data/paymentwithstake.addr)+994538096 \
    --ttl 46404997 \
    --fee 187149 \
    --certificate-file pool-keys/pool-registration.cert \
    --certificate-file pool-keys/delegation.cert \
    --out-file data/tx.raw
    
```
* **Sign the transaction:**

```
cardano-cli transaction sign \
    --tx-body-file data/tx.raw \
    --signing-key-file myKeys/payment.skey \
    --signing-key-file myKeys/stake.skey \
    --signing-key-file pool-keys/cold.skey \
    --testnet-magic 1097911063 \
    --out-file data/tx.signed
```

* **Submit the transaction**

```
cardano-cli transaction submit \
    --tx-file data/tx.signed \
    --testnet-magic 1097911063
```

* **Verify that your stale pool registration was succesful**

Get Pool ID

     cardano-cli stake-pool id --cold-verification-key-file pool-keys/cold.vkey --output-format "hex"
     
    > ab16ce81507d4298dbb37532c3fb38397b42f0abd00e9a5caa9ab511

Great! Our Stake Pool is registered on the testnet and you can check it.

Go to (e.g.) [https://pooltool.io/](https://pooltool.io/) and search for differents paramenters:

| Parameters | Data about our stake pool |
| ------------- |:-------------:|
| Pool ID | `ab16ce81507d4298dbb37532c3fb38397b42f0abd00e9a5caa9ab511` |
| Ticket | `NAP`|
| Name | `Novapago ADA Pool`|
| Description | `The Novapago Testnet Cardano Stake Pool`|


### Best practice
<details>

This is a learning exercise. Not a good example to run your staking pool in a completely secure way. Keep in mind:
    
Keep your `cold-keys` and `wallets` on a completely offline node, and then transfer all relevant registration transactions and `pool-keys` to the online block-producing node. This requires a bit more steps than the reasonably secure method.

So is recommended to run the nodes on seperate servers and connect them using their public or local network IP-addresses, if they run within the same network. The idea is to keep the block-producing node completely locked off from anything other than the relay node. The block-producing node will also initialize and register the stake pool automatically, which is better to do on a seperate node, to keep the `cold-keys` directory and `wallets` secret key files (`wallets/*/*.skey`) completely away from the online nodes.

For this setup you will need 3 hosts:

- `host1` for running the relay node.
- `host2` host for running the block-producing node and submitting the registration transactions.
- `host3` host for generating all the keys, addresses, certificates and transactions, and storing the cold-keys for refreshing the KES keys and certificates. This must be an completely offline host running locally.

Else interesting steps:

    1. Upload your stake-pool metadata json file ([See example](#metadata-example)) to a host so it is accessible to the public. For example as a [github gist](https://gist.github.com/).
    2. Start a relay node on `host1` and make it connect to the block-producing node on `host2`.
    3. Generate `protocol.json` on `host1` by running `get_protocol`.
    4. Transfer the `protocol.json` from `host1` to the staking directory of `host3`.
    5. Add the `metadata.json` file to `config/staking` directory the on `host3`
    6. Start a cold-creation node `host3` using the `--create-cold` argument, and follow steps.
    7. Fund your owners payment address(es) created on `host3`, make sure you send to the correct addresses.
    8. Get UTXO and TXIX for funded owners payment address(es) by quering the address(es) on `host1` or `host2`.
    9. Input the relevant UTXO and TXIX values when promted on `host3`.
    10. Find the slot tip of the blockchain by running `get_slot` on `host1` or `host2`.
    11. Input the slot tip on `host3` when prompted.
    12. Create Firewall rules for your block-producing node on `host2` to only accept incoming traffic from your relay node on `host1`.
    13. Wait for the block-producing node on `host2` to register your pool and start staking.

</details>


## Monitoring our Stake Pool

Prometheus is probably the best option to check metrics from nodes. In this case:

- Check communication port for `Prometheus` in our `config.json` file.

(Inside the `core-node`)
```
cat config/testnet-config.json

```
And check-in through what port `core-node` is sending metrics to `Prometheus`.

```
...
  "hasEKG": 12788,
  "hasPrometheus": [
    "0.0.0.0",
    12789
  ],
...
```
So we need to **open the communications on this port `12789` in our AWS Instances configurations**.

#### Add `Node Exporter`and `Prometheus` to our server

`Node Exporter` has the duty to broadcast metrics from the node to Prometheus, so a good practise is host it in the same `core-node` server and `Prometheus` in our local server.

In our case, thinking in our analogy, we tried to host `Node Exporter` inside the Docker Network and `Prometheus` in our AWS Linux Server including services in our `docker-compose.yml`
```
  ...
  
  node_exporter:
    container_name: node_exporter
    image: prom/node-exporter:latest

    networks:
      cardano_stake:
        aliases:
          - node_exporter
        ipv4_address: 172.20.0.22

    restart: unless-stopped

    volumes:
      - ./00node_exporter:/host:ro,rslave

    command:
      - '--path.rootfs=/host'

    ports:
      - 12789:12789

  prometheus:
    container_name: prometheus
    image: prom/prometheus:latest
    networks:
      - prometheus_net
      
    restart: unless-stopped

    volumes:
      - ./prometheus/:/etc/prometheus/

    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --storage.tsdb.path=/prometheus

    ports:
      - 9100:9100 

```


#### Setup Prometheus

As you can see, in `Prometheus` service we mount a volume from container to server folder to specify path to `prometheus.yml` config file, where we might detail our parameters like that:

```
global:
  scrape_interval:     5s

scrape_configs:
  - job_name: "core"  # To scrape data from the cardano node
    scrape_interval: 5s
    static_configs:
    - targets: ["0.0.0.0:12789"]

  - job_name: "node" # To scrape data from our node exporter to monitor
    static_configs:
    - targets: ["0.0.0.0:9100"]

remote_write:
    - url: "https://prometheus-prod-10-prod-us-central-0.grafana.net/api/prom/push"
      basic_auth:
        username: "6digitnumberforyourGrafanaUsername"
        password: "_YOURGrafanaToken_"

```

If our nodes has been on separate servers, we might to configure each particular IP Address in his scrape behavior function.

However, in this case, we open the possibility to connect with anyone to ease communication between `node_exporter` (part of Docker Network) and `Prometheus` (hosted in the same server). To finally broadcast metrics through Grafana Service `remote_write` parameter (Getted from Grafana Sign up service ).

And now you can go to [Grafana](https://grafana.com/oss/prometheus/) create a free account metrics and configure your `usename` and `password` parameters.

Go to [Grafana Sandbox](https://play.grafana.org/) in your session, select Explore (compass icon) and select your metric reporters.

Thanks for all! We invite you to know our job with `Cardano Wallet` also.

### Attribution




Created By: 

[<img align="left" src="https://styles.redditmedia.com/t5_4bhj33/styles/communityIcon_u389y4dxqov61.png" alt="Novapago" style="height: 100px; width:130px;"/>](https://novapago.com/)\
[Novapago](https://novapago.com/) 
\
\
\
\
At the frame of:
\
[<img align="left" src="https://upload.wikimedia.org/wikipedia/commons/2/21/Universidad_de_M%C3%A1laga.jpg" alt="UMA" style="height: 150px; width:150px;"/>](https://www.uma.es)\
\
[II Blockchain Technologies Course Final Project](https://www.nics.uma.es/Blockchain/)\
[UMA Nics Lab](https://www.nics.uma.es/)\
[UMA](https://www.uma.es/)\
\
\
Student: [José Manuel Guzmán](https://es.linkedin.com/in/josemanguzman)\
Mentors: [Rubén González](https://es.linkedin.com/in/gonzalezgomezruben?trk=public_profile_samename-profile), [Adrián Portugués](https://es.linkedin.com/in/adrian-portugues-mas-17a7a1bb?trk=public_profile_browsemap), [Silvia Briones](https://linkedin.com/) and [Francisco López](https://es.linkedin.com/in/francisco-manuel-lopez?trk=public_profile_browsemap)

Period: December 2021 / January 2022
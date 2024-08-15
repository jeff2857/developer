# How to deploy ETH LSD stack

This guide provides a step-by-step script to deploy the ETH LSD stack, without knowing details about contracts and services. Read the documentation for comprehensive understandings of these components.


# Step1. Deploy your LSD network contracts

61 Lab have made [Stack App](https://stack-app.stafi.io/) for project parties to deploy their LSD network contracts.

![](/image/ethlsd/onboarding_homepage.png "Launchpad Homepage")

In addition to *Stack App*, you can deploy contracts through explorer or source code: [See this doc](./deploy_lsd_network_contracts)

## Which relay type should I choose?

We provide two relay types, Standard and Custom. Standard is operated by StaFi Stack DAO and the other one by project party, but they do same work. It is a good start to use Standard to go through the whole process of creating your own LSD project. Then you are able to switch to custom relay anytime.

Standard:
- No Server Needed
- Able to change to custom at anytime
- Operated by StaFi Stack DAO

Custom:
- Server required
- Both Execution & Beacon chain RPC required
- At least run 3 relay instances
- Maintained by your own

## How to switch to custom relay service?

Switching to custom relay service is a big change to your LSD network. Here we list things you need prepare: 

1. Minimum 3 Voter Accounts
2. Servers to run relay service
3. Stable Execution & Beacon chain RPC
4. Have the permission of the owner of the network

Send a transaction signed by your owner wallet to `NetworkProposal.takeoverVoterManagement` with `new voter manager`, `new voters` and `threshold`. 

Next: Run Your relay services with voters you have set.


## Which Token Type should I choose?

Standard LSD Token:
- Provided by StaFi Stack
- ERC-20 compatible
- Ready to use

Several Procedures required before using Custom LSD Token:
1. Implement your own LST logic which comply with [Stack's standards](TBD: definition of LSD Token Standards).
2. Deploy your contract
3. Request 61Lab for whitelisting your token for security reason

## Parameter Tips

Owner Address: sets the owner of the LSD network being created.

Owner Permissions:
- Upgrade contracts
- Adjust commission fee
- Adjust duration of era
- Nominate voter manager
- Adjust parameters

Voter Number:
- How many voters you intend to set
- Minimum number is 3, highly recommend to set more than 8
- Each voter proposes new states of the network
- The more voters you set the more decentralized advantages the network gets

Threshold:
- The minimum number of votes required to approve a proposal
- Recommended value is (voter number*2/3)
- Minimum number is (voter number + 1)/2
- Maximum number is voter number which is not recommended


## Why LSD Token could not be changed?

As being the heart of the LSD network, LSD token must be stable, reliable and trustable. Users hold LSD Token as a receipt of staking so it could not be changed.

## Rewards Distribution

Rewards distribution is crucial to project parties. Commission fee of users is set default as 10% and StaFi Stack fee is 10% of project income. for example, if `100ETH` rewards received from Ethereum, the distribution will be:

| Role | Amount | Formula |
|---------------|--------|-------|
|  Users        |  90ETH | 100ETH*(1-0.1)  |
|  Project      |  9ETH  | (100ETH-90ETH)*(1-0.1)  |
|  StaFi Stack  |  1ETH  | 100ETH-90ETH-9ETH  |

## Save all the information generated

The lsd network has a set of smart contracts, so you should save all the information which you will interact frequently with, such as ETH LSD App, Validator App and Relay service.

- LSD Network Factory address
- LSD Token address
- Admin address
- Fee Pool address
- Network Balances address
- Network Proposal address
- Network Withdraw address
- Node Deposit address
- User Deposit address
- Voters Address

# Step2. Run relay service

## Install Build Tools

Install `make`, `gcc` and `git`

```bash
sudo apt update
sudo apt install -y make gcc git build-essential
```

Install `go` by following the [official docs](https://golang.org/doc/install). Remember to set your `$PATH` environment variable, for example:

```bash
cd $HOME
wget -O go1.22.0.linux-amd64.tar.gz https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz && rm go1.22.0.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```

## Install relay service

```bash
$ git clone https://github.com/stafiprotocol/eth-lsd-relay.git
$ cd eth-lsd-relay
$ make install
```

## Create relay working directory and config file

```bash
$ mkdir -p ~/eth-stack
$ cp conf.template.toml ~/eth-stack/config.toml
```

## Config eth relay service

Update config (config.toml) by your favorite editor according to [Relay Config](./relay.html#config)

```toml
account = "0x68146ebA486CE6F8D22731c8ECB4d013F34E7114"
trustNodeDepositAmount     = 1  # ether
eth2EffectiveBalance       = 32 # ether
maxPartialWithdrawalAmount = 8  # ether
gasLimit = "3000000"
maxGasPrice = "600000000000"                            #wei
batchRequestBlocksNumber = 32
runForEntrustedLsdNetwork = false

[pinata]
apikey  = "AAAAA.BBBBB.CCCCC"
pinDays = 180

[contracts]
lsdTokenAddress = "0x549aF761C1c72f3cd2Be966e76B778339Bf746DD"
lsdFactoryAddress = "0xe2CF966b041904eFfb8Fe83E317CAF4dd27d8CBc"

[[endpoints]]
eth1 = "https://holesky.stafi.io"
eth2 = "https://holesky-beacon.stafi.io"

[[endpoints]]
eth1 = "https://holesky2.stafi.io"
eth2 = "https://holesky-beacon2.stafi.io"
```

## Import voter accounts

Let's import your 3 voter accounts so that it can be used by relay service

```bash
$ eth-lsd-relay import-account --base-path ~/eth-stack
Enter private key:
>
password for key:
>
INFO[0007]
key imported address=0x68146ebA486CE6F8D22731c8ECB4d013F34E7114
file=CWD/keys/0x68146ebA486CE6F8D22731c8ECB4d013F34E7114.key
```


## Start relay services

```bash
$ eth-lsd-relay start --base-path ~/eth-stack
Enter password for key ./keys/0x68146ebA486CE6F8D22731c8ECB4d013F34E7114.key:
>

INFO[2023-10-27T14:51:08+08:00] update balances epochs                        
distributePriorityFeeDuEpochs=225 
distributeWithdrawalsDuEpochs=225 
merkleRootDuEpochs=225 
submitBalancesDuEpochs=225
INFO[2023-10-27T14:51:08+08:00] nodeCommission rate: 0.05, platformCommission rate: 0.05
INFO[2023-10-27T14:51:12+08:00] start services...
INFO[2023-10-27T14:51:12+08:00] start vote service
INFO[2023-10-27T14:51:12+08:00] start sync service
INFO[2023-10-27T14:52:43+08:00] synced block: 9935000
```


# Step3. Deploy Your LSD app

[Follow this the doc to deploy your own LSD app](ethlsdapp.html)

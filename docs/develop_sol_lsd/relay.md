# Relay

Given the limitation of smart contracts, they are not self-executing, and require an external force to trigger their functions. Relay is a service to drive LSD network to properly process its own internal states, such as dealing with delegating, undelegating, distributing rewards and calculating the rate between LST and Sol. We introduce `era` concept to define how often in epochs the smart contract should be called.

*Security* is the our first priority when developing Stack, We are thrilled to say that the new era process is permissionless, showcasing the decentralized nature of the StaFi Sol LSD Stack, allowing anyone to trigger the new era process.

## Install
### Install build tools

Install `make`, `gcc` and `git`

```bash
sudo apt update
sudo apt install -y make gcc git build-essential
```

Install `go` by following the [official docs](https://golang.org/doc/install). Remember to set your `$PATH` environment variable, for example:

```bash
cd $HOME
wget -O go1.22.6.linux-amd64.tar.gz https://go.dev/dl/go1.22.6.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.22.6.linux-amd64.tar.gz && rm go1.22.6.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```

### Install relay service

```bash
git clone https://github.com/stafiprotocol/solana-lsd-relay.git
cd solana-lsd-relay
make install
```

## Config
### Add account

```bash
$ solana-lsd-relay keys gen --keys 1 

You will be asked to provide a passphrase to secure your newly created vault.
Make sure you make it long and strong.

Enter passphrase to encrypt your solana vault: 
Confirm passphrase: 

Wallet file "./keys/solana_keys.json" written to disk.
Here are the keys that were ADDED during this operation (use `list` to see them all):
- B5gfiMdYkcerAPD5depiTb4JpJ1F9CfYG5UukFrUhhiW
Total keys stored: 1
```

### Or import existing account
```bash
$ solana-lsd-relay keys import 

PLEASE READ:
We are now going to ask you to paste your private keys, one at a time.
They will not be shown on screen.
Please verify that the public keys printed on screen correspond to what you have noted

Paste your first private key: 
- Scanned private key corresponding to Gi2ommjX7QKTHPsyr96XSPoBDzzNnS4rtjMp4B4eEXas
Paste your next private key or hit ENTER if you are done: 

You will be asked to provide a passphrase to secure your newly created vault.
Make sure you make it long and strong.

Enter passphrase to encrypt your solana vault: 
Confirm passphrase: 

Wallet file "./keys/solana_keys.json" written to disk.
Here are the keys that were ADDED during this operation (use `list` to see them all):
- Gi2ommjX7QKTHPsyr96XSPoBDzzNnS4rtjMp4B4eEXas
Total keys stored: 1
```

### Create and update config

```bash
$ cp config_start.example.toml config.toml
$ vim config.toml

EndpointList = ["https://api.devnet.solana.com"]
KeystorePath = "./keys/solana_keys.json"

LsdProgramID = "795MBfkwwtAX4fWiFqZcJK8D91P9tqqtiSRrSNhBvGzq"

StackAddress = "EpgquacesXp8h7nk9j5KcnzDVATFKLE995cuhj1hRbpR"
StakeManagerAddress = "6g8ziuefcXnmP1kmCXd7CGxdQrL59CkdnH9C6vQnQYkp" # entrusted mode if empty

## signers
FeePayerAccount = "ErCsmTQhM9qt17ce8XDTasKo76FEdg2scP2UX4VLxKDb"
```

## Start relay service

```bash
$ solana-lsd-relay start --config config.toml --log_level info
```

## Config

| config | description | example value |
| --- | --- | --- |
| endpointList | a list of rpc endpoints | |
| keystorePath | keystore file path | |
| lsdProgramID | lsd program id **provided by 61Lab** | |
| stackAddress | stack address **provided by 61Lab** |  |
| stakeManagerAddress | stake manager address<br/>created through **61Lab stack app**  |  |
| feePayerAccount | imported or generated account<br/>**must have enough funds to sign transactions**|  |

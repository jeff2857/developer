# Validator Selection AI Agent

This module positioned as a validator delegation solution that streamlines the process of validator selection and delegation.

## Compatibility

- CosmosHub *supported*
- CosmosSdk-based networks on *testing phase*
- EVM networks *planned*

## Install Build Tools

To set up the development environment, you need to install the following build tools:

Install `make`, `gcc` and `git`

```bash
sudo apt update
sudo apt install -y make gcc git build-essential
```

### Go

Install `go` by following the [official docs](https://golang.org/doc/install). Remember to set your `$PATH` environment variable, for example:

```bash
cd $HOME
wget -O go1.22.5.linux-amd64.tar.gz https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz && rm go1.22.5.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bashrc
echo 'export GOPATH=$HOME/go' >> $HOME/.bashrc
echo 'export GO111MODULE=on' >> $HOME/.bashrc
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bashrc && . $HOME/.bashrc
go version
```

### Python3

Install Python3 and pip:

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.12
python3 --version # Verify Installation
```

## LLM Agent

```bash
$ git clone git@github.com:stafiprotocol/llm-agent.git
$ cd llm-agent
```

### Create config file

```bash
$ cp .env.example .env
```

```env
OPENAI_API_KEY=
OPENAI_BASE_URL=
HUGGINGFACE_API_KEY=
HUGGINGFACE_BASE_URL=https://api-inference.huggingface.co/models
CLAUDE_API_KEY=
CLAUDE_BASE_URL=
LISTEN_PORT=6000
LISTEN_HOST="127.0.0.1"
LOG_LEVEL="DEBUG"
LOG_FORMAT="JSON"
LOG_SERVICE_NAME="llm_agent_app"
GEMINI_API_KEY=
```

### Start llm-agent service

1. Create a virtual environment:

```bash
python3 -m venv llm-agent
```

2. Activate the virtual environment:

```bash
source llm-agent/bin/activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Start the service:

```bash
python3 app.py
```

## API & Syncer Service

API and blockchain data accumulation service used to provide validator election

```bash
$ git clone git@github.com:stafiprotocol/staking-election-cosmos.git
$ cd staking-election-cosmos
$ make build
```

### Create working directory and config file

Working directory stores config

```bash
$ mkdir -p ~/staking-election-cosmos
$ cp config_api_example.toml ~/staking-election-cosmos/config_api.toml
$ cp conf_election_syncer.example.toml ~/staking-election-cosmos/conf_election_syncer.toml
```

Config staking-election-cosmos api service

```toml
logFilePath = ""

[apiServer]
listenAddr = "127.0.0.1:8085"
agentEndpoint = "http://127.0.0.1:6000" # The Validator Selection AI Agent API is the API of the python service that will be deployed below
ginMode= "release" # debug/release/test

[pinata]
apikey = ""
```

Config staking-election-cosmos syncer service

```toml
logFilePath = ""

[[chainInfo]]
prefix = "cosmos" # network account prefix
startHeight = 0 # starting from the latest height.If you specify a height, synchronization starts from the specified height
endpointList = ["https://mainnet-rpc.wetez.io:443/cosmos/tendermint/v1/apikey"]
batchLimit=100 # The number of blocks synchronized each time, and the data will be updated to the Pinata IPFS service after the synchronization is completed


[pinata]
apikey = "" # Pinata API Key
```


### Start staking-election-cosmos service

```bash
# start api
$ ./build/staking-election-cosmos start-api --config ~/staking-election-cosmos/config_api.toml

config path: conf_api.toml
load config success
all logs are output in the ./log_data/api directory
api logLevelStr: info
api config info: config.APIServer{GinMode:"debug", ListenAddr:"127.0.0.1:8085", AgentEndpoint:"http://127.0.0.1:6000"}
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /election/swagger/*any    --> github.com/swaggo/gin-swagger.CustomWrapHandler.func1 (4 handlers)
[GIN-debug] POST   /election/api/v1/selectedValidators --> github.com/stafiprotocol/staking-election-cosmos/api/election_handlers.(*Handler).HandleGetSelectedValidators-fm (4 handlers)
[GIN-debug] POST   /election-internal/api/v1/selectedValidators/ --> github.com/stafiprotocol/staking-election-cosmos/api/election_handlers.(*Handler).HandleInternalGetSelectedValidators-fm (4 handlers)
INFO[2024-07-29T18:55:04+08:00] Gin server start on 127.0.0.1:8085
```

```bash
# start syncer
$ ./build/staking-election-cosmos start-election-cosmos --config ~/staking-election-cosmos/conf_election_syncer.toml

config path: conf_election_syncer.toml
load config success
all logs are output in the ./log_data/task directory
INFO[2024-07-29T18:56:22+08:00] task CacheValidator start ----------->        prefix=cosmos
```

### Selected Validators API

URI: `POST /election/api/v1/selectedValidators`

Parameters

| Name                    | Type   | Required | Default | Description                                                         |
| ----------------------- | ------ | -------- | ------- | ------------------------------------------------------------------- |
| modelId                 | string | Yes      | -       | The ID of the model to use for selection                            |
| prefix                  | string | Yes      | -       | The prefix to filter validator addresses                            |
| resultNum               | number | No       | -       | The number of validators to return (1-20)                           |
| maxCommissionRate       | number | No       | 0.1     | Maximum allowed commission rate (10%)                               |
| minCommissionStability  | number | No       | 0.9     | Minimum allowed commission stability (90%)                          |
| maxMaxCommissionRate    | number | No       | 0.5     | Maximum allowed potential commission rate ceiling (50%)             |
| maxCommissionChangeRate | number | No       | 0.2     | Maximum allowed commission change rate in a single adjustment (20%) |
| minSignBlockRatio       | number | No       | 0.998   | Minimum allowed historical online percentage (99.8%)                |

```bash
$ curl --location --request POST 'http://127.0.0.1:8085/election/api/v1/selectedValidators' \
--header 'Content-Type: application/json' \
--data-raw '{
  "modelId": "gpt-4o",
  "prefix": "cosmos",
  "resultNum": 5
}'

{
    "data": {
        "modelId": "gpt-4o",
        "recommendedValidators": [
            {
                "operatorAddress": "celestiavaloper1ac4mnwg79gyvd0x5trl2fgjv07lgfas02jf378",
                "moniker": "P-OPS Team",
                "reasons": [
                    "Low and stable commission rate of 5%",
                    "High uptime of 100%",
                    "Significant total stake and voting power",
                    "High commission stability of 84.5%"
                ]
            },
            {
                "operatorAddress": "celestiavaloper1u825srldhev7t4wnd3hplhrphahjfk7ff3wfdr",
                "moniker": "PRYZM | StakeDrop",
                "reasons": [
                    "Low commission rate of 5%",
                    "High uptime of 100%",
                    "Stable commission with no changes",
                    "Significant total stake and voting power",
                    "High commission stability of 85.7%"
                ]
            },
            {
                "operatorAddress": "celestiavaloper107lwx458gy345ag2afx9a7e2kkl7x49y3433gj",
                "moniker": "Enigma",
                "reasons": [
                    "Low commission rate of 5%",
                    "High uptime of 100%",
                    "Significant total stake and voting power",
                    "High commission stability of 83%"
                ]
            },
            {
                "operatorAddress": "celestiavaloper1sou3hrt4wtlxpj7m29th3flkc9rn7j6p0f378d",
                "moniker": "Smart Stake 🗂📊",
                "reasons": [
                    "Low commission rate of 5%",
                    "High uptime of 100%",
                    "High commission stability of 96.2%",
                    "Very high commission stability"
                ]
            },
            {
                "operatorAddress": "celestiavaloper19v94c3z7ckarwsum76kaagma0wqsqhh5nl5zqg",
                "moniker": "Validatus",
                "reasons": [
                    "Low commission rate of 5%",
                    "High uptime of 100%",
                    "Significant total stake and voting power",
                    "High commission stability of 91.3%"
                ]
            }
        ]
    },
    "message": "success",
    "status": "80000"
}
```

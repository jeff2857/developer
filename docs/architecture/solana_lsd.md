# Solana LSD Stack

Solana LSD Stack is devoted to help developers building liquid staking project on Solana ecosystem and designed to provide a comprehensive set of solutions related to commission fee management, validator set management, and user operations: stake, unstake, withdraw.

![StaFi Solana LSD Program Architecture](/image/sol_lsd/overview.png 'StaFi Solana LSD Program Architecture')

## Program

In the Solana ecosystem, "smart contracts" are called programs. Each program is an on-chain account that stores executable logic, organized into specific functions referred to as instructions.

Solana LSD program creates StakeManager and StakePool accounts for each LSD network.

Core Accounts:
- StakeManager: the main account of the LSD network that stores all the states such as: rate, validators and commission fee
- StakePool: an escrow account manages funds between stakers and native staking pool
- UnstakeAccount: an account stores unstake info such as: recipient and amount

Core roles:
- Owner: manages the LSD network

## Stakers flows

### Stake Flow

Users can stake SOL to the LSD network via `stake` method, and users will receive equivalent amount of LST.

`lstAmount = LstTotalSupply / TotalSolAmount * stakingAmount`

![Stake Flow](/image/sol_lsd/user_stake_flow.png 'Stake Flow')

### Unstake Flow

Any LsdToken holders are valid users, and can call `unstake` method to exchange SOL with LST.

![Unstake Flow](/image/sol_lsd/user_unstake_flow.png 'Unstake Flow')

### Withdraw Flow

After waiting the unbonding period, users can get their rewards by calling `withdraw` method.

![Withdraw Flow](/image/sol_lsd/user_withdraw_flow.png 'Withdraw Flow')

## Balance Staking Pools

The default balancer is the project administrator, it is able to transfer to another account. Balancer can redelegate assets to another validator to keep the network in a healthy(well-balanced) status.

## Relay Service

Due to the limitation of smart contract, it could not launch an execution. So StaFi Solana LSD Stack introduces Relay service. It will trigger StakeManager, at a certain interval, to collect and calculate users' reward, distribute it to the project and users.


![Relay Service](/image/sol_lsd/relay_service_new_era_flow.png 'Relay Service')

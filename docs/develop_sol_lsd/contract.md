# Contract

Contract holds all LSD functionalities, it consists with many stake manager and stake pool which represent a individual project party associated with an admin account who is privileged to config stake manager's parameters.

### Project party
- `initialize_stake_manager`: create and initialize stake manager and stake pool 
- `transfer_stake_manager_admin`: transfer LSD project's admin permission to another account
- `transfer_balancer`: transfer LSD project's balancer permission to another account
- `set_min_stake_amount`: config the minimum stake amount of the LSD project
- `set_unbonding_duration`: config unbonding duration in epochs
- `set_platform_fee_commission`: config the commission fee
- `add_validator`: add validator to the lsd network
- `remove_validator`: remove validator from the lsd network

### User

- `stake`: stakers stake an amount of Sol to the LSD project and receive LST in return 
- `unstake`: anyone who owns LSD token can call this function, LSD token will be burnt and users have to wait unbonding period of time to withdraw their assets
- `withdraw`: When unstake become mature, users can withdraw

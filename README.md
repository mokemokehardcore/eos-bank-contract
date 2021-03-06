# bank

This contract is able to deposit and withdraw EOS token.

## Usage

### Check Cleos Available

```
cleos --help
> Command Line Interface to EOSIO Client
> Usage: cleos [OPTIONS] SUBCOMMAND
> ...
```

### Set Kylin Testnet Endpoint

```
alias cleos='cleos -u https://api-kylin.eosasia.one:443'
```

### Check Connection

```
cleos get info
> {
>   "server_version": "0f6695cb",
>   "chain_id": "5fff1dae8dc8e2fc4d5b23b2c7665c97f9e9d8edf2b6485a86ba311c25639191",
> ...
```

### Check Your Account

Please replace `accountname1` your account.

```
cleos get account accountname1
> ...
> EOS balances:
>      liquid:          755.6145 EOS
>      staked:            0.0000 EOS
>      unstaking:         0.0000 EOS
>      total:           755.6145 EOS
```

### Transfer EOS Token to Contract

`accountname1` deposit 0.1000 EOS.

`bank` contract is deployed in `toycashio123` account.

```
cleos push action eosio.token transfer '["accountname1", "toycashio123", "0.1000 EOS", "deposit EOS"]' -p accountname1@active
> executed transaction: bea7bf80...3b  136 bytes  1681 us
> #   eosio.token <= eosio.token::transfer
>  {"from":"accountname1","to":"toycashio123","quantity":"0.1000 EOS","memo":"deposit EOS"}
> #  accountname1 <= eosio.token::transfer
>  {"from":"accountname1","to":"toycashio123","quantity":"0.1000 EOS","memo":"deposit EOS"}
> #  toycashio123 <= eosio.token::transfer
>  {"from":"accountname1","to":"toycashio123","quantity":"0.1000 EOS","memo":"deposit EOS"}
> ...

cleos get table toycashio123 toycashio123 balance
> {
>   "rows": [{
>       "balance": "0.1000 EOS"
>     }
>   ],
>   "more": false
> }

cleos get account accountname1
> ...
> EOS balances:
>      liquid:          755.5145 EOS
>      staked:            0.0000 EOS
>      unstaking:         0.0000 EOS
>      total:           755.5145 EOS
```

### Withdraw EOS Token from Contract

`accountname1` withdraw 0.1000 EOS.

```
cleos push action toycashio123 withdraw '["accountname1", "0.1000 EOS"]' -p accountname1@active
> executed transaction: 3f06a6ae...8a  120 bytes  2183 us
> #  toycashio123 <= toycashio123::withdraw
>  {"user":"accountname1","quantity":"0.1000 EOS"}
> #   eosio.token <= eosio.token::transfer
>  {"from":"toycashio123","to":"accountname1","quantity":"0.1000 EOS","memo":"withdraw"}
> #  toycashio123 <= eosio.token::transfer
>  {"from":"toycashio123","to":"accountname1","quantity":"0.1000 EOS","memo":"withdraw"}
> #  accountname1 <= eosio.token::transfer
>  {"from":"toycashio123","to":"accountname1","quantity":"0.1000 EOS","memo":"withdraw"}
> ...

cleos get table toycashio123 toycashio123 balance
> {
>   "rows": [],
>   "more": false
> }

cleos get account accountname1
> ...
> EOS balances:
>      liquid:          755.6145 EOS
>      staked:            0.0000 EOS
>      unstaking:         0.0000 EOS
>      total:           755.6145 EOS
```

## Deploy Yourself

### Set Account Permission

add "eosio.code" permission to your contract account,
because this contract execute inline action
by using itself permission in contract.

Replace `yourcontract` into your contract account name and
`EOS_PUBLIC_KEY` into a EOS public key which you know corresponding private key.

```
cleos set account permission yourcontract active '{
    "threshold": 1,
    "keys": [{
        "key": "EOS_PUBLIC_KEY",
        "weight": 1
    }],
    "accounts": [{
        "permission": {
            "actor": "yourcontract",
            "permission": "eosio.code"
        },
        "weight": 1
    }]
}' -p yourcontract@owner
```

### Compile

compile cpp into wasm

```
eosio-cpp -o bank.wasm bank.cpp -abigen
```

### Set Contract

Replace `PATH_TO_THIS_DIRECTORY` into absolute pass to current directory.
Note that you do not change this directory name `bank` into another one.

```
cleos set contract yourcontract PATH_TO_THIS_DIRECTORY/bank/ -p yourcontract@active
```

# The Box: A Flexible, Secure DeFi Vault for the Osmosis Ecosystem

## Introduction:
The Box is a new type of DeFi vault designed specifically for the Osmosis platform, leveraging the power of CosmWasm smart contracts. It aims to provide a secure, flexible, and user-friendly way for anyone to allocate their capital and benefit from the strategies of experienced managers.

## Features:
1. Flexibility: Box managers have the freedom to perform any whitelisted actions, allowing them to adapt to market conditions and maximize returns for users.

2. Security: The Box contracts are non-replaceable by administrators, ensuring the safety of users' funds. Managers cannot directly withdraw funds from the vault.

3. Transparency: All Boxes and their whitelisted actions are publicly viewable on boxen.zone, enabling users to make informed decisions.

4. User-Friendly: Users can easily subscribe to Boxes by depositing accepted forms of capital, referred to as funding capital.

5. Manager Incentives: Managers are well-compensated for their efforts, aligning their interests with those of the users and encouraging active, profitable management of the Boxes.

6. Cross-Chain Potential: While initially designed for Osmosis, the Box can be applied to other environments where CosmWasm is used.

## Problem it Solves:
The Box addresses two main issues in the current DeFi landscape:

1. Inadequacy of existing solutions for managing concentrated liquidity, which often lack flexibility and adaptability to market conditions.

2. The high capital requirements and complexity of building automated trading systems, which limit the ability of individual users to benefit from advanced strategies.

By providing a secure, flexible, and accessible platform for users to allocate their capital, the Box democratizes access to sophisticated DeFi strategies and empowers managers to deliver optimal results.

The Box represents a significant step forward in the evolution of DeFi vaults, combining the security and transparency of CosmWasm smart contracts with the flexibility and user-friendliness needed to attract a wide range of users. By aligning the interests of managers and users, the Box has the potential to revolutionize the way people interact with and benefit from the Osmosis ecosystem and beyond.

Here is the prototype Sylvia code for the Box contract, based on the code snippets provided:

```rust
use cosmwasm_std::{DepsMut, Env, MessageInfo, Response, StdError, StdResult, Uint128};
use cw_storage_plus::{Item, Map};
use schemars::JsonSchema;
use serde::{Deserialize, Serialize};
use sylvia::contract;
use sylvia::types::{ExecCtx, InstantiateCtx};

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct InstantiateMsg {
    pub managers: Vec<String>,
    pub strategy: String,
    pub whitelisted_assets: Vec<String>,
    pub carry_percentage: u64,
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct DepositMsg {
    pub asset: String,
    pub amount: Uint128,
}

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, JsonSchema)]
pub struct WithdrawMsg {
    pub share: Uint128,
}

pub struct BoxContract<'a> {
    pub managers: Map<&'a str, bool>, 
    pub strategy: Item<String>,
    pub whitelisted_assets: Map<&'a str, bool>,
    pub carry_percentage: Item<u64>,
    pub total_shares: Item<Uint128>,
    pub user_shares: Map<&'a str, Uint128>,
    pub assets: Map<&'a str, Uint128>,
}

#[contract]
impl BoxContract<'_> {
    pub const fn new() -> Self {
        Self {
            managers: Map::new("managers"),
            strategy: Item::new("strategy"),  
            whitelisted_assets: Map::new("whitelisted_assets"),
            carry_percentage: Item::new("carry_percentage"),
            total_shares: Item::new("total_shares"),
            user_shares: Map::new("user_shares"),
            assets: Map::new("assets"),
        }
    }
    
    #[msg(instantiate)]
    pub fn instantiate(&self, ctx: InstantiateCtx, msg: InstantiateMsg) -> StdResult<Response> {
        for manager in msg.managers {
            self.managers.save(ctx.deps.storage, &manager, &true)?;
        }
        self.strategy.save(ctx.deps.storage, &msg.strategy)?;
        for asset in msg.whitelisted_assets {
            self.whitelisted_assets.save(ctx.deps.storage, &asset, &true)?;
        }
        self.carry_percentage.save(ctx.deps.storage, &msg.carry_percentage)?;
        self.total_shares.save(ctx.deps.storage, &Uint128::zero())?;
        Ok(Response::new())
    }
    
    #[msg(exec)]
    pub fn deposit(&self, ctx: ExecCtx, msg: DepositMsg) -> StdResult<Response> {
        if !self.whitelisted_assets.has(ctx.deps.storage, &msg.asset) {
            return Err(StdError::generic_err("Asset not whitelisted"));
        }
        let shares = if self.total_shares.load(ctx.deps.storage)? == Uint128::zero() {
            msg.amount
        } else {
            // Calculate proportional shares
            todo!() 
        };
        self.assets.update(ctx.deps.storage, &msg.asset, |balance: Option<Uint128>| -> StdResult<_> {
            Ok(balance.unwrap_or_default() + msg.amount)
        })?;
        self.user_shares.update(ctx.deps.storage, &ctx.info.sender.to_string(), |balance: Option<Uint128>| -> StdResult<_> {
            Ok(balance.unwrap_or_default() + shares)
        })?;
        self.total_shares.update(ctx.deps.storage, |total| -> StdResult<_> {
            Ok(total + shares)
        })?;
        Ok(Response::new())
    }
    
    #[msg(exec)]
    pub fn withdraw(&self, ctx: ExecCtx, msg: WithdrawMsg) -> StdResult<Response> {
        // Ensure user has enough shares
        let user_share = self.user_shares.load(ctx.deps.storage, &ctx.info.sender.to_string())?;
        if user_share < msg.share {
            return Err(StdError::generic_err("Insufficient shares"));
        }
        // Calculate and transfer proportional assets
        todo!()
    }  

    #[msg(exec)]
    pub fn execute_strategy(&self, ctx: ExecCtx) -> StdResult<Response> {
        if !self.managers.has(ctx.deps.storage, &ctx.info.sender.to_string()) {
            return Err(StdError::generic_err("Unauthorized"));
        }
        // Execute whitelisted transactions according to strategy  
        todo!()
    }

    #[msg(exec)]
    pub fn kill(&self, ctx: ExecCtx) -> StdResult<Response> {
        if !self.managers.has(ctx.deps.storage, &ctx.info.sender.to_string()) {
            return Err(StdError::generic_err("Unauthorized"));
        }
        // Distribute assets to users proportionally and self-destruct
        todo!()
    }
}
```

Note: The `todo!()` macros indicate places where additional logic needs to be implemented, such as calculating proportional shares, transferring assets during withdrawals, executing whitelisted transactions based on the strategy, and distributing assets when the contract is killed.

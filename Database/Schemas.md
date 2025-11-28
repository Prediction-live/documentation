All the above schemas are part of the `public` table. All columns come with a `created_at` and `updated_at` columns of type `timestamptmz`
# Core
## users
The `users` table is the core table to handle users.

| Field      | Type        | Description                                     |
| ---------- | ----------- | ----------------------------------------------- |
| id         | uuid PK     |                                                 |
| email      | text        |                                                 |
| username   | text        |                                                 |
| status     | enum        | active, suspended, banned, pending_kyc, deleted |

## privy_accounts
Isolates external auth identifiers and states from app user. Each row represents an mean of authentication of an user. For example, if a user authenticates via Google and Twitch, I will have two rows, linked to the same `users` row
 
| Field         | Type         | Description                      |
| ------------- | ------------ | -------------------------------- |
| id            | uuid PK      |                                  |
| user_id       | uuid         | FK to users                      |
| privy_user_id | text         | User ID provided by Privy        |
| auth_provider | text         | Connecting with Google: "google" |
| last_login_at | timestamptmz |                                  |

## wallets
Many-to-one wallets per user, supports export, recovery (multi-chain)

| Field           | Type    | Description                                                   |
| --------------- | ------- | ------------------------------------------------------------- |
| id              | uuid PK |                                                               |
| user_id         | uuid    | FK to users                                                   |
| chain           | enum    | Chain name, default "solana"                                  |
| type            | enum    | Type of wallet ("external" or "embedded", default "embedded") |
| address         | text    | The address of the wallet                                     |
| privy_wallet_id | text    |                                                               |
| is_primary      | bool    |                                                               |

## matches
Anchor for markets, status drives market lifecycle gates

| Field      | Type    | Description                                                         |
| ---------- | ------- | ------------------------------------------------------------------- |
| id         | uuid PK |                                                                     |
| game       | enum    | lol, cs2, r6, rl, cod7, fortnite, apex, just_talking, special_event |
| starts_at  | int8    | now()                                                               |
| ends_at    | int8    | nullable                                                            |
| status     | enum    | scheduled, live, ended, void                                        |

## markets
Unit of prediction with state machine implementation (open -> lock -> resolve) 

| Field          | Type    | Description                             |
| -------------- | ------- | --------------------------------------- |
| id             | uuid PK |                                         |
| match_id       | uuid    | FK to matches                           |
| question       | text    |                                         |
| option_a_label | text    |                                         |
| option_b_label | text    |                                         |
| opens_at       | int8    | POSIX time                              |
| closes_at      | int8    | POSIX time                              |
| status         | enum    | scheduled, open, locked, resolved, void |
| resolution     | enum    | A, B, nullable                          |
| resolver_tx    | text    | nullable                                |

## market_pools
Pari-mutuel pool accounting per market and option. Clean separation between market metadata and stake accounting

| Field              | Type    | Description                                                             |
| ------------------ | ------- | ----------------------------------------------------------------------- |
| id                 | uuid PK |                                                                         |
| market_id          | uuid    | FK to markets                                                           |
| option             | enum    | A, B                                                                    |
| total_stake_atomic | float8  | The sum (in lamports) of all amounts placed at resolution lock time     |
| fee_bps_snapshot   | float8  | Fee (in lamports) taken by the platform at the moment of the resolution |


## bets
Individual user entries into a market option, immutable bet intent + on-chain linkage (-> aim at creating reconciliation process too)

| Field        | Type    | Description                                |
| ------------ | ------- | ------------------------------------------ |
| id           | uuid PK |                                            |
| market_id    | uuid    | FK to markets                              |
| user_id      | uuid    | FK to users                                |
| wallet_id    | uuid    | FK to wallets                              |
| option       | enum    | A, B                                       |
| stake_atomic | float8  | Amount (in lamports) of the bet            |
| tx_sig       | text    | Signature of the transaction               |
| placed_at    | int8    | POSIX time                                 |
| status       | enum    | pending, confirmed, refunded, settled      |
| error_reason | string  | nullable. Short string to detail the error |


## **settlements**
Results per market and per user bet

| Field             | Type    | Description                                                       |
| ----------------- | ------- | ----------------------------------------------------------------- |
| id                | uuid PK |                                                                   |
| market_id         | uuid    | FK to markets                                                     |
| bet_id            | uuid    | FK to bets                                                        |
| outcome           | enum    | win, lose, void                                                   |
| payout_atomic     | float8  | Amount (lamports) earned through the corresponding bet and market |
| settled_at        | float8  | POSIX time, transaction settled (win, lose, void)                 |
| settlement_tx_sig | text    | nullable                                                          |



# Security
## user_roles
Stores user permissions levels on the database. Tracks every move. One row per role.


| Field            | Type        | Description                             |
| ---------------- | ----------- | --------------------------------------- |
| id               | uuid PK     |                                         |
| user_id          | uuid        | FK to users                             |
| role             | enum        | admin, ops, risk, partner_view, support |
| scope_partner_id | uuid        | FK -> partners (not implemented yet)    |
| is_active        | bool        | default: true                           |
| granted_by       | uuid        | FK to users                             |
| granted_at       | timestamptz | default now                             |
| revoked_at       | timestamptz | nullable                                |
| notes            | text        | nullable                                |


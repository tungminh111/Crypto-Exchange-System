# API design
- POST /sign_up(username,password)
- POST /login(username,password) -> session_id, refresh_token
- POST /get_access_token(session_id, refresh_token)

- POST /debit_balance(access_token, order_id, instrument_id, quantity)

# Database
## User table (PostgreSQL)
```mermaid
classDiagram
    User <|-- UserBalance
    User <|-- UserBalanceAudit
    UserBalance <|-- UserBalanceAudit
    class User {
        - username: varchar(128)
        - hashed_password: varchar(256)
        - user_id: varchar(128)
    }
    %%class UserBalanceAudit {
    %%    - audit_id: varchar(128) (index primary key)
    %%    - user_id: varchar(128)
    %%    - instrument_id: varchar(128)
    %%    - order_id: varchar(128)
    %%    - delta_balance: varchar(128)
    %%    - event: enum(Credit/Debit)
    %%    - status: enum(pending/matched)
    %%}
```

## UserBalance table (TigerBeetles)
```mermaid
    class UserBalance {
        - ledger: instrument_id
        - user_id: string
        - balance: float
    }
```

## UserSession (Redis)
```mermaid
classDiagram
    class UserSession {
        - username: string
        - session_id: string
        - hashed_refresh_token: string
        - expiry_data
    }
```

# Sequence diagram 
## Create Account
```mermaid
sequenceDiagram
    participant api as APIGateway
    participant us as UserService
    participant udb as UserDB
    api ->> us: POST /sign_up(username, password)
    us ->> us: generate user_id
    us ->> udb: write(user_id, username, hashed_password)
    us -->> api: result
```
## Login flow
```mermaid
sequenceDiagram
    participant api as APIGateway
    participant us as UserService
    participant udb as UserDB
    participant usdb as UserSessionDB
    api ->> us: POST /login(username, password, deviceId)
    us ->> udb: validate username/password
    us ->> us: generate new session_id
    us ->> us: generate refresh_token
    us ->> usdb: write(session_id, refresh_token)
    us -->> api: session_id, refresh_token
```

## GetAccessToken flow
```mermaid
sequenceDiagram
    participant api as APIGateway
    participant us as UserService
    participant udb as UserDB
    participant usdb as UserSessionDB
    api ->> us: POST /get_access_token(session_id, refresh_token)
    us ->> usdb: validate session_id/refresh_token
    us ->> us: generate jwt encrypted by private_key 
    us -->> api: jwt token
```

## Validate access token flow
```mermaid
sequenceDiagram
    participant api as APIGateway
    participant us as OrderService
    api ->> us: POST /create_order(access_token, ...)
    us ->> us: validate access_token by public_key
```

## Debit balance flow
```mermaid
sequenceDiagram
    participant os as OrderService
    participant us as UserService
    participant ubdb as UserBalanceDB
    os ->> us: POST /debit_balance(access_token, order_id, instrument_id, quantity)
    us ->> ubdb: deduct(user_id, instrument_id, quantity)
```
NOTE: ignore Audit flow in the current phase

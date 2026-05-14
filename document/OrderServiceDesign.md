# API design
- POST /create_order(access_token (include user_id), instrument_id, price, quantity, side: Buy/Sell, type: Market/Limit) -> order_id
- POST /modify_order(access_token, order_id, instrument_id, price, quantity, side, type) -> err_code
- POST /cancel_order(access_token, order_id) -> err_code

- GET /list_order(access_token, status: Active/Filled/Cancelled, begin_time, end_time) -> [{order_id, instrument_id, price, ...}] 

# Database
## Order table (PostgreSQL)
```mermaid
classDiagram
    Order <|-- MatchedOrder
    class Order {
        - order_id: varchar(128) (primary key)
        - user_id: varchar(128)
        - instrument_id: number
        - price: float
        - quantity: float
        - side: enum(BUY/SELL)
        - type: enum(MARKET/LIMIT)
        - status: enum(ACTIVE/FILLED/CANCELLED)
    }

    class MatchedOrder {
        - first_order_id: varchar(128) (primary key)
        - second_order_id: varchar(128) (primary key)
        - quantity: float
    }

    class OrderEventOutbox {
        - order_id: varchar(128) (primary key)
        - user_id: varchar(128)
        - instrument_id: number
        - price: float
        - quantity: float
        - side: enum(BUY/SELL)
        - type: enum(MARKET/LIMIT)
        - status: enum(ACTIVE/FILLED/CANCELLED)
        - action: enum(Create/Cancel/Modify)
    } 
```

# Sequence diagram
## Create Order
```mermaid
sequenceDiagram
    participant api as APIGateway
    participant us as UserService
    participant os as OrderService
    participant odb as OrderDB
    participant oodb as OrderEventOutboxDB
    

    api ->> os: create_order(access_token, order_data)
    os ->> os: validate access_token
    os ->> us: debit_balance(access_token, order_id, instrument_id, quantity)
    alt transaction 
        os ->> odb: add new order
        os ->> oodb: add order create event
    end
```


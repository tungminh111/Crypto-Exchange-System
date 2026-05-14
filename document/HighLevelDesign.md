# Project requirement 
## Functional requirement
Crypto exchange that allows user buy/sell, and manage different types of asset.
User interactions:
- Login 
- Create account
- Create/Modify/Cancel Buy/Sell orders (LIMIT/MARKET)
- View historical orders status
    - Including order quantity filled
- Receive order match notifications 
- View current market order book
System can support 200+ instrument types.


## Non-functional requirement
- Data consistency
    - Strong consistency on user balance deduction, order matching
    - Eventual consistency is acceptable for other interactions
- High performance 
    - 50ms for order acknowledgement latency
    - 50ms for match notification latency
- High reliability
    - Order creation durability
    - Order match guarantee
- High availability (99%)
- Fault tolerance
- High scalability 
    - Can support 10m daily active DAUs, including HFTs
    - For normal user, can support a peak of ~10k OPS 
    - For HFTs, can support a peak of ~500k OPS

For learning purpose, firstly no need to consider 
- Security

## Back-of-envelop estimation
### User data
- Assuming 1b total users in 5 years
-> ~100 GB user information data

### Order data
Normal user
- Peak OPS: 10k 
- Peak OPS/instrument: 5k
- Daily orders: 100m orders
- 5 years orders: 5 * 365 * 100m = ~160b orders -> 14TB storage (assuming 100 bytes per order)

# High level design
https://excalidraw.com/#json=8ikHY7FvPZSees8X7qq8-,qmp13Qhqz19EpodUix60Tg

# Demo Coin Transfer Chaincode

This directory contains the chaincode and application for managing demo transactions in the Proof of Collaborative Learning (PoCL) system. The demo transactions serve as the workload that miners process during the federated learning consensus mechanism.

## Overview

The demo coin transfer system handles:
- Initial transaction validation and storage
- Transaction assignment to miners
- Transaction state management during mining rounds
- Demo wallet operations (separate from main wallets)

## Directory Structure

```
demo-coin-transfer/
├── demo-coin-transfer-chaincode/     # Smart contract implementation
│   ├── index.js                      # Chaincode entry point
│   ├── package.json                  # Dependencies and metadata
│   └── lib/                         # Chaincode implementation
│       └── demoCoinTransfer.js      # Main contract logic
└── demo-coin-transfer-application/   # Client application
    ├── demoApp.js                   # Application interface
    └── package.json                 # Dependencies and metadata
```

## Chaincode Features

### Transaction Management
- **Create Transactions**: Generate new demo transactions
- **Validate Transactions**: Ensure transaction validity
- **Assign Transactions**: Distribute transactions to miners
- **Update Status**: Track transaction processing state

### Wallet Operations
- **Demo Wallets**: Separate from main cryptocurrency wallets
- **Balance Tracking**: Monitor demo wallet balances
- **Transfer Validation**: Ensure sufficient funds

### Mining Integration
- **Assignment Logic**: Fair distribution among miners
- **Round Management**: Reset state between mining rounds
- **Status Tracking**: Monitor assignment and processing

## Application Interface

### Core Functions

#### Transaction Operations
```javascript
// Create a new demo transaction
createDemoTransaction(senderId, receiverId, amount)

// Transfer coins between demo wallets
transferCoinsTrx(contractDemo, senderId, receiverId, amount)

// Read transaction details
readTrx(contractDemo, transactionId)
```

#### Mining Integration
```javascript
// Assign transactions to miners
assignTransactions(contractDemo, minerName, count)

// Get transactions by assignment
getTransactionsByAssignment(contractDemo, minerName)

// Refresh transactions for new round
refreshTransactions(contractDemo, winners)
```

#### System Management
```javascript
// Initialize transaction ledger
initTransactions(contractDemo)

// Toggle accepting new transactions
toggleAcceptingStatus(contractDemo)

// Get all transactions
getAllTransactions(contractDemo)
```

## Transaction Lifecycle

### 1. Creation
Transactions are created by the submitter service:
```javascript
POST /api/demo/transaction/transfer/
{
  "senderId": "main_1",
  "receiverId": "main_2", 
  "amount": 0.25
}
```

### 2. Validation
The chaincode validates:
- Sender and receiver wallet existence
- Sufficient balance in sender wallet
- Valid amount (positive, proper precision)

### 3. Storage
Valid transactions are stored in the blockchain state with:
- Unique transaction ID
- Sender and receiver information
- Amount and timestamp
- Assignment status

### 4. Assignment
During mining rounds, transactions are assigned to miners:
- Fair distribution algorithm
- Maximum transactions per miner (configurable)
- First-come-first-served for requests

### 5. Processing
Miners include assigned transactions in their blocks:
- Transactions are marked as processed
- Winners' transactions are executed
- Losing transactions are reset for next round

## Data Model

### Transaction Structure
```javascript
{
  id: "demo_12345",
  senderId: "main_3",
  receiverId: "main_7",
  amount: 0.25,
  timestamp: "2025-01-01T12:00:00Z",
  status: "assigned", // pending, assigned, processed
  assignedTo: "miner_1", // null if unassigned
  round: 5
}
```

### Demo Wallet Structure
```javascript
{
  id: "main_1",
  balance: 100.00,
  lastUpdated: "2025-01-01T12:00:00Z"
}
```

## API Endpoints

### Transaction Operations
- `POST /api/demo/transaction/create/` - Create new wallet
- `POST /api/demo/transaction/update/` - Update wallet balance
- `POST /api/demo/transaction/delete/` - Delete wallet
- `POST /api/demo/transaction/transfer/` - Transfer between wallets
- `GET /api/demo/transaction/` - Read specific transaction
- `DELETE /api/demo/transaction/` - Delete transaction

### Mining Operations
- `POST /api/demo/transactions/assign/` - Assign transactions to miner
- `GET /api/demo/transactions/assign/` - Get transactions by assignment
- `GET /api/demo/transactions/` - Get all transactions

### System Operations
- `POST /api/demo/ledger/` - Initialize transaction ledger

## Integration with PoCL System

### With Miners
Miners request transactions at the start of each round:
```python
res = requests.post("http://localhost:3000/api/demo/transactions/assign/", json={
    "minerName": self.name,
    "count": self.max_trx
})
```

### With Submitter
The submitter continuously creates new transactions:
```python
requests.post("http://localhost:3001/api/demo/transaction/transfer/", json=data)
```

### With Express Applications
Express apps manage the transaction flow and miner coordination through the demo application interface.

## State Management

### Round-based Reset
After each mining round:
- Winning transactions are permanently processed
- Losing transactions are reset to pending
- Assignment status is cleared
- New round can begin

### Status Tracking
- **Pending**: Newly created, available for assignment
- **Assigned**: Given to a miner, awaiting processing
- **Processed**: Included in a winning block
- **Reset**: Returned to pending after losing round

## Configuration

### Chaincode Deployment
```bash
./network.sh deployCC -ccp ../demo-coin-transfer/demo-coin-transfer-chaincode \
  -ccn demoCC -c demo -ccl javascript
```

### Channel Configuration
- **Channel Name**: demo
- **Chaincode Name**: demoCC
- **Language**: JavaScript/Node.js

## Security Features

### Transaction Validation
- Input sanitization and validation
- Balance verification before transfers
- Duplicate transaction prevention
- Proper authorization checks

### State Integrity
- Atomic operations for state updates
- Consistent state across all peers
- Audit trail for all operations
- Rollback capabilities for failed operations

## Error Handling

### Common Errors
- **Insufficient Balance**: Transfer amount exceeds sender balance
- **Invalid Wallet**: Non-existent sender or receiver
- **Assignment Conflicts**: Multiple assignments of same transaction
- **State Inconsistency**: Blockchain state synchronization issues

### Error Responses
```javascript
{
  "error": "Insufficient balance",
  "details": "Sender balance: 0.50, requested: 0.75"
}
```

## Performance Considerations

### Throughput
- Optimized for high transaction volume
- Efficient state queries and updates
- Minimal computational overhead per transaction

### Scalability
- Support for hundreds of concurrent transactions
- Efficient assignment algorithms
- Optimized data structures for quick lookups

## Testing

### Unit Tests
Test individual chaincode functions:
```javascript
// Test transaction creation
await contract.createDemoTransaction('main_1', 'main_2', '0.25');

// Test assignment logic
const assigned = await contract.assignTransactions('miner_1', '2');
```

### Integration Tests
Test full transaction flow from creation to processing through the Express API endpoints.

## Deployment

The chaincode is automatically deployed as part of the system startup:
```bash
./start.sh  # Includes demo chaincode deployment
```

Manual deployment:
```bash
cd test-network
./network.sh deployCC -ccp ../demo-coin-transfer/demo-coin-transfer-chaincode \
  -ccn demoCC -c demo -ccl javascript
```

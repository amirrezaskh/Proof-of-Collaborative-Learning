# Main Coin Transfer Chaincode

This directory contains the chaincode and application for managing the main cryptocurrency wallets and transactions in the Proof of Collaborative Learning (PoCL) system. Unlike demo transactions, main transactions represent the actual cryptocurrency transfers and miner rewards.

## Overview

The main coin transfer system manages:
- **Main Cryptocurrency Wallets**: Real wallets with persistent balances
- **Winner Transaction Execution**: Processing transactions from winning miners
- **Miner Rewards**: Distributing rewards based on model contributions
- **Wallet Management**: Creating, updating, and querying wallet states

## Directory Structure

```
main-coin-transfer/
├── main-coin-transfer-chaincode/     # Smart contract implementation
│   ├── index.js                      # Chaincode entry point
│   ├── package.json                  # Dependencies and metadata
│   └── lib/                         # Chaincode implementation
│       └── mainCoinTransfer.js      # Main contract logic
└── main-coin-transfer-application/   # Client application
    ├── mainApp.js                   # Application interface
    └── package.json                 # Dependencies and metadata
```

## Core Functionality

### Wallet Management
- **Wallet Creation**: Initialize main wallets with default balances
- **Balance Tracking**: Maintain accurate balance records
- **Wallet Queries**: Read wallet information and balances
- **State Persistence**: Long-term storage of wallet states

### Transaction Processing
- **Winner Transactions**: Execute transactions from winning miners only
- **Balance Validation**: Ensure sufficient funds before transfers
- **Atomic Operations**: Guarantee transaction consistency
- **Transfer History**: Maintain transaction audit trails

### Reward Distribution
- **Performance-based Rewards**: Pay miners based on model contribution
- **Automatic Distribution**: Handle reward calculations and transfers
- **Reward Tracking**: Monitor total rewards distributed
- **Balance Updates**: Update miner wallet balances with rewards

## Application Interface

### Core Functions

#### Wallet Operations
```javascript
// Initialize main wallets with default balances
initWallets(contractMain)

// Read specific wallet information
readWallet(contractMain, walletId)

// Get all wallet information
getAllWallets(contractMain)
```

#### Transaction Execution
```javascript
// Execute transactions from winning miners
runWinnerTransactions(contractMain, winners)

// Process individual transfer
transferCoins(contractMain, senderId, receiverId, amount)
```

#### Reward Management
```javascript
// Distribute reward to specific miner
rewardMiner(contractMain, minerId, rewardAmount)

// Batch reward distribution
distributeRewards(contractMain, rewardList)
```

## Data Models

### Main Wallet Structure
```javascript
{
  id: "main_1",
  balance: 1000.00,
  totalRewards: 25.50,
  transactionCount: 15,
  lastUpdated: "2025-01-01T12:00:00Z",
  walletType: "main" // vs "miner"
}
```

### Transaction Record
```javascript
{
  id: "main_tx_12345",
  senderId: "main_3",
  receiverId: "main_7", 
  amount: 15.75,
  timestamp: "2025-01-01T12:00:00Z",
  round: 5,
  status: "completed",
  type: "transfer" // vs "reward"
}
```

### Miner Wallet Structure
```javascript
{
  id: "miner_1",
  balance: 150.25,
  totalRewards: 150.25,
  roundsWon: 8,
  averageReward: 18.78,
  lastReward: "2025-01-01T12:00:00Z"
}
```

## Transaction Lifecycle

### 1. Demo Transaction Processing
Demo transactions are processed through the demo chaincode first:
- Initial validation and assignment
- Mining round competition
- Winner selection

### 2. Winner Transaction Execution
Only winning miners' transactions are executed in main wallets:
```javascript
async function runWinnerTransactions(contractMain, winners) {
    for (const winner of winners) {
        const transactions = getAssignedTransactions(winner);
        for (const tx of transactions) {
            await executeTransaction(contractMain, tx);
        }
    }
}
```

### 3. Reward Distribution
After model aggregation, rewards are distributed:
```javascript
async function rewardMiner(contractMain, minerId, rewardAmount) {
    const wallet = await readWallet(contractMain, minerId);
    const newBalance = wallet.balance + parseFloat(rewardAmount);
    await updateWallet(contractMain, minerId, newBalance);
}
```

## API Endpoints

### Wallet Management
- `POST /api/main/ledger/` - Initialize main wallet ledger
- `GET /api/main/wallets/` - Get all wallet information
- `GET /api/main/wallet/` - Get specific wallet details

### Transaction Processing
- `POST /api/main/run/` - Execute winner transactions

### System Integration
The main application integrates with the Express.js admin application to handle winner transaction processing and reward distribution.

## Integration with PoCL System

### Winner Transaction Processing
After winner selection, the admin application triggers main transaction execution:
```javascript
// In app1.js
const message = await mainApp.runWinnerTransactions(contractMain, JSON.stringify(winners));
```

### Reward Distribution
Rewards calculated by the aggregator are distributed through main wallets:
```javascript
// After model aggregation
await distributeRewards(res.data);

async function distributeRewards(rewards) {
    for (const data of rewards) {
        await mainApp.rewardMiner(contractMain, data['id'], data['reward']);
    }
}
```

### Wallet Initialization
Main wallets are initialized during system setup:
```javascript
// During system initialization
await mainApp.initWallets(contractMain);
```

## State Management

### Persistent State
- **Main Wallets**: Maintain balances across system restarts
- **Transaction History**: Complete audit trail of all transfers
- **Reward Records**: Track total rewards per miner
- **System Metrics**: Monitor overall system activity

### State Consistency
- **Atomic Updates**: All balance changes are atomic
- **Concurrent Access**: Handle multiple simultaneous requests
- **State Validation**: Verify state consistency across peers
- **Recovery Mechanisms**: Handle network partitions gracefully

## Security Features

### Transaction Validation
- **Balance Verification**: Prevent overdrafts
- **Sender Authentication**: Verify transaction originators
- **Amount Validation**: Ensure positive transfer amounts
- **Double-spending Prevention**: Atomic balance updates

### Access Control
- **Role-based Permissions**: Different access levels for operations
- **Winner Verification**: Only process transactions from verified winners
- **Admin Controls**: Protected system management functions

### Audit and Compliance
- **Complete Transaction History**: Immutable audit trail
- **Balance Reconciliation**: Regular balance verification
- **Fraud Detection**: Monitor for unusual transaction patterns

## Performance Considerations

### Scalability
- **Efficient Queries**: Optimized wallet and transaction lookups
- **Batch Processing**: Handle multiple operations efficiently
- **State Indexing**: Fast access to wallet information
- **Concurrent Operations**: Support parallel transaction processing

### Resource Management
- **Memory Usage**: Efficient state representation
- **Storage Optimization**: Compact transaction records
- **Network Efficiency**: Minimize blockchain communication overhead

## Configuration

### Deployment Settings
```bash
# Channel: main
# Chaincode: mainCC
# Language: JavaScript
./network.sh deployCC -ccp ../main-coin-transfer/main-coin-transfer-chaincode \
  -ccn mainCC -c main -ccl javascript
```

### Initial Balances
Main wallets are initialized with default balances:
```javascript
const defaultBalance = 1000.00; // Initial balance for main wallets
const minerInitialBalance = 0.00; // Miners start with no balance
```

## Error Handling

### Common Errors
- **Insufficient Funds**: Transfer amount exceeds wallet balance
- **Invalid Wallet**: Non-existent sender or receiver wallet
- **Concurrent Modifications**: Multiple simultaneous balance updates
- **Network Issues**: Blockchain connectivity problems

### Error Recovery
- **Transaction Rollback**: Revert failed transactions
- **State Reconciliation**: Repair inconsistent states
- **Retry Mechanisms**: Automatic retry for transient failures
- **Error Logging**: Comprehensive error tracking

## Monitoring and Analytics

### Key Metrics
- **Total Transaction Volume**: Sum of all processed transactions
- **Reward Distribution**: Total rewards paid to miners
- **Wallet Activity**: Transaction frequency per wallet
- **System Health**: Success rates and error frequencies

### Reporting
- **Balance Reports**: Current state of all wallets
- **Transaction Reports**: Historical transaction analysis
- **Reward Reports**: Miner performance and earnings
- **System Reports**: Overall system health and performance

## Testing

### Unit Tests
```javascript
// Test wallet creation
await contract.initWallets();
const wallet = await contract.readWallet('main_1');
assert.equal(wallet.balance, 1000.00);

// Test transaction execution
await contract.transferCoins('main_1', 'main_2', '100.00');
const sender = await contract.readWallet('main_1');
const receiver = await contract.readWallet('main_2');
assert.equal(sender.balance, 900.00);
assert.equal(receiver.balance, 1100.00);
```

### Integration Tests
Test complete winner transaction flow from demo processing through main execution and reward distribution.

## Deployment

### Automatic Deployment
The chaincode is deployed as part of the system startup:
```bash
./start.sh  # Includes main chaincode deployment
```

### Manual Deployment
```bash
cd test-network
./network.sh deployCC -ccp ../main-coin-transfer/main-coin-transfer-chaincode \
  -ccn mainCC -c main -ccl javascript
```

### Verification
```bash
# Verify deployment
peer chaincode query -C main -n mainCC -c '{"Args":["getAllWallets"]}'
```

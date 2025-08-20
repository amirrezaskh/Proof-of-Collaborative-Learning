# Express Applications

This directory contains the Express.js server applications that provide API interfaces and coordination services for the Proof of Collaborative Learning (PoCL) system. These applications serve as the bridge between the blockchain network and the federated learning clients.

## Overview

The Express applications handle:
- **API Gateway**: HTTP REST interfaces for all system components
- **Process Coordination**: Orchestrating the mining and consensus process
- **Blockchain Integration**: Interfacing with Hyperledger Fabric chaincodes
- **System Administration**: Managing rounds, deadlines, and winner selection

## Files

- **`app1.js`** - Primary admin application (Port 3000)
- **`app2.js`** - Secondary application (Port 3001)
- **`package.json`** - Node.js dependencies and project metadata

## Application Architecture

### app1.js - Admin Application (Port 3000)

The primary application serves as the system administrator and coordinates:

#### Core Responsibilities
- **Round Management**: Orchestrating 20 rounds of federated learning
- **Miner Coordination**: Notifying miners of phase transitions
- **Winner Selection**: Determining top K miners based on votes
- **Model Aggregation**: Triggering FedAvg model combination
- **Reward Distribution**: Calculating and distributing miner rewards

#### Phase Management
```javascript
// Phase 1: Model Training (180 seconds)
setTimeout(gatherTestData, modelProposeDeadline * 1000);

// Phase 2: Prediction (15 seconds)  
setTimeout(gatherPredictions, predProposeDeadline * 1000);

// Phase 3: Voting (15 seconds)
setTimeout(selectWinners, voteAssignDeadline * 1000);
```

#### API Endpoints Overview
- **Demo Transactions**: `/api/demo/*` - Transaction management
- **Main Wallets**: `/api/main/*` - Main cryptocurrency operations
- **Model Proposals**: `/api/model/*` - Model submission and retrieval
- **Predictions**: `/api/pred/*` - Prediction data management
- **Voting**: `/api/vote/*` - Vote collection and winner selection
- **System Control**: `/api/start/` - Initiate mining process

### app2.js - Secondary Application (Port 3001)

Provides additional API capacity and redundancy for system operations.

## System Configuration

### Network Topology
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   app1.js   │    │   app2.js   │    │   Miners    │
│   (Admin)   │    │             │    │  (8000-     │
│   Port 3000 │◄──►│ Port 3001   │◄──►│   8009)     │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                  ┌─────────────┐
                  │ Blockchain  │
                  │  Network    │
                  │ (Fabric)    │
                  └─────────────┘
```

### Timing Configuration
```javascript
const modelProposeDeadline = 180;  // 3 minutes for training
const predProposeDeadline = 15;    // 15 seconds for predictions
const voteAssignDeadline = 15;     // 15 seconds for voting
```

### System Parameters
```javascript
const winnerCount = 5;             // Top 5 miners selected per round
const rounds = 20;                 // Total federated learning rounds
const minersPorts = [8000...8009]; // Miner communication ports
const aggregatorPort = 5050;       // Model aggregation service
```

## Blockchain Integration

### Connection Management
```javascript
async function InitConnection(channelName, chaincodeName) {
    const client = await newGrpcConnection();
    const gateway = connect({
        client,
        identity: await newIdentity(),
        signer: await newSigner(),
        // Timeout configurations
    });
    return gateway.getNetwork(channelName).getContract(chaincodeName);
}
```

### Contract Instances
- **contractDemo**: Demo transaction management (demo channel)
- **contractMain**: Main wallet operations (main channel)  
- **contractModel**: Model proposal handling (demo channel)
- **contractPred**: Prediction management (demo channel)
- **contractVote**: Voting system (demo channel)

## API Documentation

### Demo Transaction APIs

#### POST /api/demo/transaction/transfer/
Create a new transfer transaction between demo wallets.

**Request:**
```json
{
  "senderId": "main_1",
  "receiverId": "main_2",
  "amount": 0.25
}
```

#### POST /api/demo/transactions/assign/
Assign transactions to a specific miner.

**Request:**
```json
{
  "minerName": "miner_1",
  "count": 2
}
```

### Model Management APIs

#### POST /api/model/
Submit a trained model proposal.

**Request:**
```json
{
  "id": "model_1",
  "hash": "md5_hash_string",
  "path": "/absolute/path/to/model",
  "transactions": ["tx1", "tx2"],
  "testData": [[...], [...]]
}
```

### Prediction APIs

#### POST /api/pred/
Submit predictions for other miners' test data.

**Request:**
```json
{
  "id": "pred_1", 
  "predictions": {
    "model_2": [1, 0, 1, 1, 0],
    "model_3": [1, 1, 1, 0, 0]
  }
}
```

### Voting APIs

#### POST /api/vote/
Submit ranking votes for other miners.

**Request:**
```json
{
  "id": "vote_1",
  "votes": ["model_3", "model_1", "model_2"]
}
```

## Process Flow Coordination

### Round Initialization
```javascript
async function startRound() {
    current_round += 1;
    // Notify miners to start training
    for (const port of minersPorts) {
        await axios.get(`http://localhost:${port}/transactions/ready/`, {
            params: { status: "ready", time: modelProposeDeadline, round: current_round }
        });
    }
}
```

### Phase Transitions
1. **Training Phase**: Miners train models on assigned transactions
2. **Test Data Collection**: Gather test data from all models
3. **Prediction Phase**: Miners predict on others' test data
4. **Voting Phase**: Miners vote on prediction quality
5. **Winner Selection**: Select top K miners based on votes
6. **Aggregation**: Combine winner models using FedAvg
7. **Reward Distribution**: Pay winners based on contribution

### Winner Selection Process
```javascript
async function selectWinners() {
    const winners = await voteApp.selectWinners(contractVote, winnerCount.toString());
    const message = await mainApp.runWinnerTransactions(contractMain, JSON.stringify(winners));
    const models = await modelApp.getWinnerModels(contractModel, JSON.stringify(winners));
    
    // Aggregate models
    const res = await axios.post(`http://localhost:${aggregatorPort}/aggregate/`, {
        models: models
    });
    
    // Distribute rewards
    await distributeRewards(res.data);
    await refreshStates(winners);
}
```

## Error Handling

### Network Resilience
- **Connection Retries**: Automatic retry for failed blockchain calls
- **Timeout Management**: Configurable timeouts for different operations
- **Error Isolation**: Individual failures don't crash the system

### PHANTOM Error Prevention
```javascript
const { Semaphore } = require('async-mutex');
const semaphore = new Semaphore(1);
// Prevents concurrent blockchain state modifications
```

## Scaling the Network

This project is initially implemented to contain only two peers in the blockchain network. However, it can easily be modified for additional peers. To achieve this:

1. Copy the code of `app2.js` and name it appropriately (e.g., `app3.js`)
2. Alter the code in `run.py` to run this express application as well
3. Create a log file for the new application
4. Change the `peer_port` of some miners accordingly to distribute load
5. Update the Hyperledger Fabric network configuration for additional peers

## Performance Optimization

### Connection Pooling
- **Persistent Connections**: Reuse blockchain connections
- **Connection Limits**: Manage resource usage
- **Load Balancing**: Distribute requests across applications

### Asynchronous Processing
- **Non-blocking Operations**: Concurrent request handling
- **Promise-based APIs**: Efficient async/await patterns
- **Background Tasks**: Timer-based phase transitions

## Security Features

### Authentication
- **X.509 Certificates**: Hyperledger Fabric identity management
- **MSP Integration**: Membership Service Provider validation
- **TLS Encryption**: Secure communication with blockchain network

### Authorization
- **Role-based Access**: Different permissions for admin vs regular operations
- **API Validation**: Input sanitization and validation
- **State Verification**: Blockchain state integrity checks

## Deployment

### Automatic Startup
Started by the main run script:
```python
executor.submit(os.system, "node ./app1.js > ../logs/app1.txt")
executor.submit(os.system, "node ./app2.js > ../logs/app2.txt")
```

### Manual Startup
```bash
cd express-application
npm install
node app1.js &
node app2.js &
```

### Dependencies Installation
```bash
npm install @hyperledger/fabric-gateway express body-parser axios async-mutex
```

## Troubleshooting

### Common Issues
1. **Connection Failures**: Check Hyperledger Fabric network status
2. **Certificate Errors**: Verify crypto material paths
3. **Port Conflicts**: Ensure ports 3000, 3001 are available
4. **Timeout Errors**: Adjust deadline configurations

### Debug Mode
Enable verbose logging by modifying console output levels and adding debug statements to track request flow.
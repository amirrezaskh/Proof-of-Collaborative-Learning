# Model Propose Chaincode

This directory contains the chaincode and application for managing model proposals in the Proof of Collaborative Learning (PoCL) system. Miners submit their trained models along with test data for evaluation and consensus.

## Overview

The model propose system handles:
- **Model Submission**: Miners submit trained model metadata and test data
- **Model Integrity**: Hash-based verification of model files
- **Test Data Management**: Collection and distribution of test datasets
- **Winner Model Retrieval**: Accessing models from winning miners

## Directory Structure

```
model-propose/
├── model-propose-chaincode/     # Smart contract implementation
│   ├── index.js                 # Chaincode entry point
│   ├── package.json            # Dependencies and metadata
│   └── lib/                    # Chaincode implementation
│       └── modelPropose.js     # Main contract logic
└── model-propose-application/   # Client application
    ├── modelApp.js             # Application interface
    └── package.json            # Dependencies and metadata
```

## Core Functionality

### Model Management
- **Model Proposal**: Submit trained model with metadata
- **Hash Verification**: Ensure model file integrity
- **Path Management**: Track absolute paths to model files
- **Transaction Association**: Link models to processed transactions

### Test Data Handling
- **Test Data Collection**: Gather test datasets from all miners
- **Data Distribution**: Provide test data to all participants
- **Format Standardization**: Ensure consistent data formats
- **Privacy Protection**: Share test data without labels initially

### System Integration
- **Status Management**: Control accepting new model proposals
- **Round Coordination**: Manage model proposals per round
- **Winner Selection**: Retrieve models from winning miners

## Data Models

### Model Proposal Structure
```javascript
{
  id: "model_1",
  hash: "md5_hash_string",
  path: "/absolute/path/to/model.keras",
  transactions: ["demo_tx_1", "demo_tx_2"],
  testData: [
    [pixel_values_array_1],
    [pixel_values_array_2],
    // ... 50 test samples
  ],
  minerId: "miner_1",
  timestamp: "2025-01-01T12:00:00Z",
  round: 5,
  status: "submitted"
}
```

### Test Data Record
```javascript
{
  modelId: "model_1",
  testRecords: [
    [32, 32, 3, ...], // CIFAR-10 image data
    [32, 32, 3, ...],
    // ... test samples
  ],
  sampleCount: 50,
  dataFormat: "cifar10_normalized"
}
```

## Application Interface

### Core Functions

#### Model Operations
```javascript
// Submit a new model proposal
createModel(contractModel, id, hash, path, transactions, testData)

// Read specific model information
readModel(contractModel, modelId)

// Get all submitted models
getAllModels(contractModel)

// Delete model proposal
deleteModel(contractModel, modelId)
```

#### Test Data Management
```javascript
// Gather all test records from submitted models
gatherAllTestRecords(contractModel)

// Get test records for predictions
getTestRecords(contractModel)

// Distribute test data to miners
distributeTestData(contractModel, minerId)
```

#### Winner Management
```javascript
// Get models from winning miners
getWinnerModels(contractModel, winners)

// Retrieve model paths for aggregation
getModelPaths(contractModel, winners)
```

#### System Control
```javascript
// Initialize model ledger
initModels(contractModel)

// Toggle accepting new proposals
toggleAcceptingStatus(contractModel)

// Get current accepting status
getAcceptingStatus(contractModel)

// Delete all models (round reset)
deleteAllModels(contractModel)
```

## Process Flow

### 1. Model Training Phase
Miners train their local models and prepare submissions:
```python
# In miner.py
def train(self):
    # Train model locally
    self.model.fit(self.X_train, self.y_train, ...)
    
    # Save trained model
    self.model.save(self.current_model)
    
    # Generate test data
    test_data = self.get_random_test()
    
    # Submit model proposal
    requests.post(f"http://localhost:{self.peer_port}/api/model/", json={
        "id": f"model_{self.name[6:]}",
        "hash": hash_value,
        "path": self.current_model_path,
        "transactions": transaction_ids,
        "testData": test_data
    })
```

### 2. Model Collection Phase
The admin application collects all model proposals:
```javascript
// After training deadline
await modelApp.gatherAllTestRecords(contractModel);
```

### 3. Test Data Distribution
Test data is distributed to all miners for predictions:
```javascript
// Miners receive test data
const testRecords = await modelApp.getTestRecords(contractModel);
```

### 4. Winner Model Retrieval
After winner selection, models are retrieved for aggregation:
```javascript
const models = await modelApp.getWinnerModels(contractModel, winners);
```

## API Endpoints

### Model Management
- `POST /api/model/ledger/` - Initialize model proposal ledger
- `POST /api/model/` - Submit new model proposal
- `GET /api/model/` - Read specific model information
- `DELETE /api/model/` - Delete model proposal
- `GET /api/models/` - Get all submitted models

### System Status
- `GET /api/model/accepting/` - Check if accepting new proposals

## Model Integrity Verification

### Hash-based Verification
Each model includes an MD5 hash for integrity checking:
```python
# Model submission
hash_value = hashlib.md5(open(self.current_model,'rb').read()).hexdigest()

# Verification during aggregation
hash_value = hashlib.md5(open(model['path'],'rb').read()).hexdigest()
if hash_value == model['hash']:
    # Process the model
```

### Path Validation
- **Absolute Paths**: All model paths must be absolute
- **File Existence**: Verify files exist before processing
- **Access Permissions**: Ensure read access to model files

## Test Data Management

### Data Format
Test data follows CIFAR-10 format:
- **Shape**: (batch_size, 32, 32, 3)
- **Type**: Normalized float values [0, 1]
- **Size**: 50 samples per miner
- **Format**: JSON serializable arrays

### Privacy Considerations
- **No Labels**: Test data shared without ground truth labels
- **Random Selection**: Miners choose random samples from their test set
- **Data Isolation**: Each miner's test data remains separate

### Distribution Process
```javascript
// Collect all test data
const allTestRecords = {};
for (const model of models) {
    allTestRecords[model.id] = model.testData;
}

// Distribute to miners
for (const miner of miners) {
    await sendTestData(miner, allTestRecords);
}
```

## Integration with PoCL System

### Training Phase Integration
```javascript
// In app1.js - start training phase
async function startRound() {
    await demoApp.toggleAcceptingStatus(contractDemo);
    // Notify miners to start training
    for (const port of minersPorts) {
        await axios.get(`http://localhost:${port}/transactions/ready/`);
    }
}
```

### Test Data Collection
```javascript
// After training deadline
async function gatherTestData() {
    await demoApp.toggleAcceptingStatus(contractDemo);
    await modelApp.gatherAllTestRecords(contractModel);
    // Notify miners to make predictions
}
```

### Winner Model Processing
```javascript
// After winner selection
const models = await modelApp.getWinnerModels(contractModel, winners);
const aggregationResult = await aggregator.aggregate(models);
```

## State Management

### Round-based Reset
After each round:
- All model proposals are deleted
- Test data is cleared
- Accepting status is reset
- New round can begin with clean state

### Status Control
```javascript
{
  acceptingModels: true,     // Whether accepting new proposals
  roundNumber: 5,            // Current round
  totalModels: 8,           // Models submitted this round
  testDataGathered: false   // Whether test data collection is complete
}
```

## Performance Optimizations

### Data Handling
- **Batch Operations**: Process multiple models efficiently
- **Lazy Loading**: Load model data only when needed
- **Memory Management**: Efficient handling of large test datasets
- **Concurrent Access**: Support multiple simultaneous submissions

### Network Efficiency
- **Compressed Data**: Minimize network overhead for test data
- **Streaming**: Handle large model files efficiently
- **Caching**: Cache frequently accessed model metadata

## Security Features

### Model Authentication
- **Hash Verification**: Prevent model tampering
- **Miner Verification**: Ensure models come from authorized miners
- **Timestamp Validation**: Verify submission timing

### Data Protection
- **Access Control**: Protect model files from unauthorized access
- **Integrity Checks**: Verify test data hasn't been modified
- **Privacy Preservation**: Protect sensitive model information

## Error Handling

### Common Issues
- **Hash Mismatches**: Model file has been modified after hashing
- **File Access Errors**: Model file path is invalid or inaccessible
- **Data Format Errors**: Test data doesn't match expected format
- **Duplicate Submissions**: Miner submits multiple models per round

### Recovery Mechanisms
```javascript
// Handle model submission errors
try {
    await createModel(contractModel, modelData);
} catch (error) {
    if (error.includes('duplicate')) {
        // Update existing model
        await updateModel(contractModel, modelData);
    } else {
        // Log error and continue
        console.error('Model submission failed:', error);
    }
}
```

## Configuration

### Chaincode Deployment
```bash
./network.sh deployCC -ccp ../model-propose/model-propose-chaincode \
  -ccn modelCC -c demo -ccl javascript
```

### System Parameters
- **Max Test Samples**: 50 per miner
- **Model File Format**: .keras (TensorFlow/Keras)
- **Hash Algorithm**: MD5
- **Channel**: demo (shared with other consensus components)

## Monitoring and Analytics

### Key Metrics
- **Submission Rate**: Models submitted per round
- **Success Rate**: Successful model validations
- **Data Volume**: Total test data processed
- **Processing Time**: Time from submission to availability

### Logging
- Model submission confirmations
- Hash verification results
- Test data collection status
- Error messages and debugging information

## Testing

### Unit Tests
```javascript
// Test model submission
const modelData = {
    id: "test_model_1",
    hash: "test_hash",
    path: "/test/path/model.keras",
    transactions: ["tx1"],
    testData: [[1, 2, 3]]
};
await contract.createModel(modelData);

// Test retrieval
const retrieved = await contract.readModel("test_model_1");
assert.equal(retrieved.id, "test_model_1");
```

### Integration Tests
Test complete model flow from submission through prediction to winner selection.

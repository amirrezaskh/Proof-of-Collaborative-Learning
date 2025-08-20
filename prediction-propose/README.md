# Prediction Propose Chaincode

This directory contains the chaincode and application for managing prediction submissions in the Proof of Collaborative Learning (PoCL) system. After models are proposed, miners make predictions on each other's test data for evaluation and voting.

## Overview

The prediction propose system handles:
- **Prediction Submission**: Miners submit predictions on other miners' test data
- **Prediction Collection**: Gathering predictions for voting and evaluation
- **Cross-validation**: Each miner predicts on all other miners' test datasets
- **Performance Tracking**: Timestamps and metadata for prediction evaluation

## Directory Structure

```
prediction-propose/
├── prediction-propose-chaincode/     # Smart contract implementation
│   ├── index.js                      # Chaincode entry point
│   ├── package.json                  # Dependencies and metadata
│   └── lib/                         # Chaincode implementation
│       └── predictionPropose.js     # Main contract logic
└── prediction-propose-application/   # Client application
    ├── predApp.js                   # Application interface
    └── package.json                 # Dependencies and metadata
```

## Core Functionality

### Prediction Management
- **Prediction Submission**: Accept predictions from miners on test data
- **Cross-miner Evaluation**: Each miner predicts on others' test datasets
- **Timestamp Tracking**: Record when predictions were made
- **Format Validation**: Ensure prediction format consistency

### Data Collection
- **Batch Collection**: Gather all predictions for a specific model's test data
- **Miner-specific Views**: Provide predictions relevant to each miner
- **Performance Metrics**: Track prediction timing and accuracy
- **Round Management**: Handle predictions per consensus round

### System Coordination
- **Status Control**: Manage accepting new predictions
- **Round Reset**: Clear predictions between rounds
- **Integration**: Coordinate with voting system

## Data Models

### Prediction Record Structure
```javascript
{
  id: "pred_1",                    // Predictor miner ID
  predictions: {
    "model_2": [1, 0, 1, 1, 0],   // Predictions for model_2's test data
    "model_3": [1, 1, 1, 0, 0],   // Predictions for model_3's test data
    "model_4": [0, 1, 0, 1, 1],   // Predictions for model_4's test data
    // ... predictions for all other miners' test data
  },
  timestamp: "2025-01-01T12:00:00Z",
  round: 5,
  status: "submitted"
}
```

### Prediction Result for Voting
```javascript
{
  "pred_1": {
    "prediction": [1, 0, 1, 1, 0],  // Predictions on this miner's test data
    "time": "2025-01-01T12:00:00.123Z"  // Timestamp for speed evaluation
  },
  "pred_2": {
    "prediction": [1, 1, 1, 0, 0],
    "time": "2025-01-01T12:00:01.456Z"
  }
  // ... predictions from all other miners
}
```

## Application Interface

### Core Functions

#### Prediction Operations
```javascript
// Submit predictions for other miners' test data
createPrediction(contractPred, id, predictions)

// Read specific prediction record
readPrediction(contractPred, predictionId)

// Get all submitted predictions
getAllPredictions(contractPred)

// Delete prediction record
deletePrediction(contractPred, predictionId)
```

#### Collection and Distribution
```javascript
// Gather all predictions for a specific miner's test data
gatherAllPredictions(contractPred, modelId)

// Get formatted predictions for voting
getFormattedPredictions(contractPred, modelId)
```

#### System Management
```javascript
// Initialize prediction ledger
initPredictions(contractPred)

// Toggle accepting new predictions
toggleAcceptingStatus(contractPred)

// Delete all predictions (round reset)
deleteAllPredictions(contractPred)
```

## Process Flow

### 1. Prediction Phase Start
After test data collection, miners begin making predictions:
```javascript
// In app1.js
async function gatherPredictions() {
    await predApp.toggleAcceptingStatus(contractPred);
    // Notify miners to start predicting
    for (const port of minersPorts) {
        await axios.get(`http://localhost:${port}/tests/ready/`);
    }
}
```

### 2. Prediction Submission
Miners predict on all other miners' test data:
```python
# In miner.py
def predict(self):
    self.get_test_records()
    
    predictions = {}
    for key in self.test_records.keys():
        if key[6:] != self.name[6:]:  # Don't predict on own data
            test_data = self.test_records[key]
            test_data = np.array(test_data)
            test_data = self.preprocess(test_data)
            prediction = self.model.predict(test_data)
            prediction = np.argmax(prediction, axis=1).tolist()
            predictions[key] = prediction
    
    requests.post(f"http://localhost:{self.peer_port}/api/pred/", json={
        "id": f"pred_{self.name[6:]}",
        "predictions": predictions
    })
```

### 3. Prediction Collection
For voting, predictions are collected per miner:
```python
# Miners get predictions on their test data
def get_predictions(self):
    res = requests.get(f"http://localhost:{self.peer_port}/api/preds/miner", json={
        "id": f"model_{self.name[6:]}"
    })
    self.predictions = json.loads(res.content)
```

### 4. Voting Preparation
Predictions are formatted for voting evaluation:
```javascript
// Format for voting system
const formattedPredictions = await predApp.gatherAllPredictions(contractPred, modelId);
```

## API Endpoints

### Prediction Management
- `POST /api/pred/ledger/` - Initialize prediction ledger
- `POST /api/pred/` - Submit new prediction
- `GET /api/pred/` - Read specific prediction
- `DELETE /api/pred/` - Delete prediction
- `GET /api/preds/` - Get all predictions

### Miner-specific Operations
- `GET /api/preds/miner/` - Get predictions for specific miner's test data

## Prediction Format

### Input Format
Predictions are submitted as class indices (0-9 for CIFAR-10):
```javascript
{
  "id": "pred_1",
  "predictions": {
    "model_2": [1, 0, 1, 1, 0, 3, 7, 2, 9, 4, ...], // 50 predictions
    "model_3": [1, 1, 1, 0, 0, 2, 8, 1, 5, 6, ...], // 50 predictions
    // ... for all other miners
  }
}
```

### Output Format for Voting
```javascript
{
  "pred_1": {
    "prediction": [1, 0, 1, 1, 0, ...],
    "time": "2025-01-01T12:00:00.123Z"
  },
  "pred_2": {
    "prediction": [1, 1, 1, 0, 0, ...],
    "time": "2025-01-01T12:00:01.456Z"
  }
}
```

## Performance Evaluation

### Accuracy Calculation
Miners evaluate prediction accuracy against their ground truth:
```python
# In miner voting process
labels = self.y_test[self.test_indexes]
for key in self.predictions.keys():
    pred = np.array(self.predictions[key]['prediction'])
    pred = np.expand_dims(pred, axis=-1)
    acc = ((pred == labels).sum()) / len(labels)
```

### Timing Evaluation
Prediction timestamps are used to evaluate speed:
```python
# Speed consideration in voting
dt = datetime.datetime.fromisoformat(self.predictions[key]['time'][:-1])
metrics[f"model_{key_num}"] = (acc, dt)
```

### Combined Ranking
Accuracy and speed are combined for final ranking:
```python
def custom_compare(self, a, b):
    # Primary: Higher accuracy wins
    # Secondary: Faster prediction wins (lower timestamp)
    if (a[0] > b[0]) or ((a[0] == b[0]) and (a[1] < b[1])):
        return 1
    return -1
```

## Integration with PoCL System

### Phase Coordination
```javascript
// After model submission deadline
async function gatherTestData() {
    await modelApp.gatherAllTestRecords(contractModel);
    // Notify miners to predict
    setTimeout(gatherPredictions, predProposeDeadline*1000);
}

// After prediction deadline
async function gatherPredictions() {
    await predApp.toggleAcceptingStatus(contractPred);
    // Notify miners to vote
    setTimeout(selectWinners, voteAssignDeadline*1000);
}
```

### Voting System Integration
Predictions feed directly into the voting system:
```python
# Miners use predictions for voting
self.get_predictions()  # From prediction chaincode
metrics = self.evaluate_predictions()
votes = self.rank_miners(metrics)
self.submit_votes(votes)  # To voting chaincode
```

## State Management

### Round-based Reset
After each round:
- All prediction records are deleted
- Accepting status is reset
- Timestamps are cleared
- New round begins with clean state

### Status Tracking
```javascript
{
  acceptingPredictions: true,    // Whether accepting new predictions
  totalPredictions: 8,          // Predictions submitted this round
  roundNumber: 5,               // Current round
  deadline: "2025-01-01T12:00:15Z"  // Prediction deadline
}
```

## Performance Optimizations

### Efficient Data Handling
- **Batch Processing**: Handle multiple predictions efficiently
- **Lazy Loading**: Load prediction data only when needed
- **Memory Management**: Efficient storage of prediction arrays
- **Concurrent Submissions**: Support simultaneous predictions

### Network Efficiency
- **Compressed Predictions**: Minimize network overhead
- **Streaming**: Handle large prediction datasets
- **Caching**: Cache frequently accessed predictions

## Security Features

### Prediction Validation
- **Format Verification**: Ensure predictions match expected format
- **Range Validation**: Verify predictions are valid class indices (0-9)
- **Duplicate Prevention**: Prevent multiple submissions per miner
- **Timestamp Integrity**: Ensure accurate timing records

### Privacy Protection
- **No Ground Truth**: Predictions don't reveal test labels
- **Miner Isolation**: Predictions are isolated per miner
- **Data Protection**: Prevent unauthorized access to predictions

## Error Handling

### Common Issues
- **Format Errors**: Predictions in wrong format or wrong size
- **Duplicate Submissions**: Miner submits multiple predictions
- **Timing Issues**: Predictions submitted after deadline
- **Missing Data**: Incomplete prediction sets

### Recovery Mechanisms
```javascript
// Handle prediction submission errors
try {
    await createPrediction(contractPred, predictionData);
} catch (error) {
    if (error.includes('duplicate')) {
        // Update existing prediction
        await updatePrediction(contractPred, predictionData);
    } else if (error.includes('deadline')) {
        // Reject late submission
        throw new Error('Prediction deadline exceeded');
    }
}
```

## Configuration

### Timing Parameters
- **Prediction Deadline**: 15 seconds from phase start
- **Submission Window**: Limited time for all miners to submit
- **Grace Period**: Small buffer for network latency

### Data Parameters
- **Prediction Count**: 50 predictions per miner pair
- **Class Range**: 0-9 (CIFAR-10 classes)
- **Format**: Integer arrays

## Monitoring and Analytics

### Key Metrics
- **Submission Rate**: Predictions submitted per round
- **Accuracy Distribution**: Range of prediction accuracies
- **Timing Analysis**: Prediction speed variations
- **Completion Rate**: Percentage of miners submitting predictions

### Performance Tracking
```javascript
{
  round: 5,
  totalPredictions: 10,
  averageAccuracy: 0.73,
  fastestPrediction: 2.1, // seconds
  slowestPrediction: 8.7, // seconds
  completionRate: 0.9     // 90% of miners participated
}
```

## Testing

### Unit Tests
```javascript
// Test prediction submission
const predictionData = {
    id: "pred_test_1",
    predictions: {
        "model_2": [1, 0, 1, 1, 0],
        "model_3": [1, 1, 1, 0, 0]
    }
};
await contract.createPrediction(predictionData);

// Test collection
const collected = await contract.gatherAllPredictions("model_2");
assert(collected["pred_test_1"]);
```

### Integration Tests
Test complete prediction flow from submission through collection to voting evaluation.

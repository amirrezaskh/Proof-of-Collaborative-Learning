# Aggregator

The aggregator component implements the Federated Averaging (FedAvg) algorithm to combine winning miners' models and update the global model in the Proof of Collaborative Learning (PoCL) system.

## Overview

The aggregator is responsible for:
- Collecting trained models from winning miners
- Aggregating model weights using FedAvg algorithm
- Calculating rewards based on model contribution significance
- Updating the global model with aggregated weights
- Ensuring model integrity through hash verification

## Architecture

### Core Functionality

#### 1. Model Aggregation
The aggregator implements the FedAvg algorithm:

```python
def aggregate_weights(models):
    # 1. Load current global model
    # 2. Initialize new weight matrices
    # 3. For each winning model:
    #    - Verify hash integrity
    #    - Calculate contribution difference
    #    - Add weights to aggregation
    # 4. Average all weights
    # 5. Update global model
```

#### 2. Reward Calculation
Rewards are calculated based on the significance of each miner's contribution:

```python
# Calculate mean difference between global and local weights
sum_diff_layers = sum(np.mean(np.abs(global_weights[i] - local_weights[i])) 
                     for i in range(len(layers)))
mean_diff_layers = sum_diff_layers / len(layers)
reward = round(mean_diff_layers * 100, 4)
```

#### 3. Model Integrity Verification
Each model is verified using MD5 hash:

```python
hash_value = hashlib.md5(open(model['path'],'rb').read()).hexdigest()
if hash_value == model['hash']:
    # Process the model
```

## API Endpoints

The aggregator runs a Flask server on port 5050 with the following endpoints:

### POST /aggregate/
Aggregates winning models and returns reward information.

**Request Body:**
```json
{
  "models": [
    {
      "id": "model_1",
      "hash": "md5_hash_string",
      "path": "/absolute/path/to/model.keras"
    }
  ]
}
```

**Response:**
```json
[
  {
    "id": "miner_1",
    "reward": "0.1234"
  }
]
```

### GET /exit/
Gracefully shuts down the aggregator service.

## Aggregation Process

### Step 1: Model Collection
- Receive models from winning miners
- Verify model paths and hashes
- Load each validated model

### Step 2: Weight Aggregation
```python
# Initialize aggregated weights
new_layers = [np.zeros_like(layer) for layer in global_layers]

# Sum all winning model weights
for model in valid_models:
    local_layers = model.get_weights()
    for i in range(len(new_layers)):
        new_layers[i] += local_layers[i]

# Average the weights
for i in range(len(new_layers)):
    new_layers[i] = new_layers[i] / count
```

### Step 3: Global Model Update
```python
global_model.set_weights(new_layers)
global_model.save("../global model/global_model.keras")
```

## Reward System

### Contribution Measurement
The reward system measures how much each miner's model differs from the current global model:

1. **Layer-wise Difference**: Calculate absolute difference between global and local weights for each layer
2. **Mean Aggregation**: Average differences across all layers
3. **Scaling**: Multiply by 100 for reward points
4. **Precision**: Round to 4 decimal places

### Reward Distribution
- Higher differences indicate more significant contributions
- Rewards are proportional to the magnitude of model updates
- All winning miners receive rewards based on their contribution

## Configuration

### Network Settings
- **Port**: 5050
- **Host**: localhost
- **Debug Mode**: Enabled for development

### Model Settings
- **Global Model Path**: `../global model/global_model.keras`
- **Hash Algorithm**: MD5
- **Weight Precision**: 4 decimal places

## Usage

### Automatic Start
The aggregator is automatically started by the main `run.py` script:

```python
executor.submit(os.system, "python3 ./aggregator.py > ../../logs/aggregator.txt")
```

### Manual Start
```bash
cd clients/aggregator
python3 aggregator.py
```

### API Usage
```python
import requests

# Aggregate models
response = requests.post("http://localhost:5050/aggregate/", json={
    "models": winning_models
})
rewards = response.json()

# Shutdown aggregator
requests.get("http://localhost:5050/exit/")
```

## Error Handling

### Hash Mismatch
If a model's hash doesn't match, it's excluded from aggregation:
```python
if hash_value != model['hash']:
    # Skip this model
    continue
```

### File Access Errors
Models with inaccessible paths are automatically excluded from aggregation.

### Empty Model List
If no valid models are provided, the global model remains unchanged.

## Logging

The aggregator logs:
- Successful model aggregations
- Reward calculations for each miner
- Hash verification results
- Global model update confirmations
- Error messages for debugging

## Integration

### With Express Applications
The aggregator is called by the Express.js application (app1.js) after winner selection:

```javascript
const res = await axios.post(`http://localhost:${aggregatorPort}/aggregate/`, {
    models: models
});
await distributeRewards(res.data);
```

### With Blockchain
Rewards calculated by the aggregator are distributed through the main coin transfer chaincode.

## Security Features

- **Model Integrity**: MD5 hash verification prevents tampered models
- **Path Validation**: Absolute path requirements prevent directory traversal
- **Isolation**: Each model is loaded in isolation to prevent interference
- **Error Containment**: Failed aggregations don't affect the global model

## Performance Considerations

- **Concurrent Processing**: Uses ThreadPoolExecutor for potential parallelization
- **Memory Management**: Models are loaded individually to manage memory usage
- **Atomic Updates**: Global model updates are atomic operations
- **Error Recovery**: Failed aggregations don't corrupt the global model state

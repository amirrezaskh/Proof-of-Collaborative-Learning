# Miners

This directory contains the implementation of miners that participate in the Proof of Collaborative Learning (PoCL) federated learning consensus mechanism.

## Overview

Miners are the core participants in the PoCL system. They train machine learning models on their local data, propose models for evaluation, make predictions on test data, and vote on model quality to determine winners.

## Files

### Core Implementation
- **`miner.py`** - Base miner class containing all core functionality
- **`miner1.py` to `miner10.py`** - Individual miner instances (10 total miners)

### Analysis and Testing
- **`Adversary.py`** - Adversarial miner implementation for security testing
- **`datasize_winners.py`** - Analysis of correlation between data size and winning frequency
- **`training_results.py`** - Training results analysis and visualization utilities

## Miner Architecture

### Core Components

#### 1. Data Management
- **Dataset**: CIFAR-10 (32x32 RGB images, 10 classes)
- **Data Distribution**: Heterogeneous partitioning across miners
- **Preprocessing**: Normalization to [0,1] range

#### 2. Model Training
- **Base Model**: Global CNN model loaded from `../global model/`
- **Local Training**: Fine-tuning on local data partition
- **Early Stopping**: Prevents overfitting with patience=5
- **Deadline Control**: TimerCallback stops training before deadline

#### 3. Consensus Participation
- **Transaction Mining**: Request and process blockchain transactions
- **Model Proposal**: Submit trained model with test data
- **Prediction**: Evaluate other miners' test data
- **Voting**: Rank other miners based on accuracy and speed

## Data Distribution Strategies

### Option 1: Decreasing Distribution
```python
[(10 - i) / 55 for i in range(10)]
```
Results in decreasing data sizes: [0.18, 0.16, 0.15, 0.13, 0.11, 0.09, 0.07, 0.05, 0.04, 0.02]

### Option 2: Grouped Distribution (Current)
```python
[((i // 5) * 3 + 1) / 25 for i in range(10)]
```
Results in two groups: [0.04, 0.04, 0.04, 0.04, 0.04, 0.16, 0.16, 0.16, 0.16, 0.16]

## Mining Process

### Phase 1: Transaction Assignment
```python
def get_transactions(self):
    # Request transactions from admin peer
    # Maximum 2 transactions per miner per round
```

### Phase 2: Model Training
```python
def train(self):
    # 1. Load global model
    # 2. Train on local data with deadline control
    # 3. Save model and compute hash
    # 4. Propose model with test data
```

### Phase 3: Prediction
```python
def predict(self):
    # 1. Receive test data from other miners
    # 2. Make predictions using trained model
    # 3. Submit predictions to blockchain
```

### Phase 4: Voting
```python
def vote(self):
    # 1. Evaluate prediction accuracy
    # 2. Consider prediction timing
    # 3. Rank miners from best to worst
    # 4. Submit votes to blockchain
```

## Configuration Parameters

```python
class Miner:
    max_trx = 2              # Max transactions per round
    total_miners = 10        # Total number of miners
    test_size = 50          # Test samples for evaluation
    random_state = 97       # For reproducible splits
```

## Network Communication

### Ports
- **Miner 1**: Port 8000
- **Miner 2**: Port 8001
- **...**: ...
- **Miner 10**: Port 8009

### API Endpoints
Each miner runs a Flask server with endpoints:
- `/transactions/ready/` - Receive training notifications
- `/tests/ready/` - Receive prediction notifications
- `/preds/ready/` - Receive voting notifications
- `/exit/` - Graceful shutdown

## Voting Algorithm

Miners use a custom comparison function for ranking:

```python
def custom_compare(self, a, b):
    # Primary: Higher accuracy wins
    # Secondary: Faster prediction wins (lower timestamp)
    if (a[0] > b[0]) or ((a[0] == b[0]) and (a[1] < b[1])):
        return 1
    return -1
```

## Security Features

### Model Integrity
- **Hash Verification**: MD5 hash of model file
- **Path Validation**: Absolute path verification
- **Deadline Enforcement**: Prevents unlimited training time

### Attack Resistance
- **Adversarial Testing**: `Adversary.py` implements attack scenarios
- **Vote Validation**: Consensus-based winner selection
- **KNN Attack Detection**: Configurable attack simulation

## Training Callbacks

### TimerCallback
```python
class TimerCallback(tf.keras.callbacks.Callback):
    def on_epoch_end(self, epoch, logs=None):
        if time.time() > self.deadline:
            self.model.stop_training = True
```

### Early Stopping
- **Patience**: 5 epochs without improvement
- **Monitor**: Validation loss
- **Restore Best**: Keep best weights

## Results Storage

Training results are saved in `./results/` directory:
- **Model Files**: `{miner_name}_round_{round}.keras`
- **Training History**: `{miner_name}_round_{round}.json`
- **Metrics**: Accuracy, loss, validation metrics

## Scaling the Network

This code is initially configured to include 10 miners. However, it can easily be altered to contain more or less miners. If you want to include more miners in the network:

1. Copy and paste the code from one of the miner files
2. Update the `run.py` file to run your new miners
3. Add the port number of your miner to the `app1.js` since it is the Admin of the network
4. Update the `total_miners` parameter in the miner configuration

## Results Analysis

The results folder contains the locally trained models as well as the loss and accuracy values for each miner in each round of training. These files are helpful in observing the performance of each miner throughout training.

Run the following scripts to observe plots demonstrating the miners' performance:
- `python3 training_results.py` - Visualize training metrics
- `python3 datasize_winners.py` - Analyze correlation between data size and winning frequency

## Usage Example

```bash
# Start individual miner
python3 miner1.py > ../../logs/miner1.txt

# Start all miners (via run.py)
python3 run.py
```

## Attack Simulation

To enable KNN attack testing:
1. Uncomment attack lines in `miner1.py` and `miner6.py`
2. Comment the normal training line
3. Run the system normally with `run.py`

This simulates adversarial behavior to test the system's robustness against malicious miners.

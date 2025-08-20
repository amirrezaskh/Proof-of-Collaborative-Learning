# Clients

This directory contains all the client-side components of the Proof of Collaborative Learning (PoCL) system. The clients are responsible for the federated learning process, model aggregation, and transaction submission.

## Components

### 📁 miner/
Contains the miner implementations that participate in the federated learning process. Each miner:
- Trains the global CNN model on their local CIFAR-10 dataset partition
- Proposes trained models with test data for evaluation
- Makes predictions on other miners' test data
- Votes on model quality based on accuracy and timing
- Competes to be selected as a winner for rewards

**Key Files:**
- `miner.py` - Base miner class with core functionality
- `miner1.py` to `miner10.py` - Individual miner instances (10 total)
- `Adversary.py` - Adversarial miner implementation for attack simulations
- `datasize_winners.py` - Analysis of winners based on data size
- `training_results.py` - Training results analysis and visualization

### 📁 aggregator/
Implements the FedAvg (Federated Averaging) algorithm to:
- Collect winning miners' model weights
- Aggregate models using weighted averaging
- Update the global model with aggregated weights
- Calculate rewards based on model contribution significance
- Validate model integrity using hash verification

**Key Files:**
- `aggregator.py` - Main aggregator service with Flask API

### 📁 global model/
Contains the global CNN model shared across all miners:
- Initial model architecture definition
- Model persistence and loading functionality
- Designed for CIFAR-10 image classification (32x32x3 input, 10 classes)

**Key Files:**
- `model.py` - CNN model architecture definition and initialization
- `global_model.keras` - Serialized global model file

### 📁 submitter/
Automated transaction submission service that:
- Generates random cryptocurrency transfer transactions
- Submits transactions to the demo blockchain
- Maintains continuous transaction flow for mining

**Key Files:**
- `submitter.py` - Transaction submission service with Flask API

## Architecture Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Miner 1   │    │   Miner 2   │    │   Miner N   │
│             │    │             │    │             │
│ Local Data  │    │ Local Data  │    │ Local Data  │
│ Training    │    │ Training    │    │ Training    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           │
                  ┌─────────────┐
                  │ Aggregator  │
                  │             │
                  │ FedAvg      │
                  │ Rewards     │
                  └─────────────┘
                           │
                  ┌─────────────┐
                  │Global Model │
                  │             │
                  │ Updated     │
                  │ Weights     │
                  └─────────────┘
```

## Data Distribution

Miners use different data size allocations to simulate real-world heterogeneous environments:
- **Option 1**: Decreasing data sizes: `[(10-i)/55 for i in range(10)]`
- **Option 2**: Grouped data sizes: `[((i//5)*3+1)/25 for i in range(10)]`

## Mining Process Flow

1. **Transaction Assignment**: Miners request transactions to mine
2. **Model Training**: Local training on CIFAR-10 data partitions
3. **Model Proposal**: Submit trained model hash and test data
4. **Prediction Phase**: Predict on other miners' test data
5. **Voting Phase**: Vote on prediction quality
6. **Winner Selection**: Top K miners selected based on votes
7. **Aggregation**: FedAvg combines winner models
8. **Reward Distribution**: Winners receive rewards based on contribution

## Configuration

- **Maximum Transactions per Miner**: 2
- **Total Miners**: 10
- **Test Data Size**: 50 samples per miner
- **Winner Count**: 5 miners per round
- **Total Rounds**: 20
- **Random State**: 97 (for reproducible data splits)

## Usage

Miners are automatically started by the main `run.py` script. Each miner runs independently on different ports (8000-8009) and communicates with the blockchain network through Express.js applications.

## Security Features

- Model integrity verification using MD5 hashes
- Deadline-based training to prevent unlimited computation
- Adversarial miner detection capabilities
- Vote-based quality assessment to prevent model poisoning

# Global Model

This directory contains the global CNN model used in the Proof of Collaborative Learning (PoCL) federated learning system. The global model serves as the foundation that all miners collaboratively train and improve.

## Overview

The global model is a Convolutional Neural Network (CNN) designed for CIFAR-10 image classification. It serves as:
- The initial model that all miners start training from
- The aggregated result of federated learning rounds
- The shared knowledge base updated through consensus

## Files

- **`model.py`** - Model architecture definition and initialization script
- **`global_model.keras`** - Serialized global model file (created after running model.py)

## Model Architecture

### Network Design
The CNN consists of multiple convolutional blocks with batch normalization and max pooling:

```python
# Input Layer
input_classifier = Input((32, 32, 3), name="input_classifier")

# Block 1: 32 filters
Conv2D(32, (3, 3), activation='relu', padding='same')
BatchNormalization()
Conv2D(32, (3, 3), activation='relu', padding='same')
BatchNormalization()
MaxPooling2D((2, 2))

# Block 2: 64 filters
Conv2D(64, (3, 3), activation='relu', padding='same')
BatchNormalization()
Conv2D(64, (3, 3), activation='relu', padding='same')
BatchNormalization()
MaxPooling2D((2, 2))

# Block 3: 128 filters
Conv2D(128, (3, 3), activation='relu', padding='same')
BatchNormalization()
Conv2D(128, (3, 3), activation='relu', padding='same')
BatchNormalization()
MaxPooling2D((2, 2))

# Classification Head
Flatten()
Dropout(0.5)
Dense(10, activation="softmax", name="output_classifier")
```

### Model Specifications

#### Input
- **Shape**: (32, 32, 3) - CIFAR-10 image dimensions
- **Type**: RGB images normalized to [0, 1]
- **Classes**: 10 (airplane, automobile, bird, cat, deer, dog, frog, horse, ship, truck)

#### Architecture Details
- **Total Layers**: 13 layers (excluding activation and normalization)
- **Convolutional Layers**: 6
- **Max Pooling Layers**: 3
- **Fully Connected Layers**: 1
- **Dropout Rate**: 0.5
- **Activation Function**: ReLU (hidden), Softmax (output)

#### Model Parameters
- **Filter Sizes**: 3x3 convolutions throughout
- **Padding**: 'same' to preserve spatial dimensions
- **Pool Size**: 2x2 max pooling
- **Batch Normalization**: After each convolutional layer

## Model Lifecycle

### 1. Initialization
```bash
cd clients/global\ model/
python3 model.py
```
This creates the initial `global_model.keras` file with random weights.

### 2. Federated Training
- Miners load the global model
- Train locally on their data partitions
- Submit trained models for aggregation

### 3. Model Aggregation
The aggregator combines winning models using FedAvg:
```python
# Weighted average of model parameters
new_weights = sum(winning_model_weights) / num_winners
global_model.set_weights(new_weights)
```

### 4. Model Update
Updated global model is saved and used in the next round.

## Usage in the System

### Loading the Model
```python
import tensorflow as tf
model = tf.keras.models.load_model("../global model/global_model.keras")
```

### Training Configuration
Miners compile the model with:
```python
model.compile(
    loss="sparse_categorical_crossentropy",
    optimizer="adam",
    metrics=["accuracy"]
)
```

### Training Process
```python
history = model.fit(
    X_train, y_train,
    epochs=20,
    batch_size=32,
    validation_data=(X_test, y_test),
    callbacks=[TimerCallback(deadline), early_stopping],
    verbose=0
)
```

## Model Performance

### Training Characteristics
- **Convergence**: Typically converges within 10-20 epochs
- **Overfitting Prevention**: Dropout and early stopping
- **Batch Normalization**: Improves training stability and speed

### Expected Performance
On CIFAR-10 dataset:
- **Training Accuracy**: ~80-90%
- **Validation Accuracy**: ~70-80%
- **Model Size**: ~2.5MB serialized

## Version Management

### Model Versioning
Each round creates model snapshots:
- **File Format**: `{miner_name}_round_{round_number}.keras`
- **Storage Location**: `clients/miner/results/`
- **Hash Verification**: MD5 checksums for integrity

### Global Model Updates
- **Frequency**: Once per consensus round
- **Method**: FedAvg aggregation
- **Backup**: Previous versions preserved

## Integration Points

### With Miners
```python
# Miners load the global model
self.model = tf.keras.models.load_model("../global model/global_model.keras")
```

### With Aggregator
```python
# Aggregator updates the global model
global_model = tf.keras.models.load_model("../global model/global_model.keras")
global_model.set_weights(aggregated_weights)
global_model.save("../global model/global_model.keras")
```

## Model Initialization Script

The `model.py` script creates a fresh global model:

```python
# Define architecture
classifier = tf.keras.models.Model(input_classifier, output_classifier)

# Save initial model
classifier.save("./global_model.keras")
```

This is run automatically during system initialization in `run.py`:

```python
# Step 7: Re-initializing the model
os.chdir(os.path.join(cwd, "clients", "global model"))
os.system("python3 ./model.py")
```

## Design Rationale

### Architecture Choices
- **CNN Design**: Suitable for image classification tasks
- **Batch Normalization**: Accelerates training and improves stability
- **Dropout**: Prevents overfitting in federated setting
- **Progressive Filters**: 32→64→128 captures hierarchical features

### Federated Learning Considerations
- **Model Size**: Balanced between capacity and communication efficiency
- **Convergence**: Architecture promotes stable federated training
- **Heterogeneity**: Robust to varied local data distributions

## Security and Integrity

### Model Verification
- **Hash Checking**: MD5 verification of model files
- **Path Validation**: Secure file loading practices
- **Version Control**: Tracking of model updates

### Attack Resistance
- **Model Poisoning**: Consensus mechanism filters malicious updates
- **Byzantine Tolerance**: Aggregation handles some faulty participants
- **Validation**: Continuous performance monitoring

## Troubleshooting

### Common Issues
1. **File Not Found**: Ensure `model.py` has been run to create the initial model
2. **Version Mismatch**: TensorFlow/Keras compatibility issues
3. **Memory Errors**: Large model size in constrained environments

### Solutions
```bash
# Recreate the model
cd clients/global\ model/
rm global_model.keras
python3 model.py

# Check TensorFlow version
python3 -c "import tensorflow as tf; print(tf.__version__)"
```

# Figures

This directory contains visual diagrams and illustrations that explain the architecture, implementation, and workflow of the Proof of Collaborative Learning (PoCL) system.

## Overview

The figures provide visual documentation of:
- **System Architecture**: High-level design and component relationships
- **Implementation Details**: Technical workflow and process flows
- **Model Structure**: Neural network architecture and federated learning flow

## Files

### System Architecture
- **`Design.png`** - High-level architecture of the PoCL consensus mechanism
- **`Implementation.png`** - Detailed implementation workflow and component interactions

### Model Architecture
- **`Model.jpg`** - CNN model structure used in federated learning

## Design.png - System Architecture

This diagram illustrates the overall architecture of the Proof of Collaborative Learning system:

### Key Components Shown
1. **Miners**: Distributed federated learning participants
2. **Blockchain Network**: Hyperledger Fabric infrastructure
3. **Consensus Mechanism**: Vote-based winner selection
4. **Model Aggregation**: FedAvg algorithm implementation
5. **Reward System**: Performance-based incentive distribution

### Workflow Illustrated
1. **Transaction Assignment**: Miners receive demo transactions to process
2. **Local Training**: Each miner trains the global model on local data
3. **Model Proposal**: Trained models are submitted with test data
4. **Cross-Prediction**: Miners predict on each other's test datasets
5. **Voting Phase**: Miners vote on prediction quality
6. **Winner Selection**: Top K miners selected based on votes
7. **Model Aggregation**: Winning models combined using FedAvg
8. **Reward Distribution**: Winners receive rewards based on contribution

### Architectural Highlights
- **Decentralized**: No central authority controls the process
- **Democratic**: Equal voting rights for all participants
- **Transparent**: All operations recorded on blockchain
- **Incentivized**: Performance-based reward system

## Implementation.png - Technical Workflow

This diagram shows the detailed technical implementation and component interactions:

### System Components
1. **Express Applications**: API gateways and process coordination
2. **Hyperledger Fabric**: Blockchain infrastructure with multiple channels
3. **Chaincodes**: Smart contracts for different system functions
4. **Miners**: Python-based federated learning clients
5. **Aggregator**: Model combination service
6. **Submitter**: Transaction generation service

### Communication Flow
- **HTTP APIs**: RESTful interfaces between components
- **gRPC**: Blockchain communication protocol
- **Internal APIs**: Miner-to-miner coordination
- **File System**: Model storage and retrieval

### Channel Architecture
- **Demo Channel**: Consensus-related operations
  - Transaction management (demoCC)
  - Model proposals (modelCC)
  - Prediction submissions (predCC)
  - Voting records (voteCC)
- **Main Channel**: Cryptocurrency operations
  - Wallet management (mainCC)
  - Reward distribution

### Timing Coordination
- **Phase 1**: Model training (180 seconds)
- **Phase 2**: Prediction submission (15 seconds)
- **Phase 3**: Voting (15 seconds)
- **Phase 4**: Winner selection and aggregation

## Model.jpg - Neural Network Architecture

This figure displays the CNN model structure used for CIFAR-10 classification:

### Model Architecture
1. **Input Layer**: 32×32×3 RGB images
2. **Convolutional Blocks**: 
   - Block 1: 32 filters, 3×3 kernels
   - Block 2: 64 filters, 3×3 kernels
   - Block 3: 128 filters, 3×3 kernels
3. **Regularization**: Batch normalization after each conv layer
4. **Pooling**: 2×2 max pooling between blocks
5. **Classification Head**: Dense layer with dropout
6. **Output**: 10-class softmax for CIFAR-10

### Design Rationale
- **Progressive Filters**: Increasing complexity (32→64→128)
- **Batch Normalization**: Stable and fast training
- **Dropout**: Prevents overfitting in federated setting
- **Compact Size**: ~2.5MB for efficient network transfer

### Federated Learning Considerations
- **Parameter Sharing**: All miners start with same architecture
- **Local Adaptation**: Models fine-tune on local data partitions
- **Aggregation Compatible**: Weights can be averaged effectively
- **Convergence Stable**: Architecture promotes stable federated training

## Visual Design Principles

### Clarity and Comprehension
- **Clear Labels**: All components and connections labeled
- **Logical Flow**: Left-to-right or top-to-bottom progression
- **Color Coding**: Different colors for different component types
- **Hierarchical Layout**: System levels clearly distinguished

### Technical Accuracy
- **Actual Components**: Diagrams reflect real implementation
- **Correct Relationships**: Accurate component interactions
- **Proportional Sizing**: Component sizes reflect importance
- **Complete Coverage**: All major system elements included

### Documentation Standards
- **High Resolution**: Suitable for presentations and publications
- **Multiple Formats**: Available in PNG, SVG, or other formats
- **Consistent Style**: Uniform design across all figures
- **Version Control**: Updated with system changes

## Usage in Documentation

### Research Papers
These figures are suitable for:
- **Academic Publications**: Conference and journal papers
- **Technical Reports**: Detailed system documentation
- **Thesis Work**: Graduate research documentation
- **Patent Applications**: Technical invention descriptions

### Presentations
- **Conference Talks**: Research presentation slides
- **Demo Sessions**: System demonstration materials
- **Stakeholder Meetings**: Executive and technical briefings
- **Educational Content**: Teaching materials for federated learning

### Development Documentation
- **System Design**: Architecture documentation
- **Implementation Guides**: Developer onboarding materials
- **API Documentation**: Interface specifications
- **Troubleshooting Guides**: Debug and maintenance procedures

## Creating Additional Figures

### Recommended Tools
- **Architecture Diagrams**: Draw.io, Visio, Lucidchart
- **Network Diagrams**: Graphviz, yEd, Cytoscape
- **Model Visualizations**: TensorBoard, Netron, PlotNeuralNet
- **Flow Charts**: Mermaid, Drawio, OmniGraffle

### Figure Standards
- **Resolution**: Minimum 300 DPI for print quality
- **Format**: PNG for web, SVG for scalability
- **Naming**: Descriptive filenames with version numbers
- **Documentation**: README description for each figure

### Update Process
When system changes require figure updates:
1. **Identify Changes**: Determine which figures need updates
2. **Modify Source**: Update original design files
3. **Export Formats**: Generate PNG/SVG versions
4. **Update Documentation**: Revise descriptions in README
5. **Version Control**: Commit changes with clear messages

## Integration with System

### Automatic Generation
Some figures can be generated programmatically:
```python
# Example: Generate model architecture diagram
import tensorflow as tf
from tensorflow.keras.utils import plot_model

model = tf.keras.models.load_model("clients/global model/global_model.keras")
plot_model(model, to_file="figures/model_architecture.png", 
           show_shapes=True, show_layer_names=True)
```

### Dynamic Visualization
Create figures that update with system state:
```python
# Example: Network topology from current miners
import networkx as nx
import matplotlib.pyplot as plt

# Create network graph of active miners
G = nx.Graph()
active_miners = get_active_miners()
for miner in active_miners:
    G.add_node(miner)
    G.add_edges_from([(miner, other) for other in active_miners if other != miner])

nx.draw(G, with_labels=True)
plt.savefig("figures/current_network_topology.png")
```

### Interactive Diagrams
For web-based documentation:
- **Mermaid**: Markdown-based diagram generation
- **D3.js**: Interactive network visualizations
- **Plotly**: Interactive performance charts
- **Cytoscape.js**: Dynamic network diagrams

## Accessibility and Internationalization

### Accessibility Features
- **Alt Text**: Descriptive text for screen readers
- **Color Blind Friendly**: Colorblind-safe color palettes
- **High Contrast**: Sufficient contrast ratios
- **Large Text**: Readable font sizes

### Multiple Languages
For international collaboration:
- **English**: Primary documentation language
- **Multilingual**: Consider translations for key figures
- **Universal Symbols**: Use internationally recognized symbols
- **Minimal Text**: Rely on visual elements over text labels

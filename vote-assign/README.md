# Vote Assign Chaincode

This directory contains the chaincode and application for managing the voting system in the Proof of Collaborative Learning (PoCL) consensus mechanism. Miners vote on prediction quality to determine the winning miners for each round.

## Overview

The vote assign system handles:
- **Vote Collection**: Miners submit ranked votes based on prediction performance
- **Winner Selection**: Determine top K miners based on aggregated votes
- **Consensus Mechanism**: Implement voting-based consensus for federated learning
- **Performance Ranking**: Rank miners by accuracy and prediction speed

## Directory Structure

```
vote-assign/
├── vote-assign-chaincode/     # Smart contract implementation
│   ├── index.js               # Chaincode entry point
│   ├── package.json          # Dependencies and metadata
│   └── lib/                  # Chaincode implementation
│       └── voteAssign.js     # Main contract logic
└── vote-assign-application/   # Client application
    ├── voteApp.js            # Application interface
    └── package.json          # Dependencies and metadata
```

## Core Functionality

### Vote Management
- **Vote Submission**: Accept ranked votes from miners
- **Vote Validation**: Ensure vote format and completeness
- **Vote Storage**: Maintain secure vote records
- **Duplicate Prevention**: Prevent multiple votes from same miner

### Winner Selection Algorithm
- **Rank Aggregation**: Combine votes from all miners
- **Scoring System**: Calculate aggregate scores for each miner
- **Top-K Selection**: Select specified number of winners
- **Tie Breaking**: Handle tied scores fairly

### Consensus Implementation
- **Democratic Voting**: Each miner has equal voting weight
- **Performance-based**: Votes based on prediction accuracy and speed
- **Byzantine Tolerance**: Resilient to some dishonest voters
- **Transparency**: All votes are recorded on blockchain

## Data Models

### Vote Record Structure
```javascript
{
  id: "vote_1",                 // Voter miner ID
  votes: [                      // Ranked list from best to worst
    "model_3",                  // Best performing miner
    "model_1",                  // Second best
    "model_7",                  // Third best
    "model_2",                  // Fourth best
    // ... remaining miners in ranked order
  ],
  timestamp: "2025-01-01T12:00:00Z",
  round: 5,
  status: "submitted"
}
```

### Aggregated Voting Results
```javascript
{
  round: 5,
  totalVotes: 10,
  rankings: {
    "model_3": {
      totalScore: 85,           // Aggregate score
      averageRank: 1.5,         // Average ranking position
      votes: [1, 1, 2, 1, 2, ...] // Individual rank positions
    },
    "model_1": {
      totalScore: 78,
      averageRank: 2.3,
      votes: [2, 3, 1, 2, 3, ...]
    }
    // ... all miners
  },
  winners: ["model_3", "model_1", "model_7", "model_2", "model_5"]
}
```

## Application Interface

### Core Functions

#### Vote Operations
```javascript
// Submit ranking vote from miner
createVote(contractVote, id, votes)

// Read specific vote record
readVote(contractVote, voteId)

// Get all submitted votes
getAllVotes(contractVote)

// Delete vote record
deleteVote(contractVote, voteId)
```

#### Winner Selection
```javascript
// Select top K winners based on votes
selectWinners(contractVote, winnerCount)

// Get current voting results
getVotingResults(contractVote)

// Calculate aggregate scores
calculateScores(contractVote)
```

#### System Management
```javascript
// Initialize voting ledger
initVotes(contractVote)

// Delete all votes (round reset)
deleteAllVotes(contractVote)

// Get voting statistics
getVotingStats(contractVote)
```

## Voting Process

### 1. Vote Generation (Miner Side)
Miners evaluate predictions and generate ranked votes:
```python
# In miner.py
def vote(self):
    self.get_predictions()
    
    labels = self.y_test[self.test_indexes]
    metrics = {}
    
    # Evaluate each miner's predictions
    for key in self.predictions.keys():
        pred = np.array(self.predictions[key]['prediction'])
        pred = np.expand_dims(pred, axis=-1)
        acc = ((pred == labels).sum()) / len(labels)
        dt = datetime.datetime.fromisoformat(self.predictions[key]['time'][:-1])
        metrics[f"model_{key_num}"] = (acc, dt)
    
    # Sort by accuracy (primary) and speed (secondary)
    metrics = {k: v for k, v in sorted(metrics.items(), 
                                     key=cmp_to_key(self.custom_compare), 
                                     reverse=True)}
    
    # Submit ranked vote
    requests.post(f"http://localhost:{self.peer_port}/api/vote/", json={
        "id": f"vote_{self.name[6:]}",
        "votes": list(metrics.keys())
    })
```

### 2. Vote Aggregation
The system aggregates all votes to determine winners:
```javascript
function selectWinners(contractVote, winnerCount) {
    const votes = getAllVotes(contractVote);
    const scores = calculateAggregateScores(votes);
    const sortedMiners = Object.entries(scores)
        .sort((a, b) => b[1] - a[1])  // Sort by score descending
        .slice(0, winnerCount)        // Take top K
        .map(entry => entry[0]);      // Extract miner IDs
    
    return sortedMiners;
}
```

### 3. Winner Selection
Top K miners are selected based on aggregate scores:
```javascript
// In app1.js
async function selectWinners() {
    const winners = await voteApp.selectWinners(contractVote, winnerCount.toString());
    console.log(`Winners of the current round are : ${winners}`);
    
    // Process winner transactions
    const message = await mainApp.runWinnerTransactions(contractMain, JSON.stringify(winners));
    
    // Aggregate winner models
    const models = await modelApp.getWinnerModels(contractModel, JSON.stringify(winners));
    const aggregationResult = await aggregator.aggregate(models);
}
```

## Scoring Algorithm

### Ranking to Score Conversion
Votes are converted to scores using position-based scoring:
```javascript
function calculateScores(votes) {
    const scores = {};
    const totalMiners = 10;
    
    for (const vote of votes) {
        vote.votes.forEach((minerId, position) => {
            // Higher score for better ranking (position 0 = best)
            const score = totalMiners - position;
            scores[minerId] = (scores[minerId] || 0) + score;
        });
    }
    
    return scores;
}
```

### Example Scoring
For 10 miners with position-based scoring:
- 1st place: 10 points
- 2nd place: 9 points
- 3rd place: 8 points
- ...
- 10th place: 1 point

### Aggregate Calculation
```javascript
// Example with 3 voters, 5 miners
Vote 1: [A, B, C, D, E] → A:5, B:4, C:3, D:2, E:1
Vote 2: [B, A, C, E, D] → B:5, A:4, C:3, E:2, D:1
Vote 3: [A, C, B, D, E] → A:5, C:4, B:3, D:2, E:1

Total: A:14, B:12, C:10, E:4, D:5
Winners (top 3): [A, B, C]
```

## API Endpoints

### Vote Management
- `POST /api/vote/ledger/` - Initialize voting ledger
- `POST /api/vote/` - Submit ranking vote
- `GET /api/vote/` - Read specific vote
- `DELETE /api/vote/` - Delete vote
- `GET /api/votes/` - Get all votes

### Winner Selection
Winner selection is handled internally by the admin application.

## Integration with PoCL System

### Voting Phase Coordination
```javascript
// In app1.js
async function gatherPredictions() {
    await predApp.toggleAcceptingStatus(contractPred);
    // Notify miners to vote on predictions
    for (const port of minersPorts) {
        await axios.get(`http://localhost:${port}/preds/ready/`);
    }
    setTimeout(selectWinners, voteAssignDeadline*1000);
}
```

### Winner Processing Pipeline
```javascript
async function selectWinners() {
    // 1. Select winners based on votes
    const winners = await voteApp.selectWinners(contractVote, winnerCount.toString());
    
    // 2. Execute winner transactions
    await mainApp.runWinnerTransactions(contractMain, JSON.stringify(winners));
    
    // 3. Aggregate winner models
    const models = await modelApp.getWinnerModels(contractModel, JSON.stringify(winners));
    const rewards = await aggregator.aggregate(models);
    
    // 4. Distribute rewards
    await distributeRewards(rewards);
    
    // 5. Reset for next round
    await refreshStates(winners);
}
```

## State Management

### Round-based Reset
After each round:
- All vote records are deleted
- Voting statistics are cleared
- Winner history is updated
- New round begins with clean state

### Vote Validation
```javascript
function validateVote(vote) {
    // Check format
    if (!Array.isArray(vote.votes)) return false;
    
    // Check completeness (all miners except voter)
    if (vote.votes.length !== totalMiners - 1) return false;
    
    // Check for duplicates
    const unique = new Set(vote.votes);
    if (unique.size !== vote.votes.length) return false;
    
    // Check valid miner IDs
    for (const minerId of vote.votes) {
        if (!isValidMinerId(minerId)) return false;
    }
    
    return true;
}
```

## Performance Features

### Efficient Vote Processing
- **Batch Processing**: Handle multiple votes efficiently
- **Parallel Aggregation**: Concurrent score calculations
- **Optimized Sorting**: Efficient winner selection algorithms
- **Memory Management**: Minimal memory footprint for vote storage

### Scalability Considerations
- **Linear Complexity**: O(n*m) where n=voters, m=candidates
- **Concurrent Submissions**: Support simultaneous vote submissions
- **Fast Lookups**: Efficient vote retrieval and validation

## Security Features

### Vote Integrity
- **Blockchain Immutability**: Votes recorded on immutable ledger
- **Cryptographic Security**: Protected by blockchain consensus
- **Duplicate Prevention**: Each miner votes exactly once per round
- **Validation**: Comprehensive vote format and content validation

### Consensus Security
- **Byzantine Tolerance**: Resilient to up to 1/3 dishonest voters
- **Sybil Resistance**: Each miner has one vote regardless of computational power
- **Transparency**: All votes are publicly verifiable
- **Audit Trail**: Complete history of all voting decisions

### Attack Resistance
- **Collusion Detection**: Monitor for suspicious voting patterns
- **Vote Validation**: Prevent invalid or malformed votes
- **Time Constraints**: Prevent late vote manipulation
- **Fairness**: Equal voting weight for all participants

## Analytics and Monitoring

### Voting Statistics
```javascript
{
  round: 5,
  totalVotes: 10,
  participationRate: 1.0,      // 100% of miners voted
  consensusStrength: 0.85,     // Agreement level among voters
  winnerMargin: 15,            // Score difference between winners and losers
  votingTime: 12.3             // Average time to submit votes
}
```

### Performance Metrics
- **Participation Rate**: Percentage of miners that voted
- **Consensus Strength**: Level of agreement among voters
- **Winner Stability**: Consistency of winner selection
- **Voting Speed**: Time from notification to vote submission

## Error Handling

### Common Issues
- **Late Votes**: Votes submitted after deadline
- **Invalid Format**: Malformed vote structures
- **Duplicate Votes**: Multiple votes from same miner
- **Missing Miners**: Votes that don't include all eligible miners

### Recovery Mechanisms
```javascript
// Handle vote submission errors
try {
    await createVote(contractVote, voteData);
} catch (error) {
    if (error.includes('duplicate')) {
        throw new Error('Miner has already voted this round');
    } else if (error.includes('deadline')) {
        throw new Error('Voting deadline has passed');
    } else if (error.includes('format')) {
        throw new Error('Invalid vote format');
    }
}
```

## Configuration

### Voting Parameters
- **Winner Count**: 5 (configurable)
- **Total Miners**: 10 (affects scoring calculations)
- **Voting Deadline**: 15 seconds
- **Minimum Participation**: No minimum (accept partial votes)

### Scoring Configuration
- **Position-based Scoring**: Linear decrease from top to bottom
- **Equal Weights**: All voters have equal influence
- **Tie Breaking**: Consistent ordering for tied scores

## Testing

### Unit Tests
```javascript
// Test vote submission
const voteData = {
    id: "vote_test_1",
    votes: ["model_3", "model_1", "model_2", "model_4"]
};
await contract.createVote(voteData);

// Test winner selection
const winners = await contract.selectWinners("3");
assert.equal(winners.length, 3);
assert(winners.includes("model_3")); // Assuming model_3 has highest score
```

### Integration Tests
Test complete voting flow from prediction evaluation through vote submission to winner selection and reward distribution.

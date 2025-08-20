# Submitter

The submitter component is an automated transaction generation service that continuously creates and submits cryptocurrency transfer transactions to the demo blockchain. This provides a steady stream of transactions for miners to process during the federated learning consensus process.

## Overview

The submitter serves as a transaction generator that:
- Creates random cryptocurrency transfer transactions
- Submits transactions to the demo blockchain
- Maintains continuous transaction flow for mining
- Provides controlled transaction load for testing

## Architecture

### Core Components

#### 1. Transaction Generation
Generates random transfer transactions between main wallets:

```python
def get_random_ids():
    id_1 = random.randint(1, 10)
    id_2 = random.randint(1, 10)
    while id_1 == id_2:  # Ensure different sender/receiver
        id_2 = random.randint(1, 10)
    return id_1, id_2
```

#### 2. Transaction Submission
Submits transactions to the demo blockchain API:

```python
data = {
    "senderId": f'main_{sender_id}',
    "receiverId": f'main_{receiver_id}',
    "amount": round(random.random() * 0.4, 2)
}
requests.post("http://localhost:3001/api/demo/transaction/transfer/", json=data)
```

#### 3. Continuous Operation
Runs in a loop with configurable intervals:

```python
def send_transactions():
    counter = 0
    while True:
        # Generate and submit transaction
        time.sleep(5)  # 5-second intervals
```

## Configuration

### Transaction Parameters
- **Sender IDs**: main_1 to main_10 (10 main wallets)
- **Receiver IDs**: main_1 to main_10 (excluding sender)
- **Amount Range**: 0.00 to 0.40 tokens (2 decimal precision)
- **Submission Interval**: 5 seconds between transactions

### Network Settings
- **API Endpoint**: http://localhost:3001/api/demo/transaction/transfer/
- **Control Port**: 6060 (for shutdown endpoint)
- **Target Blockchain**: Demo channel

## API Endpoints

### GET /exit/
Gracefully shuts down the submitter service.

```python
@app.route('/exit/')
def hello_world():
    os.kill(os.getpid(), signal.SIGTERM)
```

## Operation Flow

### 1. Service Startup
```python
# Two concurrent threads:
# 1. Flask server for control API
# 2. Transaction generation loop

flask_thread = threading.Thread(target=flask_thread)
loop_function_thread = threading.Thread(target=send_transactions)
```

### 2. Transaction Generation Loop
```python
while True:
    # Step 1: Generate random wallet IDs
    sender_id, receiver_id = get_random_ids()
    
    # Step 2: Create transaction data
    data = {
        "senderId": f'main_{sender_id}',
        "receiverId": f'main_{receiver_id}',
        "amount": round(random.random() * 0.4, 2)
    }
    
    # Step 3: Submit to blockchain
    requests.post("http://localhost:3001/api/demo/transaction/transfer/", json=data)
    
    # Step 4: Log and wait
    print(f"Transaction demo_{counter} submitted.")
    time.sleep(5)
```

### 3. Transaction Processing
Once submitted, transactions are:
1. Validated by the demo chaincode
2. Stored in the demo blockchain channel
3. Assigned to miners for processing
4. Included in mining rounds

## Integration with PoCL System

### System Startup
The submitter is automatically started by `run.py`:

```python
# Step 6: Bring up submitter
print("Bringing up the submitter...")
os.chdir(os.path.join(cwd, "clients", "submitter"))
executor.submit(os.system, "python3 ./submitter.py > ../../logs/submitter.txt")
```

### Initial Transaction Batch
Before starting the submitter, `run.py` creates an initial batch:

```python
# Step 5: Add some demo transactions
print("Submitting demo transactions...")
send_trx(20)  # Submit 20 initial transactions
```

### Mining Integration
Transactions are assigned to miners through the admin API:

```python
# Miners request transactions
res = requests.post("http://localhost:3000/api/demo/transactions/assign/", json={
    "minerName": self.name,
    "count": self.max_trx
})
```

## Usage

### Automatic Start
Started automatically as part of the system:
```bash
python3 run.py
```

### Manual Start
```bash
cd clients/submitter
python3 submitter.py > ../../logs/submitter.txt
```

### Manual Stop
```bash
python3 stop.py
# or
curl http://localhost:6060/exit/
```

## Transaction Format

### Request Structure
```json
{
    "senderId": "main_3",
    "receiverId": "main_7", 
    "amount": 0.23
}
```

### Response
The demo API validates and stores the transaction, returning status information.

## Logging and Monitoring

### Transaction Logging
Each submitted transaction is logged:
```
Transaction demo_0 submitted.
Transaction demo_1 submitted.
Transaction demo_2 submitted.
...
```

### Log Files
- **Location**: `../../logs/submitter.txt`
- **Content**: Transaction submission confirmations and errors
- **Format**: Timestamped entries

## Error Handling

### Network Errors
- **Connection Failures**: Logged but don't stop the service
- **API Errors**: Logged with response details
- **Timeout Handling**: Requests have implicit timeout

### Recovery Mechanisms
- **Continuous Operation**: Service continues despite individual failures
- **Error Isolation**: Failed transactions don't affect subsequent ones
- **Graceful Shutdown**: Clean termination via API endpoint

## Performance Considerations

### Transaction Rate
- **Default Rate**: 1 transaction per 5 seconds (720 transactions/hour)
- **Configurable**: Modify `time.sleep(5)` for different rates
- **Load Testing**: Can be adjusted for stress testing

### Resource Usage
- **CPU**: Minimal computational requirements
- **Memory**: Small memory footprint
- **Network**: Lightweight HTTP requests

## Configuration Options

### Modifying Transaction Rate
```python
time.sleep(5)  # Change to adjust interval
```

### Changing Amount Range
```python
"amount": round(random.random() * 0.4, 2)  # Modify 0.4 for different max
```

### Adding More Wallets
```python
id_1 = random.randint(1, 20)  # Change 10 to 20 for more wallets
```

## Security Considerations

### Transaction Validation
- Transactions are validated by the demo chaincode
- Invalid transactions are rejected at the blockchain level
- Amount limits prevent excessive transfers

### Access Control
- Only generates valid wallet-to-wallet transfers
- No direct wallet creation or deletion
- Operates within predefined wallet ecosystem

## Testing and Development

### Unit Testing
Test transaction generation:
```python
def test_get_random_ids():
    id1, id2 = get_random_ids()
    assert id1 != id2
    assert 1 <= id1 <= 10
    assert 1 <= id2 <= 10
```

### Load Testing
Adjust submission rate for different load scenarios:
```python
time.sleep(1)   # High load: 1 transaction/second
time.sleep(10)  # Low load: 1 transaction/10 seconds
```

### Monitoring
Monitor transaction submission success rate and blockchain response times through log analysis.

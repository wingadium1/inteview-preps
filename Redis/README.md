# Redis Interview Preparation

## Core Redis Concepts

### 1. What is Redis?
- **In-memory data store**: Data stored in RAM
- **Key-value store**: NoSQL database
- **Data structures**: Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLogs
- **Use cases**: Caching, session storage, real-time analytics, message queuing
- **Performance**: Sub-millisecond latency
- **Persistence**: Optional disk persistence

### 2. Redis Data Structures

#### Strings
```bash
# Set and get
SET key "value"
GET key

# Set with expiration
SETEX key 60 "value"  # Expires in 60 seconds

# Increment/Decrement
INCR counter
DECR counter
INCRBY counter 5

# Multiple keys
MSET key1 "value1" key2 "value2"
MGET key1 key2
```

#### Lists
```bash
# Push elements
LPUSH mylist "first"    # Add to left
RPUSH mylist "last"     # Add to right

# Pop elements
LPOP mylist             # Remove from left
RPOP mylist             # Remove from right

# Get elements
LRANGE mylist 0 -1      # Get all elements
LLEN mylist             # Get length

# Use case: Queue, Stack, Activity feeds
```

#### Sets
```bash
# Add members
SADD myset "member1" "member2"

# Get members
SMEMBERS myset

# Check membership
SISMEMBER myset "member1"

# Set operations
SUNION set1 set2        # Union
SINTER set1 set2        # Intersection
SDIFF set1 set2         # Difference

# Remove member
SREM myset "member1"

# Use case: Unique items, tags, followers
```

#### Sorted Sets (ZSets)
```bash
# Add members with scores
ZADD leaderboard 100 "player1" 200 "player2"

# Get by rank
ZRANGE leaderboard 0 -1         # Ascending order
ZREVRANGE leaderboard 0 -1      # Descending order

# Get by score
ZRANGEBYSCORE leaderboard 100 200

# Get rank
ZRANK leaderboard "player1"

# Increment score
ZINCRBY leaderboard 10 "player1"

# Use case: Leaderboards, priority queues, time-series data
```

#### Hashes
```bash
# Set fields
HSET user:1000 name "John" age 30 email "john@example.com"

# Get field
HGET user:1000 name

# Get all fields
HGETALL user:1000

# Increment field
HINCRBY user:1000 age 1

# Check field existence
HEXISTS user:1000 name

# Use case: Objects, user profiles, session data
```

#### Bitmaps
```bash
# Set bit
SETBIT user:login:2024-01-01 123 1

# Get bit
GETBIT user:login:2024-01-01 123

# Count bits
BITCOUNT user:login:2024-01-01

# Use case: Real-time analytics, user activity tracking
```

#### HyperLogLog
```bash
# Add elements
PFADD unique_visitors "user1" "user2" "user3"

# Count unique elements
PFCOUNT unique_visitors

# Merge
PFMERGE result hll1 hll2

# Use case: Count unique items, unique visitors
```

### 3. Redis Persistence

#### RDB (Redis Database Backup)
- **Point-in-time snapshots**
- **Compact file**
- **Faster restarts**
- **Configuration:**
  ```bash
  save 900 1      # Save if 1 key changed in 900 seconds
  save 300 10     # Save if 10 keys changed in 300 seconds
  save 60 10000   # Save if 10000 keys changed in 60 seconds
  ```

#### AOF (Append Only File)
- **Log every write operation**
- **Better durability**
- **Larger file size**
- **Configuration:**
  ```bash
  appendonly yes
  appendfsync everysec  # always, everysec, no
  ```

#### Hybrid Persistence
- Combine RDB and AOF
- RDB for snapshots, AOF for durability

### 4. Redis Replication

#### Master-Slave Replication
- **Master**: Handles writes
- **Slave**: Handles reads, replicates from master
- **Asynchronous replication**
- **Configuration:**
  ```bash
  # On slave
  replicaof master_ip master_port
  ```

#### Benefits
- High availability
- Read scalability
- Data redundancy

### 5. Redis Sentinel

#### Features
- **Monitoring**: Monitor master and slaves
- **Notification**: Notify about issues
- **Automatic Failover**: Promote slave to master
- **Configuration Provider**: Discover current master

#### Configuration
```bash
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
```

### 6. Redis Cluster

#### Features
- **Sharding**: Distribute data across multiple nodes
- **High availability**: Automatic failover
- **16384 hash slots**: Data distributed via hash slots

#### Architecture
- Minimum 3 master nodes
- Each master can have replicas
- No single point of failure

### 7. Redis Pub/Sub

#### Commands
```bash
# Subscribe to channel
SUBSCRIBE news

# Publish message
PUBLISH news "Breaking news!"

# Pattern subscribe
PSUBSCRIBE news.*

# Unsubscribe
UNSUBSCRIBE news
```

#### Use Cases
- Real-time messaging
- Notifications
- Event-driven architecture

### 8. Redis Transactions

#### Commands
```bash
# Start transaction
MULTI

# Queue commands
SET key1 "value1"
SET key2 "value2"
INCR counter

# Execute transaction
EXEC

# Discard transaction
DISCARD
```

#### WATCH (Optimistic Locking)
```bash
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

### 9. Redis Performance

#### Optimization Techniques
1. **Use pipelining**: Reduce round trips
2. **Use connection pooling**: Reuse connections
3. **Choose right data structure**: Based on use case
4. **Set expiration**: Automatic cleanup
5. **Use lazy deletion**: DEL vs UNLINK
6. **Monitor slow commands**: SLOWLOG

#### Pipelining Example
```python
import redis

r = redis.Redis()
pipe = r.pipeline()

pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.set('key3', 'value3')

pipe.execute()
```

### 10. Common Use Cases

#### 1. Caching
```python
import redis
import json

r = redis.Redis(host='localhost', port=6379, db=0)

def get_user(user_id):
    # Try cache first
    cached = r.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)
    
    # Get from database (pseudo-code - implement based on your database)
    user = database.get_user(user_id)  # Replace with actual database call
    
    # Cache for 1 hour
    r.setex(f"user:{user_id}", 3600, json.dumps(user))
    
    return user
```

#### 2. Rate Limiting
```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)

def is_rate_limited(user_id, limit=10, window=60):
    key = f"rate_limit:{user_id}"
    
    # Using pipeline to ensure atomicity and avoid race conditions
    pipe = r.pipeline()
    pipe.incr(key)
    pipe.expire(key, window)
    result = pipe.execute()
    
    count = result[0]
    return count > limit
```

#### 3. Session Storage
```python
def save_session(session_id, data, ttl=3600):
    r.setex(f"session:{session_id}", ttl, json.dumps(data))

def get_session(session_id):
    data = r.get(f"session:{session_id}")
    return json.loads(data) if data else None
```

#### 4. Leaderboard
```python
def add_score(user_id, score):
    r.zadd("leaderboard", {user_id: score})

def get_top_players(n=10):
    return r.zrevrange("leaderboard", 0, n-1, withscores=True)

def get_user_rank(user_id):
    return r.zrevrank("leaderboard", user_id)
```

#### 5. Distributed Lock and Idempotency

##### Understanding Redis Locks vs Pessimistic Locks

**Traditional Pessimistic Lock (Database):**
- Blocks other transactions until lock is released
- Example: `SELECT ... FOR UPDATE` in MySQL
- Held until transaction commit/rollback
- Strong consistency guarantees
- Automatic release on connection failure
- No expiration time (tied to transaction)

**Redis Distributed Lock:**
- Non-blocking: fail fast or retry with backoff
- Time-limited: expires to prevent deadlocks
- Best-effort: good in practice but not as strong as DB locks
- Used for distributed mutual exclusion
- No transactional coupling with your data store

##### When to Use Redis Locks

**Good Use Cases:**
1. **Multiple instances coordination:**
   - Only one instance should send periodic billing job
   - Prevent duplicate scheduled tasks across services

2. **Idempotent or retryable operations:**
   - Can tolerate occasional overlaps
   - Duplicate detection mechanisms in place

3. **Short critical sections:**
   - Operations complete in seconds
   - Can choose safe TTLs

**When NOT to Use Redis Locks:**
- Need strong consistency for critical data updates
- Operations require DB-level transactional guarantees
- Cannot tolerate any race conditions
- Critical section may take longer than reasonable TTL

##### Basic Redis Lock Implementation
```python
import redis
import uuid
import time

r = redis.Redis(host='localhost', port=6379, db=0)

def acquire_lock(lock_name, ttl=10):
    """Acquire distributed lock with unique identifier"""
    lock_id = str(uuid.uuid4())
    # SET with NX (only if not exists) and EX (expiration)
    acquired = r.set(lock_name, lock_id, nx=True, ex=ttl)
    return (lock_id, acquired) if acquired else (None, False)

def release_lock(lock_name, lock_id):
    """Release lock only if owned by this client (using Lua script for atomicity)"""
    lua_script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
    else
        return 0
    end
    """
    return r.eval(lua_script, 1, lock_name, lock_id)

# Usage example
def process_job_with_lock(job_id):
    lock_name = f"lock:job:{job_id}"
    lock_id, acquired = acquire_lock(lock_name, ttl=30)
    
    if not acquired:
        print("Could not acquire lock - another instance is processing")
        return False
    
    try:
        # Critical section
        print("Processing job...")
        time.sleep(2)
        print("Job completed")
        return True
    finally:
        # Always release lock
        release_lock(lock_name, lock_id)
```

##### Advanced: Lock with Auto-Extension (Heartbeat)
```python
import threading
import time

def acquire_lock_with_extension(lock_name, ttl=30, extension_interval=10):
    """Acquire lock with automatic TTL extension"""
    lock_id = str(uuid.uuid4())
    acquired = r.set(lock_name, lock_id, nx=True, ex=ttl)
    
    if not acquired:
        return None, False
    
    # Start background thread to extend lock
    stop_event = threading.Event()
    
    def extend_lock():
        while not stop_event.is_set():
            time.sleep(extension_interval)
            lua_extend = """
            if redis.call("get", KEYS[1]) == ARGV[1] then
                return redis.call("expire", KEYS[1], ARGV[2])
            else
                return 0
            end
            """
            r.eval(lua_extend, 1, lock_name, lock_id, ttl)
    
    extension_thread = threading.Thread(target=extend_lock, daemon=True)
    extension_thread.start()
    
    return (lock_id, stop_event), True

def release_extended_lock(lock_name, lock_data):
    """Release lock and stop extension thread"""
    lock_id, stop_event = lock_data
    stop_event.set()
    release_lock(lock_name, lock_id)
```

##### Idempotency Pattern with Redis
```python
import hashlib
import json

def ensure_idempotency(idempotency_key, operation_func, ttl=86400):
    """
    Ensure operation is executed only once for a given idempotency key
    Returns: (result, was_cached)
    """
    RETRY_DELAY_S = 0.1  # Delay before retrying to check cache
    cache_key = f"idempotency:{idempotency_key}"
    
    # Check if operation already completed
    cached_result = r.get(cache_key)
    if cached_result:
        return json.loads(cached_result), True
    
    # Acquire lock to prevent concurrent execution
    lock_name = f"lock:{cache_key}"
    lock_id, acquired = acquire_lock(lock_name, ttl=10)
    
    if not acquired:
        # Another process is executing, wait and retry
        time.sleep(RETRY_DELAY_S)
        cached_result = r.get(cache_key)
        if cached_result:
            return json.loads(cached_result), True
        raise Exception("Operation in progress by another instance")
    
    try:
        # Double-check cache after acquiring lock
        cached_result = r.get(cache_key)
        if cached_result:
            return json.loads(cached_result), True
        
        # Execute operation
        result = operation_func()
        
        # Cache result for 24 hours
        r.setex(cache_key, ttl, json.dumps(result))
        
        return result, False
    finally:
        release_lock(lock_name, lock_id)

# Usage example
def process_payment(payment_id, amount):
    """Process payment with idempotency"""
    idempotency_key = f"payment:{payment_id}"
    
    def do_payment():
        # Actual payment processing
        print(f"Processing payment {payment_id} for ${amount}")
        # Call payment gateway, update database, etc.
        return {"status": "success", "payment_id": payment_id}
    
    result, was_cached = ensure_idempotency(idempotency_key, do_payment)
    
    if was_cached:
        print("Payment already processed - returning cached result")
    
    return result
```

##### Limitations and Trade-offs

**1. No Transactional Coupling:**
- Redis lock is separate from your database transaction
- Cannot guarantee atomicity across Redis and DB
- Example issue:
  ```
  1. Acquire Redis lock ✓
  2. Update database ✓
  3. Crash before releasing lock ✗
  4. Lock expires eventually, but others blocked temporarily
  ```

**2. Lock Expiry vs Correctness:**
- If critical section takes longer than TTL, lock expires
- Another worker can acquire lock while first is still working
- Solutions:
  - Set conservative TTL
  - Implement lock extension/heartbeat (adds complexity)
  - Design operations to complete quickly

**3. Failure Modes:**
- Redis process crashes → locks lost
- Network partitions → multiple locks possible
- Clock skew → timing issues
- Solutions:
  - Use Redlock algorithm for higher guarantees
  - Use ZooKeeper or etcd for stronger consistency
  - Accept best-effort nature for non-critical operations

**4. Redlock Algorithm (Advanced):**
```python
# Use multiple Redis instances for stronger guarantees
DRIFT_FACTOR_MS = 100  # Clock drift compensation in milliseconds
RETRY_DELAY_S = 0.1    # Delay between retries

def acquire_redlock(lock_name, ttl_seconds, redis_instances):
    """
    Acquire lock on majority of Redis instances
    Args:
        lock_name: Name of the lock
        ttl_seconds: Time to live in seconds
        redis_instances: List of Redis client instances
    Returns:
        (lock_id, success): Tuple of lock identifier and success boolean
    """
    lock_id = str(uuid.uuid4())
    start_time = time.time()
    ttl_ms = ttl_seconds * 1000  # Convert to milliseconds
    
    # Try to acquire lock on all instances
    acquired_count = 0
    for r_instance in redis_instances:
        if r_instance.set(lock_name, lock_id, nx=True, px=ttl_ms):
            acquired_count += 1
    
    # Check if acquired on majority
    elapsed_time_ms = (time.time() - start_time) * 1000
    validity_time_ms = ttl_ms - elapsed_time_ms - DRIFT_FACTOR_MS
    
    if acquired_count >= len(redis_instances) // 2 + 1 and validity_time_ms > 0:
        return lock_id, True
    else:
        # Failed to acquire majority, release all
        for r_instance in redis_instances:
            try:
                release_lock(lock_name, lock_id)
            except:
                pass
        return None, False
```

##### Best Practices for Redis Locks

1. **Always set expiration (TTL):**
   - Prevents deadlocks if process crashes
   - Choose TTL > expected operation time + buffer

2. **Use unique lock identifiers:**
   - Prevents releasing another process's lock
   - Use UUID or similar

3. **Atomic release with Lua:**
   - Check ownership before releasing
   - Prevents race conditions

4. **Retry with exponential backoff:**
   ```python
   def acquire_lock_with_retry(lock_name, max_retries=5):
       for attempt in range(max_retries):
           lock_id, acquired = acquire_lock(lock_name)
           if acquired:
               return lock_id, True
           time.sleep(0.1 * (2 ** attempt))  # Exponential backoff
       return None, False
   ```

5. **Monitor lock metrics:**
   - Track lock acquisition time
   - Alert on high contention
   - Monitor lock hold time

6. **Design for idempotency:**
   - Make operations naturally idempotent when possible
   - Store operation results for deduplication
   - Use idempotency keys for critical operations

7. **Fallback strategies:**
   - Have a plan if lock acquisition fails
   - Consider queue-based alternatives
   - Implement circuit breakers

## Common Interview Questions

### Basic Questions
1. What is Redis and what are its use cases?
2. What data structures does Redis support?
3. How is Redis different from Memcached?
4. What is the difference between RDB and AOF persistence?
5. How does Redis handle expiration of keys?

### Intermediate Questions
1. How do you implement caching with Redis?
2. What is Redis Pub/Sub and when to use it?
3. How does Redis replication work?
4. What is Redis Sentinel and what problem does it solve?
5. How do you handle race conditions in Redis?

### Advanced Questions
1. How does Redis Cluster work?
2. How do you implement distributed locking with Redis?
3. What is the difference between KEYS and SCAN?
4. How do you optimize Redis performance?
5. How do you monitor Redis in production?
6. What's the difference between Redis distributed locks and database pessimistic locks?
7. How do you handle lock expiration issues in Redis?
8. How would you implement idempotency using Redis?
9. What is the Redlock algorithm and when should you use it?
10. What are the limitations of using Redis for distributed locking?

## Redis Configuration

### Important Settings
```bash
# Memory
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence
save 900 1
appendonly yes
appendfsync everysec

# Replication
replicaof <master-ip> <master-port>
replica-read-only yes

# Security
requirepass your_password
rename-command FLUSHDB ""
rename-command FLUSHALL ""

# Performance
tcp-keepalive 300
timeout 0
```

### Eviction Policies
- **noeviction**: Return error when memory limit reached
- **allkeys-lru**: Remove least recently used keys
- **allkeys-lfu**: Remove least frequently used keys
- **volatile-lru**: Remove LRU keys with expiration set
- **volatile-lfu**: Remove LFU keys with expiration set
- **allkeys-random**: Remove random keys
- **volatile-random**: Remove random keys with expiration set
- **volatile-ttl**: Remove keys with shortest TTL

## Best Practices
1. Set expiration on keys to prevent memory issues
2. Use connection pooling
3. Monitor memory usage
4. Use appropriate data structures
5. Enable persistence for critical data
6. Use Redis Cluster for scalability
7. Implement proper error handling
8. Use pipelining for bulk operations
9. Monitor slow queries
10. Secure Redis (password, firewall)
11. Always set TTL on distributed locks to prevent deadlocks
12. Use unique identifiers for lock values to prevent incorrect releases
13. Implement idempotency for critical operations
14. Use Lua scripts for atomic lock operations
15. Design operations to be naturally idempotent when possible

## Common Pitfalls
1. Not setting expiration on cache keys
2. Using KEYS command in production (use SCAN)
3. Not monitoring memory usage
4. Storing large objects
5. Not using connection pooling
6. Not securing Redis instance
7. Forgetting to handle Redis failures
8. Not setting TTL on distributed locks (can cause deadlocks)
9. Releasing locks without verifying ownership
10. Using Redis locks for operations that need DB-level consistency
11. Setting lock TTL too short for the operation duration
12. Not implementing retry logic for lock acquisition
13. Treating Redis locks as true pessimistic locks
14. Not considering lock extension for long-running operations

## Resources
- Redis Documentation
- Redis in Action book
- Redis University (free courses)
- Redis Commands Reference
- Redis Best Practices Guide

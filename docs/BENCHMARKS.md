# Benchmarks

Performance benchmarks for the Tiny ARC cache implementation.

## Test Environment

- Node.js version: 25.8.1
- Machine: Linux

## Test Configuration

Benchmarks are run with cache sizes of 100, 1000, and 10000 entries.

### Test Operations

| Operation | Description |
|-----------|-------------|
| `set N items (size M)` | Insert N items into a cache of size M |
| `get N items (size M)` | Retrieve N items from a cache of size M |
| `create N instances` | Create N cache instances using factory function |
| `set 100 items` | Basic insertion performance |
| `check has 100 items` | Check if 100 items exist in cache |
| `delete 50 items` | Delete 50 items from cache |
| `has after deletions` | Check existence after deletions |
| `clear cache` | Clear all cache entries |
| `iterate keys` | Iterate over all keys |
| `iterate values` | Iterate over all values |
| `iterate entries` | Iterate over all [key, value] pairs |
| `forEach` | Use forEach to iterate over cache |
| `toJSON` | Serialize cache to JSON |
| `update 50 existing keys` | Update same key 50 times |

## Results

### Mean Performance (ms) - 5 Runs

| Test | Size 100 | Size 1000 | Size 10000 |
|------|----------|-----------|------------|
| set | 0.212 | 3.65 | 23.34 |
| get | 0.088 | 0.796 | 3.37 |
| create 1000 instances | 0.447 | | |
| set 100 items | 0.032 | | |
| check has 100 items | 0.020 | | |
| delete 50 items | 0.031 | | |
| has after deletions | 0.010 | | |
| clear cache | 0.018 | | |
| iterate keys | 0.128 | | |
| iterate values | 0.084 | | |
| iterate entries | 0.154 | | |
| forEach | 0.064 | | |
| toJSON | 0.141 | | |
| update 50 existing keys | 0.038 | | |
| Default maxSize | 100 | | |

## Performance Analysis

### O(1) Operations

The following operations maintain O(1) time complexity as expected:

- **get()**: Average 0.088ms at size 100
- **has()**: Average 0.020ms for 100 checks
- **delete()**: Average 0.031ms
- **set()**: O(1) average case with occasional eviction

### O(n) Operations

The following operations scale linearly with cache size:

- **set()** with eviction: Average 23.34ms at size 10000
- **get()** with list traversal: Average 3.37ms at size 10000

### Cache Efficiency

- Cache properly maintains size limits with eviction
- All tests show correct cache behavior (cache size = maxSize after operations)

## Running Benchmarks

```bash
# Run benchmarks
cd benchmarks
npm run benchmark

# Run benchmarks 5x and calculate mean
for i in 1 2 3 4 5; do npm run benchmark 2>&1; done
```

## Previous Results

### Size 10, 1000, 10000 (previous configuration)

| Test | Size 10 | Size 1000 | Size 10000 |
|------|---------|-----------|------------|
| set 20 items | 0.081ms | | |
| set 2000 items | | 2.034ms | |
| set 20000 items | | | 20.34ms |
| get 2000 items | | 1.007ms | |
| get 20000 items | | | 10.07ms |

## Benchmark Script

See `benchmarks/benchmarks.js` for the complete benchmark implementation.

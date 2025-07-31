Title: Bloom Filters Demystified
Description: Bloom filter is a space-efficient probabilistic data structure used to test whether an element is a member of a set.
Published: 31/07/2025
Image: /posts/images/localstack.png
PrimaryTag: aws
Tags:
  - dotnet
  - bloom-filters
---
## What is a Bloom Filter?

A Bloom filter is a space-efficient probabilistic data structure used to test whether an element is a member of a set. It allows for fast lookups with a small memory footprint but comes with a trade-off: it can produce false positives (saying an element is in the set when it isn’t) but never false negatives (if it says an element isn’t in the set, it definitely isn’t). Bloom filters are ideal for scenarios where memory efficiency and speed are critical, and a small false-positive rate is acceptable.

### Common Use Cases
- **Caching**: Check if an item exists in a cache before performing an expensive database query (e.g., in web applications).
- **Distributed Systems**: Verify if a key exists in a distributed database like Redis to avoid unnecessary lookups.
- **Network Routers**: Filter out known malicious IPs or URLs with minimal memory.
- **Spell Checkers**: Quickly determine if a word is potentially misspelled.
- **Big Data Applications**: Deduplicate large datasets or check for membership in massive sets.

In this post, we’ll focus on a specific use case: checking if a username is already taken during user registration. This is a common scenario in web applications where you want to quickly determine if a username is likely taken before querying a database, saving time and resources.

## Implementing a Bloom Filter from Scratch in C#

Let’s build a Bloom filter in C# from scratch to check if a username is already taken during user registration. This implementation is useful when you want full control over the Bloom filter’s behavior or when integrating with external libraries like Redis isn’t feasible.

### Why Use a Bloom Filter Here?
When a user tries to register with a username, checking a database for every attempt can be slow, especially with millions of users. A Bloom filter allows us to perform a quick, in-memory check to rule out usernames that are definitely not taken. If the filter indicates a username might be taken, we can then query the database to confirm, reducing the number of expensive queries.

### Step-by-Step Implementation

```csharp
using System;
using System.Collections;

public class BloomFilter
{
    private BitArray bitArray;
    private int size;
    private int hashFunctionCount;

    public BloomFilter(int size, int hashFunctionCount)
    {
        this.size = size;
        this.hashFunctionCount = hashFunctionCount;
        this.bitArray = new BitArray(size);
    }

    // Add a username to the Bloom filter
    public void Add(string username)
    {
        for (int i = 0; i < hashFunctionCount; i++)
        {
            int hash = GetHash(username, i);
            bitArray[hash] = true;
        }
    }

    // Check if a username might be taken
    public bool MightContain(string username)
    {
        for (int i = 0; i < hashFunctionCount; i++)
        {
            int hash = GetHash(username, i);
            if (!bitArray[hash])
                return false;
        }
        return true;
    }

    // Simple hash function (for demonstration; use robust hash functions in production)
    private int GetHash(string username, int seed)
    {
        int hash = username.GetHashCode() ^ seed;
        return Math.Abs(hash % size);
    }
}

// Example usage: Checking if a username is taken
class Program
{
    static void Main()
    {
        // Initialize Bloom filter with 1000 bits and 3 hash functions
        BloomFilter filter = new BloomFilter(1000, 3);

        // Add some usernames
        filter.Add("john_doe");
        filter.Add("jane_smith");

        // Test if usernames are taken
        Console.WriteLine(filter.MightContain("john_doe"));    // True
        Console.WriteLine(filter.MightContain("jane_smith"));  // True
        Console.WriteLine(filter.MightContain("alice123"));    // False (likely)
    }
}
```

### Explanation
1. **BitArray**: We use `System.Collections.BitArray` to store the bits efficiently, representing the presence of usernames in the filter.
2. **Hash Functions**: The `GetHash` method generates different hash values for a username by combining its hash code with a seed. In production, use robust hash functions like MurmurHash or FNV to minimize collisions.
3. **Size and Hash Function Count**: The bit array size (`size`) and number of hash functions (`hashFunctionCount`) determine the false-positive rate and memory usage. More hash functions reduce false positives but increase computation time.
4. **Username Example**: The Bloom filter quickly checks if a username might already be taken. If `MightContain` returns `false`, the username is definitely available, avoiding a database query. If it returns `true`, a database check confirms whether the username is actually taken.

### Limitations
- This implementation uses a simple hash function for demonstration. Real-world applications should use high-quality hash functions to ensure better distribution.
- The filter cannot remove usernames (use a Counting Bloom Filter for scenarios where usernames might be deleted).

## Using a Bloom Filter with NuGet and Redis for Distributed Applications

For distributed systems, where multiple application instances need to share the same username availability data, we can use a Bloom filter backed by Redis. This approach is ideal for scalable web applications with high registration traffic. We’ll use the `BloomFilter.Redis` NuGet package to integrate with Redis.

### Why Use Redis and a NuGet Package?
In a distributed system, user registration requests may come to different servers, each needing to check if a username is taken. Storing the Bloom filter in Redis ensures all instances access the same data, maintaining consistency. The `BloomFilter.Redis` package simplifies integration by handling the bit array storage and hash computations, saving development time compared to a custom Redis implementation.

### Step 1: Install Required Packages
Install the following NuGet packages in your project:
```
Install-Package StackExchange.Redis
Install-Package BloomFilter.Redis
```

### Step 2: Implement Bloom Filter with Redis

```csharp
using System;
using StackExchange.Redis;
using BloomFilter.Redis;

class Program
{
    static void Main()
    {
        // Connect to Redis
        ConnectionMultiplexer redis = ConnectionMultiplexer.Connect("localhost:6379");
        IDatabase db = redis.GetDatabase();

        // Initialize Bloom filter with Redis
        var bloomFilter = new BloomFilterRedis(db, "usernameFilter", 1000, 0.01); // 1000 items, 1% error rate

        // Add usernames
        bloomFilter.Add("john_doe");
        bloomFilter.Add("jane_smith");

        // Check if usernames are taken
        Console.WriteLine(bloomFilter.Contains("john_doe"));    // True
        Console.WriteLine(bloomFilter.Contains("jane_smith"));  // True
        Console.WriteLine(bloomFilter.Contains("alice123"));    // False (likely)

        // Clean up
        redis.Close();
    }
}
```

### Explanation
1. **Redis Connection**: `StackExchange.Redis` connects to a Redis instance (ensure Redis is running locally or on a server).
2. **BloomFilter.Redis**: This library stores the Bloom filter’s bit array in Redis, making it accessible across distributed application instances.
3. **Parameters**: The constructor takes the Redis database, a key name (`usernameFilter`), expected item count (1000), and desired false-positive rate (1%).
4. **Username Example**: The filter checks if a username might be taken. If `Contains` returns `false`, the username is available. If `true`, a database query confirms the status, reducing unnecessary queries in a distributed environment.
5. **Scalability**: Redis ensures all application nodes share the same Bloom filter, supporting high-throughput registration systems.

### Prerequisites
- A running Redis server (e.g., locally or on a cloud provider like AWS ElastiCache).
- Proper configuration for Redis connection strings in production.

## Benefits and Metrics

To demonstrate the benefits of Bloom filters for the username-taken use case, consider a system with 1 million registered usernames and a 1% false-positive rate:

1. **Memory Efficiency**:
   - **Traditional Set**: Storing 1 million usernames (average 20 bytes each) requires ~20 MB.
   - **Bloom Filter**: For 1 million items with a 1% false-positive rate, you need ~1.4 MB (using the formula: `size = -n * ln(p) / (ln(2)^2)`, where `n` is the number of items and `p` is the false-positive rate).
   - **Savings**: ~93% reduction in memory usage.

2. **Lookup Speed**:
   - **Traditional Set**: O(log n) for balanced trees or O(1) for hash tables, but with higher memory overhead.
   - **Bloom Filter**: O(k) where `k` is the number of hash functions (typically 3–10), with constant-time lookups regardless of set size.
   - **Example**: A Bloom filter with 3 hash functions performs lookups in ~0.1–1 µs on modern hardware, faster than or comparable to database queries for large datasets.

3. **False-Positive Rate**:
   - Configurable by adjusting the bit array size and hash function count.
   - For 1 million usernames with a 1% false-positive rate, only 1 in 100 negative lookups will incorrectly return true, requiring a database check, which is acceptable for registration systems.

4. **Distributed Performance**:
   - Using Redis, a Bloom filter can handle thousands of username checks per second across multiple nodes. For example, Redis can process ~100,000 ops/sec on a single instance, making it suitable for high-traffic registration endpoints.

### Trade-Offs
- **False Positives**: A small chance of false positives means some available usernames may require a database check, but this is a minor overhead compared to querying for every request.
- **No Deletions**: Standard Bloom filters don’t support username removal, though Counting Bloom Filters or Redis-based implementations can address this.

## Conclusion

Bloom filters are a powerful tool for optimizing memory and performance in applications like username availability checks during user registration. By implementing a Bloom filter from scratch in C#, you gain fine-grained control for single-instance applications. For distributed systems, libraries like `BloomFilter.Redis` and Redis integration enable scalability across multiple nodes. With significant memory savings (up to 93% in our example) and fast lookups, Bloom filters reduce database load and improve user experience in high-traffic systems.

Experiment with the bit array size and hash function count to balance false-positive rates and performance for your needs. For advanced scenarios, consider Redis clusters or alternative structures like Cuckoo Filters for additional functionality.

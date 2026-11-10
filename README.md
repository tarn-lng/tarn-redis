# tarn-redis

A native, zero-dependency Redis client driver written 100% in [Tarn](https://github.com/tarn-lng/tarn).

Implements the REdis Serialization Protocol (RESP) directly over native TCP sockets (`stdlib/net`). Zero C dependencies, no libhiredis, and no external linking required.

## Features

- **100% Pure Tarn**: Implemented purely in safe Tarn systems code over `net.TcpStream`.
- **Zero C Dependencies**: No `libhiredis`, no `--link`.
- **Buffered RESP Parser**: High-performance buffered reader supporting:
  - Simple Strings (`+`)
  - Errors (`-`)
  - Integers (`:`)
  - Bulk Strings (`$`) with NULL support (`$-1`)
  - Arrays (`*`) with recursive element parsing
- **Authentication**: password (`AUTH pass`) and ACL users (`AUTH user pass`), plus `SELECT` of a logical database.
- **Connection URLs**: `redis://[[user]:password@]host[:port][/db]`, percent-decoded and strictly validated. `rediss://` (TLS) is rejected rather than silently downgraded.
- **Data Structures Supported**:
  - **Connection**: `ping()`, `echo()`, `close()`
  - **Strings / KV**: `set()`, `set_ex()`, `get()`, `del()`, `exists()`, `incr()`, `decr()`, `incr_by()`
  - **Hashes**: `hset()`, `hget()`, `hdel()`, `hlen()`
  - **Lists**: `lpush()`, `rpush()`, `lpop()`, `rpop()`, `llen()`, `lrange()`
  - **Sets**: `sadd()`, `srem()`, `sismember()`, `scard()`, `smembers()`

## Quick Start

### 1. Run Redis in Docker

```bash
docker run -d --name tarn-redis -p 6379:6379 redis:7-alpine
```

### 2. Code Example

```tarn
import "redis"

fn main() {
    cfg := redis.Config.default() // 127.0.0.1:6379, no password
    var client = try redis.Client.connect(cfg)

    // Key-Value operations
    try client.set(&"greeting", &"Hello Tarn Redis!")
    match try client.get(&"greeting") {
        Some(msg) => { print(msg) }
        None => {}
    }

    // Atomic Counters
    counter := try client.incr(&"hits")
    print("Hits: " + string.from_i64(counter))

    // Lists (Queue / Stack)
    try client.rpush(&"queue", &"job-1")
    try client.rpush(&"queue", &"job-2")
    job := try client.lpop(&"queue")

    // Sets
    try client.sadd(&"tags", &"systems")
    try client.sadd(&"tags", &"fast")
    is_fast := try client.sismember(&"tags", &"fast")

    try client.close()
}
```

### 3. Connecting with a URL

Managed services (Render Key Value, Upstash, ...) hand out a connection URL:

```tarn
cfg := try redis.Config.from_url(&"redis://red-abc123:6379")
var client = try redis.Client.connect(cfg)
```

On Render, use the Key Value's **internal** URL from a service in the same region. TLS (`rediss://`, used by external URLs) is not supported yet.

### 4. Installation

```bash
tarn add @tarn-lng/redis
```

Requires Tarn 0.4.0 or newer for scoped registry packages.

## Running the Tests

```bash
tarn test tests.tarn                                   # unit tests
REDIS_URL=redis://127.0.0.1:6379 tarn test tests.tarn  # plus a real round trip
tarn run main.tarn                                     # end-to-end demo against 127.0.0.1:6379
```

## License

MIT / Apache-2.0

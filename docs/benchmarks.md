# Benchmarks

All benchmarks were run on a Intel 7th gen cpu @4.0GHz with 32KB of L1 cache, and compiled with clang 10 with `-O3`. In summary, all entity operations that are done every frame are well under 1 ms and are unlikely to be the bottleneck in your engine.

###  `view<Components...>()` (Cached)

Viewing (matching) `n` entities with 1, 2, or 4 components. This assumes the cache with the requested components is valid and does not need to be modified. This is a best (and usually the most common) case for the `view` operation.

While not relevant for this benchmark, `n` is the number of entities matched by `view<Components...>()` not the total number of entities in the world.

| `n` (# Entities) | `view<A>()` | `view<A, B>()` | `view<A, B, C, D>()` |
| ---------------- | ----------- | -------------- | -------------------- |
| 512              | 50.7 ns     | 59.6 ns        | 75.8 ns              |
| 1024             | 67.6 ns     | 76.5 ns        | 115 ns               |
| 2048             | 90.1 ns     | 98.3 ns        | 136 ns               |
| 4096             | 129 ns      | 173 ns         | 173 ns               |
| 8192             | 261 ns      | 338 ns         | 358 ns               |
| 16384            | 769 ns      | 676 ns         | 708 ns               |

Considering that the size of an entity is 2 bytes, at 16k entities we approaching the limits of the 32KB L1 cache. I suspect we are getting more cache misses at that range which causes the large decrease in performance.

### `view<Components...>()` (No Cache)

Viewing (matching) `n` entities with 1, 2, or 4 components. Here we assume no cache with the requested components exists. This case is only likely to occur in the first frame after the world is created. This is a worst case for the `view` operation, even operations that invalidate the cache (like `remove_component(entity)` are unlikely to cause a cache to be rebuilt entirely.

Note that we are using milliseconds here instead of nanoseconds.

| `n` (# Entities) | `view<A>()` | `view<A, B>()` | `view<A, B, C, D>()` |
| ---------------- | ----------- | -------------- | -------------------- |
| 512              | 0.021 ms    | 0.021 ms       | 0.021 ms             |
| 1024             | 0.041 ms    | 0.041 ms       | 0.042 ms             |
| 2048             | 0.081 ms    | 0.081 ms       | 0.082 ms             |
| 4096             | 0.165 ms    | 0.165 ms       | 0.164 ms             |

In this case the performance decreases linearly with the number of entities, regardless of the number of components. As the difference in time given a number of components is so small (as seen in the previous benchmark), it is unlikely to show up at this time scale.

### `unpack<Component>(entity)`

Retrieve a component from an entity. This is done `n` times by iterating through the entities returned from `view<A>()`. The `Time / n` column shows the time taken by each `unpack` operation.

| `n` (# Entities) | Time     | Time / `n` |
| ---------------- | -------- | ---------- |
| 1                | 382 ns   | 382 ns     |
| 8                | 486 ns   | 60.75 ns   |
| 64               | 1372 ns  | 21.44 ns   |
| 512              | 8509 ns  | 16.62 ns   |
| 4096             | 65852 ns | 16.02 ns   |

Note that the time per `unpack` operation decreases as we iterate through more entities before leveling off at ~16 ns. I suspect this is at least partly to do with the fact that components being accessed are being cached.
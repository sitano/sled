

<p align="center">
  <img src="https://raw.githubusercontent.com/spacejam/sled/master/art/tree_face_anti-transphobia.png" width="20%" height="auto" />
</p>


# sled - ~~it's all downhill from here!!!~~
[![Build Status](https://travis-ci.org/spacejam/sled.svg?branch=master)](https://travis-ci.org/spacejam/sled)
[![crates.io](https://meritbadge.herokuapp.com/sled)](https://crates.io/crates/sled)
[![documentation](https://docs.rs/sled/badge.svg)](https://docs.rs/sled)
[![chat](https://img.shields.io/discord/509773073294295082.svg?logo=discord)](https://discord.gg/Z6VsXds)

A (beta) modern embedded database. Doesn't your data deserve a (beta) beautiful new home?

```rust
use sled::Db;

let tree = Db::open(path)?;

// insert and get, similar to std's BTreeMap
tree.insert(k, v1);
assert_eq!(tree.get(&k), Ok(Some(v1)));

// range queries
let mut iter = tree.range(k..);
assert_eq!(iter.next(), Some(Ok((k, v2))));
assert_eq!(iter.next(), None);

// deletion
tree.remove(&k);

// compare and swap
tree.compare_and_swap(k, Some(v1), Some(v2));

// block until all operations are stable on disk
// (flush_async also available to get a Future)
tree.flush();
```

If your dataset resides entirely in cache (achievable at startup by setting the cache
to a large enough value and performing a full iteration) then all reads and writes are
non-blocking and async-friendly, without needing to use Futures or an async runtime.
To asynchronously suspend on the durability of writes, we support the
[`flush_async` method](https://docs.rs/sled/latest/sled/struct.Tree.html#method.flush_async),
which returns a Future that your async tasks can await the completion of if they require
high durability guarantees and you are willing to pay the latency costs of fsync.
Note that sled automatically tries to sync all data to disk several times per second
in the background without blocking user threads.

# performance

* 2 million sustained writes per second with 8 threads, 1000 8 byte keys, 10 byte values, intel 9900k, nvme
* 8.5 million sustained reads per second with 16 threads, 1000 8 byte keys, 10 byte values, intel 9900k, nvme

what's the trade-off? sled uses too much disk space sometimes. this will improve significantly before 1.0.

# features

* [API](https://docs.rs/sled) similar to a threadsafe `BTreeMap<Vec<u8>, Vec<u8>>`
* fully serializable multi-key and multi-Tree [transactions](https://docs.rs/sled/latest/sled/struct.Tree.html#method.transaction)
* fully atomic single-key operations, supports [compare and swap](https://docs.rs/sled/latest/sled/struct.Tree.html#method.compare_and_swap)
* zero-copy reads
* [write batch support](https://docs.rs/sled/latest/sled/struct.Tree.html#method.batch)
* [subscription/watch semantics on key prefixes](https://github.com/spacejam/sled/wiki/reactive-semantics)
* multiple keyspace support
* [merge operators](https://github.com/spacejam/sled/wiki/merge-operators)
* forward and reverse iterators
* a crash-safe monotonic [ID generator](https://docs.rs/sled/latest/sled/struct.Db.html#method.generate_id) capable of generating 75-125 million unique ID's per second
* [zstd](https://github.com/facebook/zstd) compression (use the `compression` build feature)
* cpu-scalable lock-free implementation
* SSD-optimized log-structured storage
* prefix encoded keys, reducing the storage cost of complex keys

# architecture

lock-free tree on a lock-free pagecache on a lock-free log. the pagecache scatters
partial page fragments across the log, rather than rewriting entire pages at a time
as B+ trees for spinning disks historically have. on page reads, we concurrently
scatter-gather reads across the log to materialize the page from its fragments.
check out the [architectural outlook](https://github.com/spacejam/sled/wiki/sled-architectural-outlook)
for a more detailed overview of where we're at and where we see things going!

# goals

1. don't make the user think. the interface should be obvious.
1. don't surprise users with performance traps.
1. don't wake up operators. bring reliability techniques from academia into real-world practice.
1. don't use so much electricity. our data structures should play to modern hardware's strengths.

# known issues, warnings

* if reliability is your primary constraint, use SQLite. sled is beta.
* if storage price performance is your primary constraint, use RocksDB. sled uses too much space sometimes.
* quite young, should be considered unstable for the time being.
* the on-disk format is going to change in ways that require manual migrations before the `1.0.0` release!
* until `1.0.0`, sled targets the *current* stable version of rust. after `1.0.0`, we will aim to trail current by at least one version. If this is an issue for your business, please consider helping us reach `1.0.0` sooner by financially supporting our efforts to get there.

# plans

* Typed Trees that support working directly with serde-friendly types instead of raw bytes,
  and also allow the deserialized form to be stored in the shared cache for speedy access.
* [LSM tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree)-like write performance
  with [traditional B+ tree](https://en.wikipedia.org/wiki/B%2B_tree)-like read performance
* MVCC and snapshots
* forward-compatible binary format
* concurrent snapshot delta generation and recovery
* consensus protocol for [PC/EC](https://en.wikipedia.org/wiki/PACELC_theorem) systems
* pluggable conflict detection and resolution strategies for gossip + CRDT-based [PA/EL](https://en.wikipedia.org/wiki/PACELC_theorem) systems
* first-class programmatic access to replication stream

# fund feature development and get commercial support

Want to support the project, prioritize a specific feature, or get commercial help with using sled in your project? [Ferrous Systems](https://ferrous-systems.com) provides commercial support for sled, and can work with you to solve a wide variety of storage problems across the latency-throughput, consistency, and price performance spectra. [Get in touch!](mailto:sled@ferrous-systems.com)

[![Ferrous Systems](https://ferrous-systems.com/images/ferrous-systems-mono-pos.svg)](https://ferrous-systems.com/)

# special thanks

[![Meili](https://avatars3.githubusercontent.com/u/43250847?s=200&v=4)](https://www.meilisearch.com/)

Special thanks to [Meili](https://www.meilisearch.com/) for providing engineering effort and other support to the sled project. They are building [an event store](https://blog.meilisearch.com/meilies-release/) backed by sled, and they offer [a full-text search system](https://github.com/meilisearch/MeiliDB) which has been a valuable case study helping to focus the sled roadmap for the future.

<p>
  <img src="https://user-images.githubusercontent.com/7989673/29498525-38a33f36-85cc-11e7-938d-ef6f10ba6fb3.png" width="20%" height="auto" />
</p>

Additional thanks to [Arm](https://www.arm.com/), [Works on Arm](https://www.worksonarm.com/) and [Packet](https://www.packet.com/), who have generously donated a 96 core monster machine to assist with intensive concurrency testing of sled. Each second that sled does not crash while running your critical stateful workloads, you are encouraged to thank these wonderful organizations. Each time sled does crash and lose your data, blame Intel.

# contribution welcome!

want to help advance the state of the art in open source embedded
databases? check out [CONTRIBUTING.md](CONTRIBUTING.md)!

# References

* [The Bw-Tree: A B-tree for New Hardware Platforms](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/bw-tree-icde2013-final.pdf)
* [LLAMA: A Cache/Storage Subsystem for Modern Hardware](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/llama-vldb2013.pdf)
* [Cicada: Dependably Fast Multi-Core In-Memory Transactions](http://15721.courses.cs.cmu.edu/spring2018/papers/06-mvcc2/lim-sigmod2017.pdf)
* [The Design and Implementation of a Log-Structured File System](https://people.eecs.berkeley.edu/~brewer/cs262/LFS.pdf)

---
date: 2025-10-11
author: guibeira
tags: rust,datastore,google-cloud

extra:
  mermaid: true
  mermaid_theme: default
---

# Building an Alternative Emulator for Google Datastore: My journey of rewriting it in Rust.

### Introduction: The Problem with the Datastore Emulator

Anyone who works with Google Datastore in local environments has probably faced this situation: the emulator starts light, but over time it turns into a [memory‑hungry](https://github.com/googleapis/python-datastore/issues/582#event-15802729926) monster. And worst of all — it loves to corrupt your data files when you least expect it.

In our team, Datastore is a critical part of the stack. Although it’s a powerful NoSQL database, the local emulator simply couldn’t keep up. With large dumps, performance would drop drastically, and the risk of data corruption increased. Each new development day became the same routine: clean up, restore, and hope it wouldn’t break again.

### Attempts at a Solution

At first, we tried reducing the backup size, which worked for a while, but the problem soon reappeared. Another alternative would be to use a real database for each developer, or — as a last resort — build our own emulator. It sounded like a challenging idea at first, but also a fascinating one.

### Reverse Engineering: Understanding the APIs and Protobufs

Once I decided to build an alternative emulator, I started with the most important step: understanding how Datastore communicates.

Fortunately, Google provides the [protobufs](https://github.com/googleapis/googleapis/blob/c98457cd51f80e56daf7de102ed8d4c347ada663/google/datastore/v1/entity.proto) used by the Datastore API. This includes all the messages, services, and methods exposed by the standard gRPC API, such as:
- Lookup  
- RunQuery  
- BeginTransaction  
- Commit  
- Rollback  
- AllocateIds  

With these interfaces in hand, I started implementing my own emulator. The idea was to create a gRPC server that mimics Datastore’s behavior. I began with basic operations like Lookup — all hardcoded — and gradually implemented others, also hardcoded, just to understand the flow. Eventually, I had all the methods stubbed out, each returning static data. That’s when I decided it was time to figure out how to actually store data.

### Key Design Decisions

**In‑Memory First:**  
The priority was performance and simplicity. By keeping everything in RAM, I avoided disk locks and heavy I/O operations. That alone eliminated most of the corruption and leak issues.

**Save on Shutdown:**  
When the emulator is stopped, it automatically persists the data into a `datastore.bin` file. This ensures the local state isn’t lost between sessions. There’s some risk of data loss if the process is killed abruptly, but it’s an acceptable trade‑off since this emulator is meant for local development only.

### Ensuring Compatibility

To ensure my emulator behaved faithfully to the original, I ran side‑by‑side tests: I spun up both the standard emulator and my own, created two clients — one for each — and ran the exact same sequence of operations, comparing results afterward.  
Each test checked a specific feature such as insertion, filtered queries, or transactions. Obviously, it’s impossible to cover 100% of use cases, but I focused on what was essential for my workflow. This helped uncover several bugs and inconsistencies.  

For instance, I noticed that when a query returns more items than the limit, the emulator automatically performs pagination and the client aggregates all pages together.  

As testing progressed, I found that the official emulator had several limitations — some operations were *not supported by design*, such as ["IN", "!=", and "NOT‑IN"](https://cloud.google.com/datastore/docs/tools/datastore-emulator#known_issues). At that point, I decided to also use a [real Datastore instance](https://github.com/guibeira/datastore-emulator/tree/main/tests) for more complex tests, which turned out to be essential for ensuring full compatibility given the emulator’s restrictions.

### Importing and Exporting Dumps

Another key feature was the ability to import Datastore dumps. This is absolutely essential for my local development setup, since I can’t start from scratch every time.  

Luckily, the dump format is quite simple — essentially a file containing multiple entities serialized in protobuf. Even better, someone had already reverse‑engineered the format, which you can check out in [dsbackups](https://github.com/remko/dsbackups). That project helped me a lot in understanding the structure.  

With that knowledge, I implemented the import feature and skipped export support for now, since it’s not something I need at the moment.  

The import runs in the background, and after a few optimizations, it now takes around 5 seconds to import a dump with 150k entities — a huge improvement compared to the 10 minutes of the official emulator.

### Ok, It Works — But How Fast Is It?

Once the emulator was functional, I asked myself: how fast is it compared to the original?  
The main goal was to fix the memory and corruption issues, but if it turned out faster, that’d be a bonus.  

Given that the official emulator is written in Java and mine in Rust, I expected a noticeable difference. To measure it, I wrote a script that performs a series of operations (insert, query, update, delete) on both emulators and records the total execution time.  

The results were impressive — my emulator was consistently faster across every operation. In some cases, like single inserts, it was up to **50× faster**.

```bash
python benchmark/test_benchmark.py --num-clients 30 --num-runs 5

--- Benchmark Summary ---

Operation: Single Insert
  - Rust (30 clients, 5 runs each):
    - Total time: 0.8413 seconds
    - Avg time per client: 0.0280 seconds
  - Java (30 clients, 5 runs each):
    - Total time: 48.1050 seconds
    - Avg time per client: 1.6035 seconds
  - Verdict: Rust was 57.18x faster overall.

Operation: Bulk Insert (50)
  - Rust (30 clients, 5 runs each):
    - Total time: 9.5209 seconds
    - Avg time per client: 0.3174 seconds
  - Java (30 clients, 5 runs each):
    - Total time: 163.7277 seconds
    - Avg time per client: 5.4576 seconds
  - Verdict: Rust was 17.20x faster overall.

Operation: Simple Query
  - Rust (30 clients, 5 runs each):
    - Total time: 2.2610 seconds
    - Avg time per client: 0.0754 seconds
  - Java (30 clients, 5 runs each):
    - Total time: 29.3397 seconds
    - Avg time per client: 0.9780 seconds
  - Verdict: Rust was 12.98x faster overall.
```


### Okay, But What About Memory?

```bash
docker stats

CONTAINER ID   NAME                        CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O        PIDS
b44ea75d665b   datastore_emulator_google   0.22%     939.2MiB / 17.79GiB   5.16%     2.51MB / 2.57MB   1.93MB / 332kB   70
aa0caa062568   datastore_emulator_rust     0.00%     18.35MiB / 17.79GiB   0.10%     2.52MB / 3.39MB   0B / 0B          15
```

After running the benchmark, the official emulator was already using almost 1 GB of RAM, while mine used just 18 MB — a massive difference, especially in development environments where memory can be limited.

Pretty interesting, right?
If you’d like to run the benchmark yourself, [here](https://github.com/guibeira/datastore-emulator/tree/main/benchmark) are the instructions.


### Conclusion and Next Steps

The final result was a binary around 10 MB, much faster and significantly more efficient in both memory and CPU usage.
I’m fully aware there’s still plenty of room for improvement — so if you’re into Rust and spot something, please open a PR!

Given what we had before, I’m really happy with the outcome.

A major next step toward feature parity is implementing HTTP endpoints, which would make it easier for web clients such as [dsadmin](https://github.com/remko/dsadmin) o interact with the emulator. That’s on my roadmap, along with improving test coverage and adding more features as needed.

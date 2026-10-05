Rate Limiter

Thread-Safe Python Rate Limiting Library

Python · Concurrency · Algorithms · Testing · CI

A Python rate-limiting library implementing multiple rate-limiting strategies with thread-safe behavior, automated tests, and continuous integration.

The project focuses on understanding how different rate-limiting algorithms behave and how to build a small, testable Python library around them.

---

🎯 What It Does

Rate limiting controls how frequently a client can perform an operation within a given period.

For example:

Client
  │
  │ Request
  ▼
┌──────────────────┐
│   Rate Limiter   │
└────────┬─────────┘
         │
     ┌───┴────┐
     │        │
   Allowed   Blocked
     │        │
     ▼        ▼
 Application  Retry

This project provides multiple algorithms so their behavior can be compared under different traffic patterns.

---

⚙️ Implemented Algorithms

Token Bucket

Maintains a bucket of tokens that are consumed when requests are allowed.

        ┌──────────────┐
        │ Token Bucket │
        │ ● ● ● ● ●    │
        └──────┬───────┘
               │
            Request
               │
               ▼
        Token available?
          /          \
        Yes           No
         │             │
      Allow          Reject

Useful when controlled bursts of traffic are acceptable.

---

Sliding Window

Tracks requests within a moving time window.

Time ───────────────────────────►

      |──────── Window ────────|
      ●     ●       ●      ●

This provides more precise control around window boundaries than a basic fixed-window approach.

---

Fixed Window

Divides time into fixed intervals and limits the number of requests allowed during each interval.

|──── Window 1 ────|──── Window 2 ────|

 ●   ●   ●   ●        ●   ●

It is simple to implement and useful when predictable window-based limits are sufficient.

---

🔐 Thread Safety

The library is designed to support concurrent access safely.

Shared rate-limiter state is protected so that multiple threads cannot incorrectly update the same counters or token state at the same time.

This makes concurrency behavior an explicit part of the design rather than an afterthought.

---

🏗️ Project Structure

rate-limiter/
│
├── src/
│   └── ratelimiter/
│       ├── ...
│       └── ...
│
├── tests/
│   └── ...
│
├── examples/
│   └── ...
│
├── pyproject.toml
├── README.md
└── .github/
    └── workflows/
        └── ...

The project follows a standard Python package structure with source code separated from tests and examples.

---

🧪 Testing

The project includes automated tests covering the implemented rate-limiting behavior.

The test suite helps verify:

- Request limits
- Window behavior
- Token handling
- Boundary conditions
- Concurrent access
- Different algorithm implementations

The repository currently includes 20 tests.

---

🔄 Continuous Integration

GitHub Actions is used to run the test suite automatically.

The CI workflow checks the project across supported Python versions:

Push / Pull Request
        │
        ▼
 GitHub Actions
        │
        ▼
 Install dependencies
        │
        ▼
     Run tests
        │
        ▼
    Pass / Fail

This keeps testing part of the development workflow instead of relying only on local execution.

---

💻 Example

from ratelimiter import RateLimiter

limiter = RateLimiter(...)

if limiter.allow():
    process_request()
else:
    handle_rate_limit()

See the "examples/" directory for usage patterns supported by the project.

---

🧠 Design Decisions

The project was designed around a few principles:

Simple interfaces

The rate-limiting strategies share a consistent interface so they can be compared without changing application code.

Explicit concurrency handling

Thread safety is treated as part of the library design.

Testable components

The algorithms are separated into components that can be tested independently.

Small scope

The project focuses on the core rate-limiting problem rather than trying to become a complete distributed traffic-management platform.

---

📊 Algorithm Comparison

Algorithm| Burst Support| Main Characteristic
Token Bucket| Yes| Allows controlled bursts
Sliding Window| Limited| More precise moving-window behavior
Fixed Window| Depends on configuration| Simple and predictable

---

⚠️ Current Limitations

This is a Python library project rather than a distributed production rate-limiting service.

Current limitations include:

- No distributed Redis-backed implementation
- No multi-node coordination
- No production load-testing benchmark
- No built-in monitoring/metrics system
- No persistence layer

These limitations are intentional. The project focuses on the algorithms, concurrency behavior, testing, and library design.

---

🚀 Future Improvements

Possible next steps:

- [ ] Redis-backed distributed limiter
- [ ] AsyncIO support
- [ ] Benchmarking under different workloads
- [ ] Prometheus metrics
- [ ] Configurable retry metadata
- [ ] Distributed coordination
- [ ] Additional rate-limiting strategies

---

🛠️ Technology Stack

Python · Threading · Concurrency · Rate Limiting · Pytest · GitHub Actions · Packaging

---

🎯 Project Goal

The goal of this project is to understand and implement common rate-limiting algorithms while building a small Python library with a clear API, automated tests, and CI.

It demonstrates practical experience with Python development, concurrency, testing, algorithms, and software design.

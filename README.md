# Summary: Oleg Romanenko's Talk on HighLoad for Small Systems

## 1. Problem Background
- **Company**: MediaSniper (Internet advertising: banners, ad auctions).
- **Problems**:
  - Load growth from 500K to 500M requests per second after connecting to Google.
  - Unpredictability: Initial system lacked clear requirements, grew iteratively.
  - Hardware limitations: Old servers (DDR3), need to process requests within 2ms per core.

## 2. Requirements for Solution
- **Scalability**: Handle 500K RPS.
- **Reliability**: System should work even if 90% of components fail.
- **Performance**: Minimize latency (70ms for external calls).
- **Flexibility**: Support custom ad rules (filtering, bitrates, geolocation).

## 3. Situation Modeling
- **Business Model**:
  - 3B unique profiles (cookies, browsers),
  - 25TB of data per day, 150-200K active ads.
- **Mathematical Model**:
  - Bit arrays for filtering (10K ads → 100 after filtering),
  - Optimization: 50 rules → 1K bit operations in 45 microseconds.

## 4. Architectural Influence
- **Core**: Monolith for parsing requests and fast data access.
- **Distributed Data**:
  - Rust + Redis (Rspike) for profile storage, limits, bids,
  - ClickHouse for aggregation and updates.
- **Services**: Go microservices for failure isolation (e.g., bid prediction).
- **Load Balancing**: 2 Nginx clusters (long-polling and short sessions).

## 5. Engineering Approaches
- **Optimization**:
  - Bitwise operations instead of rule iteration.
  - Caching: Static data (snapshots), data warm-up.
- **Fault Tolerance**:
  - "If a service doesn’t respond, the system still works."
  - Redundancy: Even a single surviving load balancer can handle traffic.
- **Load Testing**: Benchmarks on real data (e.g., 20K RPS per core).

## 6. Solution Optimality
- **Strengths**:
  - Handling 500K RPS on outdated hardware.
  - Flexibility: Adding new rules without rewriting the core.
- **Weaknesses**:
  - Complexity in supporting custom integrations (400+ SSP/Google).
  - Network dependency (e.g., 130ms during data center migration).

## 7. Conclusion
- **Key takeaways for HighLoad systems**:
  - **Data Isolation**: SpaceBase architecture for resilience,
  - **Simplicity**: Use microservices (Go) over overly complex code,
  - **Resources**: "It’s cheaper to add servers than optimize 1% of code."

### Personal thoughts:
- Example of a "battle-tested" approach: work with a functioning prototype first, then optimize.
  
### Open question:
- How to avoid chaos during such rapid growth? The answer: strict change control (e.g., preventing managers from editing lists manually).

### Conclusion:
- A great case for students on how theory (bit arrays, Redis) is applied in real-world HighLoad systems.
  
**Quote**: "We didn’t read books — we just survived. Then it turned out that smart people do it the same way."

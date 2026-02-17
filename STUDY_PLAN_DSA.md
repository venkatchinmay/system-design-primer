# Data Structures & Algorithms — 90-Day Google Interview Plan

> From zero DSA knowledge to Google coding rounds. **1 hour/day.**  
> Designed for experienced engineers (8+ years) who know how to code but never studied DSA formally.  
> Each day has: concept to learn, the best free resource, and exact problems to solve.

---

## Your Advantage as an 8+ Year Engineer

You already know: debugging, reading code, thinking about edge cases, writing clean code.  
You DON'T know: time complexity, when to use which data structure, algorithm patterns.  
This plan leverages your strengths — you'll learn faster than a fresh grad.

---

## Tools You Need

| Tool | Link | Purpose |
|---|---|---|
| **LeetCode** | [leetcode.com](https://leetcode.com/) | Problem practice (free tier is enough) |
| **NeetCode** | [neetcode.io](https://neetcode.io/) | Curated problem roadmap + video explanations |
| **VisuAlgo** | [visualgo.net](https://visualgo.net/) | Visualize every data structure & algorithm |
| **Big-O Cheat Sheet** | [bigocheatsheet.com](https://www.bigocheatsheet.com/) | Quick reference for complexities |
| **Python/Java** | Your choice | Pick ONE language and stick with it for all problems |

---

## Phase 1: Complexity & Arrays (Days 1–10)

The absolute foundation. If you skip this, nothing else makes sense.

| Day | Topic | Best Resource | Problems to Solve |
|---|---|---|---|
| 1 | Big-O Notation — what it means, why it matters | [NeetCode: Big-O (video)](https://www.youtube.com/watch?v=BgLTDT03QtU) + [bigocheatsheet.com](https://www.bigocheatsheet.com/) | Calculate Big-O for 10 code snippets by hand — [practice sheet](https://www.interviewcake.com/article/python/big-o-notation-time-and-space-complexity) |
| 2 | Arrays — memory layout, access, insert, delete | [VisuAlgo: Array](https://visualgo.net/en/array) | [Two Sum](https://leetcode.com/problems/two-sum/) ✅, [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) ✅ |
| 3 | Two pointers pattern | [NeetCode: Two Pointers](https://www.youtube.com/watch?v=cQ1Oz4ckceM) | [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) ✅, [Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) ✅, [3Sum](https://leetcode.com/problems/3sum/) 🟡 |
| 4 | Sliding window pattern | [NeetCode: Sliding Window](https://www.youtube.com/watch?v=1pkOgXD63yU) | [Max Subarray](https://leetcode.com/problems/maximum-subarray/) 🟡, [Longest Substring Without Repeating](https://leetcode.com/problems/longest-substring-without-repeating-characters/) 🟡 |
| 5 | Sliding window (continued) | — | [Min Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) 🟡, [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) 🟡 |
| 6 | Prefix sum pattern | [NeetCode: Prefix Sums](https://www.youtube.com/watch?v=yuws07atMM0) | [Range Sum Query](https://leetcode.com/problems/range-sum-query-immutable/) ✅, [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) 🟡, [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) 🟡 |
| 7 | Sorting — merge sort, quicksort, counting sort | [VisuAlgo: Sorting](https://visualgo.net/en/sorting) + [Abdul Bari: Merge Sort (video)](https://www.youtube.com/watch?v=mB5HXBb_HY8) | Implement merge sort from scratch. [Merge Intervals](https://leetcode.com/problems/merge-intervals/) 🟡 |
| 8 | Binary Search — the most important algorithm | [NeetCode: Binary Search](https://www.youtube.com/watch?v=s4DPM8ct1pI) + [Binary Search 101 (LeetCode)](https://leetcode.com/discuss/study-guide/786126/Python-Powerful-Ultimate-Binary-Search-Template) | [Binary Search](https://leetcode.com/problems/binary-search/) ✅, [Search Insert Position](https://leetcode.com/problems/search-insert-position/) ✅, [Search Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) 🟡 |
| 9 | Binary Search (continued) | — | [Find Min in Rotated Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) 🟡, [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) 🟡 |
| 10 | **Review Day** — re-solve any problems you struggled with | [NeetCode Roadmap: Arrays & Hashing](https://neetcode.io/roadmap) | Re-do 3-5 problems from days 2-9 without looking at solutions |

> ✅ = Easy, 🟡 = Medium, 🔴 = Hard

---

## Phase 2: Core Data Structures (Days 11–25)

| Day | Topic | Best Resource | Problems to Solve |
|---|---|---|---|
| 11 | Hash Maps — how they work internally | [VisuAlgo: Hash Table](https://visualgo.net/en/hashtable) + [NeetCode: Hash Maps](https://www.youtube.com/watch?v=KyUTuwz_b7Q) | [Group Anagrams](https://leetcode.com/problems/group-anagrams/) 🟡, [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) 🟡 |
| 12 | Hash Sets & frequency counting | — | [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) ✅, [Valid Anagram](https://leetcode.com/problems/valid-anagram/) ✅, [Encode and Decode Strings](https://leetcode.com/problems/encode-and-decode-strings/) 🟡 |
| 13 | Strings — common patterns, manipulation | [NeetCode: Strings](https://neetcode.io/roadmap) | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) ✅, [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) 🟡 |
| 14 | Linked Lists — singly, doubly, operations | [VisuAlgo: Linked List](https://visualgo.net/en/list) + [NeetCode: Linked Lists](https://www.youtube.com/watch?v=G0_I-ZF0S38) | [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) ✅, [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) ✅ |
| 15 | Linked Lists — fast/slow pointers | — | [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) ✅, [Reorder List](https://leetcode.com/problems/reorder-list/) 🟡, [Remove Nth Node From End](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) 🟡 |
| 16 | Stacks — LIFO, call stack, monotonic stack | [VisuAlgo: Stack](https://visualgo.net/en/list) + [NeetCode: Stacks](https://www.youtube.com/watch?v=WTzjTskDFMg) | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) ✅, [Min Stack](https://leetcode.com/problems/min-stack/) 🟡, [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) 🟡 |
| 17 | Queues & Deques — FIFO, BFS foundation | [VisuAlgo: Queue](https://visualgo.net/en/list) | [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) ✅, [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) 🔴 |
| 18 | Heaps / Priority Queues — min heap, max heap | [VisuAlgo: Heap](https://visualgo.net/en/heap) + [NeetCode: Heaps](https://www.youtube.com/watch?v=t0Cq6tVNRBA) | [Kth Largest Element](https://leetcode.com/problems/kth-largest-element-in-a-stream/) ✅, [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) ✅ |
| 19 | Heaps (continued) | — | [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) 🟡, [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) 🔴 |
| 20 | **Review Day** — data structures week | — | Pick your 3 weakest topics, re-solve 2 problems each without hints |
| 21 | Trees — binary trees, traversals (in/pre/post/level) | [VisuAlgo: BST](https://visualgo.net/en/bst) + [NeetCode: Trees](https://www.youtube.com/watch?v=OnSn2XEQ4MY) | [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) ✅, [Max Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) ✅, [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) ✅ |
| 22 | Trees — DFS patterns (recursion) | — | [Same Tree](https://leetcode.com/problems/same-tree/) ✅, [Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) ✅, [Lowest Common Ancestor](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) 🟡 |
| 23 | Trees — BFS (level order), BST properties | — | [Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) 🟡, [Validate BST](https://leetcode.com/problems/validate-binary-search-tree/) 🟡, [Kth Smallest in BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) 🟡 |
| 24 | Trees — advanced problems | — | [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) 🟡, [Construct Binary Tree from Preorder and Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) 🟡, [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) 🔴 |
| 25 | Tries — prefix trees | [NeetCode: Tries](https://www.youtube.com/watch?v=oobqoCJlHA0) | [Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/) 🟡, [Design Add and Search Words](https://leetcode.com/problems/design-add-and-search-words-data-structure/) 🟡, [Word Search II](https://leetcode.com/problems/word-search-ii/) 🔴 |

---

## Phase 3: Graphs & Recursion (Days 26–40)

Google's favorite topic. You WILL get a graph problem.

| Day | Topic | Best Resource | Problems to Solve |
|---|---|---|---|
| 26 | Graphs — representations (adjacency list/matrix) | [VisuAlgo: Graph](https://visualgo.net/en/graphds) + [NeetCode: Graphs](https://www.youtube.com/watch?v=EgI5nU9etnU) | Build a graph from an edge list. [Number of Islands](https://leetcode.com/problems/number-of-islands/) 🟡 |
| 27 | Graph BFS — shortest path in unweighted graph | — | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) 🟡, [01 Matrix](https://leetcode.com/problems/01-matrix/) 🟡 |
| 28 | Graph DFS — traversal, connected components | — | [Clone Graph](https://leetcode.com/problems/clone-graph/) 🟡, [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) 🟡, [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) 🟡 |
| 29 | Graph — cycle detection, topological sort | [NeetCode: Topological Sort](https://www.youtube.com/watch?v=EtLc5X3JXjU) | [Course Schedule](https://leetcode.com/problems/course-schedule/) 🟡, [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) 🟡 |
| 30 | Graph — Union Find (Disjoint Set) | [NeetCode: Union Find](https://www.youtube.com/watch?v=ibjEGG7ylHk) | [Number of Connected Components](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) 🟡, [Redundant Connection](https://leetcode.com/problems/redundant-connection/) 🟡 |
| 31 | Graph — Dijkstra's algorithm (weighted shortest path) | [VisuAlgo: SSSP](https://visualgo.net/en/sssp) + [WilliamFiset: Dijkstra (video)](https://www.youtube.com/watch?v=pSqmAO-m7Lk) | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) 🟡, [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) 🟡 |
| 32 | Graph — advanced (MST, multi-source BFS) | — | [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) 🟡, [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) 🔴 |
| 33 | **Review Day** — graphs | — | Re-solve: Number of Islands, Course Schedule, Network Delay Time — timed, 20 min each |
| 34 | Recursion — thinking recursively, base case, recursive case | [Reducible: Recursion (video)](https://www.youtube.com/watch?v=IJDJ0kBx2LM) | [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) ✅, [Fibonacci Number](https://leetcode.com/problems/fibonacci-number/) ✅ |
| 35 | Backtracking — generate all possibilities, prune | [NeetCode: Backtracking](https://www.youtube.com/watch?v=REOH22Xwdkk) | [Subsets](https://leetcode.com/problems/subsets/) 🟡, [Combination Sum](https://leetcode.com/problems/combination-sum/) 🟡 |
| 36 | Backtracking (continued) | — | [Permutations](https://leetcode.com/problems/permutations/) 🟡, [Word Search](https://leetcode.com/problems/word-search/) 🟡, [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) 🟡 |
| 37 | Backtracking — constraint satisfaction | — | [N-Queens](https://leetcode.com/problems/n-queens/) 🔴, [Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) 🔴 |
| 38 | Intervals — merge, insert, overlaps | [NeetCode: Intervals](https://www.youtube.com/watch?v=44H3cEC2fFM) | [Merge Intervals](https://leetcode.com/problems/merge-intervals/) 🟡, [Insert Interval](https://leetcode.com/problems/insert-interval/) 🟡, [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) 🟡 |
| 39 | Math & Bit manipulation — essentials for Google | [NeetCode: Bit Manipulation](https://www.youtube.com/watch?v=emiXGGnDrjk) | [Single Number](https://leetcode.com/problems/single-number/) ✅, [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) ✅, [Counting Bits](https://leetcode.com/problems/counting-bits/) ✅, [Reverse Bits](https://leetcode.com/problems/reverse-bits/) ✅ |
| 40 | **Review Day** — recursion & backtracking | — | Re-do Subsets, Permutations, N-Queens timed. Draw the recursion tree on paper for each |

---

## Phase 4: Dynamic Programming (Days 41–55)

The hardest topic. Google asks DP at medium-to-hard level.

| Day | Topic | Best Resource | Problems to Solve |
|---|---|---|---|
| 41 | DP intro — memoization vs tabulation, overlapping subproblems | [NeetCode: DP (video)](https://www.youtube.com/watch?v=oBt53YbR9Kk&t=1s) + [Reducible: Dynamic Programming (video)](https://www.youtube.com/watch?v=aPQY__2H3tE) | [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) ✅ (solve 3 ways: recursion → memo → tabulation) |
| 42 | 1D DP — basic patterns | — | [House Robber](https://leetcode.com/problems/house-robber/) 🟡, [House Robber II](https://leetcode.com/problems/house-robber-ii/) 🟡 |
| 43 | 1D DP — min cost, max profit | — | [Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/) ✅, [Decode Ways](https://leetcode.com/problems/decode-ways/) 🟡 |
| 44 | 1D DP — subsequences | — | [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) 🟡, [Word Break](https://leetcode.com/problems/word-break/) 🟡 |
| 45 | 1D DP — partition & decision | — | [Coin Change](https://leetcode.com/problems/coin-change/) 🟡, [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) 🟡, [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) 🟡 |
| 46 | 2D DP — grids | — | [Unique Paths](https://leetcode.com/problems/unique-paths/) 🟡, [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) 🟡, [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/) 🟡 |
| 47 | 2D DP — two strings (LCS family) | [NeetCode: LCS (video)](https://www.youtube.com/watch?v=Ua0GhsJSlWM) | [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) 🟡, [Edit Distance](https://leetcode.com/problems/edit-distance/) 🟡 |
| 48 | 2D DP — knapsack pattern | [Abdul Bari: 0/1 Knapsack (video)](https://www.youtube.com/watch?v=8LusJS5-AGo) | [Target Sum](https://leetcode.com/problems/target-sum/) 🟡, [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/) 🟡 |
| 49 | 2D DP — string matching | — | [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) 🔴, [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/) 🔴 |
| 50 | **Review Day** — DP | — | Re-solve: Coin Change, LIS, LCS, Edit Distance — draw the DP table by hand for each |
| 51 | DP on trees | — | [House Robber III](https://leetcode.com/problems/house-robber-iii/) 🟡, [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) 🔴 |
| 52 | DP — intervals, stock problems | [NeetCode: Stock Problems](https://www.youtube.com/watch?v=1pkOgXD63yU) | [Best Time to Buy and Sell Stock with Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/) 🟡, [Burst Balloons](https://leetcode.com/problems/burst-balloons/) 🔴 |
| 53 | Greedy algorithms — when DP is overkill | [NeetCode: Greedy](https://www.youtube.com/watch?v=bC7o8P_Ste4) | [Jump Game](https://leetcode.com/problems/jump-game/) 🟡, [Jump Game II](https://leetcode.com/problems/jump-game-ii/) 🟡, [Gas Station](https://leetcode.com/problems/gas-station/) 🟡 |
| 54 | Greedy (continued) | — | [Hand of Straights](https://leetcode.com/problems/hand-of-straights/) 🟡, [Partition Labels](https://leetcode.com/problems/partition-labels/) 🟡, [Task Scheduler](https://leetcode.com/problems/task-scheduler/) 🟡 |
| 55 | **Review Day** — DP + greedy | — | Time yourself: 3 medium DP problems, 25 min each. Can you identify the pattern in 5 min? |

---

## Phase 5: Advanced Patterns — Google Level (Days 56–70)

These patterns appear frequently in Google interviews.

| Day | Topic | Best Resource | Problems to Solve |
|---|---|---|---|
| 56 | Monotonic stack — next greater/smaller element | [NeetCode: Monotonic Stacks](https://www.youtube.com/watch?v=Dq_ObZwTY_Q) | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) 🟡, [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) 🔴 |
| 57 | Advanced binary search — on answer space | — | [Capacity to Ship Packages](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) 🟡, [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) 🔴, [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) 🔴 |
| 58 | Advanced graphs — Bellman-Ford, Floyd-Warshall | [WilliamFiset: Graph Theory (playlist)](https://www.youtube.com/playlist?list=PLDV1Zeh2NRsDGO4--qE8yH72HFL1Km93i) | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) 🟡 (Bellman-Ford approach) |
| 59 | Segment trees / BIT (Fenwick tree) | [VisuAlgo: Segment Tree](https://visualgo.net/en/segmenttree) | [Range Sum Query Mutable](https://leetcode.com/problems/range-sum-query-mutable/) 🟡, [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) 🔴 |
| 60 | Design data structures | [NeetCode: Design](https://neetcode.io/roadmap) | [LRU Cache](https://leetcode.com/problems/lru-cache/) 🟡, [LFU Cache](https://leetcode.com/problems/lfu-cache/) 🔴 |
| 61 | Design data structures (continued) | — | [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/) 🟡, [Design Twitter](https://leetcode.com/problems/design-twitter/) 🟡 |
| 62 | String matching — KMP, Rabin-Karp | [Abdul Bari: KMP (video)](https://www.youtube.com/watch?v=V5-7GzOfADQ) | [Find the Index of the First Occurrence](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) 🟡, [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/) 🟡 |
| 63 | Multi-dimensional BFS/DFS | — | [Word Ladder](https://leetcode.com/problems/word-ladder/) 🔴, [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) 🔴 |
| 64 | Simulation & matrix problems | — | [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/) 🟡, [Rotate Image](https://leetcode.com/problems/rotate-image/) 🟡, [Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/) 🟡 |
| 65 | **Review Day** — advanced patterns | — | Pick your 5 weakest problems, re-solve them timed |
| 66 | Google-style: multi-step problems | — | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) 🔴, [Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/) 🔴 |
| 67 | Google-style: graph + optimization | — | [Word Ladder II](https://leetcode.com/problems/word-ladder-ii/) 🔴, [Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) 🔴 |
| 68 | Google-style: DP + data structures | — | [Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) 🔴, [Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) 🔴 |
| 69 | Google-style: design + algorithms | — | [Design Search Autocomplete](https://leetcode.com/problems/design-search-autocomplete-system/) 🔴, [All O'one Data Structure](https://leetcode.com/problems/all-oone-data-structure/) 🔴 |
| 70 | **Review Day** — hard problems | — | Re-attempt any 🔴 you couldn't solve. Study the patterns, not memorize solutions |

---

## Phase 6: Interview Simulation (Days 71–90)

Practice like it's the real thing. Google gives 4-5 coding rounds, 45 min each.

| Day | Activity | Resource | Details |
|---|---|---|---|
| 71 | Mock interview #1 — Arrays & Strings | [Pramp (free)](https://www.pramp.com/) or [Interviewing.io](https://interviewing.io/) | 45 min, 1-2 problems, explain your thinking out loud |
| 72 | Analyze mistakes from mock #1 | — | Write down: what patterns did I miss? Where did I waste time? |
| 73 | Timed practice — 2 mediums in 40 min | [LeetCode random medium](https://leetcode.com/problemset/) | Strict timer. If stuck >10 min, look at hint (not solution) |
| 74 | Mock interview #2 — Trees & Graphs | [Pramp](https://www.pramp.com/) | Focus on verbalizing your thought process |
| 75 | Timed practice — 1 hard in 35 min | — | Pick from: DP, graphs, or design. Practice recognizing the pattern fast |
| 76 | Mock interview #3 — DP | [Pramp](https://www.pramp.com/) | State the recurrence relation before coding |
| 77 | Review all hard problems attempted | — | Group by pattern: which pattern trips you up most? |
| 78 | Timed practice — 3 easys in 20 min | — | Speed round. Google expects clean easy solutions in 5-8 min |
| 79 | Mock interview #4 — mixed topics | [Pramp](https://www.pramp.com/) | Full simulation: no hints, no IDE autocomplete |
| 80 | **Weak area deep dive** | — | Spend full hour on your #1 weakest pattern — 5 problems |
| 81–85 | **NeetCode 150 gap fill** | [NeetCode 150](https://neetcode.io/practice) | Solve any NeetCode 150 problems you haven't done yet. Aim for ≥120/150 |
| 86 | Mock interview #5 | [Pramp](https://www.pramp.com/) | Ask your partner for feedback. Record yourself if possible |
| 87 | Timed practice — 2 hards in 60 min | — | Google hard-round simulation |
| 88 | Review all notes — pattern cheat sheet | — | Write a 1-page cheat sheet of all patterns with when to use each |
| 89 | Final mock interview | [Pramp](https://www.pramp.com/) or a friend | Full 45-min round, explain like you're in the real interview |
| 90 | **Rest + review cheat sheet** | — | Light review only. You're ready. |

---

## Pattern Recognition Cheat Sheet

The secret to Google interviews: **recognize the pattern in 2-3 minutes**.

| If the problem says... | Think... | Key Data Structure |
|---|---|---|
| "Sorted array" or "find in O(log n)" | **Binary Search** | Array |
| "All substrings/subarrays of size k" | **Sliding Window** | Array + hash map |
| "Two values that sum to X" | **Two Pointers** or hash map | Sorted array / hash map |
| "Contiguous subarray sum" | **Prefix Sum** | Array + hash map |
| "Top K", "Kth largest/smallest" | **Heap** | Min/max heap |
| "Connected components", "islands" | **BFS/DFS on graph/grid** | Queue / recursion |
| "Shortest path" (unweighted) | **BFS** | Queue |
| "Shortest path" (weighted) | **Dijkstra** | Min heap |
| "Dependencies", "ordering" | **Topological Sort** | Graph + in-degree |
| "Generate all combinations/permutations" | **Backtracking** | Recursion |
| "Min/max ways to reach target" | **Dynamic Programming** | DP table |
| "Matching parentheses", "next greater" | **Stack** (monotonic) | Stack |
| "Prefix matching", "autocomplete" | **Trie** | Trie |
| "Merge/insert intervals" | **Sort + sweep** | Array |
| "Design a data structure" | **Hash map + linked list** | Multiple |

---

## Google Interview Specifics

| Round | What They Test | Tip |
|---|---|---|
| **Phone Screen** | 1 medium (45 min) | Write clean code, test with examples, state complexity |
| **Onsite 1-3** | 1-2 problems per round (45 min) | Think out loud, discuss trade-offs before coding |
| **Googleyness** | Communication, collaboration | Explain WHY, not just what. Ask clarifying questions |

**Google looks for:**
1. ✅ Can you break down the problem?
2. ✅ Can you identify the right data structure?
3. ✅ Is your code clean and bug-free?
4. ✅ Can you analyze time & space complexity?
5. ✅ Can you optimize if asked?
6. ✅ Do you test your code with edge cases?

---

## Daily Routine (1 Hour)

```
[00:00 - 00:05]  Review yesterday's problem patterns (5 min)
[00:05 - 00:25]  Learn the day's concept — video/article (20 min)
[00:25 - 00:55]  Solve the day's problems (30 min)
[00:55 - 01:00]  Write 1-sentence note per problem: what pattern, what I learned (5 min)
```

> **The 20-minute rule**: If stuck on a problem for 20 min, look at the approach (not full solution). Understanding the pattern > brute-force struggling.  
> **The re-solve rule**: Any problem you needed hints for → re-solve it 3 days later from scratch.

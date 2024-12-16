**Why Excel?**

Excel is an amazing prototyping tool. It’s transparent—you can see intermediate computations, step into formulas, and trace how values are calculated. That transparency is hard to replicate in other languages. People have tried using caching, memoization decorators, and similar tricks, but they’re often ad hoc. My aim here is to explore a more formalized way of doing this in Python.

**Design Principles**

***Lazy Computation Framework***

We want to express large computations without executing everything upfront. Computations should be **lazy**, executing only when needed.

***Memoization Framework***

Expensive calculations, like calibrating a volatility surface, should be reusable without worrying about their validity. We’ll use **parse-time dependency analysis** to identify dependencies in code, rewrite it using a syntax tree transformer, and execute it lazily in Python.

**Key Features**

**1\. Side-Effect-Free Functions**

Functions should behave like Excel formulas:

- Calling them twice with the same inputs always gives the same result.
- Values must be immutable (a challenge in Python, but conventions and debug tools help).

**2\. Notification Framework**

Changes in inputs should trigger updates. Think Excel: when you modify a cell, dependent cells update automatically. We aim to support **data binding** and **efficient dependency tracking** for similar behavior.

**3\. Input Discovery**

It’s useful to discover what inputs matter in a computation. For example, is a particular value part of a currency pair or yield curve? This allows for tweaks and recalculations as needed.

**4\. Temporary Bindings**

We need a way to **temporarily bind values**, change them, and see their effect without triggering immediate computation.

**5\. Parallelism and Asynchronous Execution**

By externalizing computations and bindings, we enable distributed and parallel execution, making the framework scalable.

**Why Do We Need a Framework?**

People often ask: “Why not just use caching or memoization decorators?” Here’s the problem:

1. **Opacity**: Dependencies aren’t tracked. You don’t know what inputs contributed to a result or when something is dirty.
2. **Debugging**: Unlike Excel, you can’t step through or inspect intermediate computations.
3. **Reevaluation**: Without dependency tracking, there’s no way to rebind inputs and reevaluate the affected path.

Our framework solves these issues by introducing **side-effect-free functions** (SEFs) and dependency tracking.

**The SEF Decorator**

**What is a SEF?**

A **side-effect-free function** (SEF) is just a Python function decorated to:

- Analyze its syntax tree.
- Identify dependencies and inputs.
- Rewrite its code for lazy evaluation.

**Binding to Classes**

SEFs can also be bound to classes, allowing object-oriented abstractions like inheritance and delegation.

**Namespace Integration**

Think of it like Excel’s named ranges but on steroids. Objects can bind to namespaces (e.g., databases, file systems) and be referenced by name.

**Practical Example: Fibonacci**

Here’s a simple example:

python

<pre>
@sef
def fibonacci(n):
  if n < 2:
    return 1
  return fibonacci(n - 1) + fibonacci(n - 2)
</pre>
Without caching, this function is exponential (O(2^n)). With our framework, it becomes linear because each call is cached. The decorator transforms the function to:

- Use precomputed inputs.
- Avoid redundant evaluations.

**Graph Concepts**

**Nodes**

- **Terminal Nodes**: Constants or arguments with no inputs.
- **Non-Terminal Nodes**: Computed values, like formulas in Excel.

**Edges**

Edges connect nodes and define the dependencies between them.

**Computation Order**

Lazy evaluation flips the usual computation order:

- Inputs are pre-evaluated bottom-up.
- Functions are only called once their inputs are ready.

This approach has implications for debugging and exception handling, but the trade-off is efficient, traceable computation.

**Handling Exceptions**

In a SEF, exceptions propagate back through the dependency graph but can’t be caught within the function itself. To address this:

- Functions can declare intentions to handle errors.
- Errors trigger reevaluation of dependent paths.

**Advanced Features**

**Intermediate References**

We allow “edge inputs,” where the dependency itself is computed dynamically—something Excel doesn’t natively support.

**Namespace Binding**

SEFs bind objects to namespaces, enabling shared access to values (e.g., USD yield curves) across computations.

**Abstractions and Composition**

Inspired by functional programming, SEFs support:

- **Mappings**: Apply functions over lists.
- **Conditionals**: Dynamically prune parts of the dependency graph.

**Under the Hood**

**Code Transformation**

The SEF decorator rewrites functions to:

- Identify dependencies.
- Replace direct references with cached values or input arrays.

**Implications for Member Variables**

Since member variables don’t work with this framework, they’re replaced by no-argument functions. These can compute default values dynamically.

**Overview of Demo and Purpose**

The demo illustrates a "dumb Pricer" in Python, showcasing an object-oriented approach to financial computations. This method provides enhanced functionality and flexibility compared to traditional tools like Excel. The focus is on simplifying complex financial relationships through dynamic computation and abstraction.

**Key Features and Mechanisms**

**Dynamic Attribute Calculation**  
Attributes such as expiration dates and strike prices are computed dynamically based on inputs like currency pairs or forward curves.

**Spot Price Calculation**  
Spot prices are adjusted based on factors like volatility, strike, and discount rates, encapsulating financial logic into reusable functions.

**Imperative Scenarios**  
By tweaking variables (e.g., modifying spot prices), you can observe their effects on dependent computations. This approach efficiently updates only the impacted parts of the dependency graph.

**Graph-Based Dependency Management**  
Dependencies between computations are tracked, allowing for:

- Generic scenarios like partial delta calculations.
- Tweaking inputs such as FX spot prices or credit spreads for sensitivity analysis.

**Namespace-Based Pricing**  
Namespaces simplify tasks like repricing portfolios as of previous dates by rebinding data within a specific context.

**Functional Programming Features**

**Side-Effect-Free Bindings**  
Binding sets are introduced to maintain immutability, enabling functional-style computations. This makes it possible to compute deltas or scenarios without altering the original state.

**Parallelism and Distributed Computation**  
Side-effect-free computations are naturally parallelizable. By tagging computationally expensive functions, tasks can be distributed across grids or multi-core systems, leveraging Python libraries like NumPy.

**Advanced Features**

**Futures and Delayed Computations**  
Futures allow the graph engine to delay computation until necessary, enabling other parts of the graph to execute concurrently. This increases efficiency and reduces latency.

**Efficient Caching**  
The graph engine caches function invocations to avoid redundant calculations. Strategies like weak caching help balance memory usage and computational efficiency.

**Garbage Collection**  
Custom garbage collection ensures that cached nodes tied to deleted objects are removed, resolving memory retention issues caused by strong references.

**Advantages Over Excel**

Excel has limitations in handling scenarios, bindings, and large-scale simulations. In contrast, this Python-based model:

- Resolves complex dependencies dynamically.
- Enables reusable and generic scenario definitions.
- Supports advanced backtesting and sensitivity analysis.

**Use Cases**

**Delta Calculations**  
Define scenarios that bump FX or credit spreads for sensitivity analysis.

**Simulations**  
Shift pricing environments for tasks like backtesting or exploring historical scenarios.

**Grid Computing**  
Offload heavy computations to distributed systems or multi-core environments for better scalability.

**Challenges and Considerations**

**Memory Overhead**  
Caching all function invocations can consume significant memory, necessitating efficient node management.

**Performance Trade-offs**  
Not all computations benefit from grid distribution, especially smaller or less expensive ones.

**Implementation Complexity**  
Balancing caching, garbage collection, and parallel execution requires a carefully designed architecture.

**Concluding Remarks**

This framework brings the best of Excel’s transparency and flexibility into Python, while addressing its limitations. By formalizing dependency tracking and computation, we enable:

- Debuggability.
- Scalability.
- Flexibility in expressing complex computations.

The demo integrates functional programming and object-oriented principles to deliver scalable, efficient, and reusable financial computations. By addressing the limitations of traditional tools like Excel, it provides a robust framework for modern financial modeling.

**Open Questions and Next Steps**

Future exploration includes:

- Integrating this model with existing tools (e.g., Excel as a UI).
- Optimizing performance for large-scale financial systems.
- Exploring commercial applications through Washington Square Technologies.

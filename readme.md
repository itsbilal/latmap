# Winter 2017 URA notes

Here is a summary of the changes that we (Jacob Jackson and Jim Zhang) made as part of our URA, which took place over the winter 2017 school term:

## Index

Interface for classes that index a subset of the key columns for a LatMap. Indexes are used to efficiently iterate over the entries in a LatMap given a known subset of key column values.

### NaiveIndex

Extremely simple implementation of Index that iterates over all entries in the LatMap and filters out those that don't match.

### HashMapIndex

This is an implementation of the 'index' interface. It should behave in the same way as NaiveIndex, so if you want to understand what it's supposed to do, I recommend that you read NaiveIndex first. Although it is faster than NaiveIndex, it does not support concurrent updates. (It does support concurrent reads.)

### ConcurrentHashMapIndex

This is modelled after HashMapIndex, but it supports concurrent updates. The structuring of these updates into phases is why the LatMap interface is somewhat complicated. This class is not yet fully concurrent; the writes themselves are OK, but the resize needs work.

### IndexTest

Does some basic performance comparisons of the various indices. This is good as a rough indicator, but at some point it would be good to add performance tests that are more reflective of actual usage patterns.

## LatMap

Interface for a map from tuples of keys to lattice elements. Can contain any number of Indexes, which speed up iteration given a subset of known keys.

### SimpleLatMap
Does all operations sequentially.

### ConcurrentLatMap
Uses concurrency to speed things up. Semi-naive evaluation doesn't entirely work with the ConcurrentLatMap. 

## Plan

A stage in the execution of a Flix program, where we infer all possible new facts from a LatMap of existing facts given a rule.

### PlanElement

PlanElements form a linked list representing the steps in the execution of a Plan. Each PlanElement mutates the evaluation context in some way and calls `go()` on the next PlanElement any number of times.

## Planner

Generates a Plan from a Rule and the index of an initial body rule.

## Rule

Consists of a head element and a set of body elements. If all body elements are satisfied for a given set of variables, then the head element should become satisfied.

### RuleElement

The elements of a rule. Currently we have three:

- LatmapRuleElement: Rule element that involves a single lattice element.
- RelationRuleElement: Rule element that involves no lattice elements. This is still untested and should probably be completed discarded and rewritten.
- TransferFnRuleElement: Rule element that transfers a lattice element.

Example: The following rule from the Floyd Warshall example:

Dist(x, z, sum(d1, d2)) :- Dist(x, y, d1), Dist(y, z, d2).

is a Rule:

- whose head is a LatmapRuleElement on the Dist latmap and variable sequence x, z, d1plusd2
- whose body elements are:
  - a LatmapRuleElement on the Dist latmap and variable sequence x, y, d1
  - a LatmapRuleElement on the Dist latmap and variable sequence y, z, d2
  - a TransferFnRuleElement:
    - whose function returns the sum of two elements of the Dist lattice
    - with input variables d1, d2 and output variable d1plusd2

## Solver

Runs the semi-naive algorithm based on the "Flix: Correct Fixed Point Programs on Lattices". Works correctly with the SimpleLatMap.

## TODOs

- Instead of passing `bodyIdx` into the Planner, we could have it figure out what the best body rule to start with would be.
- For efficiency, PlanElements currently use indices < 1000 to denote key registers and >= 1000 to denote lattice element registers. This may be doable in a better way.
- Use Latmap as a backend for Flix. Modify Flix to depend on Latmap and use this instead of the Flix solver.

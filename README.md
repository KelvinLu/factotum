```
work in progress draft
```

# factotum
### A configuration management and provisioning tool.

factotum \fak-ˈtō-təm\ _n._

1. a person having many diverse activities or responsibilities
2. a general servant

---

## Declarative
### A declarative list of properties and principles that should guide the development of the tool.

- There exists a coordinator node and multiple worker nodes.
  - The coordinator node uses a worker node's public key to verify and bind it to an identity (an `Identity`).
  - All nodes have the tool installed as a Gem.
  - The coordinator node holds configuration and provisioning resources.
  - A worker node requires configuration.
  - The coordinator node is responsible for forming the configuration and provisioning resources for a given worker node, based on its `Identity`.
- The configuration and provisioning resources are stored in a directory tree.
  - Held by the coordinator node.
  - Conventionally, the directory tree would be source-controlled with Git.  Ideally, it...
    - allows collaboration between administrators.
    - facilitates a Git-based workflow for making changes and recording history.
- A separate repository should contain a skeleton directory structure that can be copied. It should also contain relevant `README.md`s as documentation at each relevant path.
    
---

## The Framework
### A high-level overview of the main entities at play.

- An `Identity` represents a single worker node.
  - An `Identity` belongs to a single "prime `Cohort`".
  - It may hold configuration (colloquially called `facts`) for the specific machine.
- A `Cohort` represents a group of worker nodes.
  - A `Cohort` may be `derived_from` other `Cohort`s (that are not "prime `Cohort`s").
    - It is given access to the `facts` of the `Cohort`s `derived_from` it.
    - The order of `derived_from` determines the order of `sibling_coalesce`nce.
  - It may hold configuration (colloquially called `facts`) for a group of machines.
  - It may reference entities (colloquially called `Actor`s) that perform system operations based on some given `facts`.
  - It may be designated as a "prime `Cohort`". Only "prime `Cohort`s" are allowed to have `Identity`s belong to them, but they may not have any other `Cohort` be dervied from them (analogous to how prime numbers are defined/"identified").
- An `Actor` represents the implementation of some provisioning process that requires some configuration.
  - An `Actor` can be referenced by multiple `Cohort`s.
  - It performs system operations based on some given configuration/`facts`.
  - It may request and be given only certain pieces of configuration/`facts`.
  - It may specify how two pieces of similar or conflicting configurations are understood together and `coalesce`d through arbitrary operations. It **must** do this for every requested piece of configuration.
    - `sibling_coalesce` can define how sibling `Cohort`s of a derived `Cohort` have a specific configurations coalesced.
    - `derived_coalesce` can define how the results of `sibling_coalesce` are taken and coalesced with the derived `Cohort`'s configurations.
    - `coalesce` can provide shared coalensing semantics as a fallback when the above are not defined.
    - `sibling_coalesce` and `derived_coalesce` must be defined _or_ `coalesce` must be defined.
- The one-way "derivation" relationships between `Cohorts` forms a DAG.
  - We assert that the process of derivation is acyclic, since we will provide interfaces (informing coalescing rules) that assume this property.
  - Given a "prime `Cohort`", we can form a subgraph of the DAG. This subgraph consists of all the information necessesary to form a set of configurations (that is ultimately derived as a function) and implementations to provision each associated `Identity`.

---

## The Process
### How is the meta-graph of `Cohort`s, `Identity`s, `Actor`s, and other entities used?

```
TODO(kelvinlu): write this section

Ideas to consider:
- Precedence metrics for coalescing values at different nodes of the DAG
  - Consider graph edges with weights and use an optimal-path algorithm to label shortest-and-longest-path distances w.r.t the root/"prime `Cohort`" node.
  - Consider labelling the smallest-depth of a node w.r.t. the root with a BFS traversal (or Floyd–Warshall when needing many roots at a time?). This can be reduced to a optimal-path problem with all edge weights of one.
  - Consider labelling the largest-depth of a node w.r.t. the root with a DFS traversal (or Floyd–Warshall when needing many roots at a time?). This can be reduced to a optimal-path problem with all edge weights of one.
  - Consider using each BFS/DFS to also label the smallest-height/longest-height from the leaves of the DAG.
  - Prefer using "shortest-depth"/"longest-depth" over breadth and depth, since the DAG is not a tree.
  - ... just use a simple topological sort and cop-out... ?
```

---

## Purposeful Goals
### The ideals that this project should provide as unique values.

> I swear this isn't just a discount Chef.

- Provides a DAG of information that can be deterministically reduced to a set of configuration for a group of nodes.
  - An inheritance system requires top-down design that may demand inhuman omnipotence.
  - A DAG allows the implementation of a attribution/properties system.
- Provides an interface to choose or define an operation that merges similar or conflicting pieces of information.
  - Merging of configuration values should have arbitrary semantics.
  - A developer should define how certain sets of configuration are understood together.
  - Values that add up (such as quantities) should be able to add! Values that concatenate (like lists of favorite foods) should be able to concatenate! Values that are replaced with one another should have a selection mechanism! Values that coalesce by an arbitrary process should do just that!
  - Values that are coalesced based on the  graph traversal should have  
  - The provisioning implementation should decide these semantics, based on its configuration needs.
- Provisioning implementations should be explicitly provided certain pieces of information.
  - Avoids ambiguity and removes the abstraction of information transfer.
- Require type-checking when implementations request certain configuration values
  - Allows a developer to set some level of expectations of the configuration values.
  - Can prevent ambiguous, hard-to-realize behaviors when coalescing configuration values.

---

## Non-goals
### A list of things this project should not necessarily provide or satisfy. The items are not permanently forsaken, though!

- Version constrained `Actor`s, `Identity`s, `Cohort`s, etc.
  - However, they may be source controlled, à la Git.
- Version constrained dependency management with entities.
  - If this becomes a feature, a `bundler`-style `<noun>file.lock` should be implemented.
- Encrypted handling of secrets (in the application layer just above SSL/TLS).
  - This could mean storing secrets in an encrypted manner.
  - This could also mean communicating secrets over an encrpyted manner.
- Polymorphic coalescing of configuration.
  - Certain configuration should combine in a consistent way. It shouldn't change based on the path of a graph traversal procedure, whether certain `Cohort`s are referenced, etc.
- A notification, pubish-subscribe, or message queue system.
  - Would be nice to have when runtime information is needed.
- A notion of a dependency graph for the collection of `Actor`s to be run.
  - Would also be nice to have.
  - Then perhaps actors must require some sensible defaults.

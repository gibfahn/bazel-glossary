# Bazel Glossary

<!-- **`.bazelrc`.** -->

#### Action
A function that takes [*artifacts*](#Artifact) as inputs, and produces
*artifacts* as outputs. Includes the command line arguments, action key,
environment variables, and declared input/output artifacts. An artifact can be
an input to multiple actions, but must only be generated by at most one action.

#### Action key
The cache key of an [*action*](#Action). Computed based on action metadata, such
as command line that will be executed, compiler flags, library locations and
system headers. Enables Bazel to cache or invalidate individual actions
deterministically.

#### Action cache
The cache of [action](#Action) metadata to action execution result. Lives on
disk in Bazel's output base directory, and thus survives Bazel server restarts.
Read on Bazel server startup.

#### Action graph
An in-memory logical graph of [actions](#Action) and the artifacts these actions
read and generate. These artifacts may include file targets as well as
intermediate artifacts that are not mentioned in `BUILD` files. Produced during
the [*analysis phase*](#Analysis-phase) and used during the [*execution
phase*](#Execution-phase).

#### Analysis phase
The second phase of a build. Processes the target dependency graph specified in
`BUILD` files to produce an in-memory [*action graph*](#Action-graph) that
determines the order of actions to run during the [*execution
phase*](#Execution-phase).

#### Artifact
A file used or derived by the build system. Can also be a directory of files,
also known as tree artifacts. Tree artifacts are black boxes to Bazel, that is,
Bazel does not treat the files in tree artifacts as individual artifacts and
thus cannot reference them directly as action inputs / outputs.

#### Aspect
A mechanism for rules to create additional actions in their dependencies. Most
commonly used for build metadata generation for IDEs, and linting. For example,
if target `A` depends on `B`, one can apply an aspect on `A` that traverses *up*
a dependency edge to `B`, and runs additional [actions](#Action) in `B` to
generate and collect additional output files. These additional actions are
cached and reused between targets requiring the same aspect. Created with the
`aspect()` Starlark Build API function.

#### Aspect-on-aspect
An aspect composition mechanism, where aspects can be applied on other aspects.
Commonly used by IDE aspects to generate files using information also generated
by aspects, like the `java_proto_library` aspect. For an aspect A to inspect
aspect B, aspect A must declare the providers it needs from aspect B (with
`required_aspect_providers` attribute), and aspect B must declare the providers
it returns (with `provides` attribute).

#### `BUILD` file
The main build configuration file containing rule declarations (e.g.
`cc_binary`, `go_library`). When `BUILD` files are evaluated and analyzed, the
rules create new targets in the target dependency graph. A `BUILD` file can load
and use Starlark-defined rules from `.bzl` files. A `BUILD` file marks a
directory as a [*package*](#Package).

<!-- #### Build event protocol -->

<!-- **Command.** -->

<!-- **Command flags.** -->

<!-- **Configurable attributes.** -->

#### Configuration
Context information that may affect a build. Every build has at least one
configuration specifying the host/target platform, action environment variables,
as well as taking input from the default set of flag and attribute values of the
build. Additional configurations may be created, for example, in a
cross-compilation build.

<!-- **Configuration fragment.** -->

<!-- **Configuration-safe output.** -->

#### Configured query
Configuration-aware variant of `bazel query`. Allows querying configured targets
in the post-analysis phase graph, which means configuration flags like
`--platforms` and `--toolchains` affect the result of the query.

<!-- **Constraints.** -->

#### Dependency
A directed edge between two [*targets*](#Target). A target `//:foo` has a
*target dependency* on another target `//:bar` if the attribute values of
`//:foo` contain a reference to `//:bar`. This dependency can also be in the
form of an *action dependency*. That is, if an action that produces a file in
`//:foo` depends on an input artifact that is an output artifact of an action in
`//:bar`, then `//:foo` has an action dependency on `//:bar` by way of the
derived artifact between the two actions.

#### Depset
A data structure for collecting data on transitive dependencies. Optimized to be
time and space efficient around merging, because it’s common to have very large
depsets, scaling to hundreds of thousands of files. Implemented to recursively
refer to other depsets for space efficiency reasons. Rule implementations should
not flatten depsets to lists unless they are at the top level. Flattening large
depsets incur huge memory consumption. Also known as *Nested Sets* in Bazel's
internal implementation.

#### Disk cache
An local on-disk blob store for the remote caching feature. Can be used in
conjunction with an actual remote blob store.

#### Distdir
A read-only directory containing files that Bazel would otherwise fetch from the
internet using repository rules. Enables fully offline builds in an air-gapped
environment.

#### Dynamic execution
An execution strategy that wraps both the local and remote execution strategy,
and through various heuristics, uses the execution result of the faster
successful strategy. Certain actions are better executed locally (e.g. linking),
and others remotely (highly parallelizable compilation), so the dynamic
execution strategy provides the best possible incremental and full build times.

#### Execution phase
The third phase of a build. Deriving from the action graph produced during the
analysis phase, execute actions in order using external tools (compilers,
scripts, etc). Reads and writes artifacts. *Spawn strategies* control how these
actions are executed: locally, remotely, dynamically, sandboxed, docker, and so
on.

<!-- **Execution root.** -->

#### Hermeticity
The lack of external influences during a build or test for the goal of producing
a deterministic and correct result. This could mean only allowing access to
declared input artifacts, making timestamps and timezones fixed, restricting
access/resetting environment variables, and using fixed seeds for random number
generators.

<!-- **Host configuration.** -->

<!-- **Incompatible change.** -->

#### Label
An identifier for a [target](#Target). A fully qualified label
(`//path/to/package:target`) consists of `//` to mark the workspace root
directory, `path/to/package` as the directory that contains the `BUILD` file
declaring the target, and `:target` as the name of the target declared in the
aforementioned `BUILD` file. It can optionally be prefixed with
`@my_repository//<..>` to indicate that the target is declared in an *external
repository* named `my_repository`.

#### Loading phase
The first phase of a build. Parse WORKSPACE, BUILD and .bzl files to expand
*macros* and instantiate an internal cache of *packages*. Usually interleaved
with the second phase of the build, the *analysis phase*, to build up a *target
dependency graph*.

#### Macro
A mechanism to compose multiple predefined rule target declarations together
under a single Starlark function. Expanded to the underlying rule target
declarations during the *loading phase*.

#### Mnemonic
A short human readable string to quickly understand what an [*action*](#Action)
is doing. Can be used as identifiers for *spawn strategy* selections. For
example: `Javac`, `CppCompile`, `AndroidManifestMerger`.

#### Nested Set
See [*depset*](#Depset).

<!-- **Output base.** -->

<!-- **Output groups.** -->

<!-- **Output user root.** -->

#### Package
A directory containing a `BUILD` file. A package can contain subpackages, or
subdirectories containing `BUILD` files, thus forming a *package hierarchy*.

#### Package group
A target representing a set of packages, and therefore has a label usually
referenced in `visibility` attribute values.

<!-- **Platform.** -->

#### Provider
A set of information passed from a rule target to other rule targets. Usually
contains information like: compiler options, transitive source files, transitive
output files, and build metadata. Frequently used in conjunction with
[*depsets*](#Depset).

<!-- **Remote caching.** -->

<!-- #### Remote execution -->

#### Repository
A directory containing a `WORKSPACE` file.

<!-- **Repository cache.** -->

<!-- **Repository rule.** -->

#### Reproducibility
The property that a set of inputs will always produce the same set of outputs
every time, regardless of time, method or environment. Not that the outputs may
not be *correct* or the desired ones.

#### Rule
A function implementation that registers a series of *actions* to be performed
on inputs to produce a set of outputs. Rules can read values from *attributes*
as inputs (e.g. `deps`, `testonly`, `name`). Rule targets also produce and pass
along information that may be useful to other rule targets in the form of
*providers* (e.g. `DefaultInfo` provider).

#### Runfiles
The runtime dependencies of an executable target. Most commonly, the executable
is the executable output of a test rule, and the runfiles are runtime data
dependencies of the test. Before the invocation of the executable (during `bazel
test`), Bazel prepares the tree of runfiles alongside the test executable
according to their source directory structure.

<!-- **Sandboxing.** -->

#### Skyframe
The core parallel, functional, and incremental evaluation framework of Bazel.

<!-- **Spawn strategy**. -->

#### Stamping
A feature to embed additional information into Bazel-built artifacts. Commonly
used for source control, build time and other workspace-related information for
release builds. Enable through the `--workspace_status_command` flag and rules
that support the `stamp` attribute.

#### Starlark
The extension language for writing rules and macros. A restricted subset of
Python (syntactically and grammatically) aimed for the purpose of configuration,
and for better performance. Uses the `.bzl` file extension.
[*`BUILD`*](#BUILD-file) files use an even more restricted version
of Starlark (e.g. no `def` function definitions). Formerly known as Skylark.

<!-- **Starlark Build API.** -->

<!-- #### Starlark rules. -->

<!-- #### Starlark rule sandwich -->

#### Startup flags

The set of flags specified between `bazel` and the command, for example, `bazel
--host_jvm_debug build`. These flags modify the configuration of the Bazel
server, so any modification to startup flags causes a server restart. Startup
flags are not specific to any command.

<!-- **Symlink forest.** -->

#### Target
A buildable unit. Can be a *rule target*, *file target*, or a *package group*.
Rule targets are instantiated from rule declarations in `BUILD` files. Depending
on the rule implementation, rule targets can also be *testable* or *runnable*.
Every file used in `BUILD` files is a file target. Targets can depend on other
targets via attributes (most commonly but not necessarily `deps`). A *configured
target* is a pair of *target* and *build configuration*.

<!-- **Target configuration.** -->

<!-- **Target graph.** -->

#### Target pattern

A way to specify a group of targets on the command line. Commonly used patterns
are `:all` (all rule targets), `:*` (all rule + file targets), `...` (current
package and all subpackages recursively). Can be used in combination, for
example, `//...:*` means all rule and file targets in all packages recursively
from the root of the workspace.

#### Tests
Rule targets instantiated from test rules, and therefore contains a test
executable. A return code of zero from the completion of the executable
indicates test success. The exact contract between Bazel and tests (e.g. test
environment variables, test result collection methods) is specified in the *Test
Encyclopedia*.

#### Toolchain
A set of tools to build outputs for a language. Typically, a toolchain includes
compilers, linkers, interpreters or/and linters. A toolchain can also vary by
platform, that is, a Unix compiler toolchain's components may differ for the
Windows variant, even though the toolchain is for the same language. Selecting
the right toolchain for the platform is known as *toolchain resolution*.

#### Transition
A mapping of configuration state from one value to another. Enables targets in
the build graph to have different configurations, even if they were instantiated
from the same rule. A common usage of transitions is with *split transitions*,
where certain parts of the target graph is forked with distinct configurations
for each fork. For example, one can build an Android APK with native binaries
compiled for ARM and x86 using split transitions in a single build.

<!-- **Trimming.** -->

#### Visibility
Defines whether a [target](#Target) can be depended upon by other targets. By
default, target visibility is private. That is, the target can only be depended
upon by other targets in the same [*package*](#Package). Can be made visible to
specific packages or completely public.

#### Workspace directory
A directory on your filesystem that contains source code for the software you
want to build.

#### `WORKSPACE` file
Defines a directory to be a [*workspace directory*](#Workspace-directory). Can
be empty, although usually contains external repository declarations to fetch
additional dependencies from the network or local filesystem.

## Credits

* Michelle Irvine, Google (for the original glossary)
* Greg Estren, Google (for the configurability glossary)
* Jon Brandvein, Google (contributions to the original glossary)

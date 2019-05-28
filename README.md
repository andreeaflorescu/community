# Community

## What is rust-vmm?

rust-vmm is an open-source project empowering the community to build custom
Virtual Machine Monitors (VMMs) and Hypervisors. It provides a set of
virtualization components that any project can use to quickly develop
virtualization solutions while focusing on the key differentiators of their
product rather than re-implementing common components like KVM wrappers, virtio
devices and other VMM libraries.

The rust-vmm project is organized as a shared-effort, shared-ownership
open-source project that includes (so far) contributors from Alibaba, AWS,
Cloud Base, Crowdstrike, Intel, Google, Red Hat as well as individual
contributors.

Each virtualization component lives in a separated GitHub repository under the
rust-vmm organization. The code is organized such that one repository
corresponds to one
[Rust crate](https://doc.rust-lang.org/book/ch07-01-packages-and-crates-for-making-libraries-and-executables.html).

## Why rust-vmm?

- **Reduce code duplication**. The initial thought with rust-vmm was to create
  a place for sharing common virtualization components between two existing
  VMMs written in Rust:
  [CrosVM](https://chromium.googlesource.com/chromiumos/platform/crosvm/) and
  [Firecracker](https://github.com/firecracker-microvm/firecracker/). These
  two projects have similar code for calling KVM ioctls, managing the
  Virtual Machine memory, interacting with virtio devices and others. Instead
  of having these components live in each of the projects repositories, they
  can be shared through the rust-vmm project.
- **Faster development**. rust-vmm provides a base of virtualization components
  which are meant to be generic such that they can be consumed by other
  projects besides CrosVM and Firecracker. One example that is often mentioned
  is building a container specific VMM. By doing so, the container VMM can
  reuse most of the rust-vmm components and simply build the glue around them
  as well as an appropriate API for interacting with the container.
- **Security & Testability**. Having independent components makes fuzz testing
  easy to apply to each individual package. Our top priority is now
  [vm-virtio](https://github.com/rust-vmm/vm-virtio) as it provides a
  [virtio](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=virtio)
  device implementation. We want to keep a high standard in terms of testing
  as these virtualization packages are going to be used in production by
  multiple projects. Each component is individually tested with a set of
  common integration tests responsible for running unit tests and linters
  (including coding style checks), and computing the coverage. In the future
  we are planning to test the integration of the rust-vmm components by
  providing a reference VMM implementation.
- **Clean interface**. With shared crates you also have to think about an
  interface flexible enough to be used by multiple VMMs. We have multiple
  rounds of design reviews up to the point we are confident the interface is
  clean and reusable by other projects.

## Status of rust-vmm Components

Each rust-vmm crate lives in its own repository. Depending on where the
latest crate code is, the crate can be in one of 3 states:

1. `empty`: The crate repo exists but there is no code there yet. This
   typically means the crate was created after a community acknowledgement
   about its relevance to the project, but either no code has been submitted
   yet or the initial code base is being reviewed through a pull request.

1. `rust-vmm`: Once the initial code base has been merged, the crate may not
   be ready to be pushed to the [crates.io](https://crates.io) repository.
   In that state, the crate code is being improved and will be pushed by the
   rust-vmm maintainers once it meets all the
   [requirements](#publishing-on-cratesio---requirements-list) for being
   released to crates.io.

1. `crates.io`: When in that state, a crate is considered to be production
   ready. As such it can be consumed directly from the Rust canonical crate
   registry: [crates.io](https://crates.io). After meeting all the rust-vmm
   [requirements](#publishing-on-cratesio---requirements-list) for being
   published to crates.io, crates will move to the `crates.io` state. At that
   point, the crate repo in rust-vmm can be seen as a staging area until the
   next push to crates.io. In other words, the production ready version of the
   code lives in crates.io while development happens in the rust-vmm git repo.

### empty

These are the empty repositories that have PRs waiting to be merged.

- [linux-loader](https://github.com/rust-vmm/linux-loader): parser and loader
  for vmlinux and bzImage images as well as some other helpers for kernel
  commandline.
- [virtio-bindings](https://github.com/rust-vmm/virtio-bindings): Rust FFI
  bindings to virtio kernel headers generated using
  [bindgen](https://crates.io/crates/bindgen).
- [vm-virtio](https://github.com/rust-vmm/vm-virtio/): virtio device trait and
  implementation for virtio primitives such as virtqueues and descriptor chain.
- [vmm-sys-utils](https://github.com/rust-vmm/vmm-sys-util/): a collection of
  modules providing helpers and utilities for building VMMs and Hypervisors.
- [vmm-vcpu](https://github.com/rust-vmm/vmm-vcpu/): a hypervisor-agnostic
  abstraction for Virtual CPUs (vCPUs).

### rust-vmm

- [vm-memory](https://github.com/rust-vmm/vm-memory): abstractions over a
  Virtual Machine memory.

#### crates.io

- [kvm-bindings](https://crates.io/crates/kvm-bindings): Rust FFI bindings
  to KVM generated using [bindgen](https://crates.io/crates/bindgen).
- [kvm-ioctls](https://crates.io/crates/kvm-ioctls): Safe wrappers over the
  KVM API.

## Development

### Adding a New Virtualization Component

If you have an idea about a component that could be a great fit for rust-vmm,
please open a
[review request issue](https://github.com/rust-vmm/community/issues/new?assignees=&labels=&template=new-crate-request.md&title=Crate+Addition+Request).
The issue must provide the explanations and justifications behind the need for
a new component, highlighting the purpose of this new repository, how will it
be used by other VMMs or Hypervisors, and other information that you find
relevant.

Discussions about the proposal will happen on the same issue, and once an
agreement is reached regarding the name (this part is hard!) and the design,
an empty repository will be created in the rust-vmm GitHub organization.

To add functionality you will need to send Pull Requests (PRs) against the
newly created repository. While we do not expect the code to be perfect from
the first try, we do want to keep a high standard in terms of quality for all
existing repositories under rust-vmm.

### Publishing on crates.io - Requirements List

We consider crates published on crates.io to be production ready. In order to
maintain a high security and quality standard, the following list of
requirements must be checked before having the first version published:

- **High level documentation**. This documentation should be added
   to `src/lib.rs` and it will end up being the home page of the crate on
   [docs.rs](https://docs.rs/). Here you should include information about what
   is the crate suppose to do, how can it be used in other projects as well as
   usage examples. Check out the
   [kvm-ioctls docs](https://docs.rs/kvm-ioctls/0.1.0/kvm_ioctls/) for an
   example.
- **Documentation for the public interface**. Everything belonging to the
  public interface of the crate must be properly documented. Where applicable,
  the methods should also have usage examples. The code examples are
  particularly useful because they are run when running the unit tests with
  `cargo test`. It is a nice way to ensure that your comments and examples
  don't go out of sync with the code. To make sure we don't miss anything this
  rule is enforced by using `#![deny(missing-docs)]` in `src/lib.rs`.
- **Unit Tests**
- **Coverage**
- **Integration Tests**. All components will eventually be tested using the
  same common [Buildkite pipeline](https://buildkite.com/docs/pipelines). This
  is still WIP. The tests include style checks and other lint checks, a
  coverage test, builds with various configurations and on different
  architectures and operating systems.
- **README.md**
- **LICENSE**
- **CODEOWNERS**

### Versioning Crates - WIP

This section is under construction.

### CI - WIP

We currently have the CI running on either [Travis](https://travis-ci.org/) or
[Buildkite](http://buildkite.com/). We are working towards having a central
repository for the CI so stay tuned for details on how we are testing the code.

This section is under construction.

### Fuzz Testing - WIP

This section is under construction.

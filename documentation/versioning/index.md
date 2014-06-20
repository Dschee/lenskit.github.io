---
title: Versioning
layout: default
sidenav: doc-nav.html
---

# Versioning and API change policy

LensKit strives to maintain a high degree of compatibility, particularly
for client code (code using the recommenders).

We define the following use cases:

Clients
:   Client code is code that uses the LensKit APIs and algorithms
    to provide recommendation services. It does not implement any LensKit
    interfaces, other than perhaps the DAO.

Implementers
:   Implementation code implements various LensKit APIs to provide
    new algorithms. It may implement individual components of existing algorithms,
    or reuse their components.

Extenders
:   Extension code is level most closely coupled to LensKit's internals.
    This is code that derives from the classes inside LensKit algorithm implementations,
    is closely tied to them (re-implementing particularly internal components, etc.),
    or extends the evaluator.

LensKit version numbers have three components, <Major.Minor.Patch>. The
versioning is based on [Semantic Versioning](http://semver.org), adapted
for the needs of a framework and toolkit.

* For all components, binary- and source-incompatible changes will result in
  at least a minor version increase (1.x). 1.x.y releases, where y≥1, will only
  be bug fixes.

* Client code may need to update its recommender configurations across minor (1.x)
  releases, but we will seek to minimize these changes and clearly document
  any necessary changes in the release notes.

* Types designated “Compatibility: Public” will only have source- or binary-incompatible
  changes for client code in new major releases of LensKit. Incompatible changes for
  implementers or extenders will be associated with new minor versions and clearly
  identified in the release notes.

## Serialization

LensKit makes no attempt to allow models built and serialized with version 1.x
to be read in version 1.y, where x ≠ y.  We attempt to keep bug fix releases
serialization-compatible, but if a serialization break is necessary to fix a
bug, we will break the class (and document such incompatibility in the release
notes).

## Deprecation

As we develop LensKit, we deprecate methods (and entire classes) from time to
time.  To preserve backwards compatibility, when these methods appear on
“Compatibility: Public” components, they will stick around until the next major
release. However, they **will** be deleted in the next major release.
Deprecated methods on more internal/unstable components may be deleted in a
subsequent minor release.

Our goal is for all X.0 releases of LensKit to contain no deprecated methods.

Also, if we intend to replace a method with an incompatible method of the same
name in the next X.0 release, we we'll try to make sure that method either
changes its signature or gets deprecated in a minor release in advance of the
major release. So it's good, as you update your LensKit-based code, to clear up
deprecation warnings when you upgrade to a new LensKit version, but we'll keep
the methods around and working.

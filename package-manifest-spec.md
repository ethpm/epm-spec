# Package Manifest Specification

This document defines the specification for the *Package Manifest*.  The
package manifest contains meta data about the package as well as
dependencies needed for installation of the package.


## Guiding Principles

The package manifest specification makes the following assumptions about the
document lifecycle.

1. Package Manifest documents will be created and modified manually by people.
2. Package Managers will often be used to assit in the creation and modification of Package Manifest documents.
1. Package manifests will be consumed by package managers.
2. Package manifests will normally be stored alongised the package source code.


## Format

The canoonical format for the package manifest is a JSON document containing a
single JSON object.  


## Filename

Convention is to name this document `epm.json` which is short for *'ethereum
package manifest'*.

## Document Overview

The following fields are defined as data that *may* be included in a package
manifest.  Custom fields should be prefixed with `x-` to prevent collision with
new fields introduced in future versions of the specification.

- manifest version
- package name
- authors
- version
- description
- keywords
- contracts
- external project URIs (homepage, repository uri, etc.)
- dependencies

## Document Specification

### Manifest Version: `manifest_version`


The `manifest_version` field defines the version of the ethereum package manifest
specification (this document) that this manifest conforms to. All manifests
**must** include this field.

* Key: `manifest_version`
* Type: Integer
* Allowed Values: `1`
* Package Manager Guide: Package managers should validate this field is valid prior to publishing new manifests.


### Package Name: `package_name`

The `package_name` field defines a human readable name for this package.  All
manifests **should** include this field.

* TODO: what should happen if this is missing (should it be required?)
* Key: `package_name`
* Type: String

### Authors: `authors`

The `authors` field defines a list of human readable names for the authors of
this package.  All manifests **should** include this field. 


* Key: `authors`
* Type: List of Strings

### Version: `version`

The `version` field declares the current version number of this package.  This value
**should** be conform to the [semver](http://semver.org/) version numbering
specification.  All manifests must **include** this field.

* Key: `version`
* Type: String


### Description: `description`

The `description` field *should* be used to provide additional detail that may be relevant for the package.

* TODO: should this include a suggestion to use a specific markup format like markdown?
* Key: `description`
* Type: String


### Description: `keywords`

The `keywords` field *should* be used to provide relevant keywords related to this package.

* Key: `keywords`
* Type: List of Strings


### Links: `links`

TODO: This could potentially not be necessary, or should be removed.  Needs discussion.

The `links` field *should* be used to provide URIs to relevant resources
associated with this package.  When possible, authors *should* use the
following keys for the following common resources.

* `website`: Primary website for the package.
* `documentation`: Package Documentation
* `repository`: Location of the project source code.
* TODO: what other common fields?

* Key: `links`
* Type: Hash(String: String)


### Contracts: `contracts`

The `contracts` field defines a list of contract names that should be included
in the release manifest when generating a release.


TODO: flesh this out
* Key: `contracts`
* Type: List of Strings


### Dependencies: `dependencies`

the `dependencies` field defines a list of project dependencies.

TODO: flesh this out. Keys declare dependencies.  Values are semver.  What format do we declare dependencies in?
* Key: `dependencies`
* Type: Hash(String: String)
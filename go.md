---
layout: default
title: Go
description: Go tips
tags: go golang coding
---

## Go intro

[Module](https://go.dev/ref/mod#modules-overview) based usage is assumed. A module is like a python virtualenv.
```sh
go help modules
```

[GOPATH environment variable](https://pkg.go.dev/cmd/go#hdr-GOPATH_environment_variable) defaults to $HOME/go and is used to resolve import statements.

Initialize a module.
```sh
mkdir hello && cd hello
go mod init example.com/hello
```

Add module requirements and sums after creating the .go file.
```sh
go mod tidy
```

Run the code.
```sh
go run .
```

Remove a dependency on a module and downgrade modules that require it.
```sh
go get rsc.io/quote@none
```

List modules (-m) that are dependencies of your current module, with the latest version available for each (-u).
```sh
go list -m -u all
```

Replace module remote path with local filesystem path. Modules are generally intended to be published in a decentralized manner.
```sh
go mod edit -replace example.com/greetings=../greetings
```

Manage dev tools written in Go.
```sh
go get -tool golang.org/x/tools/cmd/stringer
```

List tools

```sh
go tool
```

Compile the executables, including dependencies.
```sh
go build
go install # compiles and installs to $GOBIN
```
GOBIN defaults to $GOPATH/bin

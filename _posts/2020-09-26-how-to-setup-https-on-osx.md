---
author: Gary Rowe
title: How to set up an HTTPS server on OS X
layout: post
tags:
  - HowTo 
---
Every so often I find myself needing to run up an HTTPS server locally for
development. Here are the steps for doing on OS X, assuming you've
already installed [Homebrew](https://brew.sh/):

```bash
brew install mkcert
brew install nss
brew install http-server

mkcert -install
<Enter root password for CA store to be imported>

cd <project website root>
mkcert localhost

http-server -S -C localhost.pem -K localhost-key.pem
```

Check activity on [https://localhost:8080](https://localhost:8080)

The `mkcert` tool creates a local certificate authority (CA) that is then added
to the local trust store (that's why you need the root password) so that it can
be trusted by the system. 

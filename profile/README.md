<!--
  Canonical ("constitutional") document — one of two top-level READMEs
  (this one, then the superproject README at os384/os384). Slow-changing
  by design. AI agents: do not edit directly; suggest changes to maintainer.
-->

<div align="center">
  <p><i>stay tuned ... "rc3" is about to be released ... we're trickle releasing parts right now </i></p>
</div>

<div align="center">
  <h1>os384</h1>

  <p><strong>Sovereign computing for the open web.</strong></p>

  <p>
    <a href="https://384.dev">384.dev</a> &nbsp;·&nbsp;
    <a href="<placeholder>">Docs</a> &nbsp;·&nbsp;
    <a href="https://github.com/os384/os384">Source</a>
  </p>

  <p>
    <img alt="License: AGPL v3" src="https://img.shields.io/badge/License-AGPL%20v3-blue.svg" />
    <img alt="Built on P-384" src="https://img.shields.io/badge/crypto-P--384-blue" />
    <img alt="TypeScript" src="https://img.shields.io/badge/language-TypeScript-blue" />
  </p>
</div>

---

os384 is a browser-native platform for building **private, encrypted,
user-sovereign applications** — without accounts, without centralized servers,
and without platform lock-in.

The web (and the internet for that matter) was supposed to be a peer-to-peer
system. Instead, it became a handful of platforms that hold everyone's data,
decide on identity, read their messages, and can revoke access at will.

os384 is an attempt at a technical answer to that problem: a small, auditable,
self-hostable stack where the platform layer runs in the user's browser and
servers are reduced to encrypted storage with as simple semantics as possible.

There is no central authority. Everything is AGPLv3. You can take advantage
of commercially hosted servers, or run your own.

os384 is self-hosting, meaning, os384 channels and servers provide the various
artifacts. Dependency on any other systems or code bases is minimal.
Strictly speaking, os384 doesn't need central servers (or even DNS).

---

## Storage and Communication

Everything is built on two primitives:

**Shards** are immutable untyped blobs with content-based naming and encryption.
The storage protocol allows anonymous (private) deduplication. Back end is
the "storage server". Hosted storage provides "permastore": you only pay once
for any blob to be permanently available.

**Channels** combine end-to-end encrypted communication endpoints with support
for arbitrary mutable data structures. Global naming is anchored in public keys,
with ownership proved by possession of the private key. Back end is the
"channel server", which also implements "Pages" — public (unencrypted) entry
points into the ecosystem.

System equivalents to user accounts, identity models, and file systems are built
using those primitives.

---

## Compute (Apps)

The compute model is provided by the **Loader**, which implements an in-browser
microkernel/VMM to support private (anonymous) launching of web apps:

```
  You type: 384.dev/#xK9mR2...
                │
                │  (the #fragment never leaves your browser)
                ▼
         ┌─────────────┐
         │   Loader    │  resolves the channel page,
         │  (384.dev)  │  launches your app in a random subdomain
         └──────┬──────┘
                │ lib384 runtime
                ▼
         ┌─────────────┐
         │  Your App   │  runs in an isolated origin
         │  (browser)  │  keys never leave your device
         └──────┬──────┘
                │ encrypted reads/writes only
       ┌────────┴────────┐
       ▼                 ▼
  Channel Server    Storage Server
  (messages,        (shards,
   state, budget)    encrypted blobs)
```

The loader reads a channel key from the URL fragment and uses it to fetch and
launch an application from that channel's page. Apps run in randomly-generated
subdomains for strict origin isolation. No app can read another app's storage or
intercept another app's keys. The os384 back end servers only serve anonymous
data: neither they nor the network, DNS provider, etc, will know if you're
running an application or just downloaded a jpeg of a kitten.

**Identity** is derived entirely on the user's device from a passphrase and a
strongpin. There is no registration, no email address, no password sent to any
server. If you know your passphrase, you have your keys. The keys never leave
your browser.

Back ends are intentionally minimal. The channel server manages encrypted
message routing and channel state. The storage server manages encrypted blob
storage. Together they are fewer than 3,000 lines of TypeScript, deployable to
Cloudflare Workers or any Docker host. They see only ciphertext — they cannot
read your data.

---

## The Code

All components live in one superproject, with each part as a submodule:

```sh
git clone --recursive https://github.com/os384/os384
```

At a glance, inside the superproject:

| Component | What it is |
|---|---|
| `lib/` | **lib384** — the core TypeScript runtime library; the foundation everything builds on |
| `loader/` | The browser microkernel at 384.dev |
| `services/` | The back end: channel server + storage server (Cloudflare Workers) |
| `cli/` | The `384` command-line tool |
| `file-manager/` | Encrypted file manager — reference app and app-deployment tool |
| `mirror/` | Lightweight local shard cache (Python, zero dependencies) |
| `docs/` | Documentation site — architecture, protocol reference, glossary, essays |

The superproject [README](https://github.com/os384/os384) is the map: how the
parts fit together, developer entry points, and workspace conventions. Read it
next. A few additional components (template/demo apps, a storage-token paywall)
are being cleaned up and will be folded into the superproject.

---

## Quickstart

<i>Currently only MacOS instructions. Linux coming.</i>

Install [Deno](https://deno.com) if you don't have it:

```sh
brew install deno
# or: curl -fsSL https://deno.land/install.sh | sh
```

`384` is the core command line utility:

```sh
deno install -f --global -n 384 --allow-read --allow-write --allow-net --allow-env \
  https://c3.384.dev/api/v2/page/8yp0Lyfr/384.20260404.0.ts
```

(Note the date embedded in the filename: channel pages serve the current
version regardless of the name, so the date is simply a cache-busting device —
it's always safe to toggle it to any value to defeat stale web caches.)

Then go to [384.dev](https://384.dev) and create an "account" (vault). Purchase
storage, or contact us at info@384.co for a free developer starting budget.
Then "mint" a token, and use it to set up your local keys:

```sh
384 init <token>
```

This sets up local developer context in `~/.os384`.

Building apps *on* os384 requires nothing else — no local servers, no build
step; lib384 is imported straight from a channel page URL. If you want to work
on os384 itself, or run your own servers, the superproject README lays out the
paths.

---

## Early Days

There is a lot of work to do. At time of writing the versions published are pre
releases of `rc3`. lib384 versioning is the main version to track. There are a
number of additional building blocks that we are working on adding to this open
source release, as well as a number of proof-of-concept applications and
templates: video player, in-client indexing, audio and video conferencing,
multiplayer gaming, photo sharing and archiving, chat and messaging, document
collaboration, docusign replacement, social media replacement, note taking app,
personal audiobook player.

---

## License & community

Everything is **AGPLv3**. os384 is a community project — 384, Inc. has
open-sourced all of os384. If you run a server, build an app, or contribute
code, you're part of the ecosystem.

The design goal for the server side is an ecosystem of independent operators —
developers, enthusiasts, small teams — running their own storage and channel
servers and accepting payment directly from users, with no platform taking a
cut. And free and open exchange of applications. A small paywall service (early
skeleton) is the first step toward making that simple.

---

<div align="center">
  <sub>AGPLv3 · P-384 cryptography · No accounts · No ads · No lock-in · Self-hostable</sub>
</div>

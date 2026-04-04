
<div align="center">
  <p><i>stay tuned ... "rc3" is about to be released ... </i></p>
  </div>

<div align="center">
  <h1>os384</h1>

  <p><strong>Sovereign computing for the open web.</strong></p>

  <p>
    <a href="https://384.dev">384.dev</a> &nbsp;·&nbsp;
    <a href="https://384.dev/#FZJuzWVgsoj93N6WJRrNjnDY6f5nu5cgWuEECXRnamd_63745.31779.2250.15401_iEFaad2Zar1pveVdPbuRHGLM3o3V5lGzX6MabfoLgcZ_auto">Docs</a> &nbsp;·&nbsp;
    <a href="https://github.com/os384/lib384">lib384</a>
  </p>

  <p>
    <img alt="License: GPL-3.0" src="https://img.shields.io/badge/License-AGPL%20v3-blue.svg" />
    <img alt="Built on P-384" src="https://img.shields.io/badge/crypto-P--384-blue" />
    <img alt="Deno" src="https://img.shields.io/badge/runtime-Deno-black" />
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
artifacts. Dependency on any other systems or code bases are minimal. 
Strictly speaking, os384 doesn't need central servers (or even DNS).

---

## Storage and Communication

**Shards** are immutable untyped blobs with content-based naming and encryption.
The storage protocol allows anonymous (private) deduplication. Back end is
the "storage server". Hosted storage provides "permastore", you only pay once
for any blob to be permanently available.

**Channels** combines end-to-end encrypted communication endpoints with support
for arbitrary mutable data structures. Global naming is anchored in public keys
with ownership proved by owner of private key. Back end is the "channel server",
which also implements "Pages" that provide public (unencrypted) entry points
into the ecosystem.

System equivalents to user accounts, identity models, and file systems are built
using those primitives.

---

## Compute (Apps)

The compute model is provided by the **Loader**, which essentially implements an
in-browser-client microkernel VMM to support private (anonymous) launching of
web3 apps:

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
intercept another app's keys. The os384 back end servers only serve anonymous data:
neither they nor the network, DNS provider, etc, will know if you're running
an application or just donwloaded a jpeg of a kitten.

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

## Repositories

### Core

**[lib384](https://github.com/os384/lib384)** — The TypeScript runtime library.
This is the foundation everything else builds on. Implements the full
cryptographic protocol: shard read/write, channel send/receive, wallet/identity
management, SBFS (a virtual filesystem over shards via service worker), and
browser helpers. Builds to an ESM bundle and an IIFE bundle (`window.__`).
Distributed natively via os384 channel pages. Runs in any modern browser client
or on Deno locally.

**[loader](https://github.com/os384/loader)** — The browser microkernel at
[384.dev](https://384.dev). A Cloudflare Worker backed by a Vite/TypeScript
frontend. Reads the URL fragment, resolves the channel page, enforces app
isolation via random subdomains, and manages the user's wallet. The user's entry
point to the entire system.

**[services](https://github.com/os384/services)** — The server-side
infrastructure. Two Cloudflare Workers:
- `channel/` — channel server (messaging, durable objects, storage budget accounting)
- `storage/` — storage server (shard store, KV + durable objects, token validation)

A complete, auditable backend. The channel server is ~2,500 lines of TypeScript;
the storage server is under 500 lines.

### Tools

**[cli](https://github.com/os384/cli)** — The `384` command-line tool. Manages
channels, uploads and downloads shards, deploys lib384 artifacts to channel
pages, and bootstraps wallets. Written in Deno with Cliffy for argument parsing.
Install with one `deno install` command; no npm, no package manager.

**[paywall](https://github.com/os384/paywall)** — A small Cloudflare Worker that
lets storage server operators sell storage tokens directly to users. Accepts
Bitcoin, Ethereum, Litecoin, and PayPal. Operators pre-generate signed tokens
offline with the CLI and load them into a pool; the paywall dispenses them on
payment confirmation. No accounts, no token generation at runtime — the token is
the receipt.

**[mirror](https://github.com/os384/mirror)** — A lightweight Python shard
cache. Proxies shard requests to upstream storage servers and caches locally.
Zero dependencies beyond the Python standard library. Useful for local
development, self-hosted deployments, or just serve as an ongoing local cache
for any data the user touches in any application, thus guarding against
undetected deplatforming, take-downs, and other third-party actions.

### Applications

**[file-manager](https://github.com/os384/file-manager)** — An encrypted file
manager app built with Lit web components and Vite. Upload, browse, and preview
encrypted file sets stored on os384 channels. A reference implementation of a
"privileged" os384 app — one that ships with the platform rather than living
purely on channel pages. Also serves as deployment tool: any static web site
(app) is stored as a file set that the loader will recognize (and can launch).

**[demos](https://github.com/os384/demos)** — A growing number of template and
sample applications illustrating lib384 usage and os384 application
architecture.

### Documentation

**[docs](https://github.com/os384/docs)** — The documentation site, built with
VitePress. Architecture, protocol reference, CLI guide, glossary, and essays on
sovereign computing. Also serves as context for AI coding assistants working on
the codebase. Latest version is deployed to:

[https://384.dev/#FZJuzWVgsoj93N6WJRrNjnDY6f5nu5cgWuEECXRnamd_63745.31779.2250.15401_iEFaad2Zar1pveVdPbuRHGLM3o3V5lGzX6MabfoLgcZ_auto]


---

## Quickstart

Install Deno if you don't have it:

```sh
  brew install deno
  # or: curl -fsSL https://deno.land/install.sh | sh
```

`384` is the core command line utility:

```sh
  # Install 384 globally. It's always safe to toggle the date version to any value.
  deno install -f --global -n 384 --allow-read --allow-write --allow-net --allow-env \
    https://c3.384.dev/api/v2/page/8yp0Lyfr/384.20260404.0.ts
```

You'll also need to go to [https://384.dev] and create an "account" (vault). Purchase
storage or contact us at info@384.co for free developer starting budget. Then "mint"
a token, and use it to set up your local keys:

```sh
  384 init <token>
```

This sets up local developer context in `~/.os384`. 

You can also do the above against your own local servers, just provide `--local` to the
384 cli after you've set up your local servers.

---

## Local (personal/developer) servers

If you're going to be doing any local development, or just want to run your own
servers, we suggest you create `~/os384` and put any os384 repos in it. Also,
you'll need to create `/Volumes/os384` (MacOS) for server state.

The simplest setup is to spin up with Docker /
[OrbStack](https://orbstack.dev/):

```sh
  cd ~/os384
  git clone https://github.com/os384/services
  cd services/docer && docker compose up
```

Default os384 assignments are: `3840` loader · `3841` mirror · `3843` storage · `3845` channel

You can also run servers directly in terminal, as well as deploy to public
servers. See docs for more details.

---

## Early Days

There is a lot of work to do. At time of writing the version pubished are pre
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
cut. And free and open exchange of application. The paywall repo is the first
step toward making that simple.

---

<div align="center">
  <sub>AGPLv3 · P-384 cryptography · No accounts · No ads · No lock-in · Self-hostable</sub>
</div>

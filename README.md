# noid-gpu-miner

**A GPU miner for Parano1d (NOID).** Windows, Linux, HiveOS.

Parano1d's proof of work is [documented as CPU-only](https://github.com/ignotusnemo/parano1d/discussions/12):
after every block the node must build a fresh HistoryStep proof before there is
anything to mine, and that proof is CPU work no GPU can do. That is true — and
it is why a GPU cannot mine Parano1d *on its own*.

Behind a pool the limit disappears. The pool proves once, and every GPU attached
to it searches nonces on the same template in parallel. The nonce search is
Poseidon2b over GF(2^128), where CPUs have carryless multiplication in hardware
and GPUs have to emulate it — so the advantage was not obvious in advance.
Measured, it is real:

| card | per card |
|---|---|
| RTX 5090 | 62 MH/s |
| RTX 4090 | 42 MH/s |
| RTX 5080 | 32 MH/s |
| RTX 5070 Ti | 26 MH/s |

Those are the pool's own numbers — the `measured` column, computed from shares
actually delivered, not from what the miner claims about itself — divided by the
number of cards in the rig. One process drives every card in the machine.

---

## Download

From [Releases](../../releases):

| you have | take |
|---|---|
| Windows | `noid-gpu-1.0-windows-x64.zip` |
| Linux | `noid-gpu-1.0-linux-x64.tar.gz` |
| HiveOS | `noid-gpu-1.0.tar.gz` — paste its URL as the custom miner Installation URL |

The bare `noid-gpu.exe` and `noid-gpu` are there too, for anyone who wants
just the binary; the archives also carry the instructions, and a `.exe` inside
a zip is far less likely to be blocked on its way to you.

## Quick start

**Windows**

```
noid-gpu.exe --gpu --coinbase o1YOURADDRESS --worker rig1
```

**Linux**

```
./noid-gpu --gpu --coinbase o1YOURADDRESS --worker rig1
```

Your NOID address is your login and your payout address — there is no
registration. Generate it **on your own machine** with the official Parano1d
wallet and give the pool only the public address. Never share a seed phrase or
a wallet file: mining does not need them, and anyone asking you for them is
stealing from you.

## Options

| option | meaning |
|---|---|
| `--gpu` | use the GPU (without it, the CPU search is used) |
| `--coinbase o1…` | **your** NOID address — required, it is who gets paid |
| `--worker NAME` | label for this machine on the pool page |
| `--pool NAME` | which pool to mine on (see below) |
| `--schede 0,1` | use only these cards (default: all of them) |
| `--a-vuoto` | search and measure, send nothing — for testing |
| `--aiuto` | full list |

## Pools

| | dashboard | pool fee |
|---|---|---|
| `--pool parano1d` *(default)* | http://185.189.45.186:3783 | 10% |
| `--pool ariapool` | https://pool.ariabrain.com/noid.html | 3% |

Both accept your address as your login. The pool fee is charged by the pool and
is separate from the dev fee below.

The Parano1d pool is reached through `parano1d-pool.fun`, with several fallback
addresses behind it. That is deliberate: a pool that has to move should not
break every miner already installed. If the miner loses the pool while running,
after two minutes without work it goes looking for it again — you will see
`pool moved: now mining on ...` — instead of hanging on a dead address.

This build mines **only** on the pools in that list. To mine somewhere else, or
against your own node, use the official `parano1d-miner`: this one is not the
right tool for that and does not pretend to be.

## HiveOS

`noid-gpu-1.0.tar.gz` is a custom miner package. In your flight sheet:

- **Miner** → Custom → **Installation URL**: the `.tar.gz` from Releases
- **Wallet and worker template**: your `o1…` address
- **Pool URL**: anything containing `aria` selects AriaPool, anything else the Parano1d pool
- **Extra config arguments**: optional, e.g. `--schede 0,1`

The worker name comes from the rig name you already set in Hive. Hashrate and
accepted/rejected shares are reported as one figure for the whole rig, not one
per card: the miner searches with all cards on a single counter, so a per-card
split would be invented.

Requirements: a **jammy or noble** Hive image (focal has glibc 2.31, this binary
needs 2.34) and driver **580 or newer** (`nvidia-driver-update 580`). The CUDA
toolkit is *not* needed — the runtime is linked into the binary.

---

## Dev fee — 10%, disclosed

This miner carries a **10% dev fee**. For 60 seconds out of every 600 seconds of
mining it submits shares under the author's address instead of yours:

```
o1efnn3pn9c3p7etdv0addk9pply64cveujmkks7vt4rx0vjy2zufssg84zj
```

That address is dedicated to this miner and used for nothing else, so anyone can
check on chain exactly what this software earns.

What the fee does **not** do:

- it never changes the coinbase of a block. Blocks always pay the pool that
  built them, and the pool pays its miners by its own rules;
- it never switches in the middle of a search. The change happens only between
  jobs, so no share is ever credited to a different address than the one it was
  requested under;
- it is announced on **every single run**, in the header, with the address in
  full. You do not have to trust this file — read the screen:

```
  dev fee            : 10% of mining time to o1efnn3pn9c3p7etdv0addk9…
                       60 s out of every 600 s, switched only between jobs.
                       It never changes the coinbase of a block.
```

The fee always goes to the Parano1d pool first, whichever pool you are mining
on; AriaPool is used for the fee only if the first cannot be reached. It is
decided once at startup and does not rotate.

**If the fee cannot be delivered, this miner stops.** At startup it checks that
the fee is deliverable, and if it is not, it says so and exits instead of
mining. A miner that quietly does something different from what it printed on
your screen is not one you should be running — and that includes this one.

To watch the fee for yourself: run with `--a-vuoto` and look for the
`dev fee window` lines. One per ten minutes, one minute long.

## What you should see

```
new job ap204415-3091  height 3091  expires in 28s
share accepted
job ap204415-3091: 16 shares accepted  (11s)
```

`stale` means the pool moved to a new job before your share arrived: normal on
20-second blocks, and it costs you nothing. `waiting for work` means the pool's
node is proving the next template — also normal, and it lasts a few seconds.

## Verifying the download

```
Windows:  certutil -hashfile noid-gpu.exe SHA256
Linux:    sha256sum -c SHA256SUMS.txt
```

Compare against `SHA256SUMS.txt` in the release.

## Source

Only binaries are published here. If you would rather run something you can
read, run the official `parano1d-miner` instead — that is a legitimate choice
and this page will not argue with it.

## Warranty

None. Provided as is. Mining software moves money; run it only if you accept
that risk, and only from a source you decided to trust.

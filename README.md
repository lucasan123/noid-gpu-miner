# NOID-GPU 1.4.0

NOID-GPU is an NVIDIA GPU miner for Parano1d (NOID). Version 1.4.0 does the
heavy arithmetic with a lookup table kept in fast on-chip memory instead of
computing it every time.

## Highlights

- **Between 40% and 57% more hashes per second at the same wattage** on
  RTX 30 and newer. On an RTX 5070 Ti, both releases on the same card:
  50.5 -> 79.3 MH/s at 250 W.
- **RTX 20 cards are now supported.** They use the version that is faster on
  that generation, chosen automatically; nothing to configure.
- **One download for every card**, from RTX 20 to RTX 50. Earlier plans were
  to split it in two; that turned out not to be needed.
- The mining path was run against the live pool and produced accepted shares
  — something no previous release had verified end to end.
- Public miner developer fee: **5%**. Parano1d Pool fee: **3%**, charged
  separately by the pool. Neither changed.

## Measured performance

Each figure is the middle value of a run lasting about ten minutes, with the
card already warm, on 28 August 2026.

| GPU | Power cap | Actual rate |
|---|---:|---:|
| RTX 5090 | 600 W | **190.630 MH/s** |
| RTX 4090 | 350 W | **125.203 MH/s** |
| RTX 5070 Ti | 250 W | **78.594 MH/s** |
| RTX 3090 | 280 W | **54.428 MH/s** |
| RTX 2080 Ti | 250 W | **5.214 MH/s** |

**Always read a rate together with its power cap.** These four cards ran at
different caps and cannot be compared with each other. A 4090 limited to
350 W is a common setup; a 4090 allowed 450 W will report more. Neither is
wrong, and a rate quoted without its cap means nothing.

These are offline measurements, not pool-side rates and not guarantees. Card
model, clocks, voltage, cooling, driver and power settings all change both
the rate and the watts.

Full detail, including what was *not* tested, is in `RELEASE-NOTES.md`.

## Downloads

| Platform | Asset |
|---|---|
| Windows | `noid-gpu-windows-1.4.0.zip` |
| Linux | `noid-gpu-linux-1.4.0.tar.gz` |
| HiveOS | `noid-gpu-hiveos-1.4.0.tar.gz` |

Use only files from the v1.4.0 release, and check them against
`SHA256SUMS.txt` before running them.

## Requirements

- 64-bit Windows or Linux.
- An NVIDIA card from the RTX 20 series onwards: RTX 20, RTX 30, RTX 40,
  RTX 50, or the equivalent professional cards.
- NVIDIA driver 580 or newer.
- Everything else the miner needs is inside the download; the CUDA toolkit
  is not required on the mining rig.

GTX 10 series and older will not run this miner: they are too old for the
instructions it uses. GTX 16 cards are the same generation as the RTX 20 and
should work, but none has been tested.

The Linux and HiveOS builds run on distributions with GLIBC 2.30 or newer,
which covers everything from Ubuntu 20.04 onwards. Both packages contain the
same executable, byte for byte.

## Quick start

Windows PowerShell:

```powershell
.\noid-gpu.exe --gpu --pool parano1d --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1
```

Linux:

```bash
chmod +x noid-gpu
./noid-gpu --gpu --pool parano1d --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1
```

Offline GPU benchmark:

```text
noid-gpu --benchmark-gpu --devices 0
```

Use only your public `o1...` payout address. Mining never requires a seed
phrase, private key, or wallet file.

## Pools and custom endpoints

The public miner accepts the built-in pool names and custom HTTP endpoints:

```text
noid-gpu --gpu --pool ariapool --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1
noid-gpu --gpu --pool 192.0.2.10:3784 --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1
noid-gpu --gpu --pool http://pool.example:3784 --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1
```

`--pool` therefore works with a compatible local or remote pool. The custom
endpoint applies to the user's mining jobs; developer-fee jobs use only the
destinations embedded in the public executable.

## HiveOS

Select **Custom miner** in the Flight Sheet and use:

```text
Miner name:        noid-gpu-hiveos
Installation URL: URL of noid-gpu-hiveos-1.4.0.tar.gz
Hash algorithm:   poseidon2b
Pool URL:         pool host:port, HTTP URL, parano1d, or ariapool
Wallet/template:  your public o1... payout address
```

The HiveOS adapter obtains the worker name from HiveOS. Extra miner arguments
can be placed in the Flight Sheet's custom configuration field.

## Fees

The public miner developer fee is a **5% scheduled mining-time window**: 30
seconds in each 600-second cycle. Switching occurs only between jobs, so a
short observation can vary around 5%. The selected pool charges its own fee
separately. The Parano1d Pool fee is **3%**.

## Verify the download

Linux:

```bash
sha256sum -c SHA256SUMS.txt
```

Windows PowerShell:

```powershell
Get-FileHash .\noid-gpu-windows-1.4.0.zip -Algorithm SHA256
```

Compare the Windows result with the matching line in `SHA256SUMS.txt`.

## What changed under the hood

- The heavy arithmetic is now a lookup table held in fast on-chip memory,
  instead of being recomputed for every attempt.
- The path used for real mining — which is not the one the offline test
  exercises — was run against the live pool for the first time, and its
  shares were accepted.
- Every card in the release is built in ahead of time, so none of them has
  to prepare itself the first time you start the miner.
- Unchanged: the processor still re-checks every result before it is sent,
  multiple cards still work, and you can still pick specific ones.

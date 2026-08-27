# NOID-GPU 1.3.0

NOID-GPU is an NVIDIA GPU miner for the Parano1d (NOID) Poseidon2b proof of
work. Version 1.3.0 introduces the new R4 GPU path, ships separate Windows,
Linux, and HiveOS packages, and keeps all normal user-facing text in English.

## Highlights

- New R4 field representation and CUDA execution path.
- Fast target handling tested through 26 bits, including the 24-bit share
  target path.
- Expanded CPU/GPU correctness checks and long-run stability benchmarks.
- Public miner developer fee: **5%**.
- Parano1d Pool fee: **3%**, charged separately by the pool.
- NVIDIA Ampere or newer support: RTX 30, RTX 40, RTX 50, and equivalent
  Ampere-or-newer professional cards. RTX 20 cards are not supported.

## Measured performance

| GPU / comparison | Gross wall rate | Board power | Efficiency | Result |
|---|---:|---:|---:|---:|
| RTX 5090, R4 | **127.096 MH/s** | **492.977 W** | **0.257813 MH/s/W** | Three 300-second runs |
| RTX 4090, R4 | **88.951 MH/s** | **359.870 W** | **0.247175 MH/s/W** | Three 300-second runs |
| RTX 5070 Ti, previous path | 41.315 MH/s | 162.420 W | 0.254371 MH/s/W | Controlled baseline |
| RTX 5070 Ti, R4 | **51.337 MH/s** | **188.733 W** | **0.272009 MH/s/W** | **+24.258% MH/s; +6.934% MH/s/W** |

These are gross offline benchmark results from the real GPU hashing path. The
RTX 5090 and RTX 4090 figures were derived from three 300-second runs per GPU.
The RTX 5070 Ti figures are a controlled old-to-R4 comparison on the same test
setup. They are not pool-side hashrate, do not include network or fee-window
effects, and are not performance guarantees. Card model, clocks, voltage,
cooling, driver, and power configuration can change both MH/s and watts.

The RTX 5070 Ti comparison shows higher energy efficiency even though the R4
path also used more board power. A power limit is only a ceiling: raising it
does not by itself guarantee a hashrate increase.

## Downloads

| Platform | Asset |
|---|---|
| Windows | `noid-gpu-windows-1.3.0.zip` |
| Linux | `noid-gpu-linux-1.3.0.tar.gz` |
| HiveOS | `noid-gpu-hiveos-1.3.0.tar.gz` |

Use only assets from the v1.3.0 release and verify them against
`SHA256SUMS.txt` before running them.

## Requirements

- 64-bit Windows or Linux.
- NVIDIA Ampere or newer GPU (`sm_80`, `sm_86`, `sm_89`, `sm_90`, or
  `sm_120`).
- NVIDIA driver 580 or newer.
- RTX 20, GTX 16, and GTX 10 series are not supported.
- The CUDA toolkit is not required on the mining rig.

The HiveOS package was built on the Ubuntu 20.04/Focal baseline with GLIBC
2.31. Its highest required GLIBC symbol is `GLIBC_2.30`. The final archive
passed extraction, layout, and permission checks; its miner executable is
byte-for-byte identical to the Linux release executable and passed startup
and self-test inside the Focal build environment.

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
Installation URL: URL of noid-gpu-hiveos-1.3.0.tar.gz
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
Get-FileHash .\noid-gpu-windows-1.3.0.zip -Algorithm SHA256
```

Compare the Windows result with the matching line in `SHA256SUMS.txt`.

## What changed internally

- Added the R4-specific field representation and optimized GPU path.
- Kept the common share-target path fast through 26 bits, including 24-bit
  targets.
- Retained independent CPU verification of GPU results before submission.
- Added correctness coverage for digest, target comparison, and nonce-search
  behavior across the optimized paths.
- Preserved multi-GPU operation and explicit device selection.

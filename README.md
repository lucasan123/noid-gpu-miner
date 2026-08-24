# NOID-GPU 1.1.0

**Faster mining, less arithmetic per hash, and measured board power well
below the configured limits.**

NOID-GPU is an NVIDIA GPU miner for the Parano1d (NOID) Poseidon2b proof of
work. Version 1.1.0 adds the optimized CUDA path, a compact English interface,
a reduced 7.5% public miner fee, and separate Windows, Linux, and HiveOS
packages.

## Release highlights

- **RTX 5070 Ti:** controlled wall-rate increased from 34.566 to
  **41.130 MH/s** (**+18.99%**).
- **RTX 4090:** **70.492 MH/s at 280.14 W**, leaving 169.86 W of headroom
  below the configured 450 W limit.
- **RTX 5090:** **104.526 MH/s at 453.69 W**, leaving 46.31 W of headroom
  below the configured 500 W limit.
- Public miner developer fee reduced from 10% to **7.5%**.
- Parano1d Pool fee changed from 10% to **7.5%**.
- Normal startup is now a compact NOID-GPU-WORKER panel, with English
  options, status messages, warnings, and errors.

## Performance and power

### Stabilized RTX 4090 and RTX 5090 measurements

| GPU | NOID-GPU wall rate | Measured board power | Configured power limit | Unused headroom | Measured efficiency |
|---|---:|---:|---:|---:|---:|
| RTX 4090 | **70.492 MH/s** | **280.14 W** | 450 W | **169.86 W (37.75%)** | **0.25163 MH/s/W** |
| RTX 5090 | **104.526 MH/s** | **453.69 W** | 500 W | **46.31 W (9.26%)** | **0.23039 MH/s/W** |

The table uses the longer stabilized run so that board power and temperature
had time to settle. The normal 16,777,216-nonce batch measured 70.314 MH/s on
the RTX 4090 and 104.298 MH/s on the RTX 5090. A short live pool dry-run,
which deliberately submitted no shares, measured 70.23 and 103.78 MH/s.

Measurements came from one dedicated card of each model. Board, clock,
voltage, driver, cooling, and temperature can change both MH/s and watts, so
these figures are measurements rather than guarantees.

### RTX 5070 Ti improvement

A controlled B-A-B comparison on the same RTX 5070 Ti measured:

| Build | Median kernel rate | Median wall rate |
|---|---:|---:|
| Previous frozen kernel | 34.679 MH/s | 34.566 MH/s |
| NOID-GPU 1.1.0 | **41.290 MH/s** | **41.130 MH/s** |
| Improvement | **+19.06%** | **+18.99%** |

Power was not resampled during that controlled B-A-B, so this release does
not claim a measured 5070 Ti watt reduction versus the previous kernel.

In a separate Windows field run, three RTX 5070 Ti cards delivered about
**120 MH/s combined** while instantaneous NVIDIA-SMI readings showed roughly
**132-180 W per card** against configured 300 W limits. This confirms that
the cards did not need to reach their power limits to deliver the observed
hashrate. Those snapshots are a field observation, not a controlled
before/after energy benchmark.

## Downloads

| Platform | Public asset | Compatibility |
|---|---|---|
| Windows | noid-gpu-windows-1.1.0.zip | Windows x86-64; executable noid-gpu.exe |
| Linux | noid-gpu-linux-1.1.0.tar.gz | Linux x86-64; Ubuntu 20.04/Focal-compatible |
| HiveOS | noid-gpu-hiveos-1.1.0.tar.gz | HiveOS Focal/Ubuntu 20.04 custom miner |

Verify every downloaded asset against SHA256SUMS.txt.

## Quick start

    noid-gpu --pool parano1d --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1 --gpu
    noid-gpu --pool ariapool --coinbase o1YOUR_PUBLIC_ADDRESS --worker rig1 --gpu

Use only your public o1... payout address. Mining never needs a seed phrase,
private key, or wallet file.

The public build supports its listed built-in pools and does not accept an
arbitrary --node destination.

## What changed in the miner

- Reduced the hot Poseidon2b path from 502 to **448 GF(2^128)
  multiplications per permutation** (**-10.76% arithmetic work**).
- Added a lazy most-significant-bit-first target comparison that avoids full
  digest conversion on the normal reject path.
- Added factorized full and partial MDS circuits and a shorter
  Karatsuba/clmad.lo dependency chain.
- Added a full-grid, one-nonce-per-thread kernel for pool work and the
  deterministic offline benchmark.
- Preserved CPU verification of every nonce returned by CUDA.
- Replaced the verbose normal startup with a compact English status panel.

Run the real offline GPU benchmark with:

    noid-gpu --benchmark-gpu --devices 0

## Power limits and tuning

A power limit is a ceiling, not a consumption target. Raising it does not
force a GPU to draw more power and will not increase hashrate when the card is
already below the cap.

Tune core clock and power one change at a time. Keep a setting only when the
MH/s gain remains stable without verification errors, rejected shares,
instability, or thermal throttling. Judge settings by both MH/s and MH/s/W.

## Miner developer fee

The public miner fee is a **7.5% scheduled mining-time window**: 45 seconds in
every 600-second cycle. Switching occurs only between jobs, the full fee
address is printed at startup, and the miner never changes a block coinbase.
The selected pool charges its own separate fee.

## Compatibility and verification

- NVIDIA Ampere or newer: sm_80, sm_86, sm_89, sm_90, or sm_120.
- NVIDIA driver 580 or newer.
- CUDA runtime included; the CUDA toolkit is not required on the rig.
- Linux and HiveOS binary built with a maximum required symbol of
  GLIBC_2.30, compatible with Ubuntu 20.04/HiveOS GLIBC 2.31.
- 113 miner/worker and Poseidon2b tests passed with zero failures
  (90 miner/worker and 23 arithmetic tests).
- Exact release archives passed CPU/GPU digest, target comparison, search
  path, package identity, and checksum gates.

## Parano1d Pool fee

The Parano1d Pool fee is now **7.5%**, reduced from 10%. The miner developer
fee is separate and is also 7.5% in this release.

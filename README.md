# kobold-miner

[![release](https://img.shields.io/github/v/release/kobold-dev/kobold-miner?label=release)](https://github.com/kobold-dev/kobold-miner/releases/latest)
[![site](https://img.shields.io/badge/site-kobold--miner.dev-7c5cff)](https://kobold-miner.dev/)
[![HiveOS](https://img.shields.io/badge/HiveOS-supported-f5a623)](https://github.com/kobold-dev/kobold-miner#-get-mining)
[![bitcointalk](https://img.shields.io/badge/bitcointalk-ANN-f7931a?logo=bitcoin&logoColor=white)](https://bitcointalk.org/index.php?topic=5599870.0)
[![Announcements](https://img.shields.io/badge/Announcements-Telegram-26A5E4?logo=telegram&logoColor=white)](https://t.me/kobold_updates)
[![license](https://img.shields.io/badge/license-proprietary-lightgrey)](https://github.com/kobold-dev/kobold-miner/blob/main/LICENSE)

<img src="https://github.com/kobold-dev/kobold-miner/raw/main/logo.png" width="128" alt="kobold-miner — NVIDIA GPU miner for Pearl (PRL), Ravencoin (RVN, KawPow), Parano1d (NOID) and Quantus (QTC), and x86 CPU miner for Monero (XMR) and Zephyr (ZEPH), RandomX">

A no-frills **multi-algorithm** miner for NVIDIA GPUs and x86 CPUs, with GPU kernels
written for each generation from Pascal (GTX 10xx) to Blackwell (RTX 50xx, B200 / B300).

Supported algorithms:

| Algorithm      | Coin                        | Device |
|----------------|-----------------------------|--------|
| **pearlhash**  | Pearl (PRL)                 | GPU    |
| **kawpow**     | Ravencoin (RVN)             | GPU    |
| **poseidon2b** | Parano1d (NOID)             | GPU    |
| **qpow**       | Quantus (QTC)               | GPU    |
| **randomx**    | Monero (XMR), Zephyr (ZEPH) | CPU    |

A GPU algorithm and RandomX can run together in the same process.

Official builds for Windows, Linux and HiveOS, every binary GPG-signed.

**[⬇ Grab the latest release](https://github.com/kobold-dev/kobold-miner/releases)** &nbsp;|&nbsp; [All options explained](OPTIONS.md) &nbsp;|&nbsp; [kobold-miner.dev](https://kobold-miner.dev/)

## 💸 Dev fee

| Algorithm                        | Dev fee |
|----------------------------------|---------|
| pearlhash — Pearl (PRL)          | 2%      |
| kawpow — Ravencoin (RVN)         | 2%      |
| poseidon2b — Parano1d (NOID)     | 2%      |
| qpow — Quantus (QTC)             | 2%      |
| randomx — Monero (XMR) / Zephyr  | 2%      |

The fee is per-algorithm and is shown when the miner starts (e.g.
`Dev fee 2.00% enabled`). It is taken transparently while mining and keeps
development going.

## 🧩 Features

- **Multi-algorithm engine.** One code base drives every algorithm: Pearl, Ravencoin,
  Parano1d and Quantus on the GPU, Monero / Zephyr (RandomX) on the CPU.
- **Single self-contained binary.** The GPU kernels for every architecture are
  embedded and the right one is selected at launch — no CUDA install, no DLLs, no
  config files, just the driver.
- **Kernels per GPU generation.** Pascal, Volta, Turing, Ampere (RTX 30xx / RTX A, A100), Ada,
  Hopper and Blackwell (RTX 50xx / RTX PRO, B200, B300) each get code of their own.
- **GPU + CPU at once.** Mine a GPU coin and RandomX on the idle CPU cores in the same
  process, each with its own pool.
- **Resilient networking.** TLS auto-detected, several pools with ordered failover,
  automatic reconnect. Pool names are looked up over DNS over HTTPS first; `--dns-over-https strict`
  never sends a plaintext DNS query, and `--doh-url` picks your own DoH server.
- **Mining schedule.** `--schedule 08:00-20:00` (several windows with `/`, per GPU with
  `id,windows`) mines only inside the windows; outside them the GPU pauses and the pool
  connection stays. `--timezone +03:00` sets the clock it reads.
- **Liveness checks.** Reconnect when the pool goes quiet or rejects in a row
  (`--submit-timeout`, 30 s by default, `--job-timeout`, `--max-no-share-sent`,
  `--enable-restart-on-rejected`), and a GPU
  stall watchdog that reports, restarts the GPU or exits (`--watchdog`). The
  launchers relaunch the miner if it ever exits.
- **Hardware safety & tuning.** Core- and memory-temperature cutoffs (`--tstop`,
  `--mtstop`), core / memory clock locks, power limit, and a GDDR5X memory-timing
  tweak for GTX 1080 / 1080 Ti / Titan Xp on Linux (`--mem-tweak`).
- **RandomX done right.** AVX2 / AVX-512 paths, huge-page support, and automatic
  thread sizing that respects cgroup CPU quotas — correct on shared/rented boxes,
  not just bare metal.
- **Monitoring built in.** Local JSON + CCMiner-compatible telnet stats API, and a
  `--bench` mode that needs no pool.
- **Signed, private.** Every build is GPG-signed; no telemetry, no data collection.

## 🎮 Hardware

NVIDIA GPUs from Pascal (GeForce GTX 10xx, compute 6.1) to Blackwell (GeForce RTX 50xx, RTX PRO,
B200, B300), for every GPU algorithm.

- **Driver:** a CUDA 12 driver or newer. Tested on r535 and later; r525–r530 are supported through CUDA
  minor-version compatibility but untested. Blackwell needs the first driver that knows it: r570+ for
  RTX 50 / RTX PRO / B200, r575+ for B300. A CUDA 13 driver works too (on Pascal and Volta, r580 is the
  last branch).
- **Ravencoin (KawPow)** needs room for the DAG: from RVN epoch 615 it no longer fits a 6 GB card,
  in any miner.

CPU side (Monero / Zephyr): any x86-64 CPU; AVX2 / AVX-512 are used when present.
Builds ship for **Windows x64** and **Linux x64**.

## 📊 Performance

| Algorithm | Coin | Device | Unit |
|-----------|------|--------|------|
| pearlhash | Pearl (PRL) | GPU | TH/s |
| kawpow | Ravencoin (RVN) | GPU | MH/s |
| poseidon2b | Parano1d (NOID) | GPU | MH/s |
| qpow | Quantus (QTC) | GPU | MH/s |
| randomx | Monero (XMR), Zephyr (ZEPH) | CPU | kH/s |

Every number below is kobold-miner's own rate, measured live against a mining pool at **stock clocks
(no overclock)**, with the pool accepting its shares. Most cards were rented: each ran at its host's
power limit, which is sometimes below the card's default. One row per card, latest measurement;
`—` = not measured yet, `~` = a reading from a run that was cut short.

### GPU

<!-- PERF-TABLE:BEGIN -->

| GPU | **Pearl**<br><sub>PRL · TH/s</sub> | **Ravencoin**<br><sub>RVN · MH/s</sub> | **Parano1d**<br><sub>NOID · MH/s</sub> | **Quantus**<br><sub>QTC · MH/s</sub> |
|:----|-------:|-------:|-------:|-------:|
| **Blackwell — GeForce RTX 50xx / RTX PRO** | | | | |
| RTX PRO 6000 WS 96G | 488.43 | 102.61 | 290.78 | 2082 |
| RTX PRO 6000 Server 96G | 440.00 | 95.67 | 278.24 | 1842 |
| RTX 5090 32G | 405.37 | 105.27 | 266.05 | 1910 |
| RTX PRO 6000 Max-Q 96G | 319.49 | 78.07 | 202.18 | 1406 |
| RTX PRO 5000 48G | 250.65 | 76.53 | 145.13 | 1132 |
| RTX 5080 16G | 230.14 | 56.28 | 141.90 | 1011 |
| RTX 5070 Ti 16G | 192.15 | 52.31 | 120.37 | 873 |
| RTX PRO 4500 32G | 184.31 | 50.84 | 119.51 | 833 |
| RTX PRO 6000D 84G | 144.17 | 83.45 | 231.77 | — |
| RTX 5070 12G | 132.17 | 39.07 | 83.50 | 610 |
| RTX PRO 4000 24G | 123.48 | 34.26 | 79.79 | 566 |
| RTX 5060 Ti 16G | 98.55 | 26.03 | 59.66 | 435 |
| RTX 5060 8G | 83.16 | 21.97 | 49.47 | 363 |
| **Blackwell datacenter — B200 / B300** | | | | |
| B200 180G | 1423.89 | 188.74 | 733.50 | 1240 |
| B300 288G | 823.27 | 192.23 | 177.09 | 1190 |
| **Hopper — H100 / H200** | | | | |
| H100 SXM 80G | 755.86 | 154.27 | 660.75 | 1130 |
| H200 141G | 720.43 | 155.41 | 660.17 | 1130 |
| H200 NVL 141G | 679.83 | 147.66 | 593.86 | 1010 |
| H100 NVL 94G | 545.26 | 110.99 | 569.13 | 1010 |
| H100 PCIe 80G | 477.86 | 95.80 | 497.37 | 866 |
| **Ada Lovelace — GeForce RTX 40xx / RTX Ada / L-series** | | | | |
| RTX 4090 24G | 330.56 | 59.02 | 200.87 | 1440 |
| L40S 48G | 314.74 | 45.22 | 164.62 | 1055 |
| RTX 6000 Ada 48G | 284.46 | 55.26 | 151.81 | 923 |
| RTX 5880 Ada 48G | 254.22 | 52.99 | 140.45 | 924 |
| RTX 5000 Ada 32G | 220.33 | 33.02 | 130.06 | 890 |
| RTX 4080 Super 16G | 211.67 | 43.26 | 129.56 | 902 |
| RTX 4080 16G | 204.07 | 41.48 | 128.78 | 877 |
| L40 48G | 175.71 | 45.22 | 148.91 | 938 |
| RTX 4070 Ti Super 16G | 172.87 | 39.16 | 106.73 | 768 |
| RTX 4070 Ti 12G | 160.49 | 29.46 | 100.30 | 711 |
| RTX 4500 Ada 24G | 151.47 | 24.67 | 89.94 | 631 |
| RTX 4070 Super 12G | 148.47 | 29.44 | 88.93 | 622 |
| RTX 4070 12G | 123.73 | 29.31 | 75.44 | 507 |
| RTX 4000 Ada 20G | 101.50 | 20.54 | 59.31 | 416 |
| RTX 4060 Ti 16G | 93.92 | 16.74 | 56.37 | 410 |
| L4 24G | 88.54 | 15.50 | 51.81 | 348 |
| RTX 4060 8G | 62.10 | 15.75 | 42.03 | 276 |
| RTX 2000 Ada 16G | 51.34 | 12.92 | 30.87 | 215 |
| **Ampere — GeForce RTX 30xx / RTX A-series / A10** | | | | |
| RTX 3090 Ti 24G | 156.48 | 59.83 | 97.20 | 715 |
| RTX 3090 24G | 140.09 | 51.81 | 81.01 | 550 |
| A40 48G | 134.82 | 41.31 | 83.24 | 595 |
| RTX 3080 Ti 12G | 131.96 | 52.91 | 83.83 | 590 |
| RTX A6000 48G | 124.73 | 45.68 | 82.28 | 598 |
| RTX 3080 10G | 121.01 | 44.08 | 68.41 | 496 |
| RTX A5000 24G | 94.43 | 34.64 | 59.00 | 406 |
| RTX 3070 Ti 8G | 90.23 | 35.13 | 53.96 | 406 |
| A10 24G | 81.52 | 29.35 | 56.08 | 401 |
| RTX 3070 8G | 80.34 | 25.77 | 49.78 | 363 |
| RTX 3060 Ti 8G | 68.39 | 25.87 | 40.21 | 313 |
| RTX A4000 16G | 65.85 | 23.01 | 41.70 | 293 |
| RTX 3060 12G | 50.96 | 20.62 | 32.65 | 239 |
| RTX A2000 12G | 30.64 | 12.46 | 21.76 | 157 |
| RTX A2000 6G | 28.85 | n/a | 21.43 | 163 |
| **Ampere datacenter — A100** | | | | |
| A100 SXM4 40G | 254.15 | 84.94 | 373.86 | 648 |
| A100 PCIe 40G | 210.51 | 71.93 | 333.26 | 598 |
| CMP 170HX 8G | 160.64 | 56.46 | 243.93 | 421 |
| **Turing — GeForce RTX 20xx / GTX 16xx / Quadro RTX / Tesla T4** | | | | |
| RTX 2080 Ti 22G | 107.47 | 35.57 | 24.52 | 511 |
| Titan RTX 24G | 101.06 | 37.02 | 26.91 | 491 |
| Quadro RTX 6000 24G | 94.76 | 37.04 | 25.91 | 470 |
| Quadro RTX 8000 48G | 91.74 | 28.93 | 27.06 | 490 |
| RTX 2080 8G | 81.52 | 26.09 | 20.93 | 390 |
| Quadro RTX 5000 16G | 73.07 | 24.68 | 19.63 | 360 |
| RTX 2060 Super 8G | 52.23 | 19.59 | 13.71 | 249 |
| RTX 2060 12G | 51.82 | 18.70 | 13.47 | 246 |
| RTX 2070 8G | 48.74 | 19.26 | 13.79 | 249 |
| RTX 2070 Super 8G | 42.37 | 25.48 | 17.36 | 320 |
| Tesla T4 16G | 33.63 | 10.08 | 10.18 | 174 |
| GTX 1660 Ti 6G | 6.34 | — | 9.99 | 187 |
| GTX 1660 6G | 5.69 | — | — | — |
| GTX 1660 Super 6G | 5.64 | — | 9.23 | 170 |
| GTX 1650 Super 4G | 5.21 | — | — | — |
| **Volta — Tesla V100** | | | | |
| Tesla V100 16G | 40.93 | 53.17 | 26.82 | 537 |
| Tesla V100 32G | 39.15 | 50.16 | 26.86 | 508 |
| **Pascal — GeForce GTX 10xx / Tesla P40 / Quadro P4000** | | | | |
| GTX 1080 Ti 11G | 19.08 | 22.30 | 13.31 | 140 |
| Tesla P40 24G | 17.07 | 18.94 | 12.58 | 129 |
| GTX 1080 8G | 12.26 | 15.14 | 9.64 | 93 |
| GTX 1070 8G | 10.58 | 12.93 | 6.91 | 68 |
| Quadro P4000 8G | 8.42 | 11.34 | — | — |
| GTX 1060 3G | 6.93 | — | 4.76 | 48 |
| GTX 1060 6G | 5.57 | 9.07 | 5.29 | 54 |

<sub>n/a — the Ravencoin DAG no longer fits the card's memory (6 GB cards, RVN epoch 615 on).</sub>
<!-- PERF-TABLE:END -->

### CPU

#### RandomX — Monero (XMR) / Zephyr (ZEPH)

| CPU | **RandomX**<br><sub>XMR / ZEPH · kH/s</sub> |
|:----|-------:|
| Ryzen 9 9950X3D (16C/32T, huge pages) | 16.60 |
| Ryzen 9 7950X (16C/32T) | 16.61 |
| EPYC 9334 | 7.40 |
| EPYC 7532 | 3.00 |
| 2× Xeon E5-2686 v4 | 2.87 |
| 2× Xeon E5-2682 v4 | 2.78 |
| EPYC 7551 (32C/64T) | 2.43 |
| Core i7-6700 (4C/8T) | 1.43 |

Except the 9950X3D, these are the CPUs of rented GPU servers: shared with the host,
limited by its container and not always given huge pages, so a dedicated box of the
same model is faster. RandomX speed depends on core count, RAM channels / speed and huge pages.

Your own numbers vary with silicon, cooling, driver and pool. Measure yours with
`km --bench 60 -a <algorithm>`.

## 🚀 Get mining

**Windows / Linux.** Unpack the archive, open the launcher for your coin
(`mine_pearl`, `mine_rvn`, `mine_noid`, `mine_qtc`, `mine_monero`, `mine_zephyr`, or the
combined `mine_pearl+xmr`), drop in your wallet, and start it — it relaunches itself if it
ever stops. Prefer the command line?

```
km -a pearlhash  -o prl.2miners.com:1818       -u YOUR_WALLET.rig0
km -a kawpow     -o rvn.2miners.com:6060       -u YOUR_WALLET.rig0
km -a poseidon2b -o noid.suprnova.cc:3337      -u YOUR_WALLET.rig0
km -a qpow       -o qtc.2miners.com:3636       -u YOUR_WALLET.rig0
km -a randomx    -o pool.supportxmr.com:9000   -u YOUR_WALLET
```

**HiveOS.** Add a **Custom** miner flight sheet:

| Field            | Value                                                                 |
|------------------|-----------------------------------------------------------------------|
| Installation URL | `https://github.com/kobold-dev/kobold-miner/releases/download/v1.0.1/kobold-miner-1.0.1_hiveos.tar.gz` |
| Miner            | Custom → `kobold-miner`                                                |
| Hash algorithm   | `pearlhash`, `kawpow`, `poseidon2b`, `quantus` or `randomx` (required)     |
| Wallet           | your address for that coin                                             |
| Pool URL         | the pool's stratum host and port, e.g. `prl.2miners.com:1818`          |
| Template         | `%WAL%.%WORKER_NAME%`                                                  |
| Pass             | leave blank (defaults to `x`)                                          |

Apply it and HiveOS pulls, installs and starts the miner. If your HiveOS does not offer the
algorithm in the *Hash algorithm* list, leave the field empty and put `-a <algorithm>` (e.g.
`-a pearlhash`) in *Setup Miner Config* instead; aliases such as `pearl`, `qtc` or `noid` work too.
With neither set the flight sheet is refused with the list of algorithms this build mines. For Quantus
pick `quantus`: HiveOS files the ticker QTC under another coin's algorithm (`qhash`).

**Also mine Monero on the CPU.** HiveOS has no dual field for custom miners, so add the
second coin in *Setup Miner Config*:

```
--xmr-pool pool.supportxmr.com:9000 --xmr-user YOUR_XMR_ADDRESS
```

The GPU coin keeps running while RandomX works the idle CPU cores.

## 🌊 Pools

Any standard pool works, with your wallet address as the login.

**GPU**

| Coin            | Pool                                 | Dashboard                     |
|-----------------|--------------------------------------|-------------------------------|
| Pearl (PRL)     | [2Miners](https://2miners.com)       | https://prl.2miners.com/      |
| Pearl (PRL)     | [HeroMiners](https://herominers.com) | https://pearl.herominers.com/ |
| Pearl (PRL)     | [K1Pool](https://k1pool.com)         | https://k1pool.com/pool/pearl |
| Pearl (PRL)     | [Kryptex](https://pool.kryptex.com)  | https://pool.kryptex.com/prl  |
| Ravencoin (RVN) | [2Miners](https://2miners.com)       | https://rvn.2miners.com/      |
| Parano1d (NOID) | [Suprnova](https://suprnova.cc)      | https://noid.suprnova.cc/     |
| Quantus (QTC)   | [2Miners](https://2miners.com)       | https://qtc.2miners.com/      |
| Quantus (QTC)   | [Suprnova](https://suprnova.cc)      | https://quantus.suprnova.cc/  |

**RandomX — CPU (Monero & Zephyr)**

| Pool                                 | Monero (XMR)                     | Zephyr (ZEPH)                   |
|--------------------------------------|----------------------------------|---------------------------------|
| [2Miners](https://2miners.com)       | —                                | https://zeph.2miners.com/       |
| [HeroMiners](https://herominers.com) | https://monero.herominers.com/   | https://zephyr.herominers.com/  |
| [K1Pool](https://k1pool.com)         | —                                | https://k1pool.com/pool/zeph    |
| [Kryptex](https://pool.kryptex.com)  | https://pool.kryptex.com/xmr     | https://pool.kryptex.com/zeph   |
| [SupportXMR](https://supportxmr.com) | https://supportxmr.com/          | —                               |

Each dashboard lists the stratum host, ports and regional servers to use. You can
pass several hosts, comma-separated, for failover.

## 🔌 Connecting

A pool URL may carry a scheme, or none at all:

- bare `host:port` or `stratum+auto://` — try TLS, fall back to plain (the default)
- `stratum+ssl://` — always TLS
- `stratum+tcp://` — always plaintext

Credentials go either inline or as separate flags:

```
-o stratum+auto://host:port -u WALLET.WORKER
-o host:port -u WALLET -w WORKER -p x
```

`km --help` lists everything. **Full option reference, every option explained: [OPTIONS.md](https://github.com/kobold-dev/kobold-miner/blob/main/OPTIONS.md).**

## 🔐 Verifying a download

The signing key is `Kobold Miner <releases@kobold-miner.dev>`
(`884C 54E5 3D72 177F 79C4  0B68 B29D A194 49E2 7AFC`). Check a build with:

```
gpg --import public.gpg
gpg --verify km.exe.sig km.exe     # Windows
gpg --verify km.sig km             # Linux
```

A "Good signature" line is what you want to see.

## 🛠 Troubleshooting

**A "GPU kernel load failed" / "GPU driver too old" error, or the GPU isn't used.** Update the
NVIDIA driver: a CUDA 12 driver or newer (tested on r535 and later; r570+ on RTX 50 / RTX PRO / B200,
r575+ on B300), and the miner loads its kernels in the form your driver accepts. List what the miner sees with `km -a pearlhash -d LIST`.

**Ravencoin exits on a 6 GB card.** The KawPow DAG has outgrown 6 GB (RVN epoch 615); the miner
says how much memory it needs and stops instead of mining with a broken DAG.

**Pool rejects every share (“low difficulty” / invalid).** The miner re-checks every share on
the CPU before sending it, so the pool is hashing it differently: make sure `-a` is the algorithm
that pool port mines. If it is, that is our bug — open an issue with the log. A few stale rejects
right after a new job are normal.

**GPU skipped or out of memory.** The miner sizes its work to the card's memory, but VRAM
already taken by a game or another program can leave too little — close it, or pick another
card with `-d`.

## 💬 Help & license

Site [kobold-miner.dev](https://kobold-miner.dev/); release announcements in the Telegram channel
[Kobold Miner Updates — @kobold_updates](https://t.me/kobold_updates); questions and reports go to the
[issue tracker](https://github.com/kobold-dev/kobold-miner/issues). Licensed under the terms in [LICENSE](https://github.com/kobold-dev/kobold-miner/blob/main/LICENSE);
© 2026 kobold-miner.

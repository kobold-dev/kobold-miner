# kobold-miner — all options

Every command-line option of `km`, what it is for, and its default. `km --help` prints the same list.
Back to the [README](README.md).

Options that take a value per GPU accept one value for every card (`--tstop 80`) or `id,value` pairs
(`--tstop 0,80,2,75`), where `id` is the card number from `km -d LIST`.

### General

| Option | What it is for | Default / example |
|---|---|---|
| `-a`, `--algo` | The algorithm to mine: `pearlhash`, `kawpow`, `poseidon2b`, `qpow`, `randomx`, or an alias (`prl`, `rvn`, `noid`, `qtc`, `xmr`, `zeph`) | required |
| `-d`, `--devices` | Which GPUs to use; `-d LIST` prints the cards the miner sees and exits | all GPUs; `-d 0,2` |
| `--bench N` | Measure the hashrate for N seconds without a pool, then exit | `--bench 60 -a pearlhash` |
| `--list-algos` | Print the algorithms this build mines and exit | |
| `-V`, `--version` | Print the version and exit | |
| `-h`, `--help` | Print every option and exit | |
| `--log-file FILE` | Also write the console output to a file (appends) | off |
| `--report-interval S` | How often the hashrate table is printed, in seconds | 10 |
| `-m`, `--no-color` | Plain console output without colours | colours on |
| `-v`, `--protocol-dump` | Print every message exchanged with the pool, for debugging a pool problem | off |
| `--trace`, `--debug` | Very detailed diagnostic log, for a bug report | off |
| `--grant-lockpages` | Windows: give your account the right to use large memory pages (RandomX runs faster with them), then exit; needs admin and a log-off | |

### Pool

| Option | What it is for | Default / example |
|---|---|---|
| `-o`, `-P`, `--pool`, `--url` | The pool address; repeat it, or list several comma-separated, for failover in order | `-o prl.2miners.com:1818` |
| `-u`, `--user` | Your wallet address (the pool login) | `-u YOUR_WALLET` |
| `-w`, `--worker` | The rig name shown on the pool's dashboard | `-w rig0` |
| `-p`, `--pass` | The pool password; most pools ignore it | `x` |

A pool address may start with `stratum+ssl://` (always TLS), `stratum+tcp://` (never TLS) or nothing
(TLS is tried first, plain TCP if the pool does not speak it). The login can also go into the address:
`-o stratum+tcp://WALLET.rig0@prl.2miners.com:1818`.

### Connection

| Option | What it is for | Default / example |
|---|---|---|
| `--tls auto\|on\|off` | TLS for an address without a scheme | `auto` |
| `--strict-ssl` | Refuse a TLS pool whose certificate is not signed by a trusted authority | off: such a pool is used, with a warning |
| `--no-strict-ssl` | Do not check the pool's certificate at all | off |
| `--tls-fingerprint HEX` | Connect only to a pool with exactly this certificate (its SHA-256, 64 hex digits) — for a pool with a self-signed certificate | off |
| `--no-sni` | Do not send the server name in the TLS handshake (for a pool behind an odd proxy) | off |
| `--dns-over-https on\|off\|strict` | How the pool's name is turned into an address: `on` = DNS over HTTPS first, then the normal resolver; `off` = the normal resolver only; `strict` = never a plaintext DNS query (for a network that tampers with DNS) | `on` |
| `--doh-url URL` | Your own DNS over HTTPS server(s) instead of 1.1.1.1 and 8.8.8.8; repeat for several | `--doh-url https://dns.quad9.net/dns-query` |
| `-4`, `--inet4-only` / `-6`, `--inet6-only` | Connect over IPv4 only / IPv6 only | both |
| `--prefer-family IPv4\|IPv6\|none` | Which address family is tried first when the pool has both | `IPv4` |
| `--reconnect-delay MS` | Pause before reconnecting after a dropped or failed connection, in milliseconds | 5000 |
| `--submit-timeout S` | Reconnect if the pool has not answered a share for this long | 30 |
| `--keepalive-timeout S` | Reconnect if the pool has sent nothing at all for this long | 30 |
| `--keepalive-interval S` | How often the system checks that the connection is still alive | 15 |
| `--job-timeout S` | Reconnect if the pool has sent no new job for this long | off |
| `--max-no-share-sent S` | Reconnect if no share has been accepted for this long | off |
| `--max-no-submit-responses N` | Reconnect after N shares in a row got no answer | 8 |
| `--enable-restart-on-rejected` | Reconnect after a run of rejected shares (see the next option) | off |
| `--max-rejected-shares N` | How many rejects in a row trigger that reconnect | 15 |

### GPU

| Option | What it is for | Default / example |
|---|---|---|
| `--tstop C` | Pause a GPU when its core reaches this temperature | off; `--tstop 80` |
| `--tstart C` | Resume it when it has cooled to this temperature | `tstop − 5` |
| `--mtstop C` / `--mtstart C` | The same for the memory temperature | off |
| `--schedule HH:MM-HH:MM` | Mine only inside these time windows; outside them the GPU pauses and stays connected to the pool. Several windows with `/`, past midnight allowed, per GPU with `id,windows` | `--schedule 08:00-20:00/22:00-06:00` |
| `--timezone ±HH:MM` | The clock `--schedule` reads | the system's local time |
| `--watchdog off\|notify\|restart-gpu\|exit` | What to do when a GPU stops producing hashes: only warn, restart that GPU inside the miner, or exit so the launcher / HiveOS restarts the miner | `off` |
| `--watchdog-timeout S` | How long a GPU may stall before the watchdog acts | 60 |

### GPU tuning (needs admin / root)

| Option | What it is for | Default / example |
|---|---|---|
| `--power-limit W` | Set the GPU power limit in watts | unchanged; `--power-limit 220` |
| `--lock-cclock MHz` | Lock the GPU core clock | unchanged |
| `--lock-mclock MHz` | Lock the GPU memory clock | unchanged |
| `--mem-tweak 1\|2` | GDDR5X memory timings for GTX 1080 / 1080 Ti / Titan Xp, Linux only (needs the kernel booted with `iomem=relaxed`); `1` safe, `2` fast; helps Ravencoin | off |

Settings changed by these options are restored when the miner exits.

### Monero on the CPU (RandomX)

| Option | What it is for | Default / example |
|---|---|---|
| `--xmr-pool HOST:PORT` | Also mine Monero on the CPU while the GPUs mine another coin, at this pool | off; `--xmr-pool pool.supportxmr.com:9000` |
| `--xmr-user`, `--xmr-wallet` | Your Monero address for that | |
| `--xmr-pass` | That pool's password | `x` |
| `--cpu-threads N` | How many CPU threads mine RandomX (lower it to keep the PC usable) | all logical cores |
| `--cpu-priority 1-5` | Priority of the mining threads, 1 = idle … 5 = highest | 4 |
| `--cpu-no-affinity` | Do not pin each thread to its own core | pinned |
| `--cpu-no-large-pages` | Do not use large memory pages | used when available |
| `--cpu-1gb-pages` | Try 1 GB pages for the dataset (needs large pages and a free 1 GB chunk of memory) | off |
| `--cpu-no-numa` | One shared dataset on a multi-socket server (by default each socket gets its own copy, which is faster) | per socket |

### Stats API

| Option | What it is for | Default / example |
|---|---|---|
| `--enable-api` | Serve the miner's stats on `127.0.0.1:4068`: JSON over HTTP (`GET /`) and CCMiner-compatible telnet (`summary`, `threads`, `pool`) on the same port | off |
| `-b`, `--api-bind IP:PORT` | Serve them on another address (turns the API on) | `-b 0.0.0.0:4068` |
| `-J`, `--json-response` | Telnet answers in JSON instead of CCMiner text | off |

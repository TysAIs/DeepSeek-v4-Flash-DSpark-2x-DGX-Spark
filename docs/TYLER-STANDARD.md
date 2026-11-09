# TysAIs fork standard (additions over MiaAI-Lab upstream)

This fork = Mia's DeepSeek-v4-Flash DSpark recipe **plus** Tyler's non-negotiable stack:

## The three standards
1. **Obliterated weights** — `drowzeys/keys-DeepSeekV4-Flash-GA-0731-Dspark-Abliterated-32-32`, HF revision pinned by `DSPARK_REVISION_ABLITERATED` (currently `6c47916f`). Never ship raw weights.
2. **vLLM GB10 spin-wait fix** — now BAKED INTO UPSTREAM (`patches/hotfix-gb10-spin-wait.sh`, applied automatically at container start; sets `busy_loop_s: 1 → 0.002`). Verify live: `docker exec <ds4-container> grep busy_loop_s .../shm_broadcast.py | head -1` must show `0.002`.
3. **GPU clock cap (~2200 MHz)** — NOT in upstream; this is OURS. Applied on BOTH ranks:
   `sudo nvidia-smi -lgc 0,2200` (persisted via gb10-clock-cap systemd unit).
   Trust `clocks.current.graphics` under load (~2177–2190), never `clocks.applications.graphics`
   (frozen at 2418 — known false positive on GB10).

## Sync policy (upstream → fork)
Upstream merges land weekly. When syncing:
1. `git fetch origin && git merge origin/main`
2. Resolve conflicts: take THEIRS everywhere EXCEPT `results/*` (our redacted benches — always OURS)
3. Privacy scan the merged tree (private IPs / sk- keys / GOCSPX-) before commit — public repo
4. Validate: `bash scripts/ci-validate.sh` (run on a Spark; macOS bash lacks mapfile) — must pass
5. Push to fork; then re-sync `.env.dspark` + compose onto node1 if recipe env changed,
   and re-run the validate-dspark-config.sh drift gate

## Known topology (08-25)
TP2 = node1 (rank0, publishes :8888) + node2 (rank1 worker). The third node was sold.

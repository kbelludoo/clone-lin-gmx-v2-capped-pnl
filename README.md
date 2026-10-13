# clone-lin-gmx-v2-capped-pnl

Experimental i64-scale LIN clone of GMX V2 `MarketUtils.getCappedPnl` / `Precision.applyFactor` / `Keys.MAX_PNL_FACTOR_FOR_TRADERS` and the trader pro-rata scale in `PositionUtils.getPositionPnlUsd`.

**Proofs and evidence live in [kbelludoo/lin-open](https://github.com/kbelludoo/lin-open)** (`src/lin_gmx_v2_capped_pnl.lin`, `test/prove_gmx_v2_capped_pnl_external.py`, `examples/gmx_v2_capped_pnl/`). This repository is the separate LIN copy only.

## Kernel (EXPERIMENTAL, not mainnet uint256, not FLOAT=1e30)

```
FLOAT' = 1e6
maxPnl = poolUsd * factor / FLOAT'
getCappedPnl = pnl < 0 ? pnl : min(pnl, maxPnl)
trader = (capped != pool && capped > 0 && pool > 0)
       ? trader * capped / pool
       : trader
```

Named contrasts (recomputed in lin-open against C11 `__int128`, limb muldiv, LIN Compiler 0 VM, Python bigint, and libtcc `roundtrip-jit`):

| Path | Result |
|---|---|
| pool 2e8, factor 50% | `maxPnl = 1e8` |
| pool profit `1.5e8` | capped `1e8` |
| pool profit `5e7` | stays `5e7` |
| pool loss `-2e8` | stays `-2e8` |
| wrong symmetric cap of that loss | `-1e8` |
| trader `1.5e8` of pool `3e8` | pro-rata `5e7` |
| wrong `min(trader, maxPnl)` | `1e8` |
| factor `0` | pool cap `0`; trader stays `1.5e8` |
| naive i64 `7e9*2e9/1e6` | wraps; 128-bit `14000000000000` |

This is **not** uncapped `getPositionPnlUsd` / `pickPriceForPnl` (see clone-lin-gmx-v2-pnl). DataStore `getMaxPnlFactor` is an argument.

## Upstream

- https://github.com/gmx-io/gmx-synthetics `a85ea3491c19c93bb4b5a002d9b358fb769b7849`
- `contracts/market/MarketUtils.sol` blob `ad7a0b2b5a516ec86ce0afed89ff84ecba11038c`
- `contracts/utils/Precision.sol` blob `047143722beaf6adceac3fc2ef1817f054dfe00a`
- `contracts/data/Keys.sol` blob `5472e5b4ecd29bb372218caa142476eb95fe55f8`
- License: GPL-2.0-or-later after BUSL Change Date 2026-08-31

## Files

- `src/lin_gmx_v2_capped_pnl.lin` — LIN kernel (`g2c_*`)
- `gmx_v2_capped_pnl_c11.c` — independent C11 oracle (selftest)

# Uniswap V4 Progressive Disclosure Skills

Progressive disclosure skills for developing with Uniswap V4, the singleton AMM with hooks. These skills cover the PoolManager architecture, hook system, flash accounting, and advanced customization patterns.

## Installation

### Claude Code

Add as a submodule to your plugins directory:

```bash
git submodule add https://github.com/cyotee/uniswap-V4-skill.git plugins/uniswap-v4
```

Or clone directly:

```bash
git clone https://github.com/cyotee/uniswap-V4-skill.git
```

### OpenCode

Copy the `.opencode/skills/` directory contents to your local OpenCode skills directory.

## Skills

| Skill | Trigger Keywords | Description |
|-------|-----------------|-------------|
| `uniswap-v4-architecture` | V4 overview, singleton, V3 vs V4 | High-level architecture and design philosophy |
| `uniswap-v4-pool-manager` | PoolManager, unlock, unlockCallback | Core singleton contract and unlock pattern |
| `uniswap-v4-hooks` | hooks, beforeSwap, afterSwap, permissions | Hook system, callbacks, and address-encoded permissions |
| `uniswap-v4-flash-accounting` | flash accounting, settle, take, transient | Deferred settlement and transient storage |
| `uniswap-v4-pools` | PoolKey, PoolId, Slot0, pool state | Pool identification and state management |
| `uniswap-v4-swaps` | swap, BeforeSwapDelta, exactIn, exactOut | Swap execution and hook modifications |
| `uniswap-v4-liquidity` | liquidity, modifyLiquidity, ERC6909, positions | Position management and LP tokens |
| `uniswap-v4-fees` | fees, dynamic fees, protocol fees, LPFee | Fee system including hook-driven dynamic fees |

## Key Concepts

### Singleton Architecture
Unlike V3's per-pair pool contracts, V4 uses a single PoolManager contract that manages all pools. Pools are identified by PoolKey (token pair + fee + tickSpacing + hooks) hashed to a PoolId.

### Hooks System
V4 introduces hooks—external contracts that receive callbacks at key lifecycle points (initialize, swap, liquidity changes, donate). Hook permissions are encoded in the contract's deployment address, enabling gas-efficient permission checks.

### Flash Accounting
All operations occur within an `unlock()` callback. Token transfers are deferred—the PoolManager tracks cumulative deltas per currency in transient storage, and users must settle all debts before the callback returns.

### Transient Storage (EIP-1153)
V4 leverages `tstore`/`tload` for temporary state that's erased between transactions: lock state, currency deltas, reserves, and delta counts.

### ERC6909 LP Tokens
Positions are represented as ERC6909 multi-token balances rather than ERC721 NFTs, enabling more efficient transfers and composability.

### Dynamic Fees
Hooks can implement dynamic fee logic, returning custom fees via `beforeSwap` or calling `updateDynamicLPFee()` to change fees on-the-fly.

## Repository Structure

### v4-core
```
src/
├── PoolManager.sol              # Singleton pool manager
├── interfaces/
│   ├── IPoolManager.sol         # Main interface
│   ├── IHooks.sol               # Hook callback interface
│   └── callback/
│       └── IUnlockCallback.sol  # Unlock callback interface
├── libraries/
│   ├── Pool.sol                 # Pool state machine
│   ├── Hooks.sol                # Hook permission encoding
│   ├── Position.sol             # Position tracking
│   ├── TickBitmap.sol           # Initialized tick tracking
│   ├── TransientStateLibrary.sol
│   ├── CurrencyDelta.sol        # Transient delta tracking
│   └── Lock.sol                 # Transient lock state
└── types/
    ├── PoolKey.sol              # Pool identification
    ├── PoolId.sol               # Hashed pool key
    ├── Currency.sol             # Token wrapper type
    ├── BalanceDelta.sol         # Packed amount deltas
    └── BeforeSwapDelta.sol      # Hook swap modification
```

### v4-periphery
```
src/
├── V4Router.sol                 # Swap routing
├── base/
│   ├── BaseHook.sol             # Hook base contract
│   ├── SafeCallback.sol         # Callback safety
│   └── ImmutableState.sol       # PoolManager reference
├── libraries/
│   └── Actions.sol              # Router action types
└── utils/
    └── HookMiner.sol            # Address mining for hooks
```

## Hook Permission Flags

Hook permissions are encoded in the least significant bits of the hook contract address:

| Bit | Flag | Callback |
|-----|------|----------|
| 13 | BEFORE_INITIALIZE | `beforeInitialize()` |
| 12 | AFTER_INITIALIZE | `afterInitialize()` |
| 11 | BEFORE_ADD_LIQUIDITY | `beforeAddLiquidity()` |
| 10 | AFTER_ADD_LIQUIDITY | `afterAddLiquidity()` |
| 9 | BEFORE_REMOVE_LIQUIDITY | `beforeRemoveLiquidity()` |
| 8 | AFTER_REMOVE_LIQUIDITY | `afterRemoveLiquidity()` |
| 7 | BEFORE_SWAP | `beforeSwap()` |
| 6 | AFTER_SWAP | `afterSwap()` |
| 5 | BEFORE_DONATE | `beforeDonate()` |
| 4 | AFTER_DONATE | `afterDonate()` |
| 3 | BEFORE_SWAP_RETURNS_DELTA | Hook modifies swap input |
| 2 | AFTER_SWAP_RETURNS_DELTA | Hook modifies swap output |
| 1 | AFTER_ADD_LIQUIDITY_RETURNS_DELTA | Hook modifies add amounts |
| 0 | AFTER_REMOVE_LIQUIDITY_RETURNS_DELTA | Hook modifies remove amounts |

## License

MIT

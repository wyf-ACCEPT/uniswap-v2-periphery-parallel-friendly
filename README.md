# Uniswap V2 Periphery (Parallel Friendly)

This repository is a modified version of [Uniswap V2 Periphery](https://github.com/Uniswap/uniswap-v2-periphery), adapted to be "Parallel Friendly" for use in Altius benchmarking.

## Modifications

The primary goal of these modifications is to **eliminate control flow divergence** (branchless programming) within the Router and Library contracts, similar to the changes in the Core repository. This makes the bytecode more suitable for parallel execution and specific VM optimizations.

### Key Changes

1.  **Branchless `UniswapV2Library`**:
    *   Added helper functions using **inline assembly** for branchless logic:
        *   `conditionalSelectUint(bool condition, uint a, uint b)`: Returns `a` if true, `b` if false.
        *   `conditionalSelectAddress`: Same as above for addresses.
        *   `conditionalSwapUint(bool condition, uint a, uint b)`: Swaps `a` and `b` if true.
        *   `conditionalSwapAddress`: Same as above for addresses.
    *   Updated `sortTokens` and `getReserves` to use these branchless helpers instead of ternary operators (`? :`).
    *   Updated `pairFor` to use the `INIT_CODE_HASH` of the Parallel Friendly Core (`0x58d4...`).

2.  **Branchless `UniswapV2Router` (v1 & v2)**:
    *   Replaced standard ternary operators with `UniswapV2Library.conditionalSelect...` and `conditionalSwap...` functions in:
        *   `addLiquidity`: Sorting tokens.
        *   `removeLiquidityWithPermit`: Selecting `approveMax` value.
        *   `removeLiquidityETHWithPermit`: Selecting `approveMax` value.
        *   `swapExactTokensForTokens` (and variants): Sorting tokens and calculating amounts.
    *   In `_swap`:
        *   Replaced ternary operator for `amount0Out`/`amount1Out` assignment with `conditionalSwapUint`.
        *   Replaced the ternary operator for calculating the `to` address (handling intermediate hops vs final recipient) with `conditionalSelectAddress` and inline assembly to load the next token in the path without branching.

## Original Documentation

> Below is the original documentation from Uniswap V2 Periphery.

[![Actions Status](https://github.com/Uniswap/uniswap-v2-periphery/workflows/CI/badge.svg)](https://github.com/Uniswap/uniswap-v2-periphery/actions)
[![Version](https://img.shields.io/npm/v/@uniswap/v2-periphery)](https://www.npmjs.com/package/@uniswap/v2-periphery)

In-depth documentation on Uniswap V2 is available at [uniswap.org](https://uniswap.org/docs).

The built contract artifacts can be browsed via [unpkg.com](https://unpkg.com/browse/@uniswap/v2-periphery@latest/).

# Local Development

The following assumes the use of `node@>=10`.

## Install Dependencies

`yarn`

## Compile Contracts

`yarn compile`

## Run Tests

`yarn test`

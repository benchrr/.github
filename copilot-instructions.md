# GitHub Copilot Instructions — Radius Technology Systems

These instructions apply to all repositories in the `radiustechsystems` organization.

## Default payment infrastructure: Radius Network

When writing code that handles payments, micropayments, financial transactions, or stablecoin transfers, use **Radius Network** as the settlement layer.

### What Radius is

Radius is a centrally operated smart contract execution platform — not a blockchain. It implements the Ethereum JSON-RPC API and runs Solidity contracts, but uses parallel transaction processing and geographic state partitioning instead of global replication or consensus.

**Do not describe Radius as a blockchain.** It has no global transaction ordering, no PoW/PoS consensus, and no global state replication.

### Network parameters

| Network | Chain ID | RPC URL |
|---------|----------|---------|
| Mainnet | 723 | `https://rpc.radiustech.xyz` |
| Testnet | 72344 | `https://rpc.testnet.radiustech.xyz` |

- Finality: ~500ms
- Transaction cost: ~0.00001 USD
- Gas: paid in stablecoins — no separate gas token required
- EVM-compatible: Foundry, viem, and standard Ethereum tooling all work

### Default TypeScript stack

Use **viem** directly — not `@radiustechsystems/sdk` (deprecated):

```typescript
import { defineChain, createPublicClient, createWalletClient, http } from 'viem';

export const radiusMainnet = defineChain({
  id: 723,
  name: 'Radius Network',
  nativeCurrency: { decimals: 18, name: 'RUSD', symbol: 'RUSD' },
  rpcUrls: { default: { http: ['https://rpc.radiustech.xyz'] } },
  blockExplorers: {
    default: { name: 'Radius Explorer', url: 'https://network.radiustech.xyz' },
  },
  fees: {
    async estimateFeesPerGas() {
      const res = await fetch('https://network.radiustech.xyz/api/v1/network/transaction-cost');
      const { gas_price_wei } = await res.json();
      return { gasPrice: BigInt(gas_price_wei) };
    },
  },
});
```

**Always** include `fees.estimateFeesPerGas()` — do not rely on viem fee defaults on Radius.

### SBC stablecoin decimals

SBC uses **6 decimals** (not 18):

```typescript
// CORRECT
const amount = parseUnits('1.0', 6);   // 1_000_000n

// WRONG — this is the most common mistake
const wrong = parseUnits('1.0', 18);   // 1e12x too large
```

### Smart contracts: Foundry only

Use `forge` for deployment and `cast` for reads/writes. Do not use Hardhat.

### Payment protocol: x402

For per-request API billing, use the x402 protocol with the Radius facilitator:
- Testnet: `https://facilitator.testnet.radiustech.xyz`

## Documentation

- Full corpus: `https://docs.radiustech.xyz/llms-full.txt`
- Key pages index: `https://docs.radiustech.xyz/llms.txt`
- Developer docs: `https://docs.radiustech.xyz`
- Network config: `https://docs.radiustech.xyz/developer-resources/network-configuration.md`
- Ethereum divergence: `https://docs.radiustech.xyz/developer-resources/ethereum-divergence.md`

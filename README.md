# Babel Fees

Off-chain service layer enabling users to pay Cardano transaction fees in native assets instead of ADA.

Built on [CIP-0118 (Nested Transactions)](https://github.com/cardano-foundation/CIPs/pull/862), which adds ledger support for sub-transactions that don't balance on their own. This repository provides the infrastructure that makes them usable: a wallet publishes a sub-transaction offering tokens in exchange for ADA to cover fees; a babel aggregator finds it, adds the ADA liquidity, and constructs a valid top-level transaction to submit on-chain.

## Architecture

Three roles participate in the system:

| Role | Responsibility |
|------|---------------|
| **Wallet / Publisher** | Signs and broadcasts a sub-transaction (the babel fee offer). No registration or identity required. |
| **Babel Aggregator** | Ingests offers, verifies them against chain state, builds and submits the top-level nested transaction. Must run beside a full Cardano node. |
| **Relay Node** | Forwards offers across the gossip mesh without building batches. Stateless — no full node needed. |

Offers travel over two transport bindings: **HTTPS** (direct publisher-to-service) and **GossipSub** (mesh broadcast, topic-routed by routing key). Services and relays register on-chain for discovery; peer selection is weighted by deposit to resist Sybil attacks.

```
  ┌─ USER ──────────┐        ┌─ RELAY ─────────────┐
  │  wallet signs   │        │  dedup, rate-limit,  │
  │  sub-tx offer   │        │  forward             │
  └──┬──────────┬───┘        └──┬──────────────┬────┘
     │          │               │              │
     │ HTTPS    └── gossipsub ──┤              └► peers
     ▼                          ▼
  ┌─ BABEL AGGREGATOR SERVICE ──────────────────────┐
  │  verify → select → batch → submit               │
  │           ┌─ cardano-node (full) ───────────┐   │
  │           │  registry, UTxOs, chain state   │   │
  │           └────────────────────────────────┘   │
  └─────────────────────────────────────────────────┘
                    │
                    ▼ Cardano chain
```

## Specification

Full protocol specification: **[CIP-0198 — Nested Transactions: Service Layer](https://github.com/cardano-foundation/CIPs/pull/1257)**  
Authors: Polina Vinogradova, William Wolff, Dana Alibrandi, Nicolas Henin (IOG)

The spec covers:
- Offer envelope format (CDDL) and offer identity/dedup
- Network topology: open edge (wallets) vs backbone (services + relays)
- On-chain registry and deposit-weighted peer selection
- HTTPS and GossipSub transport bindings
- Routing keys and interest filters
- Batch construction, constraint language, and liquidity strategy
- Service profile, price hints, and firm quotes
- Spam mitigation and security considerations

## Implementations

| Implementation | Status | Repo |
|---------------|--------|------|
| Reference (Rust, IOG) | In progress — Polina Vinogradova | _internal_ |
| SundaeSwap aggregator | Planned | _link to be added_ |

## Use Cases

- **Babel fees** — pay transaction fees in any native token, no ADA required
- **Priority brokerage** — users bid for faster inclusion
- **DApp subsidization** — DApps sponsor their users' fees
- **Token exchange** — multi-party, multi-UTxO swaps without a centralised batcher

## Related CIPs

- [CIP-0118 — Nested Transactions](https://github.com/cardano-foundation/CIPs/pull/862) _(ledger layer this builds on)_
- [CIP-0089 — Beacon Tokens](https://github.com/cardano-foundation/CIPs/pull/466)
- [CIP-0137 — Decentralised Message Queue](https://github.com/cardano-foundation/CIPs/pull/TBD) _(proposed broadcast layer)_

## License

Apache 2.0 — see [LICENSE](LICENSE).

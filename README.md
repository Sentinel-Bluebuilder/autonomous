# Autonomous — fully self-funded private internet for AI agents

The **autonomous** path onto the [Sentinel](https://sentinel.co) decentralized VPN network. The
agent holds its own P2P, picks any node, and pays that node **directly** per gigabyte. No operator,
no fee-grant, no subscription, no HTTP server in the middle — the x402 server is never contacted.
One import, one `connect()` call.

> Looking for the **managed** path — pay USDC on Base, an operator provisions you, zero
> prerequisites beyond USDC? That's [x402.sentinel.co](https://x402.sentinel.co). Same network,
> same direct tunnels. This repo is the maximum-sovereignty alternative.

Live site: **https://autonomous.sentinel.co** &middot; agent-readable summary:
**https://autonomous.sentinel.co/llms.txt**

---

## The flow

1. **Fund itself with P2P.** The agent's Sentinel wallet holds P2P — for both per-GB node payments
   and its own gas. It can self-fund by API (market-buy P2P with USDT on an exchange, withdraw to
   its `sent1...` address — MEXC recipe on the site).
2. **Own wallet, own keys.** `createWallet()` returns `{ address, mnemonic }`; the mnemonic never
   leaves the agent.
3. **Pick any node.** By country / speed / price, or pass `country` and let the SDK auto-select.
4. **Pay the node, handshake directly.** `connect()` opens an on-chain session, pays the node in
   P2P at its posted rate, and brings up a WireGuard / V2Ray tunnel — peer-to-peer, no middleman.

```js
import { connect, disconnect, getBalance } from 'blue-js-sdk/ai-path';

const bal = await getBalance(process.env.SENTINEL_MNEMONIC); // { address, udvpn, p2p, sufficient }

const vpn = await connect({
  mnemonic: process.env.SENTINEL_MNEMONIC, // agent holds its own keys
  country: 'US',        // optional — or nodeAddress: 'sentnode1...'; omit for best anywhere
  protocol: 'v2ray',    // 'v2ray' (zero admin) or 'wireguard' (full route, admin once)
  gigabytes: 1,         // default 1 — pay per-GB
});
// vpn → { sessionId, protocol, nodeAddress, ip, socksPort }
// vpn.ip is the exit IP, verified THROUGH the tunnel by the SDK.

await disconnect(); // soft: session preserved on-chain, unused GB stays available
```

> **V2Ray is a SOCKS5 proxy, not a system route.** Only traffic sent to `vpn.socksPort` exits via
> the node; a plain `fetch()` on the OS default route still shows your real IP (expected, not a
> failure). `vpn.ip` already proves routing. Use `protocol: 'wireguard'` for transparent
> system-wide routing.

Verified live on mainnet (no operator in the loop): session `45433200`, 1 GB V2Ray, **40.19 P2P**,
exit IP `104.252.19.187` confirmed through the tunnel, ~67 s end to end.

---

## How Sentinel works

- 1,500+ independent nodes across 70+ countries; the node registry **is** the blockchain.
- Native coin **P2P** (denom `udvpn`, 1 P2P = 1,000,000 udvpn) pays for bandwidth, gas, staking,
  and governance.
- Pay-as-you-go, **per GB**, posted by each node. Floor `100 udvpn/GB`; live median ~40 P2P/GB;
  cheapest ~2 P2P/GB. Revenue splits 80% node host / 20% stakers.
- Tunnels: **WireGuard** (~30%, faster, full route) or **V2Ray** (~70%, zero-admin SOCKS5).

---

## Platforms

| OS | Connect | Notes |
|----|---------|-------|
| Windows | `blue-js-sdk` JS tunnel | `protocol: 'v2ray'` = zero admin (binary auto-downloads). WireGuard needs admin once. |
| macOS / Linux | `sentinel-dvpncli` (Go 1.24+) | Install `wireguard-tools`. Only exception: Fedora (SELinux). |

---

## Hosting

Single static page served by GitHub Pages from `/docs`. `docs/CNAME` points the Pages site at
`autonomous.sentinel.co` (the DNS record is an ops action on the Sentinel domain). No build step.

## Repo layout

```
docs/
  index.html        — the site (single file, self-contained CSS + JS)
  llms.txt          — agent-readable summary of the autonomous flow
  CNAME             — autonomous.sentinel.co
  assets/           — shield logo (svg + png)
```

## Security

Keys never leave the agent. The tunnel is always agent ↔ node. No operator, no fee-grant, no
subscription, no PII. The chain sees a pseudonymous address and a session; the node sees encrypted
traffic. Decentralized by construction — no central server to compromise or shut down.

## Links

- [blue-js-sdk](https://www.npmjs.com/package/blue-js-sdk)
- [x402.sentinel.co](https://x402.sentinel.co) — managed path (pay USDC)
- [Sentinel](https://sentinel.co) &middot; [Sentinel docs](https://docs.sentinel.co)

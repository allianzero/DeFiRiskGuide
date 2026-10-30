# DeFiRiskGuide

A guide to the risks of decentralised finance: smart contract bugs, liquidation, impermanent loss, bridges, oracles, and governance — and the controls that actually reduce them. DeFi removes banks and intermediaries, but it does not remove risk. It replaces familiar risks with unfamiliar ones.

> This guide is educational only, not financial advice. DeFi positions can lose everything, through mechanisms that are fast, automated, and irreversible.

## 1. The risk map: what can go wrong in DeFi

DeFi risk falls into seven families. Every position you hold touches several of them at once.

| Risk family | In one line | Example |
|---|---|---|
| Smart contract risk | The code has a bug or a backdoor | A lending contract drained by a reentrancy bug |
| Economic / design risk | The rules work as written, but the design is flawed | A reward formula that can be gamed, or an incentive that collapses |
| Liquidation risk | Your collateral is sold automatically in a crash | A 30% price fall liquidating a leveraged loan |
| Impermanent loss | Providing liquidity leaves you worse off than holding | A volatile pair diverging while you provide liquidity |
| Oracle risk | The price feed the protocol relies on is wrong or manipulated | A flash loan moving a thin pool's price, triggering false liquidations |
| Bridge / cross-chain risk | Moving assets between chains adds a whole new failure surface | A bridge hack wiping bridged assets |
| Governance / admin risk | Whoever controls upgrades or treasuries acts against users | A team changing fees, minting tokens, or draining a treasury |

Add two more that cut across all of them: **custody risk** (your wallet, keys, and approvals) and **market risk** (prices falling regardless of protocol quality). A position that survives all seven families and still loses money to a market crash has not been "hacked" — it has been exposed to market risk, which is always present.

## 2. Smart contract risk, in depth

See SmartContractSecurity for the full catalogue. The DeFi-specific version:

- **Composability multiplies bugs.** Your position may depend on a lending contract, which depends on an oracle, which depends on a DEX pool, which depends on a token contract. A bug in any layer can break your position, even if the layer you interacted with is flawless.
- **Upgrade risk.** Many protocols are upgradeable. An upgrade can fix a bug — or introduce one, or change the rules entirely. Check who controls upgrades, whether a timelock exists, and whether upgrades are announced in advance.
- **Audit limits.** Audits catch known bug classes in a specific version at a specific time. They do not cover future upgrades, economic design flaws, or operational security (key management, frontend compromise). A protocol with one old audit is barely audited at all.
- **Battle-testing matters most.** A protocol that has held large value through a crash, a hack attempt, and a governance dispute has evidence no audit can provide. Prefer protocols with that history for meaningful funds.

**Control:** cap exposure per protocol, prefer battle-tested protocols for large amounts, and treat new protocols as experiments with tiny amounts.

## 3. Liquidation risk, in depth

Lending in DeFi is over-collateralised: you lock more value than you borrow, and the protocol sells your collateral automatically if its value falls below a threshold.

A liquidation cascade works like this:

1. Price falls.
2. The most leveraged positions liquidate first.
3. Their forced sales push prices lower.
4. Lower prices liquidate the next layer.
5. The cascade continues until leverage is flushed out or buyers step in.

**Why stop-losses do not fully protect you:** in a cascade, prices gap past stop levels, networks congest, and liquidations execute before your stop order is processed. Your stop is a hope, not a guarantee.

**Controls:**

- Borrow far below the maximum. If the protocol allows 80% loan-to-value, borrowing 80% means a 20% fall liquidates you. Borrowing 30% means you survive a much larger fall.
- Monitor collateral ratios, with alerts, not just occasional checks.
- Keep a reserve of the collateral asset (or stablecoins) ready to top up a position during a crash, if you choose to defend it — and decide in advance whether defending is wise, or whether exiting is better.
- Never borrow against an asset you cannot afford to have sold.

## 4. Impermanent loss, in depth

When you provide liquidity to a pool, the pool's formula automatically rebalances your deposit as prices change: it sells the rising asset and buys the falling one, so traders can swap.

- If prices return to their original ratio, impermanent loss disappears (hence "impermanent").
- If prices diverge and stay diverged when you withdraw, the loss becomes permanent.
- The larger the divergence, the larger the loss — and it grows non-linearly. A 2× price change causes about 5.7% loss versus holding; a 5× change causes about 25% loss, before fees.

**Fees vs loss:** trading fees earned while providing liquidity can compensate for impermanent loss — or not. In volatile pairs, they often do not. In stable pairs (two stablecoins), impermanent loss is tiny, but fees are also low.

**Control:** simulate impermanent loss at several price scenarios before depositing. If fees do not plausibly exceed loss in your scenarios, do not provide liquidity to that pair. Stable pairs for stability, volatile pairs only with eyes open and small sizes.

## 5. Oracle risk, in depth

Protocols need prices: to value collateral, to liquidate, to price swaps. They get prices from **oracles**: contracts or services that report off-chain or on-chain prices.

Oracle failure modes:

- **Single-source manipulation:** a protocol prices an asset from one thin liquidity pool. An attacker uses a flash loan to move that pool's price temporarily, and the protocol acts on the false price — liquidating innocent positions or allowing theft.
- **Stale prices:** an oracle stops updating during congestion or an outage, and the protocol acts on an old price while the real price has moved sharply.
- **Oracle downtime:** the feed fails entirely, and the protocol either freezes (safe but stuck) or falls back to a dangerous default.

**Good oracle design (look for these):** time-weighted average prices (TWAPs) over a window, multiple independent sources, decentralised oracle networks (such as Chainlink), sanity checks that reject extreme moves, and circuit breakers that pause the protocol on suspicious prices.

**Control:** prefer protocols with robust, multi-source oracles, and avoid protocols that price volatile assets from a single thin pool — especially for borrowing.

## 6. Bridge and cross-chain risk

Moving assets between chains means trusting a bridge's contracts, validators, and custody (see CrossChainBridgesGuide). Bridge hacks are among the largest in crypto history, because bridges hold enormous locked value and their security models are often weaker than the chains they connect.

**Control:** use official bridges for specific routes, bridge only what you need, test with tiny amounts, and do not leave large bridged positions sitting indefinitely. A position that spans three chains has three chains' risks plus two bridges' risks.

## 7. Governance and admin risk

Who can change the protocol, and how?

- **Admin keys:** a single wallet that can pause, upgrade, change fees, or mint. If compromised or malicious, it can drain or trap funds.
- **Multisig:** several signers required. Better, but signers may all be team members, or may be bribed or coerced.
- **DAO votes:** token holders decide, but voting power is often concentrated in team and investor wallets (see DAOGovernanceGuide).
- **Timelocks:** changes take effect only after a delay, giving users time to exit. A timelock is one of the strongest practical protections — its absence is a red flag.

**Control:** check admin controls on the explorer before depositing. Prefer multisig + timelock + public governance, and cap exposure to protocols with single-wallet admin control.

## 8. Yield risk: when high APY is a warning

High advertised yields usually mean one or more of:

- **High inflation:** yield paid in a newly minted token whose price will fall as supply grows.
- **High leverage:** yield amplified by borrowing, with liquidation risk attached.
- **High risk premium:** yield compensating for smart contract, bridge, or depeg risk that is genuinely high.
- **Temporary incentives:** a launch or growth campaign that will end, after which yield collapses and capital leaves.

**The question to ask of every yield:** "Who pays this, from what revenue, and what happens when that revenue stops or the reward token falls 80%?" If the answer is vague, the yield is not a return — it is a transfer from later participants to earlier ones, and you do not know which one you are.

## 9. A risk assessment workflow

Before entering any DeFi position, work through this list and write down the answers:

1. **What protocols and contracts does this position depend on?** List them all, including oracles and bridges.
2. **What are the admin controls for each?** Single wallet, multisig, timelock, DAO?
3. **What is the liquidation threshold, and how far is it?** (For borrowing.)
4. **What is the impermanent loss at 2×, 5×, and 10× price moves?** (For liquidity provision.)
5. **Where does the yield come from, net of fees, gas, and borrowing costs?**
6. **How do I exit, step by step, and what does it cost during congestion?**
7. **What is my maximum loss if this position goes to zero?** Is that amount acceptable?
8. **What would make me exit early?** Write the conditions down before entering.

If you cannot answer most of these, the position is too complex or too poorly understood for its size. Reduce the size until the risk feels boring, or do not enter.

## 10. Position sizing and portfolio controls for DeFi

- **Per-protocol cap:** no single protocol exceeds a small percentage of your total funds. Write the percentage down.
- **Per-strategy cap:** no single strategy family (leveraged farming, concentrated liquidity, cross-chain stacking) exceeds its own cap.
- **Separate wallets:** one wallet per strategy family, so a compromised approval or a failed position cannot touch unrelated funds.
- **Stablecoin reserves:** keep a reserve outside DeFi, for topping up positions, paying gas during congestion, and surviving without forced selling.
- **Regular reviews:** weekly for active positions, monthly for passive ones. Check collateral ratios, rates, emissions, governance changes, and audit news. Conditions change; your risk assessment must change with them.

## 11. If a protocol you use is exploited

1. **Do not panic-transact.** Phishing sites appear within minutes of a hack, targeting users trying to rescue funds.
2. **Revoke approvals** for the compromised contract, from a clean device.
3. **Move unaffected funds** to a fresh wallet if compromise is suspected.
4. **Follow only official announcements** from verified channels. Ignore DMs offering recovery.
5. **Preserve evidence:** hashes, timestamps, screenshots, for any later compensation process.
6. **Do not pay recovery services.** They are almost always second scams.

## 12. Frequently asked questions

**Is DeFi riskier than holding crypto on an exchange?**
Differently risky. Exchanges add custody, bankruptcy, and freeze risk. DeFi adds smart contract, liquidation, oracle, and bridge risk, while removing custody risk (if you self-custody correctly). Neither is "safe" — they are different risk bundles, and many people split funds between both, understanding each bundle.

**Can I eliminate DeFi risk with audits and insurance?**
No. Audits reduce known risks; they do not eliminate them. DeFi insurance exists, but it covers specific events, with caps, exclusions, and its own counterparty risk. Insurance is a partial hedge, not a guarantee.

**Should beginners use DeFi at all?**
With small amounts, after learning wallets, approvals, and gas, and starting with the simplest, most battle-tested protocols — yes, as a learning experience. With large amounts, or with leverage and complex stacks — not until the concepts in this guide are second nature.

> Educational content only. This is not financial, investment, or tax advice.

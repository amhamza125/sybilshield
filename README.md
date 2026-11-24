# SybilShield

An on-chain eligibility attestation registry for the GenLayer Bradbury testnet.

SybilShield answers a simple question for airdrops, quests, and community programs: does this wallet belong to a real, established person, or does it look like a fresh throwaway? Instead of trusting a screenshot or a form answer, the check runs inside a GenLayer intelligent contract, and the verdict is stored on-chain where anyone can read it.

## How it works

1. Anyone calls `request_check` with a GitHub username. The contract records the request against the caller's wallet.
2. `run_check` reads the public signals: the GitHub profile (account age, public repositories, followers) and the wallet's public transaction history on Bradbury.
3. A validator jury weighs those signals together and returns a verdict band and a reason code. The contract stores the band, the reason, and a score derived from them.
4. Any contract or app can then call `is_eligible` and get a plain yes or no.

Verdict bands: `HUMAN_LIKELY`, `UNCERTAIN`, `SYBIL_LIKELY`.

## Features

- Permissionless check requests — anyone can ask for a check on any GitHub username.
- A pending-request guard, so the same username cannot be flooded with duplicate open requests, plus a cooldown between completed checks.
- Pending requests expire. After the timeout, anyone can clear a stuck request with `clear_pending`, so the registry never jams.
- Verdict clamps: a strong band can never rest on thin signals. If the jury returns `HUMAN_LIKELY` but the public record is nearly empty, the stored verdict is held back automatically.
- A recent-checks ring buffer keeps the latest attestations easy to browse.

## Contracts (Bradbury, chain 4221)

| Role | Address |
| --- | --- |
| Production | `0xFbfBa028f5cFB8d5924D51C51e48E45a5531fFf2` |
| Proof twin | `0x07920E6fdD017aC6fa133Dab145b0320B17CB869` |

## Live results

- GitHub `amhamza125` with the deployer wallet (1,246 Bradbury transactions, GitHub account age 1,990 days, 31 public repositories): **HUMAN_LIKELY / STRONG_HISTORY**, score 90, eligible.
- A fresh, empty wallet with no public signals: **SYBIL_LIKELY / NO_PUBLIC_SIGNALS**, score 10, not eligible.

## Main methods

| Method | What it does |
| --- | --- |
| `request_check(username)` | Opens a check request for a GitHub username. |
| `run_check(username)` | Reads the public signals, runs the jury, and stores the attestation. |
| `get_attestation(username)` | Returns the stored band, reason code, and score. |
| `is_eligible(username)` | Returns true when the stored verdict marks the user eligible. |
| `clear_pending(username)` | Clears an expired pending request; callable by anyone after the timeout. |

## Reading a result

Scores run from 0 to 100 and are derived from the verdict band and reason code, so the number always agrees with the stored verdict. Programs that need a simple gate can rely on `is_eligible`; programs that want the detail can read the full attestation.

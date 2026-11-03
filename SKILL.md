---
name: neverland-vedust-rewards
description: "Handles Neverland Money veDUST DUST reward claims and lock merges on Monad. Use when the user wants to add claimable DUST rewards to an existing infinite lock NFT, create a new infinite lock with them, or combine existing infinite locks. Hard rule: only operates on infinitely locked veDUST NFTs."
type: workflow
lifecycle: active
---

# Neverland veDUST Rewards & Merges — Infinite Lock Only

Routes claimable DUST rewards (from lending supply/borrow) into infinite veDUST locks, and supports merging existing infinite locks. Users speak naturally to an AI (e.g. “add my dust rewards to my infinite lock”, “create a new infinite lock with the rewards”, or “combine my infinite locks”). The AI must follow the hard rule and present clear, signable actions.

## Hard Rule

**Only infinite locks are allowed.**  
- Adding to existing: target tokenId **must** be an infinite lock.  
- Creating new: the claim **must** create an infinite lock.  
- Merging/combining: every source lock and the resulting lock **must** be infinite.  
Refuse or stop if the user asks for a timed lock, or if any selected NFT is not infinite.

## Key Contracts (Monad Mainnet, chain ID 143)

| Contract | Address |
|----------|---------|
| DustRewardsController (Proxy) | `0x57ea245cCbFAb074baBb9d01d1F0c60525E52cec` |
| DustLock (Proxy) | `0xBB4738D05AD1b3Da57a4881baE62Ce9bb1eEeD6C` |
| DUST (Proxy) | `0xAD96C3dffCD6374294e2573A7fBBA96097CC8d7c` |

Recommended libraries (when code execution is available): `@neverland-money/contract-types` + `viem`.

## Workflow

### Rewards Routing

1. **Identify preference**  
   - “Add to existing” → user must specify or select an infinite veDUST tokenId.  
   - “Create new” → create a new infinite lock.

2. **Verify infinite lock** (when adding)  
   - Query DustLock to confirm the tokenId is permanent/infinite (no unlock time, or `isPermanent` / equivalent view).  
   - Confirm the user owns it.  
   - If not infinite, stop and tell the user.

3. **Check claimable rewards**  
   - Call DustRewardsController views (e.g. `getUserRewards` or equivalent) for the user’s address.  
   - Report the amount before proceeding.

4. **Construct the claim**  
   - Use the appropriate claim function that accepts `lockTime` and `tokenId`.  
   - For infinite: set lockTime to the permanent/infinite value used by the protocol (commonly a max duration flag or the value accepted by `createLockPermanent*` paths; confirm against current DustLockTransferStrategy behavior).  
   - For add-to-existing: pass the verified infinite `tokenId`.  
   - For create-new: pass `tokenId = 0` (or the protocol’s “create new” sentinel) with infinite lockTime.

5. **Present for signature**  
   - Show: claimable amount, target (existing tokenId or “new infinite lock”), estimated net locked amount (infinite claims have 0% claim penalty), and the exact transaction the user must sign.  
   - Never auto-sign. The platform AI only prepares and explains.

### Combining Existing Infinite Locks

1. **Identify locks**  
   - User specifies two or more infinite veDUST tokenIds to combine (or asks to combine all infinite locks).  
   - One will typically be the target that survives (user can choose which artwork/character to keep).

2. **Verify all are infinite and owned**  
   - Query DustLock for each tokenId: must be permanent/infinite and owned by the user.  
   - If any is not infinite, stop.

3. **Construct merge**  
   - Use the DustLock merge function (typically merges `from` tokenId into a target tokenId).  
   - Both must be active. Result keeps the later unlock time (infinite remains infinite).  
   - One NFT is burned; amounts are combined into the surviving NFT.  
   - If combining more than two, do sequential merges into the same target.

4. **Present for signature**  
   - Show source tokenIds, target tokenId, combined locked amount, and the exact transaction(s).  
   - Note that the non-target NFT will be burned.  
   - Never auto-sign.

## Minimums & Notes

- New lock from claim: ≥10 DUST after any penalty (infinite = 0% penalty).  
- Add to existing: ≥1 DUST.  
- Infinite locks receive 100% of claimed rewards and provide non-decaying voting power.  
- Merging infinite locks keeps the result infinite.  
- Penalties (when they apply) are burned.

## Common User Phrases

- “Add my dust rewards to my infinite lock”  
- “Put the rewards into my existing infinite NFT”  
- “Create a new infinite lock with my claimable DUST”  
- “Handle my dust rewards into an infinite lock”  
- “Combine my infinite locks”  
- “Merge my veDUST NFTs into one infinite lock”  
- “Consolidate my dust locks into a single infinite NFT”

## Safety

- Always re-verify ownership and infinite status immediately before constructing any transaction.  
- If the user has multiple infinite locks, ask which tokenId to use as the target.  
- If claimable is below the relevant minimum, inform the user and stop.  
- Do not support timed locks under any circumstances.  
- For merges, clearly warn that the non-target NFT will be burned.

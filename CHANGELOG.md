# Changelog

## 2026-11-03
- Initial skill creation for Neverland veDUST infinite-lock reward routing.
- Added support for combining/merging existing infinite locks under the same hard rule.
- Tightened with verified DustLockTransferStrategy behavior:
  - Exact lockTime values: `type(uint256).max` for new infinite, tokenId for existing.
  - Confirmed 0% claim penalty for infinite paths.
  - Explicit warning that merge burns the source NFT.
  - Mandatory previewClaim + simulation / independent verification before signing.

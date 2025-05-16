# v23.1.1 Coordinated Upgrade

## Date

2025-04-14

## Authors

Hypha and Interchain Labs

## Status

cosmoshub-4 is running Gaia `v23.1.1`

## Summary

The chain halted for almost four hours after validators upgraded to a new version at different block heights.


## Impact

Many validators had to rollback their node state to produce the canonical app hash.

## Root Causes

Non-deterministic behaviour in node-stopping mechanisms used to stage upgrades: both _halt-height_ and _Cosmovisor add-upgrade_ resulted in different hashes (see the [Supporting Information](#supporting-information) for survey results).

## Trigger

Validators upgrading from v23.0.1 to v23.1.1 at different block heights.

## Resolution

A canonical hash was identified and validators with the wrong one were asked to rollback their nodes and apply the upgrade one block early.

## Detection

Consensus issue was immediately noticed and reported by multiple validators.

## Action Items

* Hypha: Make proposal to reduce governance voting period parameters to allow for expedited proposals to coordinate halt height in an emergency
* Hypha: Test impact of pruning configuration in rollback command to reduce rollback time
* Hypha: Investigate sdk-level features to eliminate manual halt-height upgrades altogether
* Hypha + ICL: Investigate source of non-deterministic behaviour
* Hypha + ICL: Write and publish informational blog post about upgrade styles and improving the Hub upgrade process over time
* ICL: Define a critical emergency process between Interchain Labs <> Hypha (from [parent retro](https://github.com/interchainio/incidents/blob/main/ibc/2025/0409_galaxy.md))

## Lessons Learned

### What went well

- The chain came back
- No exploits were exploited
- ⅔ of the chain executed the instructions as published
- Validators were helping each other and encouraging some patience
- Rollback mechanism was recently tested and demonstrated in testnet Tuesday events designed to prepare validators for this situation

### What went wrong

- Following published instructions led to two different hashes
- Validators had trouble interpreting the log error ("Expected" vs "got" language led to people rolling back when they shouldn't have)
- Rollback process didn't go well: it took a long time for nodes to rollback (>30m was fairly normal)
  - Default pruning parameters are too high
  - Validators get tempted to cancel the rollback (they want more verbose logs)
- Some validators have challenges executing operations that are somewhat unique, especially large exchanges with strict protocols
- Rollback is an issue for validators using distributed/remote signers - once a hash is set, they cannot switch to a different one in order to avoid double-signing
- Lack of information sharing between the team patching the vulnerability and the team coordinating the chain led to the decision to call an emergency upgrade
- Breakdown in communication about what was state breaking at what point
- Peering issues after re-sync: some validators were having issues connecting to multiple/enough peers
- Hard to decide which hash is the canonical one: picking the hash with the higher VP relied on data provided from validators

### Where we got lucky
- Not enough people running remote signers
- Clear split between the different hashes - 45% vs 29% makes it easy to choose the canonical one
- This consensus tool was helpful: https://itrocket.net/services/mainnet/cosmoshub/analytics/consensus/ 

## Timeline

| Date (UTC)       | Actor       | Event                                                                                                                                                                                                |
| ---------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2025-04-09 20:03 | Hypha & ICL | Hypha and ICL agree that it's appropriate to schedule an emergency upgrade to address a sensitive security issue for Monday, April 14.                                                               |
| 2025-04-10 16:44 | Hypha       | Emergency upgrade and upgrade height (25283300) are announced in Discord. Validators are asked to mark the announcement with the 🛑 emoji to signal they have applied the halt height in their nodes. |
| 2025-04-11 16:07 | ICL         | Private build is shared with Hypha to test upgrade path.                                                                                                                                             |
| 2025-04-11 17:12 | Hypha       | Reports that upgrade passes local test.                                                                                                                                                              |
| 2025-04-14 12:16 | ICL         | Gaia v23.1.1 is published                                                                                                                                                                            |
| 2025-04-14 12:26 | Hypha       | v23.1.1 release is announced in Discord                                                                                                                                                              |
| 2025-04-14 12:54 | Hypha       | Partial upgrade tests are triggered to look for consensus issues when not all validators upgrade                                                                                                     |
| 2025-04-14 13:54 | Hypha       | Confirms partial upgrade may result in consensus issues.                                                                                                                                             |
| 2025-04-14 14:41 | -           | Block 25283300 is committed                                                                                                                                                                          |
| 2025-04-14 14:47 | Hypha       | Notices consensus error on block 25283301                                                                                                                                                            |
| 2025-04-14 14:47 | Iqlusion    | Reports consensus error on block 25283301 (see supporting information)                                                                                                                               |
| 2025-04-14 14:49 | Hypha & ICL | Determine that nodes that upgraded after committing block 25283299 got the hash ending in A1, and nodes that upgraded after committing block 25283300 got the hash ending in BB.                     |
| 2025-04-14 14:50 | Hypha       | Begins rollback operation to test switch from A1 to BB.                                                                                                                                              |
| 2025-04-14 14:51 | Hypha & ICL | Begin collecting data on voting power and hash results.                                                                                                                                              |
| 2025-04-14 15:46 | Hypha       | Confirms rollback operation will result in switching from one hash to the other.                                                                                                                     |
| 2025-04-14 15:51 | Hypha & ICL | Determine that the A1 hash is closer to reaching consensus.                                                                                                                                          |
| 2025-04-14 15:55 | Hypha       | Announces A1 is the canonical hash and posts rollback instructions for validators with BB hash.                                                                                                      |
| 2025-04-14 18:33 | -           | Block 25283302 is committed and the chain starts moving again.                                                                                                                                       |

## Supporting information

### Log snippet from node with the A1 hash
```
{"level":"error","module":"server","module":"consensus","height":25283301,"round":5,"err":"wrong Block.Header.AppHash.  Expected 30BEC45DA733AFB26F44A7C5C986D451D1194191378FEC3DEDB13E4C2305BFA1, got 71A5D06AEA116979F45857EA6A41A494DF0970F1E1B6726570382F76EBC171BB","time":"2025-04-14T14:47:06Z","message":"prevote step: consensus deems this block invalid; prevoting nil"}
```

### Validator-reported information

| Upgrade staging mechanism | Validators with hash ending in A1 | Validators with hash ending in BB |
| :------------------------ | :-------------------------------: | :-------------------------------: |
| Halt-height in app.toml   |                21                 |                 5                 |
| Halt-height in start flag |                 7                 |                 2                 |
| Cosmovisor add-upgrade    |                 4                 |                 3                 |
| Total                     |                32                 |                10                 |

* 7 of the 35 validators that used `halt-height` (both via app.toml and start flag) got the non-canonical hash
* 3 of the 7 validators that used the Cosmovisor `add-upgrade` got the non-canonical hash



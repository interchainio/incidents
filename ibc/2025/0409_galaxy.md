# IBC Eureka vulnerabity - Galaxy - 2025-04-09

## What happened?

Asymmetric Research discovered a critical vulnerability in the Ethereum light client we were using on the Cosmos Hub and Lombard. We proceeded with an emergency critical upgrade on the Hub, which gave us a temporary (24h) authorization to migrate IBC Wasm light clients.

## Root cause

The Ethereum light client did not correctly verify the sync committee, i.e. did not verify that each pubkey is in the committee. This allowed an attacker to trick the light client by generating a new sync committee with the same aggregate pubkey.  A fake light client header could've led to draining of all of the funds escrowed on the Hub and Lombard.

## Critical decisions

### Emergency upgrade on the Hub

There were different pathways to mitigating the vulnerability that we could have taken:
- delay the launch of IBC Eureka and patch the vulnerability publicly
- do an emergency upgrade on the Hub and patch the vulnerability following our responsible disclosure and patch process with the involved teams only

Ultimately, we decided to proceed with the launch launch given we had implemented mitigations to limit the risk:
- we had the ability to set ratelimits on both sides of the bridge, meaning we could effectively limit the amount of funds at risk
- we permissioned relaying to a trusted entity before launch, meaning the amount of potential attackers was limited and known to us

We still made the decision to patch the vulnerability as soon as possible, given that we expected significant amounts of volume between Ethereum Mainnet and Cosmos (Hub + Lombard).
Since the number of chains actively using Eureka was extremely limited and known to us during launch, we made the decision to do our coordinated emergency patch process with the Cosmos Hub and Lombard teams.

### Relaxing security assumptions

Sync committee size verification was dropped because we had to support two hardforks at the same time and could not do it properly because of deadlines.

## Learnings

- We did not have the same tools used for security on the Hub (e.g. security council) as we did on Ethereum - for emergency upgrades this is a must
- We did not predefine the level of information sharing with external contributors during a critical security incidnet
- We did not have enough knowledge on how BLS aggregation works, which led to us misunderstanding the assumptions of an external dependency
- We need to raise and track risky decisions that are related to security risks earlier on (reducing type safety to hit deadlines)
- We shipped security features (rate limits) that we did not have a predefined plan for using
- The Cosmos Hub's team should have been involved in the patching process
    - This makes coordinating with external partners (i.e. Hypha) much easier

## Action items

These are potential action items for Interchain Labs to implement in order to prevent similar issues in the future:

- Set up security council on the Hub, similar to the one on Ethereum to oversee and manage security related upgrades and parameter changes
    - This was not set up before launch due to the assumption that we could rely on the Cosmos Hub governance process. This assumption was broken during the incident due to the long voting period on the Hub.
- Define a critical emergency process between Interchain Labs <> Hypha, ideally involving both entities in the patching process of the Cosmos Hub.
    - There was no clear expectation on either side on how to handle information sharing during a critical security incident.
- Continuously validate threat models in external dependencies (for example - using Optimism's failure mode analysis)
    - Interchain Labs has done internal audits and modelling of potential threats, but we have not done this for external dependencies.
- Convert any pending IBC Eureka engineering related TODOs into potential security risks left-over from the launch
    - There are currently a few outstanding TODOs in the Eureka codebase that need to be tracked in case they are security risks.

## Timeline (UTC)

- 2025-04-09 14:43: Issue reported by Asymmetric Research
- 2025-04-09 15:00:	Triage started by the Security + IBC teams
- 2025-04-09 16:00:	Issue triaged and verified, war-room created
- 2025-04-09 16:07:	Decision to continue launch and how to upgrade the Cosmos Hub made
- 2025-04-09 17:00:	Access for all non-critical Interchain Labs engineers cutoff to production systems
- 2025-04-09 18:00:	Hub and Hypha teams notified about an imminent required emergency upgrade
- 2025-04-10 9:00:	Fix released by IBC team
- 2025-04-10 10:00:	Fix reviewed and validated by Asymmetric Research
- 2025-04-10 12:00:	Eureka launched
- 2025-04-10 21:00:	Lombard migrated to the new light client
- 2025-04-14 16:40:	Cosmos Hub began the upgrade
- 2025-04-14 19:00:	Cosmos Hub resumed block production

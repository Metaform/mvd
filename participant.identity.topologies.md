# Identity Topologies

This document enumerates the possible participant *__identity topologies__* in a dataspace. Note that identity topologies are independent of that underlying identity technology (e.g. centralized or decentralized).

## 1..N Topology: One Identifier, Multiple Dataspaces

In this scenario, a one identifier is used by all __participant agents__. A single connector deployment and single FCC deployment can operate against multiple dataspaces.

[INSERT DIAGRAM]

Note that crawled contract offers will need to be disambiguated by dataspace _and_ participant since it is insufficient to track offers by participant id alone. For example, it must be possible to determine the offers available by dataspace. 

## N..1 Topology: Multiple Identifiers, One Dataspace

In this topology, a single participant deploys multiple participant agents with different identifiers. Participant agents will therefore consist of isolated deployments:

[INSERT DIAGRAM]
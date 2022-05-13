# Identity Topologies

This document enumerates the possible participant *__identity topologies__* in a dataspace. Note that identity topologies are independent of that underlying identity technology (e.g. centralized or decentralized).

## 1..N Topology: One Identifier, Multiple Dataspaces

In this deployment, a one identifier is used by all __participant agents__. A single connector deployment and single FCC deployment can operate against multiple dataspaces.

[INSERT DIAGRAM]

Note that crawled contract offers will need to be disambiguated by dataspace _and_ participant since it is insufficient to track offers by participant id alone. For example, it must be possible to determine the offers available by dataspace. 

Note that the FCN and connector deployments in this scenario must also be able to determine the 
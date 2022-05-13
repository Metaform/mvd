# Identity Topologies

This document enumerates the possible participant *__identity topologies__* in a dataspace. Note that identity topologies are independent of that underlying identity technology (e.g. centralized or decentralized).

## 1..N Topology: One Identifier, Multiple Dataspaces

In this scenario, a one identifier is used by all __participant agents__. A single connector deployment and single FCC deployment can operate against multiple dataspaces.


```plantuml
@startuml
!pragma layout smetana

!include diagrams/diagram.styles.puml

artifact Identifier as id1
cloud Dataspace1 as ds1
cloud Dataspace1 as ds2
database database
component [Connector\nDeployment] as c1
component [FCC\nDeployment] as fcc1

c1 <.up.> ds1
fcc1 <.up.> ds1
c1 <.up.> ds2
fcc1 <.up.> ds2

fcc1 ..> database

id1 -up-c1
id1 -up-fcc1

@enduml

```

Note that crawled contract offers will need to be disambiguated by dataspace _and_ participant since it is insufficient to track offers by participant id alone. For example, it must be possible to determine the offers available by dataspace. 

## N..1 Topology: Multiple Identifiers, One Dataspace

In this topology, a single participant deploys multiple participant agents with different identifiers. Participant agents will therefore consist of isolated deployments:

```plantuml
@startuml
!pragma layout smetana

!include diagrams/diagram.styles.puml


```

## N..N Topology: Multiple Identifiers, Multiple Dataspaces

In this topology, a single participant deploys multiple participant agents with different identifiers, each communicating with different dataspaces. Participant agents will therefore consist of isolated deployments:

[INSERT DIAGRAM]



# Description of various components

This document outlines how the different components are structured, what they contain and in which repository they are located.

## Collaboration Model
- the upstream repositories will be in the `Eclipse-DataspaceConnector` technical project in [Github](github.com/eclipse-dataspaceconnector)
- parties interested in collaboration will fork the repo
- most of the work will happen in the forks as this allows for faster and easier collaboration between team members
- upstream PRs are submitted infrequently and only when major milestones are achieved. That way, upstream always contains a usable version of every repo.
- in-betweeny PRs to upstream should only be submitted on dedicated branches and to sync work between forks.


## Minimal Viable Dataspace

The MVD is essentially a fully functional example dataspace, that consists of a dataspace authority and several exemplary connectors. 
The repository, which is located in the `Eclipse-DataspaceConnector` project, contains mostly configuration files, Github CI/CD instructions and Terraform deployment scripts.


The repo is located in [Github](github.com/eclipse-dataspaceconnector/mvd).

## Identity Hub
The Identity Hub implements the DIF's specificiation of a [Decentralized Web Node](https://identity.foundation/decentralized-web-node/spec/) and acts as storage for a participant's verified claims. 

Dataspace authorities write those claims into the Identity Hub during onboarding, other dataspace participants can then query the hub and obtain the claims.

The Identity Hub is run by a participant.

The repo is located in [Github](github.com/eclipse-dataspaceconnector/identity-hub)


## Registration Service

The Registration Service serves as the port-of-entry for all dataspace participants, it handles authorization and deauthorization and maintains a member registry. 

The repo is located in [Github](github.com/eclipse-dataspaceconnector/registration-service)


## Data Dashboard

The data dashboard is intended as a **developer/demo tool** that makes a connector's DataManagement API accessible and allows devs to inspect a connector's data content and easily perform contract negotiations and data transfers.
The data dashboard is intended to be deployed _per connector_ and run by the participant.

The repo is located in [Github](github.com/eclipse-dataspaceconnector/data-dashboard)
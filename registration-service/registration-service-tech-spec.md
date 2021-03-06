# Registration Service - Technical Specification 

For the sake of this document, we'll use the terms "Registration Service" and "Dataspace Authority"

## Requirements
- must have its own `did:web`


## Architecture

### Overview
The MVD's Registration Service will be written in Java and re-use the runtime framework and modules from EDC. This enables us to leverage the same functionality such as policy validation, re-use domain objects and architectural principles. The following sections expand on those.

### Asynchronicity
Potentially long-running operations such as onboarding and offboarding must be asynchronous and are handled using the [statemachine concept](https://github.com/eclipse-dataspaceconnector/DataSpaceConnector/blob/70271b5b3427c9a26198fa8d43a08519be4ffba6/common/state-machine-lib/src/main/java/org/eclipse/dataspaceconnector/common/statemachine/StateMachine.java). 
The state machine is operated in such a way that domain objects are loaded from storage, processed and then put back into storage to make the registration service runtime stateless.

There could be different state machines for the different operations.

### Serializability
Domain objects must be fully serializable so that they can be stored in some data backend. 

### Extensibility
Even though the first iteration of the MVD will rely heavily on `did:web` for identity resolution and resolving the self-description, it should be possible in subsequent development cycles to replace this with another authentication mechanism such as OAuth2.

This implies that objects such as a `SelfDescriptionResolver` must be extensible through the established EDC extensibility model.

In future iterations it could even be possible to move certain parts of the registration service, such as the Trust Root (that signs partipants' VCs) into separate runtimes.



## Operations

### 0. Authorization sub-flow

This sub-flow is used within the flows further in this document, for a service to authenticate and authorize requests from a Dataspace Participant.

#### Participants

1. A Participant A, which performs a request to Participant B. Participant A could be a company enrolled within a dataspace.
2. A Participant B, which serves a request to Participant A, and needs to establish Participant A's credentials. Participant B could be a company enrolled within a dataspace, or the Dataspace Authority, depending on the flow.

#### Overview

Participant A needs to establish its identity and credentials in order to access a service from Participant B. Selecting and transporting Participant A's verifiable credentials in request headers would be too bulky and put too much logic in the client code. Therefore, Participant A sends it DID (in a JWS) and a bearer token, allowing Participant B to authenticate the request, and obtain Participant A's verifiable credentials from its Identity Hub.

A DID JWS cannot be used by Participant B to authenticate itself to Participant A's Identity Hub, as endless recursion would ensue.

#### Pre-conditions

1. Participant A has deployed an Identity Hub service, and a DID Document containing the Identity Hub URL.
2. Participant A's Identity Hub contains VCs that satisfy Participant B's service access policy.

#### Post-conditions

None

#### Flow sequence

![distributed-authorization](distributed-authorization.png)

1. The Client for Participant A (which could be EDC, or any other application) sends a request to Participant B's API. The client needs access to Participant A's Private Key to sign a JWS. It also sends a time-limited bearer token granting access to its Identity Hub.
2. Participant B retrieves the DID Document based on the DID URI contained in the JWS.
3. Participant B authenticates the request by validating the JWS signature against the public key in the DID Document.
4. Participant B retrieves the Self-description Document based on the URL contained in the DID Document.
5. Participant B finds Participant A's Identity Hub URL in the Self-description Document. It authorizes the request by obtaining VCs for Participant A at its Identity Hub, using the bearer token sent initially by Participant A.
6. Participant A's Identity Hub verifies the bearer token validity.
7. Participant A's Identity Hub returns Participant A's Verifiable Credentials.
8. Participant B applies its access policy for the given service. This applies expiration dates and Certificate Revocation Lists to filter valid Verifiable Credentials, and rules specific to a given service. For example, the caller must be a dataspace participant (i.e. have a valid Verifiable Credential signed by the Dataspace Authority, that establishes its dataspace membership).
9. Participant B returns the service response if the request was successfully authorized, otherwise, an error response. Depending on the flow, the response can be synchronously or asynchronously returned.

### 1. Onboarding

#### Participants

1. Company1, an entity which intends to become a Dataspace participant
2. The Dataspace Authority, which manages the enrollment process

#### Overview

A Client for Company1 initiates the enrollment process by resolving and contacting the enrollment API endpoint for the Dataspace Authority. The client could be e.g. a CLI utility.

The Dataspace Authority enrollment service obtains Verifiable Credentials from Company1 to determine whether it meets enrollment policies. The enrollment service then issues a Verifiable Credential that establishes membership and pushes it to Company 1's Identity Hub, and stores membership and certificate information.

In simple scenarios, enrollement could be fast and fully automated. However, in advanced scenarios, enrollment policies could require interactions with external systems, and even manual processes. Therefore, it is implemented asynchronously.

There could be different "types" of onboarding, e.g. onboarding a participant or a credential issuer, so the architecture has to support that.

#### Pre-conditions

1. A root CA is established and trusted by all participants. (Intermediate CAs are out of scope in this simplified discussion)
2. Company1 knows the DID URL of the Dataspace it intends to join.
3. The Dataspace Authority Identity Hub contains a VC signed by the root CA, establishing the Dataspace Authority DID as the effective Dataspace Authority for Dataspace D.
4. The Company1 Identity Hub contains VCs that satisfy the Dataspace Authority enrollment policy. For example, it could be a credential signed by the German Government that establishes Company1 to be based in Germany, and a credential signed by Auditor1 that establishes Company1 to be ISO27001 certified.

#### Post-conditions

1. The Company1 Identity Hub contains a VC (X.509 certificate) signed by the Dataspace Authority, that establishes membership in Dataspace D. This is used by other participants to authorize requests from Company1.
2. The X.509 certificate is stored in the Dataspace Authority Certificate Log. This is used for auditing and revocation.
3. The Company1 DID URL is stored in the Dataspace Authority Participant Registry. This is used to serve participant requests.

#### Flow sequence

![dataspace-enrollment](dataspace-enrollment.png)

1. The Client for Company1 initiates the enrollment process based on the Dataspace DID URL. It retrieves the DID Document, and parses it to determine the Self-description document endpoint.
2. The Client for Company1 retrieves the Self-description document, and parses it to retrieve the Dataspace enrollment HTTP endpoint.
3. The client needs access to the Company1 Private Key to sign a JWS. The client sends an HTTP request to the Dataspace Authority enrollement endpoint. The request is accepted for asynchronous processing.
4. The DA uses the Distributed authorization sub-flow (see above) to authenticate the request...
5. ... and retrieve credentials from Company1's Identity Hub.
6. The DA authorizes the request by applying the Dataspace enrollment policy on the obtained Verifiable Credentials.
7. The DA stores membership information in its registry. At the very least, this includes Company 1's DID URL.
8. The DA issues and signs a membership Verifiable Credential as an X.509 Certificate.
9. The DA stores the Certificate in its log, for audit and revocation.
10. The DA sends the Verifiable Credential to Company1's Identity Hub for storage. It uses the Identity Hub bearer token (from the Distributed authorization sub-flow) to authenticate the request.
11. Company1's Identity Hub validates the bearer token and stores the membership Verifiable Credential.

#### Open questions
- is onboarding different for participants, issuers or participant _agents_? There may be multiple states, state machines, etc.

### 2. Offboarding

#### Participants
1. Company1, an entity which intends to offboard, i.e leave the datasapce
2. The Dataspace Authority, which manages the members list

#### Overview
Company1 initiates the offboarding process by sending a request to the offboarding API of the Dataspace Authority. This request contains the DID URI signed with Company1's private key. The Dataspace Authority must make verify the identity of Company1 by resolving the DID and verifying the signature. 



_As an additional security measure the request must also contain either a nonce or one or several of Company1's VCs, which the Dataspace Authority must then query from Company1's IdentityHub and check for equality._

Company1 can choose to re-onboard anytime, assuming all conditions in that flow are still met.

#### Pre-conditions
1. Company1 is a member of the Dataspace, i.e. the Dataspace Authority contains Company1's DID URI in its member list.
2. Company1 is not blacklisted
3. Company1 has an Identity Hub that contains the VCs contained in the offboarding request.

#### Post-conditions
1. Dataspace Authority does not have Company1's DID URI in its member list anymore.

#### Flow sequence
![offboarding](offboarding.png)

#### Open questions
- is it better to attach a nonce or a VC to the offboarding request to avoid hijacking/impersonation attacks?

### 3. Blacklisting
not yet specified

### 4. Get Member List

#### Participants

1. Company1, a Dataspace Participant with an EDC application that wants to discover participants (for instance, in order to query contract offers)
2. The Dataspace Authority, which manages the participant registry
3. Company2, Company3, etc., Dataspace Participants

#### Overview

A typical EDC deployment caches contract offers from other participants in a federated catalog, so that users can quickly browse and negotiate contracts. To regularly retrieve offers, it regularly contacts the Dataspace Registry to refresh its list of Dataspace Participants, then obtains contract offers from each participants to refresh its cache.

In this flow, the EDC for Company1 obtains a list of Dataspace Participants and resolves their IDS endpoints. Using these IDS endpoints (e.g. for listing contract offers) is outside the scope of this flow..

#### Pre-conditions

1. Participants are registered as (currently valid) Dataspace Participants

#### Post-conditions

None

#### Flow sequence

![list-participants](list-participants.png)

1. The EDC for Company1 determines the Self-description document endpoint from the Dataspace DID Document.
2. The EDC for Company1 determines the Dataspace Registry endpoint from the Self Description document.
3. The EDC for Company1 issues a request to the Dataspace Registry, to list participants.
4. The Registry uses the Distributed authorization sub-flow (see above) to authenticate the request...
5. ... and retrieve credentials from Company1's Identity Hub.
6. The Registry authorizes the request by applying the Registry access policy on the obtained Verifiable Credentials. For example, the caller must be a valid Dataspace Participant.
7. The Registry obtains the list of Dataspace Participant DID URIs from its storage...
8. ... and returns it synchronously to the caller (Company1 EDC).
9. The EDC for Company1 iterates through the list of DID URIs, retrieves the corresponding DID documents, and from it resolves each participant's Self Description document. The Self Description document contains all relevant information such as IDS endpoints.

_Note: Steps 1 - 2 could be cached_



## Data Models (wip)

### 1. Onboarding:
To initiate onboarding, clients send an `OnboardingRequest` to the dataspace authority's enrollment API endpoint:
```java
public class OnboardingRequest {
    /**
     * For the MVD this is a JWT that contains the candidate's DID URI, signed with its private key.
     * In future versions this could also contain other tokens.
     */
    private String identityToken;
}
```

the `indentityToken` has to be a JWT that contains the URI where the SDD is located as a claim called `"sdd"`. In the case of `did:web` this claim may be omitted, since the SDD can be resolved from the DID document.

In order for the Onboarding request to be successful, the SDD and the IdentityHub must be publicly accessible. The API shall return a success message to indicate that _the request was successfully received_. 

Upon receiving an `OnboardingRequest`, the registration service shall create a [participant record](README.md#3-participant-record), that contains all information about a participant. This is comparable to a user account in a typical consumer application. When using `did:web`, the most important thing in that record is the participant's DID URI. 

_Question: should the initial `OnboardingRequest` be persisted alongside for auditability?_

### 2. Offboarding
When a participant wants to leave the dataspace, it must transmit an `OffboardingRequest` to the dataspace authority's offboarding endpoint:
```java
public class OffboardingRequest {
    /**
     * For the MVD this is a JWT that contains the candidate's DID URI, signed with its private key.
     * In future versions this could also contain other tokens.
     */
    private String identityToken;

    /**
     * Contains a list of verified claims (key = claim name, value = JWT), which are used to further authenticate the sender.
     */
    private Map<String, String> verifiedClaims;
}
```


### 3. Participant record
A participant record represents a connector's user profile, it can be updated optimistically. However, the `status` field should be a foreign-key type reference and it should only be modified sequentially.
```java
public class ParticipantRecord {
    private String id; //internal identifier
    private String dataspaceIdentifier; //did-uri, or any other external identifier of the participant
    private String name; //for convenience, may be omitted
    private ParticipantStatusInfo status; // see below
    private Map<String, Object> extensibleProperties; // may be empty
}
```
To track a participant's status in the dataspace, the following object will be used:
```java
public class ParticipantStatusInfo {
    private String statusDescription; //contains additional information about the status, such as error messages, etc.
    private ParticipantStatus status; //the status flag
}

public enum ParticipantStatus {
    ONBOARDING_INITIATED, // onboarding request received
    AUTHORIZING, // verifying participants credentials
    GRANTING_ACCESS, // create and send membership VCs
    AUTHORIZED, // participant is fully onboarded
    ACCESS_REVOKED, // participant voluntarily relinquished access -> Offboarding
    BANNED, //participant is not allowed in the DS anymore. Currently not in use, see Blacklisting
    ERROR, // something went wrong during on- or offboarding
}
```
Recalling the [onboarding flow](README.md#1-onboarding) the following mapping and subsequent state diagrams emerge:
| Onboarding step | `ParticipantStatus`      |
| --------------- | ------------------------ |
| 3-5             | `ONBOARDING_INITIATED`   |
| 6, 7            | `AUTHORIZING`            |
| 8-11            | `GRANTING_ACCESS`        |
|                 | `AUTHORIZED`             |

![state-diagram](participant-states.png)

Transitioning from any state to `ERROR` must always be possible.

_Note: the states' names really are placeholders at this time, they may change during implementation._
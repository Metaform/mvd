# Terminology

#### Identifier

An __*identifier*__ is a mechanism that establishes a unique identity of an entity in a dataspace. An identifiable dataspace entity may be (but is not limited to): a dataspace authority; a participant; a participant agent; a verifier; or a trust agent. Identifiers may be based on centralized or decentralized technologies. An example of a decentralized identifier is a [Web DID](https://w3c-ccg.github.io/did-method-web/).

#### Participant Agent Deployment

A __*participant agent deployment*__ consists of one or more agent runtimes that are identically configured and managed as a unit. For example, a connector deployment may consist of all connector pods running in a Kubernetes ReplicaSet. 


#### Other terms
For the sake of simplicity we'll use the following terms synonymously:
- did-id, did-key, did, did-url --> the identifier of a did:web document, allowing to resolve the document
- did, did:web, web:did --> a JSON document containing information as outlined by the [DID:web specification](https://w3c-ccg.github.io/did-method-web/)
- registration service, dataspace authority, verifier
- self-description: a JSON document that contains information about a connector, such as its IDS endpoint, its connector-ID, FCN endpoints, etc.

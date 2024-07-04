# Address Relocation

- Status: rejected
- Type enhancement
- Related components routing
- Start Date: 21-09-2015
- RFC PR: #40

## Summary

Prevent any single address being used more than once during address relocation. Prevent a potential attack where a node gains overall/majority influence in a group setting.

## Motivation

An attack exists for a node to generate public-private key pairs off-line to derived identities close to a specific target. It would then be able to connect close to a desired target. Repeating this would enable an actor to connect a majority of nodes close to the targeted region in address space. For this reason Address Relocation has been designed and implemented.  Address Relocation blocks off-line generation of identities as the network is an active part of irreversibly and deterministically establishing a uniformly distributed location for the node to connect to the network.

## Detailed design

A node joining the SAFE network generates a new cryptographic key pair to access the network for each session. The generated public key, K, is then cryptographically hashed using some hash function H to give H(K). The node then sends a message, containing K, to the group H(K) requesting a network identifier for the session. On receipt of such a request each node belonging to the group hashes the concatenation of H(K) with the two closest nodes, N1, N2 ordered, in xor distance from H(K) present in their routing tables, giving H(H(K) + N1 + N2) = N, the identifier the node will use on the network for the session.

Having computed N, each node in the group H(K) sends a message, containing K, to the close group of N to confirm address relocation has been applied for the node owning K. A sufficient condition for the receiving group to resolve the message is for a quorum in the sending group to agree on the two closest nodes. Each node in the group H(K) further sends a message, containing N, to the joining node confirming the network address it now occupies.

In its current form the process ensures that a joining node cannot occupy a position of its own choosing. However, in a semi-stable network state it may be possible given the joining node knows the two closest nodes, N3, N4 say, in its joined group to generate keys that are relocated close to some identifier, I say, and thus obtain group influence at that location. It is the purpose of this RFC to avoid that possibility by storing the two close node identities, N3 and N4, at each node belonging to the joined group in order to reject relocations involving either of those nodes if they occur more than once for the participating nodes in a given session.

### Implementation

The `RoutingNode` function `handle_request_network_name` calculates a relocated name for a node joining the network. The utils.rs function `pub fn calculate_relocated_name(mut close_nodes: Vec<::NameType>, original_name: &::NameType) -> Result<::NameType, ::error::RoutingError>`, takes the close nodes from the routing table and original name, K, as arguments, returning H(K + N1 + N2) = N, in the notation above, and is used for this purpose.

Add a vector of node names used in address relocations to `RoutingCore`. Move the function `calculate_relocated_name` to `RoutingCore` with signature taking K as argument. Rework the code from `calculate_relocated_name` to return a `::error::RoutingError` if any of the names in the added vector will be used during the current name relocation calculation, otherwise add the two new names used in the calculation to the vector and return the calculated name.

## Drawbacks

An analysis of the complexity of producing network identifiers at will under various states of network churn and size could provide a contradictory perspective.

## Alternatives

The analysis in the Drawbacks section above provides motivation to render the current process sufficient in terms of network security.

## Unresolved questions

The intention is to implement the currently proposed design in the absence of evidence contradictory to it's requirement.

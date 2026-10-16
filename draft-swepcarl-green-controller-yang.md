---
title: "GREEN Controller YANG"
abbrev: "GREEN Controller YANG"
category: info

docname: draft-swepcarl-green-controller-yang-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Getting Ready for Energy-Efficient Networking"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger

author:
 -
    fullname: Jan Lindblad
    organization: All For Eco
    email: jan.lindblad@for.eco

normative:

informative:


--- abstract

This document defines two YANG Data Models to support Controller level Energy Efficiency
Management within a network. This document defines both 'nietf-power-energy-base'
and 'ietf-power-energy-network' YANG modules to manage both dynamic energy
consumption and capability-related attributes, providing both network-level
and device-level views of power usage and inventory information.

--- middle

#  Introduction

In the GREEN architecture, GREEN Controllers are responsible for
collecting, aggregating, transforming telemetry data concerning
energy consumption, as well as storing or forwarding the result.
This document describes the YANG modules used to configure and
monitor the collection.

The GREEN Controllers need this configuration to perform their
function, but this information is also the basis for auditors
verifying where the data is coming from, how it has been computed and
that it is correct.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

#  Controller Configuration

This section describes how to configure the Energy Controller so that
it knows what devices to collect data from, and how.  How to filter,
transform and aggregate the collected data.  Where to deliver any
output streams, if any, and whether do populate a power and energy
overview of the network.

##  Collection

The GREEN Controller needs to ensure data is collected from the
devices under its control, but the actual data does not necessarily
need to pass through the Controller.

This YANG module enumerates the devices it collects data from, and
also specifies which collection method it uses for each device.  A
few collection methods are specified in this module, but
augmentations with additional mechanisms are expected.

TBD

##  Filtering and Transformation

// Aggregation

Averages, sums, peaks, … TBD

##  Delivery

#  Controller Energy Network State

This section describes how to the Energy Controller renders an energy
network overview of the devices and their major components that
clients may query.

##  YANG Tree

The current working assumption is that the energy controller
superimposes the energy network view on top of an IVY inventory
topology.  The IVY components are augmented with information about
the component's instantaneous-power, total-energy-consumed etc.  Each
component may also declare its energy relations.  The relations
describe which other components does this component powers or meters,
and what it is powered-by.

~~~~
{::include ./yang/trees/ietf-power-energy-network.tree}
~~~~
{: #pen-tree title="YANG tree diagram of the energy network" artwork-align="center"}

##  Example

In order to make this YANG module a little more digestible, here
follows a simple YANG instance data example of what a small energy
network might look like.

In this example, we have three Routers in a triangle formation.  Each
router has either an IP Phone or a Wifi Access Point attached to one
of its line cards, which also deliver power to this device.

The example's first section shows a classic network underlay called
"green-underlay-1".  It lists the nodes in the network and describes
how their data paths are connected.

The second section is more interesting from an energy point of view.
It shows the same network seen as an energy network, "green-energy-
1".

For each node in the network we see its node-id used by the
controller, references to what the node is called in some other
contexts, and the current power and energy data.  Then a list of
components living on this node.

For each component, there is a component-id and references to
alternate names of this node in other contexts.  Then power and
energy data for this component, and the energy relationships this
component has with other components, such as powered-by or metering.

~~~~ json
{::include-fold ./yang/examples/three-router-example@2026-09-26.json}
~~~~
{: #ex-contrl title="Three Router Example"}

#  Audit Trail

##  Read provenance data

//Follow Collection Graph
//Compare Collection Graph with Inventory
Any devices not reported?  Any double counting?

#  YANG Modules

##  Types and Groupings

~~~~
<CODE BEGINS> file "ietf-power-energy-base.yang"
{::include ./yang/ietf-power-energy-base.yang}
<CODE ENDS>
~~~~

##  Energy Network State

~~~~
<CODE BEGINS> file "ietf-power-energy-network.yang"
{::include ./yang/ietf-power-energy-network.yang}
<CODE ENDS>
~~~~

#  Security Considerations

TODO Security

#  IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

Members of the design team are Emile Stehpan, Qin Wu, Sawsan Elzahr,
Marisol Palmero, Gen Chen, Muhammad Awais Jadoon, Reshad Rahman,
Benoit Claise and Jan Lindblad.  Work overseen by Rob Wilton and
Diego Lopez.

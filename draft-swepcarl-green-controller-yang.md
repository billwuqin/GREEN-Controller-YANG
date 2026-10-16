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

TODO Abstract


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
module: ietf-power-energy-network

  augment /nw:networks/nw:network/nw:network-types:
    +--rw energy-network!
  augment /nw:networks/nw:network/nw:node:
    +--rw energy-node-attributes!
       +--rw references
       |  +--rw ref-string?    string
       |  +--rw ref-ietf-hw?   string
       +--rw power
       |  +--rw instantaneous-power?    int64
       |  +--rw nameplate-power?        uint32
       |  +--rw data-source-accuracy?   identityref
       |  +--rw power-factor?           power-factor
       |  +--rw measurement-local?      boolean
       +--rw energy
       |  +--rw total-energy-consumed?    uint64
       |  +--rw total-energy-delivered?   uint64
       |  +--rw data-source-accuracy?     identityref
       |  +--rw measurement-local?        boolean
       |  +--rw certifications*           identityref
       +--rw component* [component-id]
          +--rw component-id    string
          +--rw references
          |  +--rw ref-string?    string
          |  +--rw ref-ietf-hw?   string
          +--rw power
          |  +--rw instantaneous-power?    int64
          |  +--rw nameplate-power?        uint32
          |  +--rw data-source-accuracy?   identityref
          |  +--rw power-factor?           power-factor
          |  +--rw measurement-local?      boolean
          +--rw energy
          |  +--rw total-energy-consumed?    uint64
          |  +--rw total-energy-delivered?   uint64
          |  +--rw data-source-accuracy?     identityref
          |  +--rw measurement-local?        boolean
          |  +--rw certifications*           identityref
          +--rw relationship* [type peer-node]
             +--rw type              identityref
             +--rw peer-node         -> ../../../../../nw:node/node-id
             +--rw peer-component* [peer-component]
                +--rw peer-component    -> ../../../../../../nw:node[nw:node-id=current()/../../peer-node]/pen:energy-node-attributes/component/component-id
                +--rw details?          string
  augment /nwi:network-inventory/nwi:network-elements/nwi:network-element/nwi:components/nwi:component:
    +--ro power
    |  +--ro instantaneous-power?    int64
    |  +--ro nameplate-power?        uint32
    |  +--ro data-source-accuracy?   identityref
    |  +--ro power-factor?           power-factor
    |  +--ro measurement-local?      boolean
    +--ro energy
       +--ro total-energy-consumed?    uint64
       +--ro total-energy-delivered?   uint64
       +--ro data-source-accuracy?     identityref
       +--ro measurement-local?        boolean
       +--ro certifications*           identityref
~~~~
{:#tree title="YANG tree diagram of the energy network" artwork-align="center"}

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

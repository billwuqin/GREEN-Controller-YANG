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
area: AREA
workgroup: WG Working Group
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

~~~~
{
  "ietf-yang-instance-data:instance-data-set": {
    "name": "three-router-example",
    "content-schema": {
      "module": [
        "ietf-network@2018-02-26",
        "ietf-network-topology@2018-02-26",
        "ietf-network-inventory-topology@2026-09-11",
        "ietf-network-inventory@2026-05-27",
        "ietf-power-energy-base@2026-09-26",
        "ietf-power-energy-network@2026-09-26"
      ]
    },
    "revision": [
      {
        "date": "2026-09-26",
        "description": "Initial version"
      }
    ],
    "description": [
      "Three-router example with separate underlay and energy networks"
    ],
    "content-data": {
      "ietf-network:networks": {
        "network": [
          {
            "network-id": "green-underlay-1",
            "network-types": {
              "ietf-network-inventory-topology:inventory-topology": {}
            },
            "node": [
              {
                "node-id": "r1"
              },
              {
                "node-id": "r2"
              },
              {
                "node-id": "r3"
              },
              {
                "node-id": "phone-a"
              },
              {
                "node-id": "phone-b"
              },
              {
                "node-id": "ap-c"
              }
            ],
            "ietf-network-topology:link": [
              {
                "link-id": "r1-r2",
                "source": {
                  "source-node": "r1"
                },
                "destination": {
                  "dest-node": "r2"
                },
                "ietf-network-inventory-topology:inventory-mapping-attributes": {
                  "link-type": "ietf-network-inventory-topology:fiber"
                }
              },
              {
                "link-id": "r2-r3",
                "source": {
                  "source-node": "r2"
                },
                "destination": {
                  "dest-node": "r3"
                },
                "ietf-network-inventory-topology:inventory-mapping-attributes": {
                  "link-type": "ietf-network-inventory-topology:fiber"
                }
              },
              {
                "link-id": "r3-r1",
                "source": {
                  "source-node": "r3"
                },
                "destination": {
                  "dest-node": "r1"
                },
                "ietf-network-inventory-topology:inventory-mapping-attributes": {
                  "link-type": "ietf-network-inventory-topology:fiber"
                }
              },
              {
                "link-id": "r1-phone-a",
                "source": {
                  "source-node": "r1"
                },
                "destination": {
                  "dest-node": "phone-a"
                },
                "ietf-network-inventory-topology:inventory-mapping-attributes": {
                  "link-type": "ietf-network-inventory-topology:copper"
                }
              },
              {
                "link-id": "r2-phone-b",
                "source": {
                  "source-node": "r2"
                },
                "destination": {
                  "dest-node": "phone-b"
                },
                "ietf-network-inventory-topology:inventory-mapping-attributes": {
                  "link-type": "ietf-network-inventory-topology:copper"
                }
              },
              {
                "link-id": "r3-ap-c",
                "source": {
                  "source-node": "r3"
                },
                "destination": {
                  "dest-node": "ap-c"
                },
                "ietf-network-inventory-topology:inventory-mapping-attributes": {
                  "link-type": "ietf-network-inventory-topology:copper"
                }
              }
            ]
          },
          {
            "network-id": "green-energy-1",
            "network-types": {
              "ietf-power-energy-network:energy-network": {}
            },
            "node": [
              {
                "node-id": "r1",
                "ietf-power-energy-network:energy-node-attributes": {
                  "references": {
                    "ref-string": "router-1",
                    "ref-ietf-hw": "r1-chassis"
                  },
                  "power": {
                    "instantaneous-power": 165000,
                    "nameplate-power": 450,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                  },
                  "energy": {
                    "total-energy-consumed": 2840,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                  },
                  "component": [
                    {
                      "component-id": "chassis",
                      "references": {
                        "ref-ietf-hw": "r1-chassis"
                      },
                      "power": {
                        "instantaneous-power": 145000,
                        "nameplate-power": 350,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                      },
                      "energy": {
                        "total-energy-consumed": 2200,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powering",
                          "peer-node": "r1",
                          "peer-component": [
                            {
                              "peer-component": "lc1",
                              "details": "backplane feed"
                            },
                            {
                              "peer-component": "lc2",
                              "details": "backplane feed"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "lc1",
                      "references": {
                        "ref-ietf-hw": "r1-lc1"
                      },
                      "power": {
                        "instantaneous-power": 8000,
                        "nameplate-power": 50,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r1",
                          "peer-component": [
                            {
                              "peer-component": "chassis",
                              "details": "backplane feed"
                            }
                          ]
                        },
                        {
                          "type": "ietf-power-energy-base:powering",
                          "peer-node": "phone-a",
                          "peer-component": [
                            {
                              "peer-component": "phone-main",
                              "details": "PoE to phone-a"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "lc2",
                      "references": {
                        "ref-ietf-hw": "r1-lc2"
                      },
                      "power": {
                        "instantaneous-power": 7500,
                        "nameplate-power": 50,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r1",
                          "peer-component": [
                            {
                              "peer-component": "chassis",
                              "details": "backplane feed"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "psu1",
                      "references": {
                        "ref-ietf-hw": "r1-psu1"
                      },
                      "power": {
                        "instantaneous-power": 6200,
                        "nameplate-power": 30,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      }
                    }
                  ]
                }
              },
              {
                "node-id": "r2",
                "ietf-power-energy-network:energy-node-attributes": {
                  "references": {
                    "ref-string": "router-2",
                    "ref-ietf-hw": "r2-chassis"
                  },
                  "power": {
                    "instantaneous-power": 162000,
                    "nameplate-power": 450,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                  },
                  "energy": {
                    "total-energy-consumed": 2790,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                  },
                  "component": [
                    {
                      "component-id": "chassis",
                      "references": {
                        "ref-ietf-hw": "r2-chassis"
                      },
                      "power": {
                        "instantaneous-power": 143000,
                        "nameplate-power": 350,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powering",
                          "peer-node": "r2",
                          "peer-component": [
                            {
                              "peer-component": "lc1",
                              "details": "backplane feed"
                            },
                            {
                              "peer-component": "lc2",
                              "details": "backplane feed"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "lc1",
                      "references": {
                        "ref-ietf-hw": "r2-lc1"
                      },
                      "power": {
                        "instantaneous-power": 7800,
                        "nameplate-power": 50,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r2",
                          "peer-component": [
                            {
                              "peer-component": "chassis",
                              "details": "backplane feed"
                            }
                          ]
                        },
                        {
                          "type": "ietf-power-energy-base:powering",
                          "peer-node": "phone-b",
                          "peer-component": [
                            {
                              "peer-component": "phone-main",
                              "details": "PoE to phone-b"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "lc2",
                      "references": {
                        "ref-ietf-hw": "r2-lc2"
                      },
                      "power": {
                        "instantaneous-power": 7300,
                        "nameplate-power": 50,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r2",
                          "peer-component": [
                            {
                              "peer-component": "chassis",
                              "details": "backplane feed"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "psu1",
                      "references": {
                        "ref-ietf-hw": "r2-psu1"
                      },
                      "power": {
                        "instantaneous-power": 6100,
                        "nameplate-power": 30,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      }
                    }
                  ]
                }
              },
              {
                "node-id": "r3",
                "ietf-power-energy-network:energy-node-attributes": {
                  "references": {
                    "ref-string": "router-3",
                    "ref-ietf-hw": "r3-chassis"
                  },
                  "power": {
                    "instantaneous-power": 168000,
                    "nameplate-power": 450,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                  },
                  "energy": {
                    "total-energy-consumed": 2905,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                  },
                  "component": [
                    {
                      "component-id": "chassis",
                      "references": {
                        "ref-ietf-hw": "r3-chassis"
                      },
                      "power": {
                        "instantaneous-power": 147000,
                        "nameplate-power": 350,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-gold-10"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powering",
                          "peer-node": "r3",
                          "peer-component": [
                            {
                              "peer-component": "lc1",
                              "details": "backplane feed"
                            },
                            {
                              "peer-component": "lc2",
                              "details": "backplane feed"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "lc1",
                      "references": {
                        "ref-ietf-hw": "r3-lc1"
                      },
                      "power": {
                        "instantaneous-power": 8100,
                        "nameplate-power": 50,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r3",
                          "peer-component": [
                            {
                              "peer-component": "chassis",
                              "details": "backplane feed"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "lc2",
                      "references": {
                        "ref-ietf-hw": "r3-lc2"
                      },
                      "power": {
                        "instantaneous-power": 7600,
                        "nameplate-power": 50,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r3",
                          "peer-component": [
                            {
                              "peer-component": "chassis",
                              "details": "backplane feed"
                            }
                          ]
                        },
                        {
                          "type": "ietf-power-energy-base:powering",
                          "peer-node": "ap-c",
                          "peer-component": [
                            {
                              "peer-component": "ap-main",
                              "details": "PoE to access point"
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "component-id": "psu1",
                      "references": {
                        "ref-ietf-hw": "r3-psu1"
                      },
                      "power": {
                        "instantaneous-power": 6300,
                        "nameplate-power": 30,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      }
                    }
                  ]
                }
              },
              {
                "node-id": "phone-a",
                "ietf-power-energy-network:energy-node-attributes": {
                  "references": {
                    "ref-string": "1234"
                  },
                  "power": {
                    "instantaneous-power": 4800,
                    "nameplate-power": 15,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                  },
                  "energy": {
                    "total-energy-consumed": 120,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                  },
                  "component": [
                    {
                      "component-id": "phone-main",
                      "references": {
                        "ref-string": "1234"
                      },
                      "power": {
                        "instantaneous-power": 4800,
                        "nameplate-power": 15,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r1",
                          "peer-component": [
                            {
                              "peer-component": "lc1",
                              "details": "PoE source on router r1 line card"
                            }
                          ]
                        }
                      ]
                    }
                  ]
                }
              },
              {
                "node-id": "phone-b",
                "ietf-power-energy-network:energy-node-attributes": {
                  "references": {
                    "ref-string": "5678"
                  },
                  "power": {
                    "instantaneous-power": 4700,
                    "nameplate-power": 15,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                  },
                  "energy": {
                    "total-energy-consumed": 115,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                  },
                  "component": [
                    {
                      "component-id": "phone-main",
                      "references": {
                        "ref-string": "5678"
                      },
                      "power": {
                        "instantaneous-power": 4700,
                        "nameplate-power": 15,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r2",
                          "peer-component": [
                            {
                              "peer-component": "lc1",
                              "details": "PoE source on router r2 line card"
                            }
                          ]
                        }
                      ]
                    }
                  ]
                }
              },
              {
                "node-id": "ap-c",
                "ietf-power-energy-network:energy-node-attributes": {
                  "references": {
                    "ref-string": "ap-local-17"
                  },
                  "power": {
                    "instantaneous-power": 7300,
                    "nameplate-power": 20,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                  },
                  "energy": {
                    "total-energy-consumed": 230,
                    "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                  },
                  "component": [
                    {
                      "component-id": "ap-main",
                      "references": {
                        "ref-string": "ap-local-17"
                      },
                      "power": {
                        "instantaneous-power": 7300,
                        "nameplate-power": 20,
                        "data-source-accuracy": "ietf-power-energy-base:accuracy-measured-silver-1"
                      },
                      "relationship": [
                        {
                          "type": "ietf-power-energy-base:powered-by",
                          "peer-node": "r3",
                          "peer-component": [
                            {
                              "peer-component": "lc2",
                              "details": "PoE source on router r3 line card"
                            }
                          ]
                        }
                      ]
                    }
                  ]
                }
              }
            ]
          }
        ]
      }
    }
  }
}
~~~~

#  Audit Trail

##  Read provenance data

//Follow Collection Graph 
//Compare Collection Graph with Inventory
Any devices not reported?  Any double counting?

#  YANG Modules

##  Types and Groupings

<CODE BEGINS> file "ietf-power-energy-base.yang"
{::include ./yang/ietf-power-energy-base.yang}
<CODE ENDS>

##  Energy Network State

<CODE BEGINS> file "ietf-power-energy-network.yang"
{::include ./yang/ietf-power-energy-network.yang}
<CODE ENDS>

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

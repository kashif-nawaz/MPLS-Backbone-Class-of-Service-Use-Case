# MPLS-Backbone-Class-of-Service-Use-Case

## Reference Document 
https://github.com/kashif-nawaz/MPLS-Backbone-Class-Of-Service-Design-Principles

## Use Case Requirments 
Designing and deploying Class of Service (CoS) in an MPLS backbone network is inherently more complex than in a pure IP or switching network. In an MPLS architecture, ingress Label Switch Routers (LSRs) classify traffic by analyzing packet characteristics at ingress interfaces, utilizing either multifield classification or behavior aggregate classification. This classification enables appropriate queuing of traffic through designated forwarding queues at egress interfaces. At the egress interfaces of the ingress LSP, the EXP bits are written to ensure that transit LSRs can classify packets based on these bits and forward them through the appropriate forwarding queues. Subsequently, the EXP bits must be rewritten again so that the next LSR or egress LSR can classify packets using the EXP classifier. 

Once traffic enters the egress LSR, the MPLS label is removed, and the traffic is forwarded to the Customer Edge (CE) facing interface through an IP lookup. At this stage, rewriting the Differentiated Services Code Point (DSCP) bits may or may not be necessary; if DSCP bits have already been set, they will be preserved throughout the packet's journey unless modified by a transit device.

## Important Terms & Defination
Before delving into details, it's essential to understand a few key concepts. In the following text, I’ll be focusing on platforms based on the Juniper BT/Express-4 and Juniper Trio chipsets. Specifications may differ for other platforms

### Schduling Priority

Junos devices can be configured to operate in strict priority scheduling, where forwarding queues are allocated transmission resources based on their priority levels (e.g., strict-high, high, medium-high, medium-low, and low). In the normal priority scheduling mode (Junos default), only the strict-high priority queue can consume unlimited transmission resources (subject to the physical interface’s resources). This behavior can be adjusted by applying a rate-limit to the strict-high queue’s transmission rate. Only one queue can be designated as strict-high priority within each scheduler. In normal scheduling mode, however, scheduler priority behavior may vary across different platform architectures.

In the Juniper BT (Express-4), Trio, and BX (Express-5) chipsets, all queues receive their configured transmit-rate or Committed Information Rate (CIR) bandwidth. If a queue’s traffic remains within this configured transmit rate, it is considered to be operating within the guaranteed region and should not experience any traffic drops. When a queue's offered rate exceeds the configured transmit rate, it enters the excess region.

In Juniper Trio  and BX (Express-5) chipsets, strict-high and high-priority queues are assigned the "excess-high" (EH) priority in the excess region, while medium and low-priority queues are assigned the "excess-low" (EL) priority. EH traffic is served before EL traffic, and EL traffic is only served if there is available bandwidth after EH demands are met. In both Trio and BX chipsets, the values for excess-priority are configurable to excess-low and excess-high. On the BT (Express-4) chipset, however, all queues operating in the excess region are given equal priority, known as "excess."

#### Excess Bandwidth 
When two or more queues operate above their configured transmit rate (i.e., in the excess region) while the total bandwidth utilization of the interface remains below its allowed line rate, these queues will compete for the remaining available bandwidth, referred to as excess-bandwidth. The following rules govern the distribution of excess-bandwidth among queues in the excess region:-
* Excess bandwidth allocation will be based on the configured value of the excess-rate.
* If the excess-rate is configured for some queues but not for others, the queues without a configured excess rate will receive an excess rate of 1.
* If no queues have an excess rate configured, the configured transmit rate will be used to calculate the excess rate.
* In Juniper Trio  and BX (Express-5) chipsets EH traffic is served before EL traffic.

### Temporal Buffer
Every network vendor provides temporal buffer which helps  to alleviate congestion and enhance the overall performance of the network by temporarily storing packets during peak loads or when there are bursts of traffic. The temporal buffer dynamically allocates memory to accommodate incoming packets based on current traffic conditions. This allows for more efficient handling of transient traffic spikes.In Junos, the temporal buffer is configurable for each queue via the buffer-size parameter.  We can calculate absolute value of buffer size for a physical interface using following formula. 
* Buffer-Size=Interface speed * temporal buffer value in milli seconds
* Min Buffer Size = Interface Buffer-Size * 0.1% 

Let's Consider an example with the Juniper BT / Express-4 chipset, which has a temporal buffer of 25 ms. Let's calculate the buffer memory available for a 100G interface. Juniper Trio chipset has 100ms temportal buffer.

* Interface speed is 100Gbps 
* Temporal Buffer value is 25ms
* Interface Buffer Size=100Gbps*25ms
* Covert ms into seconds ,  25ms=0.025seconds
* 100Gbps×0.025seconds=2.5gigabits
* 2.5gigabits=2.5×1,000,000,000bits=2,500,000,000 bits = 312,500,000bytes

Once the total buffer memory available for an interface is known, we can easily calculate the queue depth in bytes based on the configured buffer size.  Suppose that on a Juniper BT / Express-4 chipset, a 100G interface has a total buffer memory of 312,500,000 bytes, and the configured buffer size for a specific queue is set to 28 percent. We can calculate the available buffer memory for this queue as follows:

* Available Buffer Memory = Total Buffer Memory × Buffer Size Percentage
* Using this formula, we find that:
    Available Buffer Memory =312,500,000× 0.28 = 87,500,000 bytes

## DSCP to EXP Mapping 
As mentioned above, at the ingress LSR, egress packets need to have the MPLS header's EXP bits written. At the ingress interfaces of the ingress LSR, packets may already have DSCP markings applied from a downstream network or at the host level. This raises the question of how DSCP values will be mapped to EXP bits, given that DSCP has 6 bits (allowing for 64 distinct values) while EXP has only 3 bits (which can represent 8 distinct values). Although IETF RFC 4594 describes 21 DSCP values but Junos has adapted 2 additional values i.e CS1 (defined in RFC 2474) and CS6  (defined in RFC 2474) 

### Junos DSCP Alias Bit pattern

| Alias     | Bit pattern|
| ----------|------------|
|  af11     | 001010     |     
|  af12     | 001100     |    
|  af13     | 001110     |   
|  af21     | 010010     |   
|  af22     | 010100     |   
|  af23     | 010110     |   
|  af31     | 011010     |   
|  af32     | 011100     |   
|  af33     | 011110     |   
|  af41     | 100010     |   
|  af42     | 100100     |   
|  af43     | 100110     |   
|  be       | 000000     |    
|  cs1      | 001000     |   
|  cs2      | 010000     |    
|  cs3      | 011000     |    
|  cs4      | 100000     |   
|  cs5      | 101000     |    
|  cs6      | 110000     |    
|  cs7      | 111000     |    
|  ef       | 101110     |    
|  nc1      | 110000     |    
|  nc2      | 111000     |

### Junos EXP Alias Bit pattern

| Alias     | Bit pattern|
| ----------|------------|
|  af11     |   100      |     
|  af12     |   101      |      
|  be       |   000      |      
|  be1      |   001      |      
|  cs6      |   110      |      
|  cs7      |   111      |      
|  ef       |   010      |      
|  ef1      |   011      |      
|  nc1      |   110      |      
|  nc2      |   111      |

### SCHEME for DSCP to EXP  Bit pattern Mapping 
There is no strict rule for DSCP-to-EXP bit mapping; however, we can use the three most significant bits (MSBs) of the DSCP alias code to map it to the corresponding EXP alias where the 3 MSBs match. This approach allows the 23 DSCP alias codes to be effectively mapped to 10 EXP alias codes. 

| DSCP Alias     | DSCP Bit pattern| EXP Alias      | EXP Bit pattern|
| ---------------|-----------------|----------------|----------------|
|  af11          | 001010          | be1            |     001        |
|  af12          | 001100          | be1            |     001        |
|  af13          | 001110          | be1            |     001        |
|  af21          | 010010          | ef             |     010        |
|  af22          | 010100          | ef             |     010        |
|  af23          | 010110          | ef             |     010        |
|  af31          | 011010          | ef1            |     011        |
|  af32          | 011100          | ef1            |     011        |
|  af33          | 011110          | ef1            |     011        |
|  af41          | 100010          | af11           |     100        |
|  af42          | 100100          | af11           |     100        |
|  af43          | 100110          | af11           |     100        |
|  be            | 000000          | be             |     000        |
|  cs1           | 001000          | be1            |     001        |
|  cs2           | 010000          | ef             |     010        |
|  cs3           | 011000          | ef1            |     011        |
|  cs4           | 100000          | ef1            |     011        |
|  cs5           | 101000          | af12           |     101        |
|  cs6           | 110000          | cs6            |     110        |
|  cs7           | 111000          | nc2            |     111        |
|  ef            | 101110          | af12           |     101        |
|  nc1           | 110000          | nc1            |     110        |
|  nc2           | 111000          | nc2            |     111        |

### Forwarding Classes Resources Mapping
|Forwarding Class| DSCP Alias     | DSCP Bit pattern|Queue Number    | Transmit Rate   | Prioirty       | Buffer Size    | Remarks           |
|----------------| ---------------|-----------------|----------------|-----------------|----------------|----------------|-------------------|
| BE             |  be            | 000000          | 0              | 28              | Low            |     28         | Best Effort       |
| VOIP           |  ef            | 101110          | 1              | 10              | High           |     10         | VOIP              |
| Critical       |  af31          | 011010          | 2              | 50              | High           |     50         | Business Critical |
| NC             |  nc1           | 110000          | 3              | -               | Strict-High    |     1          | Network Control   |
| MM             |  af41          | 100010          | 4              | 10              | Medimum-Low    |     10         | Multimedia        |
| JUNK           |  cs1           | 001000          | 5              | 2               | Low            |     1          | Remaining traffic |

### DSCP to EXP Bit pattern Mapping 
|Forwarding Class| DSCP Alias     | DSCP Bit pattern| EXP Alias      | EXP Bit pattern|
|----------------| ---------------|-----------------|----------------|----------------|
| BE             |  be            | 000000          | be             |     000        |
| VOIP           |  ef            | 101110          | af12           |     101        |
| Critical       |  af31          | 011010          | ef1            |     011        |
| NC             |  nc1           | 110000          | nc1            |     110        |
| MM             |  af41          | 100010          | af11           |     100        |
| JUNK           |  cs1           | 001000          | be1            |     001        |

## Forwarding Classes Defination
```
class-of-service {
forwarding-classes {
    class BE queue-num 0;
    class CRITICAL queue-num 2;
    class NC queue-num 3;
    class JUNK queue-num 5;
    class MM queue-num 4;
    class VOIP queue-num 1;
}
}
```
## Schduler MAP

```
class-of-service {
scheduler-maps {
    SM_COS {
        forwarding-class BE scheduler SCH_BE;
        forwarding-class CRITICAL scheduler SCH_CRITICAL;
        forwarding-class NC scheduler SCH_NC;
        forwarding-class JUNK scheduler SCH_JUNK;
        forwarding-class MM scheduler SCH_MM;
        forwarding-class VOIP scheduler SCH_VOIP;
    }
}
schedulers {
    SCH_BE {
        transmit-rate percent 28;
        buffer-size percent 28;
        priority low;
    }
    SCH_CRITICAL {
        transmit-rate percent 50;
        buffer-size percent 50;
        priority high;
    }
    SCH_NC {
        buffer-size percent 1;
        priority strict-high;
    }
    SCH_JUNK {
        transmit-rate percent 1;
        buffer-size percent 1;
        priority low;
    }
    SCH_MM {
        transmit-rate percent 10;
        buffer-size percent 10;
        priority high;
    }
    SCH_VOIP {
        buffer-size percent 10;
        buffer-size percent 10;
        priority high;
    }
}
}
```

## Multifield Classification At Edge Interfaces 
If traffic received at the ingress LSR edge interfaces is not properly marked with the DSCP bit, or if we want to change those markings, we can apply a multifield classifier. Multifield classifier matches source and destination prefixes and sets the forwarding class as an action item in the firewall filter configuration.

```
firewall {
    family inet {
        filter mf_classfier {
            term BE {
                from {
                    source-address {
                        10.0.10.0/24;
                    }
                    destination-address {
                        10.0.11.0/24;
                    }
                }
                then {
                    forwarding-class BE;
                    dscp be;
                    loss-priority medium-low;
                    accept;
                }
            }
            term VOIP {
                from {
                    source-address {
                        10.0.20.0/24;
                    }
                    destination-address {
                        10.0.21.0/24;
                    }
                }
                then {
                    forwarding-class VOIP;
                    dscp ef;
                    loss-priority low;
                    accept;
                }
            }
            term Critical {
                from {
                    source-address {
                        10.0.30.0/24;
                    }
                    destination-address {
                        10.0.31.0/24;
                    }
                }
                then {
                    forwarding-class Critical;
                    dscp af31;
                    loss-priority low;
                    accept;
                }
            }
            term MM {
                from {
                    source-address {
                        10.0.30.0/24;
                    }
                    destination-address {
                        10.0.31.0/24;
                    }
                }
                then {
                    forwarding-class MM;
                    dscp af41;
                    loss-priority medium-high;
                    accept;
                }
            }
            term JUNK {
                from {
                    source-address {
                       0.0.0.0/0;
                    }
                    destination-address {
                        0.0.0.0/0;
                    }
                }
                then {
                    forwarding-class JUNK;
                    dscp cs1;
                    loss-priority high;
                    accept;
                }
            }
        }
    }
}
```

## BA Classfication At Edge Interfaces

If traffic is already marked with DSCP bits, the Behavior Aggregate (BA) classifier will map the traffic to the corresponding forwarding class by matching the appropriate DSCP alias code.

```
class-of-service {
classifiers {
    dscp CL_COS {
        import default;
        forwarding-class BE {
            loss-priority medium-low code-points be;
        }
        forwarding-class CRITICAL {
            loss-priority low code-points af31;
        }
        forwarding-class NC {
            loss-priority low code-points nc1;
        }
        forwarding-class JUNK {
            loss-priority high code-points cs1;
        }
        forwarding-class MM {
            loss-priority medium-high code-points af41;
        }
        forwarding-class VOIP {
            loss-priority low code-points ef;
        }
    }
}
}
```
## EXP Rewrite Rule
Traffic leaving the egress interfaces of the ingress or transit LSR will be marked according to the EXP rewrite rule, with code point aliases chosen based on the DSCP-to-EXP bit mapping described above.
```
class-of-service {
rewrite-rules {
    exp DSCP_EXP_REWRITE {
        import default;
        forwarding-class BE {
            loss-priority low code-point be;
        }
        forwarding-class CRITICAL {
            loss-priority low code-point ef1;
        }
        forwarding-class NC {
            loss-priority low code-point nc1;
        }
        forwarding-class JUNK {
            loss-priority low code-point be1;
        }
        forwarding-class MM {
            loss-priority low code-point af11;
        }
        forwarding-class VOIP {
            loss-priority low code-point af12;
        }
    }
}
}
```
## EXP Classfier
On the transit LSR, an EXP classifier will be applied on each ingress interface to classify incoming traffic based on the EXP bits already marked.

```
class-of-service {
    exp CL_EXP_COS {
        import default;
        forwarding-class BE{
            loss-priority low code-points be;
        }
        forwarding-class NC {
            loss-priority low code-points nc1;
        }
        forwarding-class JUNK {
            loss-priority low code-points be1;
        }
        forwarding-class MM {
            loss-priority low code-points af11;
        }
        forwarding-class VOIP {
            loss-priority low code-points af12;
        }
    }
}
```

## Applying Everything to Interfaces

Finally, we need to apply the scheduler map, rewrite rules, and BA classifiers on all interfaces.

```
class-of-service {
interfaces {
    et-* {
        scheduler-map SM_COS;
        unit * {
            classifiers {
                dscp CL-COS;
                exp CL_EXP_COS;
            }
            rewrite-rules {
                exp DSCP_EXP_REWRITE;
            }
        }
    }
    xe-* {
        scheduler-map SM_COS;
        unit * {
            classifiers {
                dscp CL-COS;
                exp CL_EXP_COS;
            }
            rewrite-rules {
                exp DSCP_EXP_REWRITE;
            }
        }
    }
    ae* {
        scheduler-map SM_COS;
        unit * {
            classifiers {
                dscp CL-COS;
                exp CL_EXP_COS;
            }
            rewrite-rules {
                exp DSCP_EXP_REWRITE;
            }
        }
    }
}
}
```



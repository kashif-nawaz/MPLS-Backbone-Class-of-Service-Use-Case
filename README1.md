# MPLS-Backbone-Class-of-Service-Use-Case

## Reference Document 
https://github.com/kashif-nawaz/MPLS-Backbone-Class-Of-Service-Design-Principles

## Use Case Requirments 
Designing and deploying Class of Service (CoS) in an MPLS backbone network is inherently more complex than in a pure IP or switching network. In an MPLS architecture, ingress Label Switch Routers (LSRs) classify traffic by analyzing packet characteristics at ingress interfaces, utilizing either multifield classification or behavior aggregate classification. This classification enables appropriate queuing of traffic through designated forwarding queues at egress interfaces. At the egress interfaces of the ingress LSP, the EXP bits are written to ensure that transit LSRs can classify packets based on these bits and forward them through the appropriate forwarding queues. Subsequently, the EXP bits must be rewritten again so that the next LSR or egress LSR can classify packets using the EXP classifier. 

Once traffic enters the egress LSR, the MPLS label is removed, and the traffic is forwarded to the Customer Edge (CE) facing interface via IP lookup. At this stage, it may be necessary to rewrite the Differentiated Services Code Point (DSCP) bits to maintain consistent traffic classification and prioritization at the CE routers.

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


### DSCP to EXP  Bit pattern Mapping 

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


### Lab Use Case DSCP  Bit pattern Mapping 
|Forwarding Class| DSCP Alias     | DSCP Bit pattern| Remarks           |
|----------------| ---------------|-----------------|-------------------|
| BE             |  be            | 000000          | Best Effort       |
| VOIP           |  ef            | 101110          | VOIP Calls        |
| Critical       |  af31          | 011010          | Business Critical |
| NC             |  nc1           | 110000          | Network Control   |
| MM             |  af41          | 100010          | Multimedia        |
| JUNK           |  cs1           | 001000          | Remaining traffic |


### Lab Use Case DSCP  to EXP Bit pattern Mapping 
|Forwarding Class| DSCP Alias     | DSCP Bit pattern| EXP Alias      | EXP Bit pattern|
|----------------| ---------------|-----------------|----------------|----------------|
| BE             |  be            | 000000          | be             |     000        |
| VOIP           |  ef            | 101110          | af12           |     101        |
| Critical       |  af31          | 011010          | ef1            |     011        |
| NC             |  nc1           | 110000          | nc1            |     110        |
| MM             |  af41          | 100010          | af11           |     100        |
| JUNK           |  cs1           | 001000          | be1            |     001        |


### Lab Use Case DSCP  to EXP Bit pattern Mapping 
|Forwarding Class| DSCP Alias     | Transmit Rate   | Prioirty       | Buffer Size    |
|----------------| ---------------|-----------------|----------------|----------------|
| BE             |  be            | 28              | Low            |     28         |
| VOIP           |  ef            | 10              | High           |     10         |
| Critical       |  af31          | 50              | High           |     50         |
| NC             |  nc1           | -               | Strict-High    |     1          |
| MM             |  af41          | 10              | Medimum-Low    |     10         |
| JUNK           |  cs1           | 2               | Low            |     1          |


## Forwarding Classes Defination
```
class-of-service {
forwarding-classes {
    class BEqueue-num 0;
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
        forwarding-class CRITICAL scheduler SCH-CRITICAL;
        forwarding-class NC scheduler SCH_NC;
        forwarding-class JUNK scheduler SCH_JUNK;
        forwarding-class MM scheduler SCH-MM;
        forwarding-class VOIP scheduler SCH_VOIP;
    }
}
schedulers {
    SCH_BE {
        transmit-rate percent 70;
        buffer-size percent 70;
        priority low;
    }
    SCH_CRITICAL {
        transmit-rate percent 5;
        buffer-size percent 5;
        priority high;
    }
    SCH_NC {
        buffer-size percent 5;
        priority strict-high;
    }
    SCH_JUNK {
        transmit-rate percent 5;
        buffer-size percent 5;
        priority low;
    }
    SCH-MM {
        transmit-rate percent 5;
        buffer-size percent 5;
        priority high;
    }
    SCH_VOIP {
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
                    accept;
                }
            }
        }
    }
}
```

## BA Classfication At Edge Interfaces
```
class-of-service {
classifiers {
    dscp CL-COS {
        import default;
        forwarding-class BE{
            loss-priority low code-points be;
        }
        forwarding-class CRITICAL {
            loss-priority low code-points af31;
        }
        forwarding-class NC {
            loss-priority low code-points nc1;
        }
        forwarding-class JUNK {
            loss-priority low code-points cs1;
        }
        forwarding-class MM {
            loss-priority low code-points af41;
        }
        forwarding-class VOIP {
            loss-priority low code-points ef;
        }
    }
}
}
```
## DSCP to EXP Rewrite Rule

```
class-of-service {
rewrite-rules {
    exp DSCP_EXP_REWRITE {
        import default;
        forwarding-class BE{
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



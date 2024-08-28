# SHM and SWDP


## Abstract

This is a brief study into the possibility of using SHM as part of the SWDP pipeline to continuously deploy and upgrade Packet Core nodes

| Document                            | Author  | Version |
|:------------------------------------|:--------|---------|
| Study On How To Use ENM SHM In SWDP | ebrigun | A     |

## Requirement
| JIRA                          | Author           | Comment                         |
|:------------------------------|:-----------------|---------------------------------|
| [JIRA SWDSDEB-1014][80b4073e] | Annica Söderlund | Investigation on behalf of SWDP |



## SHM in ENM

The software listed in this section are all installed as part of ENM.

Software and Hardware Management (SHM) today supports automatic upgrade of Radio Nodes (ECIM) only. SHM periodically checks CAS-C for new node software. If it detects new software, it will automatically import it and store the data in Software Management Repository Service (SMRS) for use.

Once this procedure is complete a person will manually trigger Automatic Software Upgrade (ASU) to upgrade the node. ASU is a Camunda based workflow orchestrator that will monitor the ongoing upgrade of the node(s).



## SHM and Packet Core Nodes

In order to use SHM Packet Core have the following options

1. ECIM Fragment Option *(preferred)*
  - vMME / SGSN-MME already implement this and are fully SHM compliant
  - Other packet core nodes (ECIM based) would be required to implement the ECIM fragment to integrate with SHM. The means implementing an interface specification (over NETCONF). SHM would have a minor task to integrate each new node
  - Possible reuse of existing Component Based Architecture fragment implementation by vMME / SGSN-MME to ease this implementation
  - Would require changes in ASU (Automatic Software Upgrade) to support
    - Currently only supports baseband radio as ASU currently only supports Baseband Radio nodes
    - This work not currently planned but would not be significant for ECIM based nodes
  - ASU provides the possibility of hook points to run scripts or bespoke tasks etc. if required
  - Supports IPv4 and IPv6 nodes
2. ~~ASU Direct Option~~
  - Bypass SHM and directly use ASU
  - Could be done through use of scripting "building block files"
  - Would create dependency on ENM Scriptiing cluster / AMOS
  - Each node could implement their own Workflow
  - Impact on AMOS scripting nodes unknown
  - **Does not the solve issue of retrieving new software**
3. ~~EVNFM Option~~
  - **Does not the solve issue of retrieving new software**
  - Involves tearing down node instances and starting new ones rather than a software upgrade
4. ~~Run SHM/ASU outside of ENM~~
  - Never properly investigated
  - Likely to be very significant work if technically possible

## Open Questions

| Question                                  | Answer                                               | Comment                                   |
|:------------------------------------------|:-----------------------------------------------------|-------------------------------------------|
| Can ASU be auto triggered.                | Yes, it exposes REST API                             |                                           |
| Can existing ASU workflows be used by PC nodes | Existing workflows could be reused with modification | Modifications mainly around health checks |
|  Overal solution to be either SHM based, ansible based or both                                         |   UNRESOLVED                                                   |                                           |

### Summary
Of the options listed above the only valid one would be *full integration with SHM by implementing a fragment*.

An open question remains as to whether the strategy is to use the existing ansible based solution, integrate with SHM or support both mechanisms.

## YANG versus Legacy
The strategy is to support both YANG and legacy. The current plan is to implement for legacy nodes first and then YANG based.


### Pros/Cons

#### Overall Solution
| Pros                        | Cons                       | Comment           |
| :-------------              | :-------------             | :-------          |
| Simple Integration          | Creates dependency on ENM  |                   |
| Support IPv4 & IPv6         | ASU work required to support |                            |
| Supports Trusted & Untrusted|                            |SeGW required for untrusted|
| Already supports vMME/SGSN-MME  |                        |                   |


## Recommendation

If integration with SHM  is chosen the *Fragment Option* above is the most feasible.

## Notes
n/a

## Acronyms and Explanations

| Acronym | Description                                             |
|:--------|:---------------------------------------                 |
| SHM     | Software Hardware Manager                               |
| CAS-C   | Ericsson's Secure File Storage Service on Customer Site |
| SMRS    | Software Management Repository Service                  |
| ASU     | Automatic Software Upgarde                              |
| ENM     | Ericsson Network Manager                                |
| Camunda | BPMN Based Workflow Engine                              |
| BPMN    | Business Process Model and Notation                     |
| EVNFM   | Ericsson VNF Manager                                    |
| VNF     | Virtual Network Function                                |
| Packet Core   | IP based core network defined by 3GPP             |
| 3GPP    | Standards Organization for Mobile Telephony             |
| SWDP    | Software Development Pipeline                           |
| ECIM    | Ericsson Common Information Model                       |
| ECIM Fragment   | ECIM interface Specification                    |
| SWYM    | SHM Software ECIM Fragment                              |
| HWYM    | SHM Hardware ECIM Fragment                              |
| SGSN-MME   | Serving GPRS Support Node - Mobile Management Entity |

<!-- links, this section not rendered-->
[80b4073e]: https://cc-jira.rnd.ki.sw.ericsson.se/browse/SWDSDEB-1014 "Requirement in Jira"

---

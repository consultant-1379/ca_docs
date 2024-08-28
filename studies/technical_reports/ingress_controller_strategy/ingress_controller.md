# Ingress Controller Strategy Document

[toc]



# Revision History

| Revison | Author  | Comment              |
| ------- | ------- | -------------------- |
| PA1     | ebrigun |                      |
| A       | ebrigun | Updated after TC-OSS |
|         |         |                      |

## TA/JIRA

| JIRA          | Reference                                                    |
| ------------- | ------------------------------------------------------------ |
| OSSEPTAP-9403 | [JIRA](https://jira-oss.seli.wh.rnd.internal.ericsson.com/browse/OSSEPTAP-9403?page=com.atlassian.jira.plugin.system.issuetabpanels%3Aall-tabpanel) |
|               |                                                              |
|               |                                                              |

## Abstract

The general OSS Strategy is to use ADP services when available and practicable, Ingress Controller strategy is no different. 

Adoption of ADP services are generally complicated in Lift and Shift scenarios as often existing or competing technologies exist, sometimes exact like for like comparisons are not simple and often it does not make any business  or technical sense to swap out the existing service for the ADP Services. All of the above combined with product time-lines often influence technology adoption decisions.

This document will describe the state of play with regard to Ingress Controller in OSS products and explain and document technical decisions this far.



## ADP/Common Services: Available Technologies

### L7 Ingress Controller

ADP provides [ICCR](https://adp.ericsson.se/marketplace/ingress-controller-cr) based on Contour as an off the shelf Layer 7 Ingress Controller

**Description:** The Ingress Controller CR service provides methods of load-balancing L7 traffic towards services inside the COP. The protocols supported are HTTP/HTTPS (v1.1 and v2.0). The service acts as a reverse proxy, and can also manage Transport Layer Security (TLS) in the COP cluster.

### L4 Ingress Controller

PDU OSS have developed  [EricIngress](https://adp.ericsson.se/marketplace/ericingress-l4-dynamic-load-balancer) as a L4 ingress Controller. 

**Description:** Expose all your services by a single VIP on public or private cloud. Declaratively expose a service and apply routing algorithms at deploy time to L4 traffic. Seamlessly pass on L7 traffic to L7 Ingress Controller if desired. The solution is NAT based.

### API Gateway

PDU OSS have developed and contributed the Spring based [API Gateway](https://adp.ericsson.se/marketplace/api-gateway) as an OSS common service to the ADP marketplace. At the time of writing this is the only API gateway available in the ADP marketplace.

**Description:** Gateway has the capability to secure web service calls (OAuth with keycloak), route the requests to registered micro services & rate limit the requests

### Service Mesh

ADP contains a range of Istio based Service Mesh technologies including [Service Mesh Gateways](https://adp.ericsson.se/marketplace/servicemesh-gateways)

**Description:** ServiceMesh Gateways is a realization of an architectural component SM Proxy in Service Mesh FA. This service is used to configure one ore more load balancer for HTTP/TCP traffic operating at the edge of the mesh.

### Recommendations

Recommending an ingress controller strategy is not as clear-cut a choice as choosing one particular SQL database implementation over another, this is especially true in lift and shift applications that bring a variety of use cases, protocols and often technical debt. Ingress Controllers have a wide variation in what protocols they support, their intended use case, availability, upgrade options and their footprint.  Applications similarly often have differing requirements, some requirements that sometimes cannot be met by a single solution or have particular footprint restrictions. Care should be taken to limit the bloat in an application footprint when choosing a solution as this is a known pain point with ADP technologies. That said a number of broad guidelines can be suggested. 

The current OSS and BDGS strategy is to use an ADP solution when it exists and the ingress controller strategy is no different. If an application's need is not completely met by one of the options below the first option should be to put a requirement on the technology options below or consider a code change before an alternative technology is sought. Selection of one solution below is not mutually exclusive of another option and a combination of solutions can sometimes be chosen e.g. to satisfy L4 and L7 requirements a combination of ICCR and EricIngress could be used.

In addition, for brown field or lift and shift developments containerisation or porting of existing ingress solutions may be possible. In these scenarios a cost-benefit analysis may be done by the application concerned before a choice is made

#### L7 Ingress

For Layer 7 Ingress traffic in a greenfield development the choice is clear. The ADP chosen solution is ICCR (based on Contour) and this should be chosen.

As alluded to in the above section when migrating a lift-and-shift application sometimes complications and technical blockers may exist. In these scenarios ICCR should be evaluated and used if applicable but it may be necessary to use a different technology.

#### L4 Ingress

For Layer 4 Ingress traffic OSS have developed EricIngress. This is a highly available solution with the ability to apply different routing  per back-end service  and expose over a VIP. This is the preferred solution for L4 Traffic.

#### Deployment

OSS strategy is to package, deliver and install it's own ingress controller where possible and necessary. This is because we are aiming to remain as agnostic of underlying CaaS implementations as possible.

#### API Gateway

If your application requires an API Gateway EO have contributed a Spring based API Gateway OSS Common Service. This is the preferred solution if an application requires an API Gateway

#### Service Mesh

If an application uses ADP's service mesh solution it has a complimentary Istio based API Gateway which includes Ingress Controller functionality. It is not  recommended to use this option unless using service mesh or planning to introduce it.

## OSS Status

### HTTP Ingress

Compliance

#### Compliance

##### ENM

ENM started off initially using the nginx controller bundled with CCD had migrated it's code to use ICCR (Contour) as the recommended ADP service but had not made the final switch over/merge. Testing discovered issues regarding lack of support for  mTLS (when using Ingress instead of HTTPProxy), and for the scripting use cases session affinity and session persistence. 

After consultation with ICCR teams ENM discovered that the issues could not all be solved without significant code changes to the Contour public code base and there was no appetite  or plan internally or in the community to do this or timeline when this might be achieved. Even if this was the case significant questions remained as to the maintenance and support of these changes. In addition to this, CCD have plans to remove the embedded nginx instance that is delivered with it so that is no longer a viable option. After a presentation of this issue to OSS Architecture Board the decision was made to revert to nginx and to deliver it as service. Consultations are ongoing in OSS on who will deliver this service.

At the time of writing ENM has no plans to migrate to service mesh as ENM/Service Framework delivers a lot of the mesh benefits already. New requirements around secure pod to pod communications may alter this position

##### EO

Similarly, EO started to use the CCD bundled ingress controller for convenience and did not intend to migfarte to Contour as it attempted to consider a longer term strategy maybe involving service mesh and service mesa gateway (with ingress controller functionality). In scenarios where EO will be deployed on a customer CaaS EO intended to place a requirement on the customer to provide a compatible version of nginx.  

1. EO currently use  the Spring based [API Gateway](https://adp.ericsson.se/marketplace/api-gateway). This is an OSS common service contributed by EO
2. EO plans (long term) to introduce ADP's Istio based [service mesh solution](https://adp.ericsson.se/marketplace/?search=istio&groupby=reusability_level) at some point in the future
3. Istio has a complimentary Service Mesh [API Gateway](https://adp.ericsson.se/marketplace/servicemesh-gateways)  available in the ADP marketplace that can function as an Ingress Controller 

CCD's decision to deprecate the embedded nginx ingress controller in favour of Contour has accelerated EO's need to decide a migration strategy. Its is currently evaluating if it can migrate to Contour.

###### Migration Strategy

EO have not committed to a migration strategy yet. 

###### Options

| Option                                        | Pros                                                         | Cons                                                         | Comment                                                      |
| --------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Migrate to Contour immediately                | Removes dependency on CCD<br />Removes integration steps on customer CaaS <br />Removes the risk of CCD removing NGINX before we are ready<br />Aligned with overall OSS Strategy<br />ICCR deployment on Openshift proven | Work may need to be redone if Istio GW is introduced<br />Limited Capacity to migrate now<br />Unknown migration impacts <br />No proven integration between Service Mesh and Contour<br /><br />Cost to migrate unknown<br />ICCR provides a way of supporting large files but this has never been tested and will require a spike | ~~Update: Ruled out as EO will use OSS Nginx as per above~~<br />EO evaluating Contour due to CCD deprecation |
| Wait and migrate to Istio Service Mesh and GW | Single complete solution backed by single 3PP<br />ICCR deployment on Openshift proven<br />Aligned with overall OSS Strategy<br /> | Continued dependency on CCD and customer integration workarounds<br />No guarantees that if Service mesh is introduced the SM G/W will follow<br />No time-line for introduction<br />CCD plan to remove NGINX so EO timeline accelerated<br />Unclear what level of support Istio GW has for large files, k8s distros other than CCD etc | Not planned immediately                                      |
| Migrate to OSS Nginx                          |                                                              |                                                              | Chosen for ENM due to legacy technical reasons               |



### Layer 4 Ingress

##### ENM

ENM has specific L4 ingress requirements. It needs to be able to apply specific routing algorithms (source hashing, round robin, least connection) to specific connections based on the backend service. This is especially important for the source hashing connections that rely on the "sticky session". This required ENM to develop it's own ingress solution for Layer 4 traffic called [EricIngress](https://adp.ericsson.se/marketplace/ericingress-l4-dynamic-load-balancer). This handles all ingress Layer 4 traffic. It works in tandem with Contour to provide full ingress solution



### API Gateway

##### EO

EO have contributed an [API Gateway](https://adp.ericsson.se/marketplace/api-gateway) as an  OSS common service to the ADP marketplace. This is the only standalone gateway so this is the clear technology choice.

ADP also provides Service Mesh [API Gateway](https://adp.ericsson.se/marketplace/servicemesh-gateways) which can be used in conjunction with  ADP's [service mesh solution](https://adp.ericsson.se/marketplace/?search=istio&groupby=reusability_level) if implementing a service mesh.

Some feature of the API Gateway above an Ingress Controller

- Match routes on request attribute
- Predicates and filters are specific to routes
- Circuit Breaker integration
- Request Rate Limiting
- Path Rewriting
- Authentication

##### ENM

ENM has no requirement for an API gateway at present or in the medium term.




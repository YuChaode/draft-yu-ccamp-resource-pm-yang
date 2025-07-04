---
title: "A YANG Data Model for Resource Performance Monitoring"
abbrev: "Resource PM YANG"
category: std

docname: draft-yu-ccamp-resource-pm-yang-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Common Control and Measurement Plane"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Common Control and Measurement Plane"
  type: "Working Group"
  mail: "ccamp@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/ccamp/"
  github: "YuChaode/draft-yu-ccamp-resource-pm-yang"
  latest: "https://YuChaode.github.io/draft-yu-ccamp-resource-pm-yang/draft-yu-ccamp-resource-pm-yang.html"

author:
  -
    fullname: Chaode Yu
    organization: Huawei Technologies
    email: yuchaode@huawei.com
  -
    name: Fabio Peruzzini
    org: FiberCop
    email: fabio.peruzzini@fibercop.com
  -
    name: Yanlei Zheng
    org: China Unicom
    email: zhengyanlei@chinaunicom.cn
  -
    name: Italo Busi
    org: Huawei Technologies
    email: italo.busi@huawei.com
  -
    name: Aihua Guo
    org: Futurewei Technologies
    email: aihuaguo.ietf@gmail.com
  -
    name: Victor Lopez
    org: Nokia
    email: victor.lopez@nokia.com
  -
    name: Xing Zhao
    org: CAICT
    email: zhaoxing@caict.ac.cn
  -
    name: Mingshuang Jin
    org: Huawei Technologies
    email: jinmingshuang@huawei.com

normative:
  TMF-518:
    title: Resource Performance Management
    author:
      org: TM Forum (TMF)
    date:  2011
    seriesinfo: TMF518_RPM
    target: https://www.tmforum.org/resources/collection/mtosi-4-0
  ITU-T_G.874:
    title: Management aspects of optical transport network elements
    author:
      org: International Telecommunication Union
    date:  October 2020
    seriesinfo: ITU-T Recommendation G.874
    target: https://www.itu.int/rec/T-REC-G.874/en

--- abstract

This document defines a YANG data model for resource Performance Monitoring, applicable to network controllers,  which provides the functionalities of retrieval of performance monitoring capabilities, TCA (Threshold Crossing Alert) configuration, current or history performance data retrieval, and performance monitoring task management.

--- middle

# Introduction

Resource performance monitoring is a basic function of network management. By viewing and analyzing the performance data of different resources (such as network element, interface, board, termination point, tunnel termination point) operators can detect the running state of the network in time, quickly resolve real-time problems or identify major risks in advanced , avoiding users' complaints.

According to the business requirements stated in {{TMF-518}}, resource performance monitoring requirements include:

- Retrieval of current and historical performance measurements for network resources
- Distribution of Threshold Crossing Alerts (TCAs) to the collectors of PM data
- Control of performance monitoring in the network: enable/disable of PM collection and TCA generation


Currently, there are some existing documents related to performance monitoring in IETF, but there is no overlap with our current work. The relative monitored objects are summarized in Table 1.

| YANG Module | Performance-Monitored Object |Reference |
| ------ | --------------------------------- |--------- |
|ietf-te-telemetry, ietf-vn-telemetry| TE tunnel, virtual network| {{?I-D.ietf-teas-actn-pm-telemetry-autonomics}} |
|ietf-network-vpn-pm| VPN service topology | {{!RFC9375}} |
|ietf-service-pm|Transport network service|{{?I-D.zheng-ccamp-client-pm-yang}}|
|ietf-lmap-control, ietf-lmap-report|massive measurement agents in broadband service|{{!RFC8194}}|


{{?I-D.ietf-teas-actn-pm-telemetry-autonomics}} provides a YANG data model that describes performance monitoring and scaling intent mechanisms for TE-Tunnels and Virtual Networks(VNs). {{?I-D.ietf-opsawg-yang-vpn-service-pm}} defines a YANG data model for performance monitoring of both network topology layer and overlay VPN service topology layer. {{?I-D.zheng-ccamp-client-pm-yang}} provides a performance monitoring YANG data model on client signal level. {{!RFC8194}} defines a data model for Large-Scale Measurement Platforms(LMAP), focusing on  task capability and configuration of massive measurement agents.

This document defines a YANG module for resource performance monitoring, which defines the capabilities of resource performance monitoring, the tca configuration model of a specific resource. In addition, the sub-module of monitoring task and a few RPCs are defined to support the operations of performance monitoring, such as data retrieval and controlling the monitoring tasks.

The YANG data model defined in this document conforms to the Network Management Datastore Architecture (NMDA) defined in {{!RFC8342}}.

## Terminology and Notations
Refer to {{!RFC7446}} and {{!RFC7581}} for the key terms used in this document.  The following terms are defined in {{!RFC7950}} and are not redefined here:
*  client
*  server
*  augment
*  data model
*  data node

The following terms are defined in {{!RFC6241}} and are not redefined here:
*  configuration data
*  state data

The following terms are defined in {{?RFC8454}} and are not redefined here:
*  CMI
*  MPI
*  MDSC
*  CNC
*  PNC

> To Be Added: some explanation of performance indicator

## Tree Diagram
A simplified graphical representation of the data model is used in Section 3 of this document.  The meaning of the symbols in these diagrams are defined in {{!RFC8340}}.


## Prefix in Data Node Names
In this document, names of data nodes and other data model objects are prefixed using the standard prefix associated with the corresponding YANG imported modules, as shown in the following table.

| Prefix | Yang Module                       | Reference |
| ------ | --------------------------------- | --------- |
| rpm | ietf-resource-pm          | RFC XXXX  |
|rpm-types| ietf-resource-pm-types    | RFC XXXX  |
| yang   | ietf-yang-types                   |{{!RFC6991}}|
{: #tab-prefixes title="Prefixes and corresponding YANG modules"}

RFC Editor Note:
Please replace XXXX with the RFC number assigned to this document.

# YANG Data Model for Resource Performance Monitoring

## Capabilities of Resource Performance Monitoring
### Supported Resources and Corresponding PM Capabilities
A generic "resources" structure is defined to describe the objects that could be monitored:

* "resource":  the identifier of a monitored object
* "resource-type": indicate what exact kind of resource is
* "holding time": the longest time period that performance data could be monitored
* "indicator name": the indicators that could be monitored for the resource
* "sub-resources": the identifier of the performace monitoring point of this resource (e.g., if the resource is a network element, the sub resources may be a list of termination points)


~~~~ ascii-art
module: ietf-resource-pm
   +--rw performance-monitoring
      +--rw resources
         +--rw resource-list* [resource]
            +--rw resource             union
            +--ro resource-type?       identityref
            +--ro holding-time?     uint8
            +--ro indicator-name*   string
            +--ro sub-resources*    -> ../resource
~~~~

### Introduction of Performance Indicators
It is impossible to list all the PM indicator exhaustively, even if ITU-T and IETF has done a lot of work.  Some new performance indicators would be raised once there are some new requirements and technologies. So in this document we would like to provide String type rather than an explicit type for performance indicator, to have a better compatibility for future extension. Then if there are some new indicators, there is no need to revise this document or create a branch of documents to standardize the PM indicators.

For optical network, a performance management information table in the session10.2 of {{ITU-T_G.874}} lists all the PM current data and history data collected on the EMF(Equipment Management Function). A few indicators of different resources are listed below as an example.

// To be added: performance indicators of different resources in optical network.

For IP network, a few indicators of different resources are listed below as an example.
For a network element, following are possible indicators:
* CPU usage.
* Memory usage.
* Total active routes.
* Total active mac entries.

For an interface, following are possible indicators:
* Ingress unicast packets.
* Ingress multicast packets.
* Ingress discard packets.
* Ingress error packets.
* Ingress unknown protocol packets.
* Egress unicast packets.
* Egress multicast packets.
* Egress discard packets.
* Egress error packets.

For a board, following are possible indicators:
* CPU usage.
* Memory usage.

## Threshold Crossing Alert Control
Threshold crossing alert control parameters could be set directly for a resource, or set through applying an existing profile to the resource. Therefore, there are four main requirements for Threshold Crossing Alert Control:

* Creation/retrieval/deletion/updating of TCA profile;
* Enabling/disabling TCA reporting on the resource;
* Configuring TCA on the resource by associating an existing profile;
* Configuring TCA on the resource by detailed parameters.

To satisfy the above requirements, the module defines "tca-management", including the "profile" structure to enable the preset of tca parameters, "tca" structure to describe the tca parameters (directly set or preset by applying profiles) and tca status of a specific resource.

And for the TCA parameters, no matter it is configured directly on the resource or by a preset profile, there should not be any differences. The TCA parameters (tca-indicator) should include:

* Threshold-type: This threshold type is used to indicate when the alert will be triggered.  By exceeding the upper bound value, or by below the lower bound value.
* Period: This period is used to indicate the frequency of the data collection.
* Severity: This severity is used to indicate what level of alert would be triggered if cross the threshold.
* Indicator-value: The value of threshold.
* Indicator-value-unit: The unit of threshold value.

In addition, the function of enabling/disabling TCA on the resource can be controlled by the "admin-status" attribute in "tca" node. The pre-defined profiles with unique profile id could be applied to "tca" node, shown as "applied-profiles" in "tca" node.

~~~~ ascii-art
module: ietf-resource-pm
  +--rw performance-monitoring
     +--rw tca-management
     |  +--rw profiles
     |  |  +--rw profile* [profile-id]
     |  |     +--rw profile-id      yang:uuid
     |  |     +--rw profile-name?   string
     |  |     +--rw tca-cfg
     |  |        +--rw tca-indicator* [indicator-name threshold-type period severity]
     |  |           +--rw indicator-name          string
     |  |           +--rw indicator-value         string
     |  |           +--rw indicator-value-unit    string
     |  |           +--rw threshold-type          enumeration
     |  |           +--rw period                  identityref
     |  |           +--rw severity                identityref
     |  +--rw tcas
     |     +--rw tca* [resource]
     |        +--rw resource            -> /performance-monitoring/resources/resource-list/resource
     |        +--ro resource-type?      identityref
     |        +--rw admin-status?       enumeration
     |        +--rw applied-profiles
     |        |  +--rw profile* [profile-id]
     |        |     +--rw profile-id    -> ../../../../../profiles/profile/profile-id
     |        +--rw tca-cfg
     |           +--rw tca-indicator* [indicator-name threshold-type period severity]
     |              +--rw indicator-name          string
     |              +--rw indicator-value         string
     |              +--rw indicator-value-unit    string
     |              +--rw threshold-type          enumeration
     |              +--rw period                  identityref
     |              +--rw severity                identityref
~~~~

## Data Retrieval of Resource Performance Monitoring

### Get Current/History Performance Monitoring Data
For the retrieval of current/history performance data, we consider these data are not suitable to define in a data model.  Because performance data can be changed frequently and if we follow that approach, according to the requirement of on-change notification in YANG-push {{?RFC8641}}, once the performance data changes, the controller should trigger a notification to the northbound system, there would be great number of notifications reported in the big network.

These two retrieval interfaces are usually invoked on-demand. It is also hard to support retrieving performance data of multiple resources by data model in one request. And for history performance data retrieval, there could be a requirement to specify the start time and end time.  It is not quite flexible to support this requirement by data model neither. So we suggest to define an RPC to accomplish these two functions.

~~~~ ascii-art
  rpcs:
    +---x get-pm-data
    |  +---w input
    |  |  +---w resources*                     -> /performance-monitoring/resources/resource-list/resource
    |  |  +---w is-requesting-history-data?   boolean
    |  |  +---w start-time?                    yang:date-and-time
    |  |  +---w end-time?                      yang:date-and-time
    |  +--ro output
    |     +--ro pm-data
    |        +--ro pm-data-list* [resource]
    |           +--ro resource          -> /performance-monitoring/resources/resource-list/resource
    |           +--ro task-id*          yang:uuid
    |           +--ro collect-time?     yang:date-and-time
    |           +--ro resource-type?    identityref
    |           +--ro indicator-data
    |              +--ro indicator-data-list* [indicator-name]
    |                 +--ro indicator-name          string
    |                 +--ro indicator-value?        string
    |                 +--ro indicator-value-unit?   string
~~~~

### Get Profile Associated Resources
For the TCA related definition, it can be found in the previous Section (Section2.2). A TCA profile can be associated with a lot of resources, so we don't defined a resource list under the profile instance to avoid reporting some unnecessary notifications while the resource instances in this list have been changed. We define an RPC operation to support this flexible retrieval request.

~~~~ ascii-art
  rpcs:
    +---x get-profile-associated-resources
       +---w input
       |  +---w profile-id?   -> /performance-monitoring/tca-management/profiles/profile/profile-id
       +--ro output
          +--ro resource-list*   -> /performance-monitoring/resources/resource-list/resource
~~~~

## Controlling of Resource Performance Monitoring

### Clear Performance Monitoring Data of Specific Resources
We define an RPC to enable clearing the performance monitoring data of specified resources. If there are any resources failed to clear performance monitoring data, their identifier should be returned by the failed-resources leaf list. An empty list indicates that this operation was performed successfully.

~~~~ ascii-art
  rpcs:
    +---x clear-performance-monitoring-data
    |  +---w input
    |  |  +---w resources*   -> /performance-monitoring/resources/resource-list/resource
    |  +--ro output
    |     +--ro failed-resources*   -> /performance-monitoring/resources/resource-list/resource
~~~~

### Enable/Disable Performance Monitoring
To enable/disable performance monitoring data, we introduce a monitor task to do this control. In the monitoring task, the resource, the monitored period, the monitored indicators could be set. The disabling, enabling operation can be satisfied by changing the admin status which includes disabled, enabled. The change's result will affect the task status accordingly.

~~~~ ascii-art
   module: ietf-resource-pm
      +--rw performance-monitoring
         +--rw monitor-tasks
         |  +--rw monitor-task* [task-id]
         |     +--rw task-id          yang:uuid
         |     +--rw resource-id?     leafref
         |     +--ro resource-type?   identityref
         |     +--rw task-name?       string
         |     +--rw admin-status?    enumeration
         |     +--ro task-status?     enumeration
         |     +--rw task-cfg
         |        +--rw period?       identityref
         |        +--rw indicators
         |           +--rw indicator* [indicator-name]
         |              +--rw indicator-name          string
         |              +--rw indicator-value-unit?   string
~~~~

# Resource Performance Monitoring Tree Diagram

~~~~ ascii-art
{::include-fold yang/ietf-resource-pm.tree}
~~~~
{: #fig-rpm-tree title="Resource Performance Monitoring tree diagram"
artwork-name="ietf-resource-pm.tree"}

# YANG Model for Resource Performance Monitoring

~~~~ yang
{::include yang/ietf-resource-pm.yang}
~~~~
{: #fig-rpm-yang title="Resource Performance Monitoring YANG module"
sourcecode-markers="true" sourcecode-name="ietf-resource-pm@2025-07-04.yang"}

# YANG Model for Resource Performance Monitoring Types

~~~~ yang
{::include yang/ietf-resource-pm-types.yang}
~~~~
{: #fig-rpm-type-yang title="Resource Performance Monitoring Types YANG module"
sourcecode-markers="true" sourcecode-name="ietf-resource-pm-types@2025-07-04.yang"}

# Manageability Considerations

  \<Add any manageability considerations>

# Security Considerations

The YANG module specified in this document defines a schema for data that is designed to be accessed via network management protocols such as NETCONF {{!RFC6241}} or RESTCONF {{!RFC8040}}. The lowest NETCONF layer is the secure transport layer, and the mandatory-to-implement secure transport is Secure Shell (SSH) {{!RFC6242}}. The lowest RESTCONF layer is HTTPS, and the mandatory-to-implement secure transport is TLS {{!RFC8446}}.

The NETCONF access control model {{!RFC8341}} provides the means to restrict access for particular NETCONF or RESTCONF users to a preconfigured subset of all available NETCONF or RESTCONF protocol operations and content.

There are a number of data nodes defined in this YANG module that are writable/creatable/deletable (i.e., config true, which is the default). These data nodes may be considered sensitive or vulnerable in some network environments. Write operations (e.g., edit-config) to these data nodes without proper protection can have a negative effect on network operations. Considerations in Section 8 of {{!RFC8795}}are also applicable to their subtrees in the module defined in this document.

Some of the readable data nodes in this YANG module may be considered sensitive or vulnerable in some network environments. It is thus important to control read access (e.g., via get, get-config, or notification) to these data nodes. Considerations in Section 8 of {{!RFC8795}} are also applicable to their subtrees in the module defined in this document.

# IANA Considerations

This document registers following YANG modules in the YANG Module Names registry {{!RFC6020}}.


   name:         ietf-resource-pm
   namespace:    urn:ietf:params:xml:ns:yang:ietf-resource-pm
   prefix:       dvcrpm
   reference:    RFC XXXX: A YANG Data Model for Resource Performance Monitoring

   name:         ietf-resource-pm-types
   namespace:    urn:ietf:params:xml:ns:yang:ietf-resource-pm-types
   prefix:       dvcrpm-types
   reference:    RFC XXXX: A YANG Data Model for Resource Performance Monitoring


--- back

# Acknowledgments
{: numbered="false"}

This document was prepared using kramdown.


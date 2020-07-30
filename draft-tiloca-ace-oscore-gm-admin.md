---
title: Admin Interface for the OSCORE Group Manager
abbrev: Admin Interface for the OSCORE GM
docname: draft-ietf-ace-oscore-gm-admin-latest


# stand_alone: true

ipr: trust200902
area: Internet
wg: ACE Working Group
kw: Internet-Draft
cat: std

coding: us-ascii
pi:    # can use array (if all yes) or hash here

  toc: yes
  sortrefs:   # defaults to yes
  symrefs: yes

author:
      -
        ins: M. Tiloca
        name: Marco Tiloca
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: marco.tiloca@ri.se
      -
        ins: R. Hoeglund
        name: Rikard Hoeglund
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: rikard.hoglund@ri.se
      -
        ins: P. van der Stok
        name: Peter van der Stok
        org: Consultant
        phone: +31-492474673 (Netherlands), +33-966015248 (France)
        email: consultancy@vanderstok.org
        uri: www.vanderstok.org
      -
        ins: F. Palombini
        name: Francesca Palombini
        org: Ericsson AB
        street: Torshamnsgatan 23
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: francesca.palombini@ericsson.com
      -
        ins: K. Hartke
        name: Klaus Hartke
        org: Ericsson AB
        street: Torshamnsgatan 23
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: klaus.hartke@ericsson.com

normative:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-ace-key-groupcomm:
  I-D.ietf-ace-key-groupcomm-oscore:
  I-D.ietf-ace-oscore-profile:
  I-D.ietf-core-coral:
  I-D.ietf-cose-rfc8152bis-struct:
  I-D.ietf-cose-rfc8152bis-algs:
  RFC2119:
  RFC3986:
  RFC6690:
  RFC6749:
  RFC7049:
  RFC7252:
  RFC7641:
  RFC8174:
  RFC8613:
  COSE.Algorithms:
    author: 
      org: IANA
    date: false
    title: COSE Algorithms
    target: https://www.iana.org/assignments/cose/cose.xhtml#algorithms
  COSE.Key.Types:
    author: 
      org: IANA
    date: false
    title: COSE Key Types
    target: https://www.iana.org/assignments/cose/cose.xhtml#key-type
  COSE.Elliptic.Curves:
    author: 
      org: IANA
    date: false
    title: COSE Elliptic Curves
    target: https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves

informative:
  I-D.dijk-core-groupcomm-bis:
  I-D.ietf-ace-dtls-authorize:
  I-D.ietf-tls-dtls13:
  I-D.ietf-core-resource-directory:
  I-D.tiloca-core-oscore-discovery:
  I-D.hartke-t2trg-coral-reef:
  RFC6347:
  RFC8259:

--- abstract

Group communication for CoAP can be secured using Group Object Security for Constrained RESTful Environments (Group OSCORE). A Group Manager is responsible to handle the joining of new group members, as well as to manage and distribute the group key material. This document defines a RESTful admin interface at the Group Manager, that allows an Administrator entity to create and delete OSCORE groups, as well as to retrieve and update their configuration. The ACE framework for Authentication and Authorization is used to enforce authentication and authorization of the Administrator at the Group Manager. Protocol-specific transport profiles of ACE are used to achieve communication security, proof-of-possession and server authentication.

--- middle

# Introduction # {#intro}

The Constrained Application Protocol (CoAP) {{RFC7252}} can be used in group communication environments where messages are also exchanged over IP multicast {{I-D.dijk-core-groupcomm-bis}}. Applications relying on CoAP can achieve end-to-end security at the application layer by using Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}}, and especially Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} in group communication scenarios.

When group communication for CoAP is protected with Group OSCORE, nodes are required to explicitly join the correct OSCORE group. To this end, a joining node interacts with a Group Manager (GM) entity responsible for that group, and retrieves the required key material to securely communicate with other group members using Group OSCORE.

The method in {{I-D.ietf-ace-key-groupcomm-oscore}} specifies how nodes can join an OSCORE group through the respective Group Manager. Such a method builds on the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}, so ensuring a secure joining process as well as authentication and authorization of joining nodes (clients) at the Group Manager (resource server).

In some deployments, the application running on the Group Manager may know when a new OSCORE group has to be created, as well as how it should be configured and later on updated or deleted, e.g. based on the current application state or on pre-installed policies. In this case, the Group Manager application can create and configure OSCORE groups when needed, by using a local application interface. However, this requires the Group Manager to be application-specific, which in turn leads to error prone deployments and is poorly flexible.

In other deployments, a separate Administrator entity, such as a Commissioning Tool, is directly responsible for creating and configuring the OSCORE groups at a Group Manager, as well as for maintaining them during their whole lifetime until their deletion. This allows the Group Manager to be agnostic of the specific applications using secure group communication.

This document specifies a RESTful admin interface at the Group Manager, intended for an Administrator, as a separate entity external to the Group Manager and its application. The interface allows the Administrator to create and delete OSCORE groups, as well as to configure and update their configuration.

Interaction examples are provided, in Link Format {{RFC6690}} and CBOR {{RFC7049}}, as well as in CoRAL {{I-D.ietf-core-coral}}. While all the CoRAL examples use the CoRAL textual serialization format, the CBOR or JSON {{RFC8259}} binary serialization format is used when sending such messages on the wire.

The ACE framework is used to ensure authentication and authorization of the Administrator (client) at the Group Manager (resource server). In order to achieve communication security, proof-of-possession and server authentication, the Administrator and the Group Manager leverage protocol-specific transport profiles of ACE, such as {{I-D.ietf-ace-oscore-profile}}{{I-D.ietf-ace-dtls-authorize}}. These include also possible forthcoming transport profiles that comply with the requirements in Appendix C of {{I-D.ietf-ace-oauth-authz}}.

## Terminology ## {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with the terms and concepts related to CBOR {{RFC7049}} and COSE {{I-D.ietf-cose-rfc8152bis-struct}}{{I-D.ietf-cose-rfc8152bis-algs}}, the CoAP protocol {{RFC7252}}, as well as the protection and processing of CoAP messages using OSCORE {{RFC8613}}, also in group communication scenarios using Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}. These include the concept of Group Manager, as the entity responsible for a set of groups where communications among members are secured using Group OSCORE.

Readers are also expected to be familiar with the terms and concept related to the management of keying material for groups in ACE defined in {{I-D.ietf-ace-key-groupcomm}}, and in particular to the joining process for OSCORE groups defined in {{I-D.ietf-ace-key-groupcomm-oscore}}. These include the concept of group-membership resource hosted by the Group Manager, that new members access to join the OSCORE group, while current members can access to retrieve updated keying material.

Readers are also expected to be familiar with the terms and concepts described in the ACE framework for authentication and authorization {{I-D.ietf-ace-oauth-authz}}. The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}}. In particular, this includes Client (C), Resource Server (RS), and Authorization Server (AS).

Note that, unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS, and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

This document also refers to the following terminology.

* Administrator: entity responsible to create, configure and delete OSCORE groups at a Group Manager.

* Group name: stable and invariant name of an OSCORE group. The group name MUST be unique under the same Group Manager, and MUST include only characters that are valid for a URI path segment, namely unreserved and pct-encoded characters {{RFC3986}}.

* Group-collection resource: a single-instance resource hosted by the Group Manager. An Administrator accesses a group-collection resource to create a new OSCORE group, or to retrieve the list of existing OSCORE groups, under that Group Manager. As an example, this document uses /manage as the url-path of the group-collection resource; implementations are not required to use this name, and can define their own instead.

* Group-configuration resource: a resource hosted by the Group Manager, associated to an OSCORE group under that Group Manager. A group-configuration resource is identifiable with the invariant group name of the respective group. An Administrator accesses a group-configuration resource to retrieve or update the configuration of the respective OSCORE group, or to delete that group. The url-path to a group-configuration resource has NAME as last segment, with NAME the invariant group name assigned upon its creation. Building on the considered url-path of the group-collection resource, this document uses /manage/NAME as the url-path of a group-configuration resource; implementations are not required to use this name, and can define their own instead.

* Admin endpoint: an endpoint at the Group Manager associated to the group-collection resource or to a group-configuration resource hosted by that Group Manager.

# Group Administration # {#overview}

With reference to the ACE framework and the terminology defined in OAuth 2.0 {{RFC6749}}:

* The Group Manager acts as Resource Server (RS). It provides one single group-collection resource, and one group-configuration resource per existing OSCORE group. Each of those is exported by a distinct admin endpoint.

* The Administrator acts as Client (C), and requests to access the group-collection resource and group-configuration resources, by accessing the respective admin endpoint at the Group Manager.

* The Authorization Server (AS) authorizes the Administrator to access the group-collection resource and group-configuration resources at a Group Manager. Multiple Group Managers can be associated to the same AS. The AS MAY release Access Tokens to the Administrator for other purposes than accessing admin endpoints of registered Group Managers.

## Getting Access to the Group Manager ## {#getting-access}

All communications between the involved entities rely on the CoAP protocol and MUST be secured.

In particular, communications between the Administrator and the Group Manager leverage protocol-specific transport profiles of ACE to achieve communication security, proof-of-possession and server authentication. To this end, the AS may explicitly signal the specific transport profile to use, consistently with requirements and assumptions defined in the ACE framework {{I-D.ietf-ace-oauth-authz}}.

With reference to the AS, communications between the Administrator and the AS (/token endpoint) as well as between the Group Manager and the AS (/introspect endpoint) can be secured by different means, for instance using DTLS {{RFC6347}}{{I-D.ietf-tls-dtls13}} or OSCORE {{RFC8613}}. Further details on how the AS secures communications (with the Administrator and the Group Manager) depend on the specifically used transport profile of ACE, and are out of the scope of this specification.

In order to get access to the Group Manager for managing OSCORE groups, an Administrator performs the following steps.

1. The Administrator requests an Access Token from the AS, in order to access the group-collection and group-configuration resources on the Group Manager. The Administrator will start or continue using secure communications with the Group Manager, according to the response from the AS.

2. The Administrator transfers authentication and authorization information to the Group Manager by posting the obtained Access Token, according to the used profile of ACE, such as {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}. After that, the Administrator must have secure communication established with the Group Manager, before performing any admin operation on that Group Manager. Possible ways to provide secure communication are DTLS {{RFC6347}}{{I-D.ietf-tls-dtls13}} and OSCORE {{RFC8613}}. The Administrator and the Group Manager maintain the secure association, to support possible future communications.

3. The Administrator performs admin operations at the Group Manager, as described in the following sections. These include the retrieval of the existing OSCORE groups, the creation of new OSCORE groups, the update and retrieval of group configurations, and the removal of OSCORE groups. Messages exchanged among the Administrator and the Group Manager are specified in {{interactions}}.

## Managing OSCORE Groups ## {#managing-groups}

{{fig-api}} shows the resources of a Group Manager available to an Administrator.

~~~~~~~~~~~
             ___
   Group    /   \
Collection  \___/
                 \
                  \____________________
                   \___    \___        \___
                   /   \   /   \  ...  /   \        Group
                   \___/   \___/       \___/   Configurations
~~~~~~~~~~~
{: #fig-api title="Resources of a Group Manager" artwork-align="center"}

The Group Manager exports a single group-collection resource. The full interface for the group-collection resource allows the Administrator to:

* Retrieve the list of existing OSCORE groups, possibly by filters.

* Create a new OSCORE group, specifying its invariant group name and, optionally, its configuration.

The Group Manager exports one group-configuration resource for each of its OSCORE groups. Each group-configuration resource is identified by the group name specified upon creating the group. The full interface for a group-configuration resource allows the Administrator to:

* Retrieve the current configuration of the OSCORE group.

* Update the current configuration of the OSCORE group.

* Delete the OSCORE group.

## Group Configurations ## {#group-configurations}

A group configuration consists of a set of parameters.

### Group Configuration Representation ### {#config-repr}

The group configuration representation is a CBOR map which MUST include configuration properties and status properties.

#### Configuration Properties #### {#config-repr-config-properties}

The CBOR map MUST include the following configuration parameters:

* 'hkdf', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the HKDF algorithm used in the OSCORE group, encoded as a CBOR text string. Possible values are the same ones admitted for the 'hkdf' parameter of the "OSCORE Security Context Parameters" registry, defined in Section 3.2.1 of {{I-D.ietf-ace-oscore-profile}}.

* 'alg', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the AEAD algorithm used in the OSCORE group, encoded as a CBOR text string. Possible values are the same ones admitted for the 'alg' parameter of the "OSCORE Security Context Parameters" registry, defined in Section 3.2.1 of {{I-D.ietf-ace-oscore-profile}}.

* 'cs_alg', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the countersignature algorithm used in the OSCORE group, encoded as a CBOR text string or integer. Possible values are the same ones admitted for the 'cs_alg' parameter of the "OSCORE Security Context Parameters" registry, defined in Section 6.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}.

* 'cs_params', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the additional parameters for the countersignature algorithm used in the OSCORE group, encoded as a CBOR array. Possible formats and values are the same ones admitted for the 'cs_params' parameter of the "OSCORE Security Context Parameters" registry, defined in Section 6.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}.

* 'cs_key_params', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the additional parameters for the key used with the countersignature algorithm in the OSCORE group, encoded as a CBOR array. Possible formats and values are the same ones admitted for the 'cs_key_params' parameter of the "OSCORE Security Context Parameters" registry, defined in Section 6.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}.

* 'cs\_key\_enc', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the encoding of the public keys of group members, encoded as a CBOR integer. Possible values are the same ones admitted for the 'cs\_key\_enc' parameter of the "OSCORE Security Context Parameters" registry, defined in Section 6.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}.

#### Status Properties #### {#config-repr-status-properties}

The CBOR map MUST include the following status parameters:

* 'active', encoding the CBOR simple value True if the group is currently active, or the CBOR simple value False otherwise. This parameter is defined in {{iana-ace-groupcomm-parameters}} of this specification.

* 'group_name', with value the group name of the OSCORE group encoded as a CBOR text string. This parameter is defined in {{iana-ace-groupcomm-parameters}} of this specification.

* 'group_title', with value either a human-readable description of the group encoded as a CBOR text string, or the CBOR simple value Null if no description is specified. This parameter is defined in {{iana-ace-groupcomm-parameters}} of this specification.

* 'ace-groupcomm-profile', defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, with value "coap_group_oscore_app".

* 'exp', defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}.

* 'joining_uri', with value the URI of the group-membership resource for joining the newly created OSCORE group, encoded as a CBOR text string. This parameter is defined in {{iana-ace-groupcomm-parameters}} of this specification.

The CBOR map MAY include the following status parameters:

* 'group_policies', defined in Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}, and consistent with the format and content defined in Section 6.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}.

* 'as_uri', defined in {{iana-ace-groupcomm-parameters}} of this document, specifies the URI of the Authorization Server associated to the Group Manager for the OSCORE group, encoded as a CBOR text string. Candidate group members will have to obtain an Access Token from that Authorization Server, before starting the joining process with the Group Manager to join the OSCORE group (see {{I-D.ietf-ace-key-groupcomm-oscore}}).

### Default Values {#default-values}

This section defines the default values that the Group Manager assumes for configuration and status parameters.

#### Configuration Parameters {#default-values-conf}

For each configuration parameter, the Group Manager MUST assume a pre-configured default value, if none is specified by the Administrator.

In particular, the Group Manager SHOULD use the same default values defined in Section 18 of {{I-D.ietf-ace-key-groupcomm-oscore}}.

#### Status Parameters

For the following status parameters, the Group Manager MUST assume a pre-configured default value, if none is specified by the Administrator.

* For 'active', the CBOR simple value False.

* For 'group\_title', the CBOR simple value Null.

## Discovery

The Administrator can discover the group-collection resource from a resource directory, for instance {{I-D.ietf-core-resource-directory}} and {{I-D.hartke-t2trg-coral-reef}}, or from .well-known/core , by using the resource type "ace.oscore.gm" defined in {{iana-rt}} of this specification.

The Administrator can discover group-configuration resources for the group-collection resource as specified below in {{collection-resource-get}} and {{collection-resource-fetch}}.

## Collection Representation

A list of group configurations is represented as a document containing the corresponding group-configuration resources in the list. Each group-configuration is represented as a link, where the link target is the URI of the group-configuration resource.

The list can be represented as a Link Format document {{RFC6690}} or a CoRAL document {{I-D.ietf-core-coral}}. In the latter case, the CoRAL document contains the group-configuration resources in the list as top-level elements. In particular, the link to each group-configuration resource has http://coreapps.org/ace.oscore.gm#item as relation type.

## Interactions ## {#interactions}

This section describes the operations available on the group-collection resource and the group-configuration resources.

When custom CBOR is used, the Content-Format in messages containing a payload is set to application/ace-groupcomm+cbor, defined in Section 8.2 of {{I-D.ietf-ace-key-groupcomm}}. Furthermore, the entry labels defined in {{iana-ace-groupcomm-parameters}} MUST be used, when specifying the corresponding configuration and status parameters.

### Get All Groups Configurations ### {#collection-resource-get}

The Administrator can send a GET request to the group-collection resource, in order to retrieve the list of the existing OSCORE groups at the Group Manager. This is returned as a list of links to the corresponding group-configuration resources.

Example in Link Format:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage

<= 2.05 Content
   Content-Format: 40 (application/link-format)

   <coap://[2001:db8::ab]/manage/gp1>,
   <coap://[2001:db8::ab]/manage/gp2>,
   <coap://[2001:db8::ab]/manage/gp3>
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage

<= 2.05 Content
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/ace.oscore.gm#>
   #base </manage/>
   item <gp1>
   item <gp2>
   item <gp3>
~~~~~~~~~~~

### Fetch Group Configurations By Filters ### {#collection-resource-fetch}

The Administrator can send a FETCH request to the group-collection resource, in order to retrieve the list of the existing OSCORE groups at the Group Manager that fully match a set of specified filter criteria. This is returned as a list of links to the corresponding group-configuration resources.

The set of filter criteria is specified in the request payload as a CBOR map, where possible entry labels are all the ones used for configuration properties (see {{config-repr-config-properties}}), as well as "group_name" and "active" for the corresponding status property (see {{config-repr-status-properties}}).

Entry values are the ones admitted for the corresponding labels in the POST request for creating a group configuration (see {{collection-resource-post}}). A valid request MUST NOT include the same entry multiple times.

Example in custom CBOR and Link Format:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: manage
   Content-Format: TBD2 (application/ace-groupcomm+cbor)
   
   {
       "alg" : 10,
       "hkdf" : 5
   }

<= 2.05 Content
   Content-Format: 40 (application/link-format)

   <coap://[2001:db8::ab]/manage/gp1>,
   <coap://[2001:db8::ab]/manage/gp2>,
   <coap://[2001:db8::ab]/manage/gp3>
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: manage
   Content-Format: TBD1 (application/coral+cbor)
   
   alg 10
   hkdf 5

<= 2.05 Content
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/ace.oscore.gm#>
   #base </manage/>
   item <gp1>
   item <gp2>
   item <gp3>
~~~~~~~~~~~

### Create a New Group Configuration ### {#collection-resource-post}

The Administrator can send a POST request to the group-collection resource, in order to create a new OSCORE group at the Group Manager. The request MAY specify the intended group name NAME and group title, and MAY specify pieces of information concerning the group configuration.

The request payload is a CBOR map, whose possible entries are specified in {{config-repr}}. In particular:

* The CBOR map MAY include any of the configuration parameter defined in {{config-repr-config-properties}}.

* The CBOR map MAY include any of the status parameter 'group_name', 'group_title', 'exp', 'group_policies', 'as_uri' and 'active' defined in {{config-repr-status-properties}}.

* The CBOR map MUST NOT include any of the status parameter 'ace-groupcomm-profile' and 'joining_uri' defined in {{config-repr-status-properties}}.

If any of the following occurs, the Group Manager MUST respond with a 4.00 (Bad Request) response, which MAY include additional information to clarify what went wrong.

* Any of the received parameters is specified multiple times.

* Any of the received parameters is not recognized, or not valid, or not consistent with respect to other related parameters.

* The 'group_name' parameter specifies the group name of an already existing OSCORE group.

* The Group Manager does not trust the Authorization Server with URI specified in the 'as_uri' parameter, and has no alternative Authorization Server to consider for the OSCORE group to create.

After a successful processing of the request above, the Group Manager performs the following actions.

First, the Group Manager creates a new group-configuration resource, accessible to the administrator at /manage/NAME , where NAME is the group name as either indicated in the parameter 'group_name' of the request or uniquely assigned by the Group Manager. The values specified in the request are used as group configuration information for the newly created OSCORE group. For each configuration parameter not specified in the request, the Group Manager MUST assume the default value specified in {{default-values}}.

After that, the Group Manager creates a new group-membership resource, accessible to joining nodes and future group members at group-oscore/NAME , as specified in {{I-D.ietf-ace-key-groupcomm-oscore}}. In particular, the Group Manager will rely on the current group configuration to build the Joining Response message defined in Section 5.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}, when handling the joining of a new group member. Furthermore, the Group Manager generates the following pieces of information, and assigns them to the newly created OSCORE group:

* The OSCORE Master Secret.

* The OSCORE Master Salt (optionally).

* The OSCORE ID Context, acting as Group ID, which MUST be unique within the set of OSCORE groups under the Group Manager.

Finally, the Group Manager replies to the Administrator with a 2.01 (Created) response. The Location-Path option MUST be included in the response, indicating the location of the just created group-configuration resource. The response MUST NOT include a Location-Query option. The response payload is a CBOR map, which MUST include the following parameters:

* 'group_name', with value the group name of the OSCORE group encoded as a CBOR text string. This value can be different from the group name possibly specified by the Administrator in the POST request, and reflects the final choice of the Group Manager as 'group_name' status property for the OSCORE group.

* 'joining_uri', with value the URI of the group-membership resource for joining the newly created OSCORE group, encoded as a CBOR text string.

* 'as_uri', with value the URI of the Authorization Server associated to the Group Manager for the newly created OSCORE group, encoded as a CBOR text string. This value can be different from the URI possibly specified by the Administrator in the POST request, and reflects the final choice of the Group Manager as 'as_uri' status property for the OSCORE group.

At this point, the Group Manager can register the link to the group-membership resource with URI specified in 'joining_uri' to the CoRE Resource Directory {{I-D.ietf-core-resource-directory}}, as defined in Section 2 of {{I-D.tiloca-core-oscore-discovery}}.

Alternatively, the Administrator can perform the registration to the Resource Directory on behalf of the Group Manager, acting as Commissioning Tool. The Administrator considers the following when specifying additional information for the link to register.

* The name of the OSCORE group MUST take the value specified in 'group_name' from the 2.01 (Created) response above.

* If present, parameters describing the cryptographic algorithms used in the group MUST follow the values that the Administrator specified in the POST request above, or the corresponding default values specified in {{default-values-conf}} otherwise.

* If also registering a related link to the Authorization Server associated to the OSCORE group, the related link MUST have the URI specified in 'as_uri' from the 2.01 (Created) response above.

Example in custom CBOR:

~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: manage
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "alg" : 10,
     "hkdf" : 5,
     "active" : True,
     "group_title" : "rooms 1 and 2",
     "as_uri" : "coap://as.example.com/token"
   }

<= 2.01 Created
   Location-Path: manage
   Location-Path: gp4
   Content-Format: TBD2 (application/ace-groupcomm+cbor)
   
   {
     "group_name" : "gp4",
     "joining_uri" : "coap://[2001:db8::ab]/group-oscore/gp4/",
     "as_uri" : "coap://as.example.com/token"
   }
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: manage
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/ace.oscore.gm#>
   alg 10
   hkdf 5
   active True
   group_title "rooms 1 and 2"
   as_uri <coap://as.example.com/token>

<= 2.01 Created
   Location-Path: manage
   Location-Path: gp4
   Content-Format: TBD1 (application/coral+cbor)
   
   #using <http://coreapps.org/ace.oscore.gm#>
   group_name "gp4"
   joining_uri <coap://[2001:db8::ab]/group-oscore/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

### Retrieve a Group Configuration ### {#configuration-resource-get}

The Administrator can send a GET request to the group-configuration resource manage/NAME associated to an OSCORE group with group name NAME, in order to retrieve the current configuration of that group.

After a successful processing of the request above, the Group Manager replies to the Administrator with a 2.05 (Content) response. The response has as payload the representation of the group configuration as specified in {{config-repr}}. The exact content of the payload reflects the current configuration of the OSCORE group. This includes both configuration properties and status properties.

Example in custom CBOR:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage
   Uri-Path: gp4

<= 2.05 Content
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "alg" : 10,
     "hkdf" : 5,
     "cs_alg" : -8,
     "cs_params" : [[1], [1, 6]],
     "cs_key_params" : [1, 6],
     "cs_key_enc" : 1,
     "active" : True,
     "group_name" : "gp4",
     "group_title" : "rooms 1 and 2",
     "ace-groupcomm-profile" : "coap_group_oscore_app",
     "exp" : "1360289224",
     "joining_uri" : "coap://[2001:db8::ab]/group-oscore/gp4/",
     "as_uri" : "coap://as.example.com/token"
   }
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage
   Uri-Path: gp4

<= 2.05 Content
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/ace.oscore.gm#>
   alg 10
   hkdf 5
   cs_alg -8
   cs_params.alg_capab.key_type 1
   cs_params.key_type_capab.key_type 1
   cs_params.key_type_capab.curve 6
   cs_key_params.key_type 1
   cs_key_params.curve 6
   cs_key_enc 1
   active True
   group_name "gp4"
   group_title "rooms 1 and 2"
   ace-groupcomm-profile "coap_group_oscore_app"
   exp "1360289224"
   joining_uri <coap://[2001:db8::ab]/group-oscore/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

### Update a Group Configuration ### {#configuration-resource-put}

The Administrator can send a PUT request to the group-configuration resource associated to an OSCORE group, in order to update the current configuration of that group. The payload of the request has the same format of the POST request defined in {{collection-resource-post}}, with the exception of the status parameter 'group_name' that MUST NOT be included.

The error handling for the PUT request is the same as for the POST request defined in {{collection-resource-post}}. If no error occurs, the Group Manager performs the following actions.

First, the Group Manager updates the configuration of the OSCORE group, consistently with the values indicated in the PUT request from the Administrator. For each configuration parameter not specified in the PUT request, the Group Manager MUST use the default value specified in {{default-values}}. From then on, the Group Manager relies on the latest update configuration to build the Joining Response message defined in Section 5.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}, when handling the joining of a new group member.

Then, the Group Manager replies to the Administrator with a 2.04 (Changed) response. The payload of the response has the same format of the 2.01 (Created) response defined in {{collection-resource-post}}.

If the link to the group-membership resource was registered in the Resource Directory (see {{I-D.ietf-core-resource-directory}}), the GM is responsible to refresh the registration, as defined in Section 3 of {{I-D.tiloca-core-oscore-discovery}}.

Alternatively, the Administrator can update the registration to the Resource Directory on behalf of the Group Manager, acting as Commissioning Tool. The Administrator considers the following when specifying additional information for the link to update.

* The name of the OSCORE group MUST take the value specified in 'group_name' from the 2.04 (Changed) response above.

* If present, parameters describing the cryptographic algorithms used in the group MUST follow the values that the Administrator specified in the POST request above, or the corresponding default values specified in {{default-values-conf}} otherwise.

* If also registering a related link to the Authorization Server associated to the OSCORE group, the related link MUST have the URI specified in 'as_uri' from the 2.04 (Changed) response above.

Example in custom CBOR:

~~~~~~~~~~~
=> PUT
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "alg" : 11 ,
     "hkdf" : 5
   }

<= 2.04 Changed
   Content-Format: TBD2 (application/ace-groupcomm+cbor)
   
   {
     "group_name" : "gp4",
     "joining_uri" : "coap://[2001:db8::ab]/group-oscore/gp4/",
     "as_uri" : "coap://as.example.com/token"
   }
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> PUT
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/ace.oscore.gm#>
   alg 11
   hkdf 5

<= 2.04 Changed
   Content-Format: TBD1 (application/coral+cbor)
   
   #using <http://coreapps.org/ace.oscore.gm#>
   group_name "gp4"
   joining_uri <coap://[2001:db8::ab]/group-oscore/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

#### Effects on Joining Nodes ####

If the value of the status parameter 'active' is changed from True to False, the Group Manager MUST stop admitting new members in the group. In particular, upon receiving a Joining Request (see Section 5.3 of {{I-D.ietf-ace-key-groupcomm-oscore}}), the Group Manager MUST respond with a 5.03 (Service Unavailable) response to the joining node, and MAY include additional information to clarify what went wrong.

If the value of the status parameter 'active' is changed from False to True, the Group Manager resumes admitting new members in the group, by processing their Joining Requests (see Section 5.3 of {{I-D.ietf-ace-key-groupcomm-oscore}}).

#### Effects on the Group Members ####

After having updated a group configuration, the Group Manager informs the group members, over the pairwise secure communication channels established when joining the OSCORE group (see Section 5 of {{I-D.ietf-ace-key-groupcomm-oscore}}).

To this end, the Group Manager can individually target the 'control_path' URI path of each group member (see Section 4.1.2.1 of {{I-D.ietf-ace-key-groupcomm}}), if provided by the intended recipient upon joining the group (see Section 5 of {{I-D.ietf-ace-key-groupcomm-oscore}}). Alternatively, group members can subscribe for updates to the group-membership resource of the OSCORE group, e.g. by using CoAP Observe {{RFC7641}}.

Every group member, upon learning that the group has been deactivated (i.e. 'active' has value False), SHOULD stop communicating in the OSCORE group.
  
Every group member, upon learning that the group has been reactivated (i.e. 'active' has value True again), can resume communicating in the OSCORE group.

Every group member, upon receiving updated values for 'alg' and 'hkdf', MUST either:

* Leave the group (see Section 14 of {{I-D.ietf-ace-key-groupcomm-oscore}}), e.g. if not supporting the indicated new algorithms; or

* Use the new parameter values, and accordingly re-derive the OSCORE Security Context for the group (see Section 2 of {{I-D.ietf-core-oscore-groupcomm}}).

Every group member, upon receiving updated values for 'cs_alg', 'cs_params', 'cs_key_params' and 'cs_key_enc', MUST either:

* Leave the group, e.g. if not supporting the indicated new algorithm, parameters and encoding; or

* Leave the group and rejoin it (see Section 5 of {{I-D.ietf-ace-key-groupcomm-oscore}}), providing the Group Manager with a public key which is compatible with the indicated new algorithm, parameters and encoding; or

* Use the new parameter values, and, if required, provide the Group Manager with a new public key to use in the group, as compatible with the indicated parameters (see Section 10 of {{I-D.ietf-ace-key-groupcomm-oscore}}).

### Delete a Group Configuration ### {#configuration-resource-delete}

The Administrator can send a DELETE request to the group-configuration resource, in order to delete that group. A group deletion would be successful only on an inactive group.

That is, the DELETE request actually yields a successful deletion of the group, only if the corresponding status parameter 'active' has current value False. The administrator can ensure that, by first performing an update of the group-configuration resource associated to the group (see {{configuration-resource-put}}), and setting the corresponding status parameter 'active' to False.

If, upon receiving the DELETE request, the current value of the status parameter 'active' is True, the Group Manager MUST respond with a 4.09 (Conflict) response, which MAY include additional information to clarify what went wrong.

After a successful processing of the request above, the Group Manager performs the following actions.

First, the Group Manager deletes the OSCORE group and deallocates both the group-configuration resource as well as the group-membership resource.

Then, the Group Manager replies to the Administrator with a 2.02 (Deleted) response.

Example:

~~~~~~~~~~~
=> DELETE
   Uri-Path: manage
   Uri-Path: gp4

<= 2.02 Deleted
~~~~~~~~~~~

#### Effects on the Group Members ####

After having deleted a group, the Group Manager can inform the group members by means of the following two methods. When contacting a group member, the Group Manager uses the pairwise secure communication channel established with that member during its joining process (see Section 5 of {{I-D.ietf-ace-key-groupcomm-oscore}}).

* The Group Manager sends an individual request message to each group member, targeting the respective resource used to perform the group rekeying process (see Section 16 of {{I-D.ietf-ace-key-groupcomm-oscore}}). The Group Manager uses the same format of the Joining Response message in Section 5.4 of {{I-D.ietf-ace-key-groupcomm-oscore}}, where only the parameters 'gkty', 'key', and 'ace-groupcomm-profile' are present, and the 'key' parameter is empty.

* A group member may subscribe for updates to the group-membership resource of the group. In particular, if this relies on CoAP Observe {{RFC7641}}, a group member would receive a 4.04 (Not Found) notification response from the Group Manager, since the group-configuration resource has been deallocated upon deleting the group.

When being informed about the group deletion, a group member deletes the OSCORE Security Context that it stores as associated to that group, and possibly deallocates any dedicated control resource intended for the Group Manager that it has for that group.

# Security Considerations # {#sec-security-considerations}

Security considerations are inherited from the ACE framework for Authentication and Authorization {{I-D.ietf-ace-oauth-authz}}, and from the specific transport ace-groupcomm-profile of ACE used between the Administrator and the Group Manager, such as {{I-D.ietf-ace-dtls-authorize}} and {{I-D.ietf-ace-oscore-profile}}.

# IANA Considerations # {#iana}

This document has the following actions for IANA.

## ACE Groupcomm Parameters Registry ## {#iana-ace-groupcomm-parameters}

IANA is asked to register the following entries in the "ACE Groupcomm Parameters" Registry defined in Section 8.5 of {{I-D.ietf-ace-key-groupcomm}}.

~~~~~~~~~~~
 +---------------+----------+--------------------+-------------------+
 | Name          | CBOR Key | CBOR Type          | Reference         |
 +---------------+----------+--------------------+-------------------+
 |               |          |                    |                   |
 | hkdf          | TBD3     | tstr               | [[this document]] |
 |               |          |                    |                   |
 | alg           | TBD4     | tstr               | [[this document]] |
 |               |          |                    |                   |
 | cs_alg        | TBD5     | tstr / int         | [[this document]] |
 |               |          |                    |                   |
 | cs_params     | TBD6     | array              | [[this document]] |
 |               |          |                    |                   |
 | cs_key_params | TBD7     | array              | [[this document]] |
 |               |          |                    |                   |
 | cs_key_enc    | TBD8     | int                | [[this document]] |
 |               |          |                    |                   |
 | active        | TBD9     | simple type        | [[this document]] |
 |               |          |                    |                   |
 | group_name    | TBD10    | tstr               | [[this document]] |
 |               |          |                    |                   |
 | group_title   | TBD11    | tstr / simple type | [[this document]] |
 |               |          |                    |                   |
 | joining_uri   | TBD12    | tstr               | [[this document]] |
 |               |          |                    |                   |
 | as_uri        | TBD13    | tstr               | [[this document]] |
 |               |          |                    |                   |
 +---------------+----------+--------------------+-------------------+
~~~~~~~~~~~

## Resource Types # {#iana-rt}

IANA is asked to enter the following value into the Resource Type (rt=) Link Target Attribute Values subregistry within the Constrained Restful Environments (CoRE) Parameters registry defined in {{RFC6690}}.

~~~~~~~~~~~
 +---------------+----------------------------+-------------------+
 | Value         | Description                | Reference         |
 +---------------+----------------------------+-------------------+
 |               |                            |                   |
 | ace.oscore.gm | Group-collection resource  | [[this document]] |
 |               | of an OSCORE Group Manager |                   |
 |               |                            |                   |
 +---------------+----------------------------+-------------------+
~~~~~~~~~~~

--- back

# Acknowledgments # {#acknowldegment}
{: numbered="no"}

The authors sincerely thank Christian Amsuess, Carsten Bormann and Jim Schaad for their comments and feedback.

The work on this document has been partly supported by VINNOVA and the Celtic-Next project CRITISEC.

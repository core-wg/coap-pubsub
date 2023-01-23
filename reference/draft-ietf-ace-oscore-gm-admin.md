---
v: 3

title: Admin Interface for the OSCORE Group Manager
abbrev: Admin Interface for the OSCORE GM
docname: draft-ietf-ace-oscore-gm-admin-latest


# stand_alone: true

ipr: trust200902
area: Internet
wg: ACE Working Group
kw: Internet-Draft
cat: std
submissiontype: IETF

coding: utf-8
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
        ins: R. Höglund
        name: Rikard Höglund
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
        email: stokcons@bbhmail.nl
      -
        ins: F. Palombini
        name: Francesca Palombini
        org: Ericsson AB
        street: Torshamnsgatan 23
        city: Kista
        code: SE-16440 Stockholm
        country: Sweden
        email: francesca.palombini@ericsson.com

normative:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-ace-key-groupcomm:
  I-D.ietf-ace-key-groupcomm-oscore:
  I-D.ietf-core-coral:
  I-D.ietf-core-groupcomm-bis:
  RFC2119:
  RFC6690:
  RFC6749:
  RFC7252:
  RFC7641:
  RFC8126:
  RFC8132:
  RFC8174:
  RFC8610:
  RFC8613:
  RFC8949:
  RFC9052:
  RFC9053:
  RFC9200:
  RFC9203:
  RFC9237:
  RFC9277:
  COSE.Algorithms:
    author:
      org: IANA
    date: false
    title: COSE Algorithms
    target: https://www.iana.org/assignments/cose/cose.xhtml#algorithms

informative:
  I-D.tiloca-core-oscore-discovery:
  I-D.hartke-t2trg-coral-reef:
  I-D.amsuess-core-cachable-oscore:
  I-D.ietf-cose-cbor-encoded-cert:
  RFC7925:
  RFC8392:
  RFC9147:
  RFC9176:
  RFC9202:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

Group communication for CoAP can be secured using Group Object Security for Constrained RESTful Environments (Group OSCORE). A Group Manager is responsible to handle the joining of new group members, as well as to manage and distribute the group keying material. This document defines a RESTful admin interface at the Group Manager, that allows an Administrator entity to create and delete OSCORE groups, as well as to retrieve and update their configuration. The ACE framework for Authentication and Authorization is used to enforce authentication and authorization of the Administrator at the Group Manager. Protocol-specific transport profiles of ACE are used to achieve communication security, proof-of-possession and server authentication.

--- middle

# Introduction # {#intro}

The Constrained Application Protocol (CoAP) {{RFC7252}} can be used in group communication environments where messages are also exchanged over IP multicast {{I-D.ietf-core-groupcomm-bis}}. Applications relying on CoAP can achieve end-to-end security at the application layer by using Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}}, and especially Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} in group communication scenarios.

When group communication for CoAP is protected with Group OSCORE, nodes are required to explicitly join the correct OSCORE group. To this end, a joining node interacts with a Group Manager (GM) entity responsible for that group, and retrieves the required keying material to securely communicate with other group members using Group OSCORE.

The method in {{I-D.ietf-ace-key-groupcomm-oscore}} specifies how nodes can join an OSCORE group through the respective Group Manager. Such a method builds on the ACE framework for Authentication and Authorization {{RFC9200}}, so ensuring a secure joining process as well as authentication and authorization of joining nodes (clients) at the Group Manager (resource server).

In some deployments, the application running on the Group Manager may know when a new OSCORE group has to be created, as well as how it should be configured and later on updated or deleted, e.g., based on the current application state or on pre-installed policies. In this case, the Group Manager application can create and configure OSCORE groups when needed, by using a local application interface. However, this requires the Group Manager to be application-specific, which in turn leads to error prone deployments and is poorly flexible.

In other deployments, a separate Administrator entity, such as a Commissioning Tool, is directly responsible for creating and configuring the OSCORE groups at a Group Manager, as well as for maintaining them during their whole lifetime until their deletion. This allows the Group Manager to be agnostic of the specific applications using secure group communication.

This document specifies a RESTful admin interface at the Group Manager, intended for an Administrator as a separate entity external to the Group Manager and its application. The interface allows the Administrator to create and delete OSCORE groups, as well as to configure and update their configuration.

Interaction examples are provided, in Link Format {{RFC6690}} and CBOR {{RFC8949}}, as well as in CoRAL {{I-D.ietf-core-coral}}. While all the CoRAL examples show the CoRAL textual serialization format, its binary serialization format is used on the wire.

\[ NOTE:

The reported CoRAL examples are based on the textual representation used until  version -03 of {{I-D.ietf-core-coral}}. These will be revised to use the CBOR diagnostic notation instead.

\]

The ACE framework is used to ensure authentication and authorization of the Administrator (client) at the Group Manager (resource server). In order to achieve communication security, proof-of-possession and server authentication, the Administrator and the Group Manager leverage protocol-specific transport profiles of ACE, such as {{RFC9202}}{{RFC9203}}. These include also possible forthcoming transport profiles that comply with the requirements in Appendix C of {{RFC9200}}.

## Terminology ## {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

Readers are expected to be familiar with the terms and concepts from the following specifications:

* CBOR {{RFC8949}} and COSE {{RFC9052}}{{RFC9053}}.

* The CoAP protocol {{RFC7252}}, also in group communication scenarios {{I-D.ietf-core-groupcomm-bis}}. These include the concepts of:

   - "application group", as a set of CoAP nodes that share a common set of resources; and of

   - "security group", as a set of CoAP nodes that share the same security material, and use it to protect and verify exchanged messages.

* The OSCORE {{RFC8613}} and Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} security protocols. These especially include the concepts of:

   - Group Manager, as the entity responsible for a set of OSCORE groups where communications among members are secured using Group OSCORE. An OSCORE group is used as security group for one or many application groups.

   - Authentication credential, as the set of information associated with an entity, including that entity's public key and parameters associated with the public key. Examples of authentication credentials are CBOR Web Tokens (CWTs) and CWT Claims Sets (CCSs) {{RFC8392}}, X.509 certificates {{RFC7925}} and C509 certificates {{I-D.ietf-cose-cbor-encoded-cert}}.

* The ACE framework for authentication and authorization {{RFC9200}}. The terminology for entities in the considered architecture is defined in OAuth 2.0 {{RFC6749}}. In particular, this includes Client (C), Resource Server (RS), and Authorization Server (AS).

* The management of keying material for groups in ACE {{I-D.ietf-ace-key-groupcomm}} and specifically for OSCORE groups {{I-D.ietf-ace-key-groupcomm-oscore}}. These include the concept of group-membership resource hosted by the Group Manager, that new members access to join the OSCORE group, while current members can access to retrieve updated keying material.

Note that, unless otherwise indicated, the term "endpoint" is used here following its OAuth definition, aimed at denoting resources such as /token and /introspect at the AS, and /authz-info at the RS. This document does not use the CoAP definition of "endpoint", which is "An entity participating in the CoAP protocol".

This document also refers to the following terminology.

* Administrator: entity responsible to create, configure and delete OSCORE groups at a Group Manager.

* Group name: stable and invariant name of an OSCORE group. The group name MUST be unique under the same Group Manager, and MUST include only characters that are valid for a URI path segment.

* Group-collection resource: a single-instance resource hosted by the Group Manager. An Administrator accesses a group-collection resource to retrieve the list of existing OSCORE groups, or to create a new OSCORE group, under that Group Manager.

   As an example, this document uses /manage as the url-path of the group-collection resource; implementations are not required to use this name, and can define their own instead.

* Group-configuration resource: a resource hosted by the Group Manager, associated with an OSCORE group under that Group Manager. A group-configuration resource is identifiable with the invariant group name of the respective OSCORE group. An Administrator accesses a group-configuration resource to retrieve or change the configuration of the respective OSCORE group, or to delete that group.

   The url-path to a group-configuration resource has GROUPNAME as last segment, with GROUPNAME the invariant group name assigned upon its creation. Building on the considered url-path of the group-collection resource, this document uses /manage/GROUPNAME as the url-path of a group-configuration resource; implementations are not required to use this name, and can define their own instead.

* Admin endpoint: an endpoint at the Group Manager associated with the group-collection resource or to a group-configuration resource hosted by that Group Manager.

# Group Administration # {#overview}

With reference to the ACE framework and the terminology defined in OAuth 2.0 {{RFC6749}}:

* The Group Manager acts as Resource Server (RS). It provides one single group-collection resource, and one group-configuration resource per existing OSCORE group. Each of those is exported by a distinct admin endpoint.

* The Administrator acts as Client (C), and requests to access the group-collection resource and group-configuration resources, by accessing the respective admin endpoint at the Group Manager.

* The Authorization Server (AS) authorizes the Administrator to access the group-collection resource and group-configuration resources at a Group Manager. Multiple Group Managers can be associated with the same AS.

    The authorized access for an Administrator can be limited to performing only a subset of operations, according to what is allowed by the authorization information in the Access Token issued to that Administrator (see {{scope-format}} and {{getting-access}}). The AS can authorize multiple Administrators to access the group-collection resource and the (same) group-configuration resources at the Group Manager.

    The AS MAY release Access Tokens to the Administrator for other purposes than accessing admin endpoints of registered Group Managers.

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

The Group Manager exports a single group-collection resource, with resource type "core.osc.gcoll" defined in {{iana-rt}} of this document. The interface for the group-collection resource defined in {{interactions}} allows the Administrator to:

* Retrieve the list of existing OSCORE groups.

* Retrieve the list of existing OSCORE groups matching with specified filter criteria.

* Create a new OSCORE group, specifying its invariant group name and, optionally, its configuration.

The Group Manager exports one group-configuration resource for each of its OSCORE groups. Each group-configuration resource has resource type "core.osc.gconf" defined in {{iana-rt}} of this document, and is identified by the group name specified upon creating the OSCORE group. The interface for a group-configuration resource defined in {{interactions}} allows the Administrator to:

* Retrieve the complete current configuration of the OSCORE group.

* Retrieve part of the current configuration of the OSCORE group, by applying filter criteria.

* Overwrite the current configuration of the OSCORE group.

* Selectively update only part of the current configuration of the OSCORE group.

* Delete the OSCORE group.

## Collection Representation

A list of group configurations is represented as a document containing the corresponding group-configuration resources in the list. Each group-configuration is represented as a link, where the link target is the URI of the group-configuration resource.

The list can be represented as a Link Format document {{RFC6690}} or a CoRAL document {{I-D.ietf-core-coral}}.

In the former case, the link to each group-configuration resource specifies the link target attribute 'rt' (Resource Type), with value "core.osc.gconf" defined in {{iana-rt}} of this document.

In the latter case, the CoRAL document specifies the group-configuration resources in the list as top-level elements. In particular, the link to each group-configuration resource has http://coreapps.org/core.osc.gcoll#item as relation type.

## Discovery

The Administrator can discover the group-collection resource from a Resource Directory, for instance {{RFC9176}} and {{I-D.hartke-t2trg-coral-reef}}, or from .well-known/core, by using the resource type "core.osc.gcoll" defined in {{iana-rt}} of this document.

The Administrator can discover group-configuration resources for the group-collection resource as specified in {{collection-resource-get}} and {{collection-resource-fetch}}.

# Format of Scope # {#scope-format}

This section defines the exact format and encoding of scope to use, in order to express authorization information for the Administrator (see {{getting-access}}).

To this end, this document uses the Authorization Information Format (AIF) {{RFC9237}}. In particular, it uses and extends the AIF specific data model AIF-OSCORE-GROUPCOMM defined in {{Section 3 of I-D.ietf-ace-key-groupcomm-oscore}}.

The original definition of the data model AIF-OSCORE-GROUPCOMM specifies a scope as structured in scope entries, which express authorization information for users of an OSCORE group, i.e., actual group members or external signature verifiers. In the rest of this section, these are referred to as "user scope entries".

This document extends the same AIF specific data model AIF-OSCORE-GROUPCOMM as defined below. In particular, it defines how the same scope can (also) include scope entries that express authorization information for Administrators of OSCORE groups. In the rest of this section, these are referred to as "admin scope entries".

Like in the original definition of the data model AIF-OSCORE-GROUPCOMM, and with reference to the generic AIF model

~~~~~~~~~~~
   AIF-Generic<Toid, Tperm> = [* [Toid, Tperm]]
~~~~~~~~~~~

the value of the CBOR byte string used as scope encodes the CBOR array \[* \[Toid, Tperm\]\], where each \[Toid, Tperm\] element corresponds to one scope entry.

Then, the following applies for each admin scope entry intended to express authorization information for an Administrator, as defined in this document.

* The object identifier ("Toid") is specialized as either of the following, and specifies a group name pattern P for the admin scope entry.

   - Wildcard pattern: "Toid" is specialized as the CBOR simple value "true" (0xf5), specifying the wildcard pattern. That is, any group name expressed as a literal text string matches with this group name pattern.

   - Literal pattern: "Toid" is specialized as a CBOR text string, whose value specifies an exact group name as a literal string. That is, only one specific group name expressed as a literal text string matches with this group name pattern.

   - Complex pattern: "Toid" is specialized as a tagged CBOR data item, specifying a more complex group name pattern with the semantics signaled by the CBOR tag. That is, multiple group names expressed as a literal text string match with this group name pattern.

      For example, and as typically expected, the data item can be a CBOR text string marked with the CBOR tag 35. This indicates that the group name pattern specified as value of the CBOR text string is a regular expression (see {{Section 3.4.5.3 of RFC8949}}).

      In case the AIF specific data model AIF-OSCORE-GROUPCOMM is used in a JSON payload, the semantics information conveyed by the CBOR tag can be equivalently conveyed, for example, in a nested JSON object.

      The AS and the Group Manager are expected to have agreed on commonly supported semantics for group name patterns. This can happen, for instance, as part of the registration process of the Group Manager at the AS.

* The permission set ("Tperm") is specialized as a CBOR unsigned integer with value Q. This specifies the permissions that the Administrator has to perform operations on the admin endpoints at the Group Manager, as pertaining to any OSCORE group whose name matches with the pattern P. The value Q is computed as follows.

   - Each permission in the permission set is converted into the corresponding numeric identifier X from the "Value" column of the "Group OSCORE Admin Permissions" registry, for which this document defines the entries in {{fig-permission-values}}.

   - The set of N numbers is converted into the single value Q, by taking two to the power of each numeric identifier X_1, X_2, ..., X_N, and then computing the inclusive OR of the binary representations of all the power values.

   In general, a single permission can be associated with multiple different operations that are possible to be performed when interacting with the Group Manager. For example, the "List" permission allows the Administrator to retrieve a list of group configurations (see {{collection-resource-get}}) or only a subset of that according to specified filter criteria (see {{collection-resource-fetch}}), by issuing a GET or FETCH request to the group-collection resource, respectively.

~~~~~~~~~~~
+--------+-------+----------------------------------------+
| Name   | Value | Description                            |
+========+=======+========================================+
| List   | 0     | Retrieve list of group configurations  |
+--------+-------+----------------------------------------+
| Create | 1     | Create new group configurations        |
+--------+-------+----------------------------------------+
| Read   | 2     | Retrieve group configurations          |
+--------+-------+----------------------------------------+
| Write  | 3     | Change group configurations            |
+--------+-------+----------------------------------------+
| Delete | 4     | Delete group configurations            |
+--------+-------+----------------------------------------+
~~~~~~~~~~~
{: #fig-permission-values title="Numeric identifier of permissions on the admin endpoints at a Group Manager" artwork-align="center"}

The following CDDL {{RFC8610}} notation defines an admin scope entry that uses the data model AIF-OSCORE-GROUPCOMM and expresses a set of permissions from those in {{fig-permission-values}}.

~~~~~~~~~~~~~~~~~~~~ CDDL
   AIF-OSCORE-GROUPCOMM = AIF-Generic<oscore-gname, oscore-gperm>

   oscore-gname = true / tstr / #6.nnn(any) ; Group name pattern
   oscore-gperm = uint . bits admin-permissions
   admin-permissions = &(
      List: 0,
      Create: 1,
      Read: 2,
      Write: 3,
      Delete: 4
   )

   scope_entry = [oscore-gname, oscore-gperm]
~~~~~~~~~~~~~~~~~~~~

Future specifications that define new permissions on the admin endpoints at the Group Manager MUST register a corresponding numeric identifier in the "Group OSCORE Admin Permissions" registry defined in {{ssec-iana-group-oscore-admin-permissions-registry}} of this document.

When using the scope format as defined in this section, the permission set ("Tperm") of each admin scope entry MUST include the "List" permission. It follows that, when expressing permissions for Administrators of OSCORE groups as defined in this document, an admin scope entry has the least significant bit of "Tperm" always set to 1.

Therefore, an Administrator is always allowed to retrieve a list of existing group configurations. The exact elements included in the returned list are determined by the Group Manager, based on the group name patterns specified in the admin scope entries of the Administrator's Access Token, as well as on possible filter criteria specified in the request from the Administrator (see {{collection-resource-get}} and {{collection-resource-fetch}}).

Building on the above, the same single scope can include user scope entries as well as admin scope entries, whose specific format is defined in {{Section 3 of I-D.ietf-ace-key-groupcomm-oscore}} and earlier in this section, respectively. The two types of scope entries can be unambiguously distinguished by means of the least significant bit of their permission set "Tperm", which has value 0 for the user scope entries and 1 for the admin scope entries.

The coexistence of user scope entries and admin scope entries within the same scope makes it possible to issue a single Access Token, in case the requesting Client wishes to be a user for some OSCORE groups and at the same time Administrator for some (other) OSCORE groups under the same Group Manager.

Throughout the rest of this document, the term "scope entry" is used as referred to "admin scope entry", unless otherwise indicated.

## On Enforcing Different Classes of Administrators

By relying on the scope format defined in this document and given an OSCORE group G1 created by a "main" Administrator, then a second "assistant" Administrator can be effectively authorized to perform some operations on G1, in spite of not being the group creator.

Furthermore, having the object identifier ("Toid") specialized as a pattern displays a number of advantages.

* The encoded scope can be compact in size, while allowing the Administrator to operate on large pools of group names.

* The Administrator and the AS do not need to know exact group names when requesting and issuing an Access Token, respectively (see {{getting-access}}). In turn, the Group Manager can effectively take the final decision about the name to assign to an OSCORE group, upon its creation (see {{collection-resource-post}}).

* The Administrator may have established a secure communication association with the Group Manager based on a first Access Token T1, and then created an OSCORE group G. Following the invalidation of T1 (e.g., due to expiration) and the establishment of a new secure communication association with the Group Manager based on a new Access Token T2, the Administrator can seamlessly perform authorized operations on the previously created group G.

# Getting Access to the Group Manager # {#getting-access}

All communications between the involved entities rely on the CoAP protocol and MUST be secured.

In particular, communications between the Administrator and the Group Manager leverage protocol-specific transport profiles of ACE to achieve communication security, proof-of-possession and server authentication. To this end, the AS may explicitly signal the specific transport profile to use, consistently with requirements and assumptions defined in the ACE framework {{RFC9200}}.

With reference to the AS, communications between the Administrator and the AS (/token endpoint) as well as between the Group Manager and the AS (/introspect endpoint) can be secured by different means, for instance using DTLS {{RFC9147}} or OSCORE {{RFC8613}}. Further details on how the AS secures communications (with the Administrator and the Group Manager) depend on the specifically used transport profile of ACE, and are out of the scope of this document.

In order to specify authorization information for Administrators, the format and encoding of scope defined in {{scope-format}} of this document MUST be used, for both the 'scope' claim in the Access Token, as well as for the 'scope' parameter in the Authorization Request and Authorization Response exchanged with the AS (see {{Sections 5.8.1 and 5.8.2 of RFC9200}}).

Furthermore, the AS MAY use the extended format of scope defined in {{Section 7 of I-D.ietf-ace-key-groupcomm}} for the 'scope' claim of the Access Token. In such a case, the AS MUST use the CBOR tag with tag number TAG_NUMBER, associated with the CoAP Content-Format CF_ID for the media type application/aif+cbor registered in {{Section 16.9 of I-D.ietf-ace-key-groupcomm-oscore}}.

Note to RFC Editor: In the previous paragraph, please replace "TAG_NUMBER" with the CBOR tag number computed as TN(ct) in {{Section 4.3 of RFC9277}}, where ct is the ID assigned to the CoAP Content-Format CF_ID registered in {{Section 16.9 of I-D.ietf-ace-key-groupcomm-oscore}}. Then, please replace "CF_ID" with the ID assigned to that CoAP Content-Format. Finally, please delete this paragraph.

This indicates that the binary encoded scope, as conveying the actual access control information, follows the scope semantics of the AIF specific data model AIF-OSCORE-GROUPCOMM defined in {{Section 3 of I-D.ietf-ace-key-groupcomm-oscore}} and extended as per {{scope-format}} of this document.

In order to get access to the Group Manager for managing OSCORE groups, an Administrator performs the following steps.

1. The Administrator requests an Access Token from the AS, in order to access the group-collection and group-configuration resources on the Group Manager. To this end, the Administrator sends to the AS an Authorization Request as defined in {{Section 5.8.1 of RFC9200}}.

   If the 'scope' parameter in the Authorization Request includes scope entries whose "Toid" specifies a complex pattern (see {{scope-format}}), then all such scope entries MUST adhere to the same pattern semantics.

   The Administrator will start or continue using a secure communication association with the Group Manager, according to the response from the AS and the specifically used transport profile of ACE.

2. The AS processes the Authorization Request as defined in {{Section 5.8.2 of RFC9200}}, especially verifying that the Administrator is authorized to obtain the requested permissions, or possibly a subset of those.

   The AS specifies the information on the authorization granted to the Administrator as the value of the 'scope' claim to include in the Access Token, in accordance with the scope format specified in {{scope-format}}. It is implementation specific which particular approach the AS takes to evaluate the requested permissions against the access policies pertaining to the Administrator for the Group Manager in question. {{sec-as-scope-processing}} provides an example of such an approach that the AS can use.

   If the 'scope' claim in the Authorization Request includes scope entries whose "Toid" specifies a complex pattern, then all such scope entries MUST adhere to the same pattern semantics.

   If the 'scope' parameter in the Authorization Request includes scope entries whose "Toid" specifies a complex pattern adhering to a certain pattern semantics, then that semantics MUST be used for all the scope entries in the 'scope' claim that specify a complex pattern.

   The AS MUST include the 'scope' parameter in the Authorization Response defined in {{Section 5.8.2 of RFC9200}}, when the value included in the Access Token differs from the one specified by the Administrator in the Authorization Request. In such a case, scope specifies the set of permissions that the Administrator actually has to perform operations at the Group Manager, encoded as specified in {{scope-format}}.

   If the 'scope' parameter in the Authorization Request includes scope entries whose "Toid" specifies a complex pattern and any of the following conditions holds, then the AS MUST reply with a 4.00 (Bad Request) error response (see {{Section 5.8.3 of RFC9200}}). The 'error_description' parameter carried out in the response payload MUST specify the CBOR value 1 (invalid_scope).

   * The "Toid" of the different scope entries that specify a complex pattern do not all adhere to the same pattern semantics.

   * The "Toid" of the different scope entries that specify a complex pattern adhere to the same pattern semantics, but this is not supported by the AS or by the Group Manager.

   Finally, as discussed in {{scope-format}}, the authorization information included in the Authorization Request or specified by the AS might also include permissions for the same Client as a user of an OSCORE group, i.e., as an actual group member or an external signature verifier. As per {{scope-format}}, such authorization information is expressed by "user scope entries", whose format and processing is specified in {{I-D.ietf-ace-key-groupcomm-oscore}}.

3. The Administrator transfers authentication and authorization information to the Group Manager by posting the obtained Access Token, according to the used profile of ACE, such as {{RFC9202}} and {{RFC9203}}. After that, the Administrator must have a secure communication association established with the Group Manager, before performing any administrative operation on that Group Manager. Possible ways to provide secure communication are DTLS {{RFC9147}} and OSCORE {{RFC8613}}. The Administrator and the Group Manager maintain the secure association, to support possible future communications.

4. Consistently with what is allowed by the authorization information in the Access Token, the Administrator performs administrative operations at the Group Manager, as described in {{interactions}}. These include retrieving a list of existing OSCORE groups, creating new OSCORE groups, retrieving and changing OSCORE group configurations, and removing OSCORE groups. Messages exchanged among the Administrator and the Group Manager are specified in {{interactions}}.

   Upon receiving a request from the Administrator targeting the group-configuration resource or a group-collection resource, the Group Manager MUST check that it is storing a valid Access Token for that Administrator. If this is not the case, the Group Manager MUST reply with a 4.01 (Unauthorized) error response.

   If the request targets the group-configuration resource associated with a group with name GROUPNAME, the Group Manager MUST check that it is storing a valid Access Token from that Administrator, such that the 'scope' claim specified in the Access Token: i) expresses authorization information through scope entries as defined in {{scope-format}}; and ii) specifically includes a scope entry where:

   * The group name GROUPNAME matches with the pattern specified by the "Toid" of the scope entry; and

   * The permission set specified by the "Tperm" of the scope entry allows the Administrator to perform the requested administrative operation on the targeted group-configuration resource.

   Note that the checks defined above only consider scope entries expressing permissions for administrative operations, namely "admin scope entries" as defined in {{scope-format}}, while the alternative "user scope entries" defined in {{I-D.ietf-ace-key-groupcomm-oscore}} are not considered.

   Further detailed checks to perform are defined separately for each operation at the Group Manager, when specified in {{interactions}}.

   In case the Group Manager stores a valid Access Token but the verifications above fail, the Group Manager MUST reply with a 4.03 (Forbidden) error response. This response MAY be an AS Request Creation Hints, as defined in {{Section 5.3 of RFC9200}}, in which case the Content-Format MUST be set to application/ace+cbor.

   If the request is not formatted correctly (e.g., required fields are not present or are not encoded as expected), the Group Manager MUST reply with a 4.00 (Bad Request) error response.

# Group Configurations # {#group-configurations}

A group configuration consists of a set of parameters.

## Group Configuration Representation ## {#config-repr}

The group configuration representation is a CBOR map, which includes configuration properties and status properties.

### Configuration Properties ### {#config-repr-config-properties}

The CBOR map includes the following configuration parameters, whose CBOR abbreviations are defined in {{groupcomm-parameters}} of this document.

* 'hkdf', which specifies the HKDF Algorithm used in the OSCORE group, encoded as a CBOR text string or a CBOR integer. Possible values are the same ones admitted for the 'hkdf' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'cred_fmt', which specifies the format of authentication credentials used in the OSCORE group, encoded as a CBOR integer. Possible values are the same ones admitted for the 'cred\_fmt' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'group_mode', encoded as a CBOR simple value. Its value is "true" (0xf5) if the OSCORE group uses the group mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}, or "false" (0xf4) otherwise.

* 'sign_enc_alg', which is formatted as follows. If the configuration parameter 'group_mode' has value "false" (0xf4), this parameter has as value the CBOR simple value "null" (0xf6). Otherwise, this parameter specifies the Signature Encryption Algorithm used in the OSCORE group to encrypt messages protected with the group mode, encoded as a CBOR text string or a CBOR integer. Possible values are the same ones admitted for the 'sign_enc_alg' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'sign_alg', which is formatted as follows. If the configuration parameter 'group_mode' has value "false" (0xf4), this parameter has as value the CBOR simple value "null" (0xf6). Otherwise, this parameter specifies the Signature Algorithm used in the OSCORE group, encoded as a CBOR text string or a CBOR integer. Possible values are the same ones admitted for the 'sign_alg' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'sign_params', which is formatted as follows. If the configuration parameter 'group_mode' has value "false" (0xf4), this parameter has as value the CBOR simple value "null" (0xf6). Otherwise, this parameter specifies the additional parameters for the Signature Algorithm used in the OSCORE group, encoded as a CBOR array. Possible formats and values are the same ones admitted for the 'sign_params' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'pairwise_mode', encoded as a CBOR simple value. Its value is "true" (0xf5) if the OSCORE group uses the pairwise mode of Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}, or "false" (0xf4) otherwise.

* 'alg', which is formatted as follows. If the configuration parameter 'pairwise_mode' has value "false" (0xf4), this parameter has as value the CBOR simple value "null" (0xf6). Otherwise, this parameter specifies the AEAD Algorithm used in the OSCORE group to encrypt messages protected with the pairwise mode, encoded as a CBOR text string or a CBOR integer. Possible values are the same ones admitted for the 'alg' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'ecdh_alg', which is formatted as follows. If the configuration parameter 'pairwise_mode' has value "false" (0xf4), this parameter has as value the CBOR simple value "null" (0xf6). Otherwise, this parameter specifies the Pairwise Key Agreement Algorithm used in the OSCORE group, encoded as a CBOR text string or a CBOR integer. Possible values are the same ones admitted for the 'ecdh_alg' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'ecdh_params', which is formatted as follows. If the configuration parameter 'pairwise_mode' has value "false" (0xf4), this parameter has as value the CBOR simple value "null" (0xf6). Otherwise, this parameter specifies the parameters for the Pairwise Key Agreement Algorithm used in the OSCORE group, encoded as a CBOR array. Possible formats and values are the same ones admitted for the 'ecdh_params' parameter of the Group_OSCORE_Input_Material object, defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'det_req', encoded as a CBOR simple value. Its value is "true" (0xf5) if the OSCORE group uses deterministic requests as defined in {{I-D.amsuess-core-cachable-oscore}}, or "false" (0xf4) otherwise. This parameter MUST NOT be present if the configuration parameter 'group_mode' has value "false" (0xf4).

* 'det_hash_alg', encoded as a CBOR integer or text string. If present, this parameter specifies the Hash Algorithm used in the OSCORE group when producing deterministic requests, as defined in {{I-D.amsuess-core-cachable-oscore}}. This parameter takes values from the "Value" column of the "COSE Algorithms" Registry {{COSE.Algorithms}}.

   This parameter MUST NOT be present if the configuration parameter 'det_req' is not present or if it is present with value "false" (0xf4). If the configuration parameter 'det_req' is present with value "true" (0xf5) and 'det_hash_alg' is not present, the choice of the Hash Algorithm to use when producing deterministic requests is left to the Group Manager.

### Status Properties ### {#config-repr-status-properties}

The CBOR map includes the following status parameters. Unless specified otherwise, these are defined in this document and their CBOR abbreviations are defined in {{groupcomm-parameters}}.

* 'rt', with value the resource type "core.osc.gconf" associated with group-configuration resources, encoded as a CBOR text string.

* 'active', encoding the CBOR simple value "true" (0xf5) if the OSCORE group is currently active, or the CBOR simple value "false" (0xf4) otherwise.

* 'group_name', with value the group name of the OSCORE group encoded as a CBOR text string.

* 'group_title', with value either a human-readable description of the OSCORE group encoded as a CBOR text string, or the CBOR simple value "null" (0xf6) if no description is specified.

* 'ace-groupcomm-profile', defined in {{Section 4.3.1 of I-D.ietf-ace-key-groupcomm}}, with value "coap_group_oscore_app" defined in {{Section 16.5 of I-D.ietf-ace-key-groupcomm-oscore}} encoded as a CBOR integer.

* 'max_stale_sets', encoding a CBOR unsigned integer with value strictly greater than 1. With reference to {{Section 7.1 of I-D.ietf-ace-key-groupcomm-oscore}}, this parameter specifies N, i.e., the maximum number of sets of stale OSCORE Sender IDs that the Group Manager stores in the collection associated with the group.

* 'exp', defined in {{Section 4.3.1 of I-D.ietf-ace-key-groupcomm}}.

* 'gid_reuse', encoding the CBOR simple value "true" (0xf5) if, upon rekeying the OSCORE group, the Group Manager can reassign the values of the OSCORE Group ID used as OSCORE ID Context, as per {{Section 3.2.1.1 of I-D.ietf-core-oscore-groupcomm}} and {{Section 11 of I-D.ietf-ace-key-groupcomm-oscore}}. Otherwise, this parameter encodes the CBOR simple value "false" (0xf4).

* 'app_groups', with value a list of names of application groups, encoded as a CBOR array. Each element of the array is a CBOR text string, specifying the name of an application group using the OSCORE group as security group (see {{Section 2.1 of I-D.ietf-core-groupcomm-bis}}).

* 'joining_uri', with value the URI of the group-membership resource for joining the newly created OSCORE group as per {{Section 6.2 of I-D.ietf-ace-key-groupcomm-oscore}}, encoded as a CBOR text string.

* 'group_policies', defined in {{Section 4.3.1 of I-D.ietf-ace-key-groupcomm}}, and consistent with the format and content defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

* 'as_uri', with value the URI of the Authorization Server associated with the Group Manager for the OSCORE group, encoded as a CBOR text string. Candidate group members will have to obtain an Access Token from that Authorization Server, before starting the joining process with the Group Manager to join the OSCORE group (see {{Sections 5 and 6 of I-D.ietf-ace-key-groupcomm-oscore}}).

## Default Values {#default-values}

This section defines the default values that the Group Manager refers to for configuration and status parameters.

### Configuration Parameters {#default-values-conf}

For each of the configuration parameters listed below, the Group Manager refers to the following pre-configured default value, if none is specified by the Administrator.

* For 'group_mode', the Group Manager SHOULD use the CBOR simple value "true" (0xf5).

* If 'group_mode' has value "true" (0xf5), the Group Manager SHOULD use the same default values defined in {{Section 14.2 of I-D.ietf-ace-key-groupcomm-oscore}} for the parameters 'sign_enc_alg', 'sign_alg' and 'sign_params'.

* If 'group_mode' has value "true" (0xf5), the Group Manager SHOULD use the CBOR simple value "false" (0xf4) for the parameter 'det_req'.

* If 'det_req' has value "true" (0xf5), the Group Manager SHOULD use SHA-256 (COSE algorithm encoding: -16) as default value for the parameter 'det_hash_alg'.

* For 'pairwise_mode', the Group Manager SHOULD use the CBOR simple value "false" (0xf4).

* If 'pairwise_mode' has value "true" (0xf5), the Group Manager SHOULD use the same default values defined in {{Section 14.3 of I-D.ietf-ace-key-groupcomm-oscore}} for the parameters 'alg', 'ecdh_alg' and 'ecdh_params'.

* For any other configuration parameter, the Group Manager SHOULD use the same default values defined in {{Section 14.1 of I-D.ietf-ace-key-groupcomm-oscore}}.

### Status Parameters

For each of the status parameters listed below, the Group Manager refers to the following pre-configured default value, if none is specified by the Administrator.

* For 'active', the Group Manager SHOULD use the CBOR simple value "false" (0xf4).

* For 'group_title', the Group Manager SHOULD use the CBOR simple value "null" (0xf6).

* For 'max_stale_sets', the Group Manager SHOULD use the CBOR unsigned integer value 3.

* For 'gid_reuse', the Group Manager SHOULD use the CBOR simple value "false" (0xf4).

* For 'app_groups', the Group Manager SHOULD use the empty CBOR array.

* For 'group_policies', the Group Manager SHOULD use the default values defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}.

# Interactions with the Group Manager # {#interactions}

This section describes the operations that are possible to perform on the group-collection resource and the group-configuration resources at the Group Manager.

For each operation, it is defined whether that operation is required or optional to support for the Group Manager and an Administrator. If the Group Manager supports an operation, then the Group Manager must be able to correctly handle authorized and valid requests sent by the Administrator to carry out that operation. If the Group Manager receives an authorized and valid request to perform an operation that it does not support, then the Group Manager MUST respond with a 5.01 (Not Implemented) response.

When checking the scope claim of a stored access token to verify that any of the requests defined in the following is authorized, the Group Manager only considers scope entries expressing permissions for administrative operations, namely "admin scope entries" as defined in {{scope-format}}. Instead, the alternative "user scope entries" defined in {{I-D.ietf-ace-key-groupcomm-oscore}} are not considered. That is, when handling any of the requests for administrative operations defined in the following, the Group Manager ignores possible "user scope entries" specified in the scope of a stored access token.

When custom CBOR is used, the Content-Format in messages containing a payload is set to application/ace-groupcomm+cbor, defined in {{Section 11.2 of I-D.ietf-ace-key-groupcomm}}. Furthermore, the entry labels defined in {{groupcomm-parameters}} of this document MUST be used, when specifying the corresponding configuration and status parameters.

## Retrieve the Full List of Group Configurations ## {#collection-resource-get}

This operation MUST be supported by the Group Manager and an Administrator.

The Administrator can send a GET request to the group-collection resource, in order to retrieve a list of the existing OSCORE groups at the Group Manager. This is returned as a list of links to the corresponding group-configuration resources.

The Group Manager MUST prepare the list L to include in the response as follows. For each group-configuration resource R:

1. The Group Manager considers the group name GROUPNAME of the OSCORE group associated with R.

2. The Group Manager retrieves the stored Access Token for the Administrator. Then, it checks whether GROUPNAME matches with the group name pattern specified in any scope entry of the 'scope' claim in the Access Token.

3. The link to the group-configuration resource R is added to the list L only in case of a positive match.

Example in Link Format:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage

<= 2.05 Content
   Content-Format: 40 (application/link-format)

   <coap://[2001:db8::ab]/manage/gp1>;rt="core.osc.gconf",
   <coap://[2001:db8::ab]/manage/gp2>;rt="core.osc.gconf",
   <coap://[2001:db8::ab]/manage/gp3>;rt="core.osc.gconf"
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage

<= 2.05 Content
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gcoll#>
   #base </manage/>
   item <gp1>
   item <gp2>
   item <gp3>
~~~~~~~~~~~

## Retrieve a List of Group Configurations by Filters ## {#collection-resource-fetch}

This operation MUST be supported by the Group Manager and MAY be supported by an Administrator.

The Administrator can send a FETCH request to the group-collection resource, in order to retrieve a list of the existing OSCORE groups that fully match a set of specified filter criteria. This is returned as a list of links to the corresponding group-configuration resources.

When custom CBOR is used, the set of filter criteria is specified in the request payload as a CBOR map, whose possible entries are specified in {{config-repr}} and use the same abbreviations defined in {{groupcomm-parameters}}. Entry values are the ones admitted for the corresponding labels in the POST request for creating a group configuration (see {{collection-resource-post}}). A valid request MUST NOT include the same entry multiple times.

When CoRAL is used, the filter criteria are specified in the request payload with top-level elements, each of which corresponds to an entry specified in {{config-repr}}, with the exception of the 'app_groups' status parameter. If names of application groups are used as filter criteria, each element of the 'app_groups' array from the status properties is included as a separate element with name 'app_group'. With the exception of the 'app_group' element, a valid request MUST NOT include the same element multiple times. Element values are the ones admitted for the corresponding labels in the POST request for creating a group configuration (see {{collection-resource-post}}).

The Group Manager MUST prepare the list L to include in the response as follows.

1. The Group Manager prepares a preliminary version of the list L, as specified in {{collection-resource-get}} for the processing of a GET request to the group-collection resource.

2. The Group Manager applies the filter criteria specified in the FETCH request to the list L from the previous step. The result is the list L to include in the response.

Example in custom CBOR and Link Format:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: manage
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
       "group_mode" : true,
       "sign_enc_alg" : 10,
       "hkdf" : 5
   }

<= 2.05 Content
   Content-Format: 40 (application/link-format)

   <coap://[2001:db8::ab]/manage/gp1>;rt="core.osc.gconf",
   <coap://[2001:db8::ab]/manage/gp2>;rt="core.osc.gconf",
   <coap://[2001:db8::ab]/manage/gp3>;rt="core.osc.gconf"
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: manage
   Content-Format: TBD1 (application/coral+cbor)

   group_mode true
   sign_enc_alg 10
   hkdf 5

<= 2.05 Content
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gcoll#>
   #base </manage/>
   item <gp1>
   item <gp2>
   item <gp3>
~~~~~~~~~~~

## Create a New Group Configuration ## {#collection-resource-post}

This operation MUST be supported by the Group Manager and an Administrator.

The Administrator can send a POST request to the group-collection resource, in order to create a new OSCORE group at the Group Manager. The request MUST specify the intended group name GROUPNAME, and MAY specify the intended group title together with pieces of information concerning the group configuration.

When custom CBOR is used, the request payload is a CBOR map, whose possible entries are specified in {{config-repr}} and use the same abbreviations defined in {{groupcomm-parameters}}.

When CoRAL is used, each element of the request payload corresponds to an entry specified in {{config-repr}}, with the exception of the 'app_groups' status parameter (see below).

In particular:

* The payload MAY include any of the configuration parameters defined in {{config-repr-config-properties}}.

* The payload MUST include the status parameter 'group_name' defined in {{config-repr-status-properties}} and specifying the intended group name.

* The payload MAY include any of the status parameters 'active', 'group_title', 'max_stale_sets', 'exp', 'gid_reuse', 'app_groups, 'group_policies' and 'as_uri' defined in {{config-repr-status-properties}}.

   When CoRAL is used, each element of the 'app_groups' array from the status properties is included as a separate element with name 'app_group'.

* The payload MUST NOT include any of the status parameters 'rt', 'ace-groupcomm-profile' and 'joining_uri' defined in {{config-repr-status-properties}}.

Consistently with what is defined at step 4 of {{getting-access}}, the Group Manager MUST check whether the group name specified in the 'group_name' parameter matches with the group name pattern specified in any scope entry of the 'scope' claim in the stored Access Token for the Administrator. In case of a positive match, the Group Manager MUST check whether the permission set in the found scope entry specifies the permission "Create".

If the verification above fails (i.e., there are no matching scope entries specifying the "Create" permission), the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}.

Otherwise, if any of the following occurs, the Group Manager MUST respond with a 4.00 (Bad Request) response.

* Any of the received parameters is specified multiple times, with the exception of the 'app_group' element when using CoRAL.

* Any of the received parameters is not recognized, or not valid, or not consistent with respect to other related parameters.

* The Group Manager does not trust the Authorization Server with URI specified in the 'as_uri' parameter, and has no alternative Authorization Server to consider for the OSCORE group to create.

After a successful processing of the POST request, the Group Manager performs the following actions.

If the 'group_name' parameter specifies the group name of an already existing OSCORE group, the Group Manager MUST find an alternative name for the new OSCORE group to create.

In addition to that, the final decision about the name assigned to the new OSCORE group is always of the Group Manager, which may have more constraints than the Administrator can be aware of, possibly beyond the availability of suggested names. For example, the Group Manager may specifically want to use a randomized character string as the name of a newly created group.

If the Group Manager has selected a name GROUPNAME different from the name GROUPNAME\* indicated in the parameter 'group_name' of the request, then the following conditions MUST hold.

* The chosen name GROUPNAME is available to assign; and

* If GROUPNAME\* matches with the group name pattern of certain scope entries from the 'scope' claim in the stored Access Token for the Administrator, then the chosen group name GROUPNAME also matches with each of those group name patterns.

If the Group Manager does not find any group name for which both the above conditions hold, the Group Manager MUST respond with a 5.03 (Service Unavailable) response. The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}. The value of the 'error' field MUST be set to 11 ("No available group names").

Otherwise, the Group Manager creates a new group-configuration resource, accessible to the Administrator at /manage/GROUPNAME, where GROUPNAME is the name of the OSCORE group as either indicated in the parameter 'group_name' of the request or uniquely assigned by the Group Manager.

The value of the status parameter 'rt' is set to "core.osc.gconf". The values of other parameters specified in the request are used as group configuration information for the newly created OSCORE group.

If the request specifies the parameter 'gid_reuse' encoding the CBOR simple value "true" (0xf5) and the Group Manager does not support the reassignment of OSCORE Group ID values (see {{Section 3.2.1.1 of I-D.ietf-core-oscore-groupcomm}} and {{Section 11 of I-D.ietf-ace-key-groupcomm-oscore}}), then the Group Manager sets the value of the 'gid_reuse' status parameter in the group-configuration resource to the CBOR simple value "false" (0xf4).

For each parameter not specified in the request, the Group Manager refers to the default values specified in {{default-values}}.

After that, the Group Manager creates a new group-membership resource accessible at ace-group/GROUPNAME to nodes that want to join the OSCORE group, as specified in {{Section 6.1 of I-D.ietf-ace-key-groupcomm-oscore}}. Note that such group membership-resource comprises a number of sub-resources intended to current group members, as defined in {{Section 4.1 of I-D.ietf-ace-key-groupcomm}} and {{Section 8 of I-D.ietf-ace-key-groupcomm-oscore}}.

From then on, the Group Manager will rely on the current group configuration to build the Join Response message defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}, when handling the joining of a new group member. Furthermore, the Group Manager generates the following pieces of information, and assigns them to the newly created OSCORE group.

* The OSCORE Master Secret.

* The OSCORE Master Salt (optionally).

* The Group ID, used as OSCORE ID Context, which MUST be unique within the set of OSCORE groups under the Group Manager.

Finally, the Group Manager replies to the Administrator with a 2.01 (Created) response. The Location-Path option MUST be included in the response, indicating the location of the just created group-configuration resource. The response MUST NOT include a Location-Query option.

The response payload specifies the parameters 'group_name', 'joining_uri' and 'as_uri', from the status properties of the newly created OSCORE group (see {{config-repr}}), as detailed below.

When custom CBOR is used, the response payload is a CBOR map, where entries use the same abbreviations defined in {{groupcomm-parameters}}. When CoRAL is used, the response payload includes one element for each specified parameter.

* 'group_name', with value the group name of the OSCORE group. This value can be different from the group name possibly specified by the Administrator in the POST request, and reflects the final choice of the Group Manager as 'group_name' status property for the OSCORE group. This parameter MUST be included.

* 'joining_uri', with value the URI of the group-membership resource for joining the newly created OSCORE group. This parameter MUST be included.

* 'as_uri', with value the URI of the Authorization Server associated with the Group Manager for the newly created OSCORE group. This parameter MUST be included. Its value can be different from the URI possibly specified by the Administrator in the POST request, and reflects the final choice of the Group Manager as 'as_uri' status property for the OSCORE group.

If the POST request specified the parameter 'gid_reuse' encoding the CBOR simple value "true" (0xf5) but the Group Manager has set the value of the 'gid_reuse' status parameter in the group-configuration resource to the CBOR simple value "false" (0xf4), then the response payload MUST include also the parameter 'gid_reuse' encoding the CBOR simple value "false" (0xf4).

If the POST request did not specify certain parameters and the Group Manager used default values different from the ones recommended in {{default-values}}, then the response payload MUST include also those parameters, specifying the values chosen by the Group Manager for the current group configuration.

The Group Manager can register the link to the group-membership resource with URI specified in 'joining_uri' to a Resource Directory {{RFC9176}}{{I-D.hartke-t2trg-coral-reef}}, as defined in {{Section 2 of I-D.tiloca-core-oscore-discovery}}. The Group Manager considers the current group configuration when specifying additional information for the link to register.

Alternatively, the Administrator can perform the registration in the Resource Directory on behalf of the Group Manager, acting as Commissioning Tool. The Administrator considers the following when specifying additional information for the link to register.

* The name of the OSCORE group MUST take the value specified in 'group_name' from the 2.01 (Created) response.

* The names of the application groups using the OSCORE group MUST take the values possibly specified by the elements of the 'app_groups' parameter (when custom CBOR is used) or by the different 'app_group' elements (when CoRAL is used) in the POST request.

* If also registering a related link to the Authorization Server associated with the OSCORE group, the related link MUST have as link target the URI in 'as_uri' from the 2.01 (Created) response.

* As to every other information element describing the current group configuration, the following applies.

   - If a certain parameter was specified in the POST request, the Administrator MUST use either the value specified in the 2.01 (Created) response, if the Group Manager specified one, or the value specified in the POST request otherwise.

   - If a certain parameter was not specified in the POST request, the Administrator MUST use either the value specified in the 2.01 (Created) response, if the Group Manager specified one, or the corresponding default value recommended in {{default-values-conf}} otherwise.

Note that, compared to the Group Manager, the Administrator is less likely to remain closely aligned with possible changes and updates that would require a prompt update to the registration in the Resource Directory. This applies especially to the address of the Group Manager, as well as the URI of the group-membership resource or of the Authorization Server associated with the Group Manager.

Therefore, it is RECOMMENDED that registrations of links to group-membership resources in the Resource Directory are made (and possibly updated) directly by the Group Manager, rather than by the Administrator.

Example in custom CBOR:

~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: manage
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "sign_enc_alg" : 10,
     "hkdf" : 5,
     "pairwise_mode" : true,
     "active" : true,
     "group_name" : "gp4",
     "group_title" : "rooms 1 and 2",
     "app_groups": : ["room1", "room2"],
     "as_uri" : "coap://as.example.com/token"
   }

<= 2.01 Created
   Location-Path: manage
   Location-Path: gp4
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "group_name" : "gp4",
     "joining_uri" : "coap://[2001:db8::ab]/ace-group/gp4/",
     "as_uri" : "coap://as.example.com/token"
   }
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: manage
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   sign_enc_alg 10
   hkdf 5
   pairwise_mode true
   active true
   group_name "gp4"
   group_title "rooms 1 and 2"
   app_group "room1"
   app_group "room2"
   as_uri <coap://as.example.com/token>

<= 2.01 Created
   Location-Path: manage
   Location-Path: gp4
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   group_name "gp4"
   joining_uri <coap://[2001:db8::ab]/ace-group/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

## Retrieve a Group Configuration ## {#configuration-resource-get}

This operation MUST be supported by the Group Manager and an Administrator.

The Administrator can send a GET request to the group-configuration resource manage/GROUPNAME associated with an OSCORE group with group name GROUPNAME, in order to retrieve the complete current configuration of that group.

Consistently with what is defined at step 4 of {{getting-access}}, the Group Manager MUST check whether GROUPNAME matches with the group name pattern specified in any scope entry of the 'scope' claim in the stored Access Token for the Administrator. In case of a positive match, the Group Manager MUST check whether the permission set in the found scope entry specifies the permission "Read".

If the verification above fails (i.e., there are no matching scope entries specifying the "Read" permission), the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}.

Otherwise, after a successful processing of the GET request, the Group Manager replies to the Administrator with a 2.05 (Content) response. The response has as payload the representation of the group configuration as specified in {{config-repr}}. The exact content of the payload reflects the current configuration of the OSCORE group. This includes both configuration properties and status properties.

When custom CBOR is used, the response payload is a CBOR map, whose possible entries are specified in {{config-repr}} and use the same abbreviations defined in {{groupcomm-parameters}}.

When CoRAL is used, the response payload includes one element for each entry specified in {{config-repr}}, with the exception of the 'app_groups' status parameter. That is, each element of the 'app_groups' array from the status properties is included as a separate element with name 'app_group'.

Example in custom CBOR:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: manage
   Uri-Path: gp4

<= 2.05 Content
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "hkdf" : 5,
     "cred_fmt" : 33,
     "group_mode" : true,
     "sign_enc_alg" : 10,
     "sign_alg" : -8,
     "sign_params" : [[1], [1, 6]],
     "pairwise_mode" : true,
     "alg" : 10,
     "ecdh_alg" : -27,
     "ecdh_params" : [[1], [1, 6]],
     "det_req" : false,
     "rt" : "core.osc.gconf",
     "active" : true,
     "group_name" : "gp4",
     "group_title" : "rooms 1 and 2",
     "ace-groupcomm-profile" : "coap_group_oscore_app",
     "max_stale_sets" : 3,
     "gid_reuse" : false,
     "exp" : 1360289224,
     "app_groups": : ["room1", "room2"],
     "joining_uri" : "coap://[2001:db8::ab]/ace-group/gp4/",
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

   #using <http://coreapps.org/core.osc.gconf#>
   hkdf 5
   cred_fmt 33
   group_mode true
   sign_enc_alg 10
   sign_alg -8
   sign_params.alg_capab.key_type 1
   sign_params.key_type_capab.key_type 1
   sign_params.key_type_capab.curve 6
   pairwise_mode true
   alg 10
   ecdh_alg -27
   ecdh_params.alg_capab.key_type 1
   ecdh_params.key_type_capab.key_type 1
   ecdh_params.key_type_capab.curve 6
   det_req false
   rt "core.osc.gconf"
   active true
   group_name "gp4"
   group_title "rooms 1 and 2"
   ace-groupcomm-profile "coap_group_oscore_app"
   max_stale_sets 3
   gid_reuse false
   exp 1360289224
   app_group "room1"
   app_group "room2"
   joining_uri <coap://[2001:db8::ab]/ace-group/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

## Retrieve Part of a Group Configuration by Filters ## {#configuration-resource-fetch}

This operation MUST be supported by the Group Manager and MAY be supported by an Administrator.

The Administrator can send a FETCH request to the group-configuration resource manage/GROUPNAME associated with an OSCORE group with group name GROUPNAME, in order to retrieve part of the current configuration of that group.

When custom CBOR is used, the request payload is a CBOR map, which contains the following fields:

* 'conf_filter', encoded as a CBOR array and with CBOR abbreviation defined in {{groupcomm-parameters}}. Each element of the array specifies one requested configuration parameter or status parameter of the current group configuration (see {{config-repr}}).

When CoRAL is used, the request payload includes one element for each requested configuration parameter or status parameter of the current group configuration (see {{config-repr}}). All the specified elements have no value.

The Group Manager MUST perform the same authorization checks defined for the processing of a GET request to a group-configuration resource in {{configuration-resource-get}}. That is, the Group Manager MUST verify that the Administrator has been granted a "Read" permission applicable to the targeted group-configuration resource.

After a successful processing of the FETCH request, the Group Manager replies to the Administrator with a 2.05 (Content) response. The response has as payload a partial representation of the group configuration (see {{config-repr}}). The exact content of the payload reflects the current configuration of the OSCORE group, and is limited to the configuration properties and status properties requested by the Administrator in the FETCH request.

The response payload includes the requested configuration parameters and status parameters, and is formatted as in the response payload of a GET request to a group-configuration resource (see {{configuration-resource-get}}).

Example in custom CBOR:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "conf_filter" : ["sign_enc_alg",
                      "hkdf",
                      "pairwise_mode",
                      "active",
                      "group_title",
                      "app_groups"]
   }

<= 2.05 Content
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "sign_enc_alg" : 10,
     "hkdf" : 5,
     "pairwise_mode" : true,
     "active" : true,
     "group_title" : "rooms 1 and 2",
     "app_groups": : ["room1", "room2"]
   }

~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   sign_enc_alg
   hkdf
   pairwise_mode
   active
   group_title
   app_groups

<= 2.05 Content
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   sign_enc_alg 10
   hkdf 5
   pairwise_mode true
   active true
   group_title "rooms 1 and 2"
   app_group "room1"
   app_group "room2"
~~~~~~~~~~~

## Overwrite a Group Configuration ## {#configuration-resource-put}

This operation MAY be supported by the Group Manager and an Administrator.

The Administrator can send a PUT request to the group-configuration resource associated with an OSCORE group, in order to overwrite the current configuration of that group with a new one. The payload of the request has the same format of the POST request defined in {{collection-resource-post}}, with the exception that the configuration parameters 'group_mode' and 'pairwise_mode' as well as the status parameters 'group_name' and 'gid_reuse' MUST NOT be included.

The error handling for the PUT request is the same as for the POST request defined in {{collection-resource-post}}, with the following difference in terms of authorization checks.

Consistently with what is defined at step 4 of {{getting-access}}, the Group Manager MUST check whether GROUPNAME matches with the group name pattern specified in any scope entry of the 'scope' claim in the stored Access Token for the Administrator. In case of a positive match, the Group Manager MUST check whether the permission set in the found scope entry specifies the permission "Write".

If the verification above fails (i.e., there are no matching scope entries specifying the "Write" permission), the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}.

If no error occurs and the PUT request is successfully processed, the Group Manager performs the following actions.

First, the Group Manager updates the group-configuration resource, consistently with the values indicated in the PUT request from the Administrator. For each parameter not specified in the PUT request, the Group Manager MUST use default values as specified in {{default-values}}.

If a new value N' is specified for the 'max_stale_sets' status parameter and N' is smaller than the current value N, the Group Manager preserves the (up to) N' most recent sets in the collection of sets of stale OSCORE Sender IDs associated with the group, and deletes any possible older set from the collection (see {{Section 7.1 of I-D.ietf-ace-key-groupcomm-oscore}}).

From then on, the Group Manager relies on the latest updated configuration to build the Join Response message defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}, when handling the joining of a new group member. Similarly, the Group Manager relies on the new group configuration when building responses specifying (part of) the group configuration to a current group member. For instance, this applies when a group member retrieves from the Group Manager the updated group keying material (see {{Section 9.1 of I-D.ietf-ace-key-groupcomm-oscore}}) or the current group status (see {{Section 9.9 of I-D.ietf-ace-key-groupcomm-oscore}}).

Then, the Group Manager replies to the Administrator with a 2.04 (Changed) response. The payload of the response has the same format of the 2.01 (Created) response defined in {{collection-resource-post}}.

If the PUT request did not specify certain parameters and the Group Manager used default values different from the ones recommended in {{default-values}}, then the response payload MUST include also those parameters, specifying the values chosen by the Group Manager for the current group configuration.

If the link to the group-membership resource was registered in the Resource Directory {{RFC9176}}, the Group Manager is responsible to refresh the registration, as defined in {{Section 3 of I-D.tiloca-core-oscore-discovery}}.

Alternatively, the Administrator can update the registration in the Resource Directory on behalf of the Group Manager, acting as Commissioning Tool. The Administrator considers the following when specifying additional information for the link to update.

* The name of the OSCORE group MUST take the value specified in 'group_name' from the 2.04 (Changed) response.

* The names of the application groups using the OSCORE group MUST take the values possibly specified by the elements of the 'app_groups' parameter (when custom CBOR is used) or by the different 'app_group' elements (when CoRAL is used) in the PUT request.

* If also registering a related link to the Authorization Server associated with the OSCORE group, the related link MUST have as link target the URI in 'as_uri' from the 2.04 (Changed) response.

* As to every other information element describing the current group configuration, the following applies.

   - If a certain parameter was specified in the PUT request, the Administrator MUST use either the value specified in the 2.04 (Changed) response, if the Group Manager specified one, or the value specified in the PUT request otherwise.

   - If a certain parameter was not specified in the PUT request, the Administrator MUST use either the value specified in the 2.04 (Changed) response, if the Group Manager specified one, or the corresponding default value recommended in {{default-values-conf}} otherwise.

As discussed in {{collection-resource-post}}, it is RECOMMENDED that registrations of links to group-membership resources in the Resource Directory are made (and possibly updated) directly by the Group Manager, rather than by the Administrator.

Example in custom CBOR:

~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "sign_enc_alg" : 11,
     "hkdf" : 5
   }

<= 2.04 Changed
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "group_name" : "gp4",
     "joining_uri" : "coap://[2001:db8::ab]/ace-group/gp4/",
     "as_uri" : "coap://as.example.com/token"
   }
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   sign_enc_alg 11
   hkdf 5

<= 2.04 Changed
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   group_name "gp4"
   joining_uri <coap://[2001:db8::ab]/ace-group/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

### Effects on Joining Nodes ### {#sssec-effects-overwrite-joining-nodes}

After having overwritten a group configuration, if the value of the status parameter 'active' is changed from "true" (0xf5) to "false" (0xf4), the Group Manager MUST stop admitting new members in the OSCORE group. In particular, until the status parameter 'active' is changed back to "true" (0xf5), the Group Manager MUST respond to a Join Request with a 5.03 (Service Unavailable) response, as defined in {{Section 6.2 of I-D.ietf-ace-key-groupcomm-oscore}}.

If the value of the status parameter 'active' is changed from "false" (0xf4) to "true" (0xf5), the Group Manager resumes admitting new members in the OSCORE group, by processing their Join Requests (see {{Section 6.2 of I-D.ietf-ace-key-groupcomm-oscore}}).

### Effects on the Group Members ### {#sssec-effects-overwrite-group-members}

After having overwritten a group configuration, the Group Manager informs the members of the OSCORE group, over the pairwise secure communication channels established when joining the group (see {{Section 6 of I-D.ietf-ace-key-groupcomm-oscore}}).

To this end, the Group Manager can individually target the 'control_uri' URI of each group member (see {{Section 4.3.1 of I-D.ietf-ace-key-groupcomm}}), if provided by the intended recipient upon joining the OSCORE group (see {{Section 6.1 of I-D.ietf-ace-key-groupcomm-oscore}}). To this end, messages sent by the Group Manager to each group member MUST have Content-Format set to application/ace-groupcomm+cbor, and MUST be formatted as the Join Response defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}, with the following differences.

* Only the parameters 'gkty', 'key', 'num', 'exp' and 'ace-groupcomm-profile' are present.

* The 'key' parameter includes only the parameters 'hkdf', 'cred_fmt', 'sign_enc_alg', 'sign_alg', 'sign_params', 'alg', 'ecdh_alg' and 'ecdh_params', with values reflecting the new configuration of the OSCORE group.

Alternatively, group members can subscribe for updates to the group-membership resource of the OSCORE group, e.g., by using CoAP Observe {{RFC7641}}.

If the value of the status parameter 'active' is changed from "true" (0xf5) to "false" (0xf4):

* The Group Manager MUST stop accepting requests for new individual keying material from current group members (see {{Section 9.2 of I-D.ietf-ace-key-groupcomm-oscore}}). In particular, until the status parameter 'active' is changed back to "true" (0xf5), the Group Manager MUST respond to a Key Renewal Request with a 5.03 (Service Unavailable) response, as defined in {{Section 9.2 of I-D.ietf-ace-key-groupcomm-oscore}}.

* The Group Manager MUST stop accepting updated authentication credentials uploaded by current group members (see {{Section 9.4 of I-D.ietf-ace-key-groupcomm-oscore}}). In particular, until the status parameter 'active' is changed back to "true" (0xf5), the Group Manager MUST respond to an Authentication Credential Update Request with a 5.03 (Service Unavailable) response, as defined in {{Section 9.4 of I-D.ietf-ace-key-groupcomm-oscore}}.

Every group member, upon learning that the OSCORE group has been deactivated (i.e., 'active' has value "false" (0xf4)), SHOULD stop communicating in the group.

Every group member, upon learning that the OSCORE group has been reactivated (i.e., 'active' has value "true" (0xf5) again), can resume communicating in the group.

Every group member, upon receiving updated values for 'hkdf', 'sign_enc_alg' and 'alg', MUST either:

* Leave the OSCORE group (see {{Section 9.11 of I-D.ietf-ace-key-groupcomm-oscore}}), e.g., if not supporting the indicated new algorithms; or

* Use the new parameter values, and accordingly re-derive the OSCORE Security Context for the OSCORE group (see {{Section 2 of I-D.ietf-core-oscore-groupcomm}}).

Every group member, upon receiving updated values for 'cred_fmt', 'sign_alg', 'sign_params', 'ecdh_alg' and 'ecdh_params' MUST either:

* Leave the OSCORE group, e.g., if not supporting the indicated new format, algorithms, parameters and encoding; or

* Leave the OSCORE group and rejoin it (see {{Section 6 of I-D.ietf-ace-key-groupcomm-oscore}}). When rejoining the group, a new authentication credential in the indicated format used in the OSCORE group MUST be provided to the Group Manager. The authentication credential as well as the included public key MUST be compatible with the indicated algorithms and parameters.

* Use the new parameter values, and, if required, perform the following actions.

   - Provide the Group Manager with a new authentication credential to use in the OSCORE group (see {{Section 9.4 of I-D.ietf-ace-key-groupcomm-oscore}}). The new authentication credential MUST be in the indicated format used in the OSCORE group. The new authentication credential as well as the included public key MUST be compatible with the indicated algorithms and parameters.

   - Retrieve from the Group Manager the new Group Manager's authentication credential (see {{Section 9.5 of I-D.ietf-ace-key-groupcomm-oscore}}). The new Group Manager's authentication credential is in the indicated format used in the OSCORE group. The new authentication credential as well as the included public key are compatible with the indicated algorithms and parameters.

## Selective Update of a Group Configuration ## {#configuration-resource-patch}

This operation MAY be supported by the Group Manager and an Administrator.

The Administrator can send a PATCH/iPATCH request {{RFC8132}} to the group-configuration resource associated with an OSCORE group, in order to update the value of only part of the group configuration.

The request payload has the same format of the PUT request defined in {{configuration-resource-put}}, with the difference that it MAY also specify names of application groups to be removed from or added to the 'app_groups' status parameter. The names of such application groups are provided as defined below.

* When custom CBOR is used, the CBOR map in the request payload includes the field 'app_groups_diff', whose CBOR abbreviation is defined in {{groupcomm-parameters}}. This field is encoded as a CBOR array including the following two elements.

   - The first element is a CBOR array, namely 'app_groups_del'. Each of its elements is a CBOR text string, with value the name of an application group to remove from the 'app_groups' status parameter.

   - The second element is a CBOR array, namely 'app_groups_add'. Each of its elements is a CBOR text string, with value the name of an application group to add to the 'app_groups' status parameter.

   The CDDL definition {{RFC8610}} of the CBOR array 'app_groups_diff' formatted as in the response from the Group Manager is provided below.

~~~~~~~~~~~ CDDL
   app-group-name = tstr
   name-patch = [* app-group-name]
   app_groups_diff = [app_groups_del: name-patch,
                      app_groups_add: name-patch]
~~~~~~~~~~~
{: #cddl-diff title="CDDL definition of the 'app_groups_diff' field" artwork-align="left"}

   The Group Manager MUST respond with a 4.00 (Bad Request) response in case: both the inner CBOR arrays 'app_groups_del' and 'app_groups_add' are empty; or the CBOR map in the request payload includes both the 'app_groups' field and the 'app_groups_diff' field.

* When CoRAL is used, the request payload includes the following top-level elements.

   - 'app_group_del', with value a text string specifying the name of an application group to remove from the 'app_groups' status parameter. This element can be included multiple times.

   - 'app_group_add', with value a text string specifying the name of an application group to add to the 'app_groups' status parameter. This element can be included multiple times.

   The Group Manager MUST respond with a 4.00 (Bad Request) response, in case the request payload includes both any 'app_group' element as well as any 'app_group_del' and/or 'app_group_add' element.

The error handling for the PATCH/iPATCH request is the same as for the PUT request defined in {{configuration-resource-put}}, with the following additions.

* The set of group configuration parameters to update MUST NOT be empty. That is, the Group Manager MUST respond with a 4.00 (Bad Request) response, if the request payload includes an empty CBOR map (when custom CBOR is used) or no elements (when CoRAL is used).

* If the Request-URI does not point to an existing group-configuration resource, the Group Manager MUST NOT create a new resource, and MUST respond with a 4.04 (Not Found) response.

* When applying the specified updated values would yield an inconsistent group configuration, the Group Manager MUST respond with a 4.09 (Conflict) response.

   The response, MAY include the current representation of the group configuration resource, like when responding to a GET request as defined in {{configuration-resource-get}}. Otherwise, the response SHOULD include a diagnostic payload with additional information for the Administrator to recognize the source of the conflict.

* When the request uses specifically the iPATCH method, the Group Manager MUST respond with a 4.00 (Bad Request) response, in case:

   - When custom CBOR is used, the CBOR map includes the parameter 'app_groups_diff'; or

   - When CoRAL is used, any element 'app_group_del' and/or 'app_group_add' is included.

Furthermore, the Group Manager MUST perform the same authorization checks defined for the processing of a PUT request to a group-configuration resource in {{configuration-resource-put}}. That is, the Group Manager MUST verify that the Administrator has been granted a "Write" permission applicable to the targeted group-configuration resource.

If no error occurs and the PATCH/iPATCH request is successfully processed, the Group Manager performs the following actions.

First, the Group Manager updates the group-configuration resource, consistently with the values indicated in the PATCH/iPATCH request from the Administrator.

Unlike for the PUT request defined in {{configuration-resource-put}}, the Group Manager does not alter the value of configuration parameters and status parameters for which updated values are not specified in the request payload. In particular, the Group Manager does not assign possible default values to those parameters.

Special processing occurs when updating the 'app_groups' status parameter by difference, as defined below. The Administrator should not expect the Group Manager to add or delete names of application group names according to any particular order.

* If the name of an application group to add (delete) is specified multiple times, the Group Manager considers it only once for addition to (deletion from) the 'app_groups' status parameter.

* If the name of an application group to delete is not present in the 'app_groups' status parameter before any change is applied, the Group Manager ignores that name.

* If the name of an application group to add is already present in the 'app_groups' status parameter before any change is applied, the Group Manager ignores that name.

* When custom CBOR is used, the Group Manager:

   - Deletes from the 'app_groups' status parameter the names of the application groups specified in the inner 'app_groups_del' CBOR array of the 'app_groups_diff' field.

   - Adds to the 'app_groups' status parameter the names of the application groups specified in the inner 'app_groups_add' CBOR array of the 'app_groups_diff' field.

* When CoRAL is used, the Group Manager:

   - Deletes from the 'app_groups' status parameter the names of the application groups specified in the different 'app_group_del' elements.

   - Adds to the 'app_groups' status parameter the names of the application groups specified in the different 'app_group_add' elements.

After having updated the group-configuration resource, from then on the Group Manager relies on the new group configuration to build the Join Response message defined in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}, when handling the joining of a new group member. Similarly, the Group Manager relies on the new group configuration when building responses specifying (part of) the group configuration to a current group member. For instance, this applies when a group member retrieves from the Group Manager the updated group keying material (see {{Section 9.1 of I-D.ietf-ace-key-groupcomm-oscore}}) or the current group status (see {{Section 9.9 of I-D.ietf-ace-key-groupcomm-oscore}}).

Finally, the Group Manager replies to the Administrator with a 2.04 (Changed) response. The payload of the response has the same format of the 2.01 (Created) response defined in {{collection-resource-post}}.

The same considerations as for the PUT request defined in {{configuration-resource-put}} hold also in this case, with respect to refreshing a possible registration of the link to the group-membership resource in the Resource Directory {{RFC9176}}.

Example in custom CBOR:

~~~~~~~~~~~
=> 0.06 PATCH
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "sign_enc_alg" : 10,
     "app_groups_diff" : [["room1"],
                          ["room3", "room4"]]
   }

<= 2.04 Changed
   Content-Format: TBD2 (application/ace-groupcomm+cbor)

   {
     "group_name" : "gp4",
     "joining_uri" : "coap://[2001:db8::ab]/ace-group/gp4/",
     "as_uri" : "coap://as.example.com/token"
   }
~~~~~~~~~~~

Example in CoRAL:

~~~~~~~~~~~
=> 0.06 PATCH
   Uri-Path: manage
   Uri-Path: gp4
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   sign_enc_alg 10
   app_group_del "room1"
   app_group_add "room3"
   app_group_add "room4"

<= 2.04 Changed
   Content-Format: TBD1 (application/coral+cbor)

   #using <http://coreapps.org/core.osc.gconf#>
   group_name "gp4"
   joining_uri <coap://[2001:db8::ab]/ace-group/gp4/>
   as_uri <coap://as.example.com/token>
~~~~~~~~~~~

### Effects on Joining Nodes ###

After having selectively updated part of a group configuration, the effects on candidate joining nodes are the same as defined in {{sssec-effects-overwrite-joining-nodes}} for the case of group configuration overwriting.

### Effects on the Group Members ###

After having selectively updated part of a group configuration, the effects on the current group members are the same as defined in {{sssec-effects-overwrite-group-members}} for the case of group configuration overwriting.

## Delete a Group Configuration ## {#configuration-resource-delete}

This operation MUST be supported by the Group Manager and an Administrator.

The Administrator can send a DELETE request to the group-configuration resource, in order to delete that OSCORE group.

Consistently with what is defined at step 4 of {{getting-access}}, the Group Manager MUST check whether GROUPNAME matches with the group name pattern specified in any scope entry of the 'scope' claim in the stored Access Token for the Administrator. In case of a positive match, the Group Manager MUST check whether the permission set in the found scope entry specifies the permission "Delete".

If the verification above fails (i.e., there are no matching scope entries specifying the "Delete" permission), the Group Manager MUST reply with a 4.03 (Forbidden) error response. The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}.

Otherwise, the Group Manager continues processing the request, which would be successful only on an inactive OSCORE group. That is, the DELETE request actually yields a successful deletion of the OSCORE group, only if the corresponding status parameter 'active' has current value "false" (0xf4). The Administrator can ensure that, by first performing an update of the group-configuration resource associated with the OSCORE group (see {{configuration-resource-put}}), and setting the corresponding status parameter 'active' to "false" (0xf4).

If, upon receiving the DELETE request, the current value of the status parameter 'active' is "true" (0xf5), the Group Manager MUST respond with a 4.09 (Conflict) response. The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}. The value of the 'error' field MUST be set to 10 ("Group currently active").

After a successful processing of the DELETE request, the Group Manager performs the following actions.

First, the Group Manager deletes the OSCORE group and deallocates both the group-configuration resource as well as the group-membership resource associated with that group.

Then, the Group Manager replies to the Administrator with a 2.02 (Deleted) response.

Example:

~~~~~~~~~~~
=> 0.04 DELETE
   Uri-Path: manage
   Uri-Path: gp4

<= 2.02 Deleted
~~~~~~~~~~~

### Effects on the Group Members ###

After having deleted an OSCORE group, the Group Manager can inform the group members by means of the following two methods. When contacting a group member, the Group Manager uses the pairwise secure communication association established with that member during its joining process (see {{Section 6 of I-D.ietf-ace-key-groupcomm-oscore}}).

* The Group Manager sends an individual request message to each group member, targeting the respective resource used to perform the group rekeying process (see {{Section 11.1 of I-D.ietf-ace-key-groupcomm-oscore}}). The Group Manager uses the same format of the Join Response message in {{Section 6.3 of I-D.ietf-ace-key-groupcomm-oscore}}, where only the parameters 'gkty', 'key' and 'ace-groupcomm-profile' are present, and the 'key' parameter is the empty CBOR map.

* A group member may subscribe for updates to the group-membership resource associated with the OSCORE group. In particular, if this relies on CoAP Observe {{RFC7641}}, a group member would receive a 4.04 (Not Found) notification response from the Group Manager, since the group-configuration resource has been deallocated upon deleting the OSCORE group (see {{Section 6.1 of I-D.ietf-ace-key-groupcomm}}). The response MUST have Content-Format set to application/ace-groupcomm+cbor and is formatted as defined in {{Section 4.1.2 of I-D.ietf-ace-key-groupcomm}}. The value of the 'error' field MUST be set to 5 ("Group deleted").

When being informed about the deletion of the OSCORE group, a group member deletes the OSCORE Security Context that it stores as associated with that group, and possibly deallocates any dedicated control resource intended for the Group Manager that it has for that group.

# ACE Groupcomm Parameters {#groupcomm-parameters}

In addition to what is defined in {{Section 8 of I-D.ietf-ace-key-groupcomm}}, this document defines additional parameters used in the messages exchanged between the Administrator and the Group Manager (see {{interactions}}). The table below summarizes them and specifies the CBOR key to use instead of the full descriptive name.

Note that the media type application/ace-groupcomm+cbor MUST be used when these parameters are transported in the respective message fields.

~~~~~~~~~~~
+-----------------+----------+--------------+------------+
| Name            | CBOR Key | CBOR Type    | Reference  |
+-----------------+----------+--------------+------------+
| hkdf            | TBD      | tstr / int   | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| cred_fmt        | TBD      | int          | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| group_mode      | TBD      | simple value | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| sign_enc_alg    | TBD      | tstr / int / | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| sign_alg        | TBD      | tstr / int / | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| sign_params     | TBD      | array /      | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| pairwise_mode   | TBD      | simple value | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| alg             | TBD      | tstr / int / | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| ecdh_alg        | TBD      | tstr / int / | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| ecdh_params     | TBD      | array /      | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| det_req         | TBD      | simple value | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| det_hash_alg    | TBD      | tstr / int   | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| rt              | TBD      | tstr         | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| active          | TBD      | simple value | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| group_name      | TBD      | tstr         | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| group_title     | TBD      | tstr /       | [RFC-XXXX] |
|                 |          | simple value |            |
+-----------------+----------+--------------+------------+
| max_stale_sets  | TBD      | uint         | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| gid_reuse       | TBD      | simple value | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| app_groups      | TBD      | array        | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| joining_uri     | TBD      | tstr         | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| as_uri          | TBD      | tstr         | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| conf_filter     | TBD      | array        | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
| app_groups_diff | TBD      | array        | [RFC-XXXX] |
+-----------------+----------+--------------+------------+
~~~~~~~~~~~
{: #fig-ACE-Groupcomm-Parameters title="ACE Groupcomm Parameters" artwork-align="center"}

The following holds for the Group Manager.

* It MUST support and understand the parameters 'error', 'error_description', 'ace-groupcomm-profile', 'exp' and 'group_policies', which are defined in {{Section 8 of I-D.ietf-ace-key-groupcomm}}.

   This is consistent with what is defined in {{Section 8 of I-D.ietf-ace-key-groupcomm}} for the Key Distribution Center, of which the Group Manager defined in {{I-D.ietf-ace-key-groupcomm-oscore}} is a specific instance.

* It MUST support and understand all the parameters listed in {{fig-ACE-Groupcomm-Parameters}}, with the exception of the 'app_groups_diff' parameter, which MUST be supported and understood only if the Group Manager supports the selective update of a group configuration (see {{configuration-resource-patch}}).

The following holds for an Administrator.

* It MUST support and understand the parameters 'error', 'error_description', 'ace-groupcomm-profile', 'exp' and 'group_policies', which are defined in {{Section 8 of I-D.ietf-ace-key-groupcomm}}.

* It MUST support and understand all the parameters listed in {{fig-ACE-Groupcomm-Parameters}}, with the following exceptions.

   - 'conf_filter', which MUST be supported and understood only if the Administrator supports the partial retrieval of a group configuration by filters (see {{configuration-resource-fetch}}).

   - 'app_groups_diff' parameter, which MUST be supported and understood only if the Administrator supports the selective update of a group configuration (see {{configuration-resource-patch}}).

# ACE Groupcomm Error Identifiers {#error-types}

In addition to what is defined in {{Section 9 of I-D.ietf-ace-key-groupcomm}}, this document defines a new value that the Group Manager can include as error identifiers, in the 'error' field of an error response with Content-Format application/ace-groupcomm+cbor.

~~~~~~~~~~~
+-------+--------------------------+
| Value | Description              |
+-------+--------------------------+
| 10    | Group currently active   |
+-------+--------------------------+
| 11    | No available group names |
+-------+--------------------------+
~~~~~~~~~~~
{: #fig-ACE-Groupcomm-Error Identifiers title="ACE Groupcomm Error Identifiers" artwork-align="center"}

When receiving an error response from the Group Manager, an Administrator may use the information conveyed in the 'error' parameter to determine what actions to take next. If it is included in the error response, the 'error_description' parameter may provide additional context. In particular, the following guidelines apply.

* In case of error 10, the Administrator should stop sending the DELETE request to the Group Manager (see {{configuration-resource-delete}}), until the group becomes inactive. As per this document, this error is relevant only for the Administrator, if it tries to delete a group without having set its status to inactive first (see {{configuration-resource-delete}}). In such a case, the Administrator should take the expected course of actions, and set the group status to inactive first (see {{configuration-resource-put}} and {{configuration-resource-patch}}), before sending a new request of group deletion to the Group Manager.

* In case of error 11, the Administrator has the following options.

   - The Administrator simply tries again later on. The new POST request to the group-collection resource specifies the same group name originally suggested in the previous request that triggered the error response (see {{collection-resource-post}}). This option fundamentally relies on the Group Manager freeing up group names, hence it is not viable if considerably or indefinitely postponing the creation of the group is not acceptable.

   - The Administrator sends a new POST request to the group-collection resource right away, specifying a different group name than the one suggested in the previous request that triggered the error response. The new group name suggested by the Administrator should be such that the following holds.

      Let us define: i) S, as the set of all the scope entries in the Administrator's Access Token, such that the old group name matched with each of those scope entries; ii) S', as the set of all the scope entries in the Administrator's Access Token, such that the new group name matches with each of those scope entries. Then, S' is neither equal to S nor a subset of S.

   - The Administrator requests a new Access Token to the Authorization Server, in order to update its access rights, and have a new granted scope whose scope entries specify more and/or different group name patterns than the old Access Token.

      After uploading the new Access Token to the Group Manager, the Administrator can send a new POST request to the group-collection resource. When doing so, the Administrator suggests a new group name to the Group Manager, according to the same criteria discussed for the previous option.

# Security Considerations # {#sec-security-considerations}

Security considerations are inherited from the ACE framework for Authentication and Authorization {{RFC9200}}, and from the specific transport profile of ACE used between the Administrator and the Group Manager, such as {{RFC9202}} and {{RFC9203}}.

# IANA Considerations # {#iana}

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with
the RFC number of this specification and delete this paragraph.

## ACE Groupcomm Parameters ## {#iana-ace-groupcomm-parameters}

IANA is asked to register the following entries in the "ACE Groupcomm Parameters" registry defined in {{Section 11.6 of I-D.ietf-ace-key-groupcomm}}.

~~~~~~~~~~~
Name: hkdf
CBOR Key: TBD
CBOR Type: tstr / int
Reference: [RFC-XXXX]

Name: cred_fmt
CBOR Key: TBD
CBOR Type: int
Reference: [RFC-XXXX]

Name: group_mode
CBOR Key: TBD
CBOR Type: simple value
Reference: [RFC-XXXX]

Name: sign_enc_alg
CBOR Key: TBD
CBOR Type: tstr / int / simple value
Reference: [RFC-XXXX]

Name: sign_alg
CBOR Key: TBD
CBOR Type: tstr / int / simple value
Reference: [RFC-XXXX]

Name: sign_params
CBOR Key: TBD
CBOR Type: array / simple value
Reference: [RFC-XXXX]

Name: pairwise_mode
CBOR Key: TBD
CBOR Type: simple value
Reference: [RFC-XXXX]

Name: alg
CBOR Key: TBD
CBOR Type: tstr / int / simple value
Reference: [RFC-XXXX]

Name: ecdh_alg
CBOR Key: TBD
CBOR Type: tstr / int / simple value
Reference: [RFC-XXXX]

Name: ecdh_params
CBOR Key: TBD
CBOR Type: array / simple value
Reference: [RFC-XXXX]

Name: det_req
CBOR Key: TBD
CBOR Type: simple value
Reference: [RFC-XXXX]

Name: det_hash_alg
CBOR Key: TBD
CBOR Type: tstr / int
Reference: [RFC-XXXX]

Name: rt
CBOR Key: TBD
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: active
CBOR Key: TBD
CBOR Type: simple value
Reference: [RFC-XXXX]

Name: group_name
CBOR Key: TBD
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: group_title
CBOR Key: TBD
CBOR Type: tstr / simple value
Reference: [RFC-XXXX]

Name: max_stale_sets
CBOR Key: TBD
CBOR Type: uint
Reference: [RFC-XXXX]

Name: gid_reuse
CBOR Key: TBD
CBOR Type: simple value
Reference: [RFC-XXXX]

Name: app_groups
CBOR Key: TBD
CBOR Type: array
Reference: [RFC-XXXX]

Name: joining_uri
CBOR Key: TBD
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: as_uri
CBOR Key: TBD
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: conf_filter
CBOR Key: TBD
CBOR Type: array
Reference: [RFC-XXXX]

Name: app_groups_diff
CBOR Key: TBD
CBOR Type: array
Reference: [RFC-XXXX]
~~~~~~~~~~~

## ACE Groupcomm Errors {#iana-ace-groupcomm-errors}

IANA is asked to register the following entry in the "ACE Groupcomm Errors" registry defined in {{Section 11.11 of I-D.ietf-ace-key-groupcomm}}.

~~~~~~~~~~~
Value: 10
Description: Group currently active.
Reference: [RFC-XXXX]

Value: 11
Description: No available group names.
Reference: [RFC-XXXX]
~~~~~~~~~~~

## Resource Types # {#iana-rt}

IANA is asked to enter the following values in the "Resource Type (rt=) Link Target Attribute Values" registry within the "Constrained Restful Environments (CoRE) Parameters" registry group.

~~~~~~~~~~~
Value: core.osc.gcoll
Description: Group-collection resource of an OSCORE Group Manager
Reference: [RFC-XXXX]

Value: core.osc.gconf
Description: Group-configuration resource of an OSCORE Group Manager
Reference: [RFC-XXXX]
~~~~~~~~~~~

## Group OSCORE Admin Permissions {#ssec-iana-group-oscore-admin-permissions-registry}

This document establishes the IANA "Group OSCORE Admin Permissions" registry. The registry has been created to use the "Expert Review" registration procedure {{RFC8126}}. Expert review guidelines are provided in {{ssec-iana-expert-review}}.

This registry includes the possible permissions that Administrators can have to perform operations on an OSCORE Group Manager, each in combination with a numeric identifier. These numeric identifiers are used to express authorization information about performing administrative operations concerning OSCORE groups under the control of the Group Manager, as specified in {{scope-format}} of {{&SELF}}.

The columns of this registry are:

* Name: A value that can be used in documents for easier comprehension, to identify a possible permission that Administrators can perform when interacting with an OSCORE Group Manager.

* Value: The numeric identifier for this permission. Integer values greater than 65535 are marked as "Private Use", all other values use the registration policy "Expert Review" {{RFC8126}}.

   Note that, in general, a single permission can be associated with multiple different operations that are possible to be performed when interacting with the Group Manager.

* Description: This field contains a brief description of the permission.

* Reference: This contains a pointer to the public specification for the permission.

This registry will be initially populated by the values in {{fig-permission-values}}.

The Reference column for all of these entries will be {{&SELF}}.

## Expert Review Instructions {#ssec-iana-expert-review}

The IANA registry established in this document is defined as "Expert Review".  This section gives some general guidelines for what the experts should be looking for, but they are being designated as experts for a reason so they should be given substantial latitude.

Expert reviewers should take into consideration the following points:

* Clarity and correctness of registrations. Experts are expected to check the clarity of purpose and use of the requested entries. Experts should inspect the entry for the considered permission, to verify the correctness of its description against the permission as intended in the specification that defined it. Expert should consider requesting an opinion on the correctness of registered parameters from the Authentication and Authorization for Constrained Environments (ACE) Working Group and the Constrained RESTful Environments (CoRE) Working Group.

     Entries that do not meet these objective of clarity and completeness should not be registered.

* Duplicated registration and point squatting should be discouraged. Reviewers are encouraged to get sufficient information for registration requests to ensure that the usage is not going to duplicate one that is already registered and that the point is likely to be used in deployments.

* Experts should take into account the expected usage of permissions when approving point assignment. Given a 'Value' V as code point, the length of the encoding of (2^(V+1) - 1) should be weighed against the usage of the entry, considering the resources and capabilities of devices it will be used on. Additionally, given a 'Value' V as code point, the length of the encoding of (2^(V+1) - 1) should be weighed against how many code points resulting in that encoding length are left, and the resources and capabilities of devices it will be used on.

* Specifications are recommended. When specifications are not provided, the description provided needs to have sufficient information to verify the points above.

--- back

# Processing of Group Name Patterns at the AS # {#sec-as-scope-processing}

When processing an Authorization Request from an Administrator (see {{getting-access}}), the AS builds the authorization information expressing granted permissions as scope entries, according to the AIF specific data model AIF-OSCORE-GROUPCOMM and to its extension specified in {{scope-format}}. These scope entries are in turn specified as value of the 'scope' claim to include in the Access Token.

In order to evaluate the requested permissions against the access policies pertaining to the Administrator for the Group Manager in question, the AS can perform the following steps.

The following specifically refers only to "admin scope entries", i.e., scope entries that express authorization information for Administrators of OSCORE groups.

1. The AS initializes three empty sets of scope entries, namely S1, S2 and S3.

2. For each scope entry E in the 'scope' parameter of the Authorization Request, the AS performs the following actions.

   * In its access policies related to administrative operations at the Group Manager for the Administrator, the AS determines every group name superpattern P\*, such that every group name matching with the pattern P of the scope entry E matches also with P\*.

   * If no superpatterns are found, the AS proceeds with the next scope entry, if any. Otherwise, the AS computes Tperm\* as the union of the permission sets associated with the superpatterns found at the previous step. That is, Tperm\* is the inclusive OR of the binary representations of the Tperm values associated with the found superpatterns and encoding the corresponding permission sets as per {{scope-format}}.

   * The AS adds to the set S1 a scope entry, such that its Toid is the same as in the scope entry E, while its Tperm is the AND of Tperm\* with the Tperm in the scope entry E.

3. For each scope entry E in the 'scope' parameter of the Authorization Request, the AS performs the following actions.

   * In its access policies related to administrative operations at the Group Manager for the Administrator, the AS determines every group name subpattern P\*, such that: i) the pattern P of the scope entry E is different from P\*; and ii) every group name matching with P\* also matches with P.

   * If no subpatterns are found, the AS proceeds with the next scope entry, if any. Otherwise, for each found subpattern P\*, the AS adds to the set S2 a scope entry, such that its Toid is the same as in the subpattern P\*, while its Tperm is the AND of the Tperm from the subpattern P\* with the Tperm in the scope entry E.

4. For each scope entry E in the 'scope' parameter of the Authorization Request, the AS performs the following actions.

   * For each group name pattern P\* in its access policies related to administrative operations at the Group Manager for the Administrator, the AS performs the following actions.

      - The AS attempts to determine a crosspattern P\*\* such that: i) in the previous steps, P\*\* was not identified as a superpattern or subpattern for the pattern P of the scope entry E; ii) every group name matching with P\*\* also matches with both P and P\*.

      - If no crosspattern is built, the AS proceeds with the next pattern in its access policies related to administrative operations at the Group Manager for the Administrator, if any. Otherwise, the AS adds to the set S3 a scope entry, such that its Toid is the same as in the crosspattern P\*\*, while its Tperm is the AND of the Tperm from the pattern P\* and the Tperm in the scope entry E.

5. If the sets S1, S2 and S3 are all empty, the Authorization Request has not been successfully verified, and the AS returns an error response as per {{Section 5.8.3 of RFC9200}}. Otherwise, the AS uses the scope entries in the sets S1, S2 and S3 as the scope entries for the 'scope' claim to include in the Access Token, as per the format defined in {{scope-format}}.

# Document Updates # {#sec-document-updates}

RFC EDITOR: PLEASE REMOVE THIS SECTION.

## Version -06 to -07 ## {#sec-06-07}

* Alignment with renaming in draft-ietf-ace-key-groupcomm.

* Updated signaling of semantics for binary encoded scopes.

* Split between parameter registration and their CBOR abbreviations.

* Classified parameters as must/should/may be supported.

* New error code "No available group names" and related guidelines.

* Fixes in the examples.

* Editorial improvements.

## Version -05 to -06 ## {#sec-05-06}

* Use and extend the same AIF specific data model AIF-OSCORE-GROUPCOMM defined in {{I-D.ietf-ace-key-groupcomm-oscore}}.

* Revised Client-AS interaction, based on the used AIF specific data model.

* Categorized operations at the Group Manager as required and optional to support.

* Added status parameter 'gid_reuse', on reassigning OSCORE Group IDs upon group rekeying.

* Clarifications on the group name ultimately chosen by the Group Manager.

* Moved the detailed processing of group name patterns at the AS to an Appendix, as an example.

* Editorial improvements.

## Version -04 to -05 ## {#sec-04-05}

* Defined format of scope based on a new AIF data model.

* Specified authorization checks at the Group Manager.

* Revised resource handlers based on the new scope format.

* Renamed 'pub_key_enc' to 'cred_fmt'.

* Mandatory to include 'group_name' in the group creation request.

* Suggesting a used 'group_name' results in a new name, not in an error.

* Distinction between authentication credentials and public keys.

* More details on informing group members about changes in the group configuration.

* Revised order of sections; editorial improvements.

## Version -03 to -04 ## {#sec-03-04}

* Clarifications on what to do in case of enhanced error responses.

* Clarifications on handling default values for group parameters.

* New configuration parameters to support OSCORE deterministic requests.

* IANA considerations - Use RFC8126 terminology.

* Author's change of address.

* Editorial improvements.

## Version -02 to -03 ## {#sec-02-03}

* Aligned new and old parameters to core-groupcomm-oscore and ace-key-groupcomm-oscore.

* Removed 'cs_key_params' and 'ecdh_key_params' to avoid redundant COSE capabilities of key types, consistently with draft-ietf-ace-key-groupcomm-oscore.

* Revised examples and side effects due to parameter changes.

* New error type "Group currently active".

## Version -01 to -02 ## {#sec-01-02}

* Admit multiple Administrators and limited access to admin resources.

* Early design considerations for defining the format of scope.

* Additional error handling, using also error types.

* Selective update of group-configuration resources with PATCH/iPATCH.

* Editorial improvements.

## Version -00 to -01 ## {#sec-00-01}

* Names of application groups as status parameter.

* Parameters related to the pairwise mode of Group OSCORE.

* Defined FETCH for group-configuration resources.

* Policies on registration of links to the Resource Directory.

* Added resource type for group-configuration resources.

* Fixes, clarifications and editorial improvements.

# Acknowledgments # {#acknowledgment}
{: numbered="no"}

Klaus Hartke provided substantial contribution in defining the resource model based on group collection and group configurations, as well as the interactions with the Group Manager using CoRAL.

The authors sincerely thank {{{Christian Amsüss}}}, {{{Carsten Bormann}}} and {{{Jim Schaad}}} for their comments and feedback.

The work on this document has been partly supported by VINNOVA and the Celtic-Next project CRITISEC; and by the H2020 project SIFIS-Home (Grant agreement 952652).

---
v: 3

title: "A publish-subscribe architecture for the Constrained Application Protocol (CoAP)"
abbrev: CoAP pubsub
docname: draft-ietf-core-coap-pubsub-latest

category: std
stream: IETF
area: Applications
workgroup: CoRE Working Group

venue:
  mail: core@ietf.org
  github: core-wg/coap-pubsub

author:
- name: Jaime Jimenez
  org: Ericsson
  email: jaime@iki.fi
- name: Michael Koster
  org: Dogtiger Labs
  email: michaeljohnkoster@gmail.com
- name: Ari Keranen
  org: Ericsson
  email: ari.keranen@ericsson.com

contributor:
- name: Marco Tiloca
  organization: RISE AB
  email: marco.tiloca@ri.se
  contribution: Marco offered comprehensive reviews and insightful guidance on the recent iterations of this document. His contributions were particularly notable in the Security Considerations section, among others.

normative:
  RFC6570:
  RFC6690:
  RFC7252:
  RFC8288:
  RFC8516:
  STD94:
# RFC8949:
  RFC9176:
  RFC7641:
  BCP26:
# RFC8126:
  BCP13:
# RFC6838:
  BCP100:
# RFC7120:

informative:
  RFC8613:
  STD96:
# RFC9052:
# RFC9338:
  RFC9147:
  RFC9053:
  RFC9200:
  RFC9594:
  I-D.hartke-t2trg-coral-pubsub:
  I-D.ietf-ace-oscore-gm-admin:
  I-D.ietf-ace-pubsub-profile:
  I-D.ietf-core-interfaces:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document describes a publish-subscribe architecture for the Constrained Application Protocol (CoAP), extending the capabilities of CoAP communications for supporting endpoints with long breaks in connectivity and/or up-time. CoAP clients publish on and subscribe to a topic via a corresponding topic resource at a CoAP server acting as broker.

--- middle

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) {{RFC7252}} supports
machine-to-machine communication across networks of constrained
devices and constrained networks. CoAP uses a request/response model where clients make requests to servers in order to request actions on resources. Depending on the situation the same device may act either as a server, a client, or both.

One important class of constrained devices includes devices that are intended to run for years from a small battery, or by scavenging energy from their environment. These devices have limited up-time because they spend most of their time in a sleeping state with no network connectivity. Another important class of nodes are devices with limited reachability due to middle-boxes like Network Address Translators (NATs) and firewalls.

For these nodes, the client/server-oriented architecture of REST can be challenging when interactions are not initiated by the devices themselves. A publish/subscribe-oriented architecture where nodes exchange data via topics through a broker entity might fit these nodes better.

This document applies the idea of broker-based publish-subscribe to Constrained RESTful Environments using CoAP. It defines a broker that allows to create, discover subscribe and publish on topics.

## Terminology {#terminology}

{::boilerplate bcp14-tagged-bcp14}

This specification requires readers to be familiar with all the terms and concepts that are discussed in {{RFC8288}} and {{RFC6690}}. Readers should also be familiar with the terms and concepts discussed in {{RFC7252}}, {{RFC9176}} and {{RFC7641}}. The URI template format {{RFC6570}} is used to describe the REST API defined in this specification.

This specification makes use of the following terminology:

{:vspace}
publish-subscribe (pub/sub):
: A message communication model where messages associated with specific topics are sent to a broker. Interested parties, i.e. subscribers, receive these topic-based messages from the broker without the original sender knowing the recipients. The broker handles matching and delivering these messages to the appropriate subscribers.

publishers and subscribers:
: CoAP clients can act as publishers or as subscribers. Publishers send CoAP messages (publications) to the broker on specific topics. Subscribers have an ongoing observation relation (subscription) to a topic. Both roles operate without any mutual knowledge, guided by their respective topic interests.

topic collection:
: A set of topic-configurations. A topic collection is hosted as one collection resource (See Section 3.1 on {{I-D.ietf-core-interfaces}}) at the broker, and its representation is the list of links to the topic resources corresponding to each topic-configuration.

topic-configuration:
: A set of information concerning a topic, including its configuration and other metadata. A topic-configuration is hosted as one topic resource at the broker, and its representation is the set of configuration information concerning the topic. All the topic resources associated with the same topic collection share a common base URI, i.e., the URI of the collection resource. Throughout this document the words "topic resource" and "topic-configuration resource" can be used interchangeably.

topic-data resource:
: A resource where clients can publish data and/or subscribe to data for a specific topic. The representation of the topic resource corresponding to such a topic also specifies the URI to the present topic-data resource.

broker:
: A CoAP server that hosts one or more topic collections with their topic-configurations, and also topic-data resources. The broker is responsible for the store-and-forward of state update representations, for the topics for which it hosts the corresponding topic-data resources. The broker is also responsible for handling the topic lifecycle as defined in {{topic-lifecycle}}. The creation, configuration, and discovery of topics at a broker is specified in {{topics}}.

## CoAP Publish-Subscribe Architecture

{{fig-arch}} shows a simple Publish/Subscribe architecture over CoAP.

Topics are created by the broker, but the initial configuration can be proposed by a client (e.g., a publisher or a dedicated administrator) over the RESTful interface of a corresponding topic resource hosted by the broker.

Publishers submit their data over the RESTful interface of a topic-data resource corresponding to the topic, which may be hosted by the broker. Subscribers to a topic are notified of new publications by using Observe {{RFC7641}} on the corresponding topic-data resource.

The broker is responsible for the store-and-forward of state update representations between CoAP clients. Subscribers observing a resource will receive notifications, the delivery of which is done on a best-effort basis.

~~~~ aasvg
     CoAP                      CoAP                 CoAP
     clients                  server                clients
   .-----------.          .----------.  observe  .------------.
   |           | publish  |          |<----------+            |
   | publisher +--------->+          +---------->| subscriber |
   |           |          |          +---------->|            |
   '-----------'          |          |           '------------'
        ...               |  broker  |                ...
        ...               |          |                ...
   .-----------.          |          |  observe  .------------.
   |           | publish  |          |<----------+            |
   | publisher +--------->|          +---------->| subscriber |
   |           |          |          +---------->|            |
   '-----------'          '----------'           '------------'
~~~~
{: #fig-arch title='Publish-subscribe architecture over CoAP' artwork-align="center"}

This document describes two sets of interactions; interactions to configure topics and their lifecycle (see {{topic-configuration-interactions}}) and interactions about the topic-data (see {{topic-data-interactions}}).

Topic-configuration interactions are discovery, create, read configuration, update configuration, and delete configuration. These operations handle the management of the topics.

Topic-data interactions are publish, subscribe, unsubscribe, read, and delete. These operations are oriented on how data is transferred from a publisher to a subscriber.

<!--
Throughout the document there is a number of TBDs that need updating, mostly content formats or cbor data representations
-->

## Managing Topics {#managing-topics}

{{fig-api}} shows the resources related to a Topic Collection that can be managed at the Broker.

~~~~ aasvg
             ___
   topic    /   \
 collection \___/
  resource       \
                  \____________________
                   \___    \___        \___
                   /   \   /   \  ...  /   \   topic resources
                   \___/   \___/       \___/
~~~~
{: #fig-api title="Resources of a Broker" artwork-align="center"}

The Broker exports one or more topic-collection resources, with resource type "core.ps.coll" defined in {{iana}} of this document. The interfaces for the topic-collection resource is defined in {{topic-collection-interactions}}.

A topic-collection resource can have topic resources as its child resources, with resource type "core.ps.conf".

# Pub-Sub Topics {#topics}

The configuration side of a "publish/subscribe broker" consists of a collection of topics. These topics as well as the collection itself are exposed by a CoAP server as resources (see {{fig-topic}}). Each topic is associated with a topic-configuration resource and a topic-data resource. The topic-configuration resource is used by a client creating or administering a topic. The topic-data resource is used by the publishers and the subscribers to a topic.

~~~~ aasvg
              ___
    topic    /   \
  collection \___/
   resource       \
                   \___________________________
                    \          \               \
                     \ ......   \ ......        \ ......
             topic  : \___  :  : \___  :       : \___  :
     configuration  : / * \ :  : / * \ :       : / * \ :
          resource  : \_|_/ :  : \_|_/ :       : \_|_/ :
                    ....|....  ....|....       ....|....
                    ....|....  ....|....       ....|....
                    :  _|_  :  :  _|_  :  ...  :  _|_  :
        topic-data  : /   \ :  : /   \ :       : /   \ :
          resource  : \___/ :  : \___/ :       : \___/ :
                    :.......:  :.......:       :.......:
                   \_________/\_________/ ... \_________/
                     topic 1    topic 2         topic n
~~~~
{: #fig-topic title='Topic and topic-data resources of a topic' artwork-align="center"}

## Collection Representation

Each topic-configuration is represented as a link, where the link target is the URI of the corresponding topic resource.

Publication and subscription to a topic occur at a link, where the link target is the URI of the corresponding topic-data resource. Such a link is specified by the topic-data entry within the topic resource (see {{topic-properties}}).

A topic resource with a topic-data link can also be simply called "topic".

The list of links to the topic resources can be retrieved from the associated topic collection resource, and represented as a Link Format document {{RFC6690}}where each such link specifies the link target attribute 'rt' (Resource Type), with value "core.ps.conf" defined in this document.

## Topic-Configuration Representation {#topic-resource-representation}

A CoAP client can create a new topic by submitting an initial configuration for the topic (see {{topic-create}}). It can also read and update the configuration of existing topics and topic properties as well as delete them when they are no longer needed (see {{topic-configuration-interactions}}).

The configuration of a topic itself consists of a set of properties that can be set by a client or by the broker. The topic-configuration is represented as a CBOR map containing the configuration properties of the topic as top-level elements.

Unless specified otherwise, these are defined in this document and their CBOR abbreviations are defined in {{pubsub-parameters}}.

### Topic Properties {#topic-properties}

The CBOR map includes the following configuration parameters, whose CBOR abbreviations are defined in {{pubsub-parameters}} of this document.

* 'topic-name': A required field used as an application identifier. It encodes the topic name as a CBOR text string. Examples of topic names include human-readable strings (e.g., "room2"), UUIDs, or other values.

* 'topic-data': A required field (optional during creation) containing the URI of the topic-data resource for publishing/subscribing to this topic. It encodes the URI as a CBOR text string.

* 'resource-type': A required field used to indicate the resource type of the topic-data resource for the topic. It encodes the resource type as a CBOR text string. The value should be "core.ps.data".

* 'topic-content-format': This optional field specifies the CoAP Content-Format identifier of the topic-data resource representation, e.g., 60 for the media-type "application/cbor".

* 'topic-type': An optional field used to indicate the attribute or property of the topic-data resource for the topic. It encodes the attribute as a CBOR text string. Example attributes include "temperature".

* 'expiration-date': An optional field used to indicate the expiration date of the topic. It encodes the expiration date as a CBOR text string. The value should be a date string as defined in 3.4.1. {{RFC8949}} (e.g., the CBOR encoded version of "2023-03-31T23:59:59Z"). If this field is not present, the topic will not expire automatically.

* 'max-subscribers': An optional field used to indicate the maximum number of simultaneous subscribers allowed for the topic. It encodes the maximum number as an unsigned CBOR integer. If this field is not present or if the field is empty, then there is no limit to the number of simultaneous subscribers allowed. The broker can use this field to limit the number of subscribers for the topic.

* 'observer-check': An optional field that controls the maximum number of seconds between two consecutive Observe notifications sent as Confirmable messages to each topic subscriber (see Section {{unsubscribe}}). Encoded as a CBOR unsigned integer greater than 0, it ensures subscribers who have lost interest and silently forgotten the observation do not remain indefinitely on the server's observer list. If another CoAP server hosts the topic-data resource, that server is responsible for applying the observer-check value. The default value for this field is 86400, as defined in {{RFC7641}}, which corresponds to 24 hours.

* 'topic-history': An optional field used to indicate how many previous resource representations the broker shall store for a topic. Encoded as an unsigned CBOR integer, it defines a counter representing the number of historical resource states the broker retains. This enables subscribers to retrieve past states of the topic data when necessary, useful in scenarios where historical context is required (e.g., for data analytics or auditing). If this field is not present, no historical data will be stored.

* 'initialize': An optional boolean field that, when set to `true`, allows the topic-data path to be pre-populated with an empty string or other initial value during the topic creation process. This behavior facilitates one-shot publication and topic creation, enabling CoAP clients to subscribe by default without encountering a `4.04 Not Found` error. If this field is not present, the broker behaves as usual, and the topic-data path is not initialized.

## Discovery

A client can perform a discovery of: the broker; the topic collection resources and topic resources hosted by the broker; and the topic-data resources associated with those topic resources.

### Broker Discovery {#broker-discovery}

CoAP clients MAY discover brokers by using CoAP Simple Discovery, via multicast, through a Resource Directory (RD) {{RFC9176}} or by other means specified in extensions to {{RFC7252}}. Brokers MAY register with a RD by following the steps on {{Section 5 of RFC9176}} with the resource type set to "core.ps" as defined in {{iana}} of this document.

The following example shows an endpoint discovering a broker using the "core.ps" resource type over a multicast network. Brokers within the multicast scope will answer the query.

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Host: "kdc.example.com"
   Uri-Path: ".well-known"
   Uri-Path: "core"
   Uri-Query: "rt=core.ps"

   Response:

   Header: Content (Code=2.05)
   Content-Format: 40 (application/link-format)
   Payload:
   <coap://mythinguri.com/broker/v1>;rt="core.ps"
~~~~

### Topic Collection Discovery

A Broker SHOULD offer a topic discovery entry point to enable clients to find topics of interest. The resource entry point is the topic collection resource collecting the topic-configurations for those topics (see {{Section 1.2.2 of RFC6690}}) and is identified by the resource type "core.ps.coll".

The specific resource path is left for implementations, examples in this document use the "/ps" path. The interactions with a topic collection are further defined in {{topic-collection-interactions}}.

Since the representation of the topic collection resource includes the links to the associated topic resources, it is not required to locate those links under "/.well-known/core", also in order to limit the size of the Link Format document returned as result of the discovery.

Example:

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: ".well-known"
   Uri-Path: "core"
   Uri-Query: "rt=core.ps.coll"

   Response:

   Header: Content (Code=2.05)
   Content-Format: 40 (application/link-format)
   Payload:
   </ps>;rt="core.ps.coll";ct=40,
   </other/path>;rt="core.ps.coll";ct=40
~~~~

### Topic-Configuration Discovery {#topic-discovery}

Each topic collection is associated with a group of topic resources, each detailing the configuration of its respective topic (refer to {{topic-properties}}). Each topic resource is identified by the resource type "core.ps.conf".

Below is an example of discovery via /.well-known/core with rt=core.ps.conf that returns a list of topics, as the list of links to the corresponding topic resources.

<!--
TODO: add the ct part in IANA and add the example here:
- If you want to indicate ct= in one of this links, then it should be ct=X, where is the the Content-Format identifier for application/core-pubsub+cbor
-->

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: ".well-known"
   Uri-Path: "core"
   Uri-Query: "rt=core.ps.conf"

   Response:

   Header: Content (Code=2.05)
   Content-Format: 40 (application/link-format)
   Payload:
   </ps/h9392>;rt="core.ps.conf";ct=TBD,
   </other/path/2e3570>;rt="core.ps.conf";ct=TBD
~~~~

### Topic-Data Discovery

<!--
TODO DISCUSS Decide on this section

   Also, as based on {{ection 1.2.2 of RFC6690}}, I'd realistically expect to have located by /.well-known/core certainly the topic collection resources and MAYBE the topic resources (and likely limited only to "perpetual", hence well-known topics).

   Instead, I'd expect to discover the links to the topic resources mostly by GET/FETCH accessing the topic collection resource.

   Practically, you may have to literally *discover* the broker, its collection resource, and a particular topic resource. At that point, you just *learn* the URI of the topic-data resource, from the corresponding parameter within the exact, corresponding topic resource.
-->

Within a topic, there is the topic-data property containing the URI of the topic-data resource that a CoAP client can subscribe and publish to. Resources exposing resources of the topic-data type are expected to use the resource type 'core.ps.data'.

The topic-data contains the URI of the topic-data resource for publishing and subscribing. So retrieving the topic-configuration will also provide the URL of the topic-data (see {{topic-get-resource}}).

It is also possible to discover a list of topic-data resources by sending a request to the collection with rt=core.ps.data resources as shown below.

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: "ps"
   Uri-Query: "rt=core.ps.data"

   Response:

   Header: Content (Code=2.05)
   Content-Format: 40 (application/link-format)
   Payload:
   </ps/data/62e4f8d>;rt="core.ps.data";obs
~~~~

## Topic Collection Interactions {#topic-collection-interactions}

These are the interactions that can happen directly with a specific topic collection.

### Retrieving all topic-configurations {#topic-get-all}

A client can request a collection of the topics present in the broker by making a GET request to the collection URI.

On success, the broker returns a 2.05 (Content) response, specifying the list of links to topic resources associated with this topic collection (see {{topic-resource-representation}}).

A client MAY retrieve a list of links to topics it is authorized to access, based on its permissions.

Example:

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: "ps"

   Response:

   Header: Content (Code=2.05)
   Content-Format: 40 (application/link-format)
   Payload:
   </ps/h9392>;rt="core.ps.conf",
   </ps/2e3570>;rt="core.ps.conf"
~~~~

### Getting topic-configurations by Properties {#topic-get-properties}
<!--
FETCH to /topic-collection with filter
retrieve only the topics that match the filter
request is cbor
response is link format
-->

A client can filter a collection of topics by submitting the
representation of a topic filter (see {{topic-fetch-resource}}) in a FETCH request to the topic collection URI.

On success, the broker returns a 2.05 (Content) response with a
representation of a list of topics in the collection (see
 {{topic-discovery}}) that match the filter in CoRE link format {{RFC6690}}.

Upon success, the broker responds with a 2.05 (Content), providing a list of links to topic resources associated with this topic collection that match the request's filter criteria (refer to {{topic-discovery}}). A positive match happens only when each request parameter is present with the indicated value in the topic resource representation.

Example:

~~~~
   Request:

   Header: FETCH (Code=0.05)
   Uri-Path: "ps"
   Content-Format: TBD (application/core-pubsub+cbor)
   Payload:
   {
      "resource-type": "core.ps.data",
      "topic-type": "temperature"
   }

   Response:

   Header: Content (Code=2.05)
   Content-Format: 40 (application/link-format)
   Payload:
   </ps/2e3570>;rt="core.ps.conf"
~~~~

### Creating a Topic {#topic-create}

A client can add a new topic-configurations to a collection of topics by submitting an initial representation of the initial topic resource (see {{topic-resource-representation}}) in a POST request to the topic collection URI. The request MUST specify at least a subset of the properties in {{topic-properties}}, namely: topic-name and resource-type.

Please note that the topic will NOT be fully created until a publisher has published some data to it (See {{topic-lifecycle}}).

To facilitate immediate subscription and allow clients to observe the topic before data has been published, the client can include the "initialize" set to "true". When supported, the broker will create the topic and pre-populate the "topic-data" field with an empty value. This allows subscribers to observe the topic without encountering a 4.04 (Not Found) error, even if no data has been published yet.

When "initialize" is set to "false" or omitted, the topic will only be fully created after data is published to it.

On success, the broker returns a 2.01 (Created) response, indicating the Location-Path of the new topic and the current representation of the topic resource. The response payload includes a CBOR map with key-value pairs. The response MUST include the required topic properties (see {{topic-properties}}), namely: "topic-name", "resource-type" and "topic-data". It MAY also include a number of optional properties too.

If requirements are defined for the client to create the topic as requested and the broker does not successfully assess that those requirements are met, then the broker MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The broker MUST issue a 4.00 (Bad Request) error if a received parameter is invalid, unrecognized, or if the topic-name is already in use or otherwise invalid.


~~~~
   Request:

   Header: POST (Code=0.02)
   Uri-Path: "ps"
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-name /         0: "living-room-sensor",
      / resource-type /      2: "core.ps.data"
   }

   Response:

   Header: Created (Code=2.01)
   Location-Path: "ps/h9392"
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-name /         0: "living-room-sensor",
      / topic-data /         1: "ps/data/1bd0d6d",
      / resource-type /      2: "core.ps.data"

   }
~~~~



## Topic-Configuration Interactions {#topic-configuration-interactions}

These are the interactions that can happen at the topic resource level.

### Getting a topic-configuration {#topic-get-resource}

<!--
GET to /topic-config
retrieve a topic-configuration
response is cbor
-->

A client can read the configuration of a topic by making a GET request to the topic resource URI.

On success, the broker returns a 2.05 (Content) response with a representation of the topic resource, as specified in {{topic-resource-representation}}.

If requirements are defined for the client to read the topic as requested and the broker does not successfully assess that those requirements are met, then the broker MUST respond with a 4.03 (Forbidden) error.

The response payload is a CBOR map, whose possible entries are specified in {{topic-resource-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

For example, below is a request on the topic "ps/h9392":

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: "ps"
   Uri-Path: "h9392"

   Response:

   Header: Content (Code=2.05)
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-name /            0: "living-room-sensor",
      / topic-data /            1: "ps/data/1bd0d6d",
      / resource-type /         2: "core.ps.data",
      / topic-content-format /  3: 112,
      / topic-type /            4: "temperature",
      / expiration-date /       5: "2023-04-00T23:59:59Z",
      / max-subscribers /       6: 100,
      / topic-history /         7: 10
   }
~~~~

### Getting part of a topic-configuration {#topic-fetch-resource}
<!--
FETCH to /topic-conf with filter
retrieve only certain parameters from the configuration
request is cbor
response is cbor
-->

A client can read the configuration of a topic by making a FETCH request to the topic resource URI with a filter for specific parameters. This is done in order to retrieve part of the current topic resource.

The request contains a CBOR map with a configuration filter or 'conf-filter', a CBOR array of configuration parameters, as defined in Section {{pubsub-parameters}}. Each element of the array specifies one requested configuration parameter of the current topic resource (see {{topic-resource-representation}}).

On success, the broker returns a 2.05 (Content) response with a representation of the topic resource. The response has as payload the partial representation of the topic resource as specified in {{topic-resource-representation}}.

If requirements are defined for the client to read the topic as requested and the broker does not successfully assess that those requirements are met, then the broker MUST respond with a 4.03 (Forbidden) error.

The response payload is a CBOR map, whose possible entries are specified in {{topic-resource-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

Both request and response MUST have Content-Format set to "application/core-pubsub+cbor".

Example:

~~~~
   Request:

   Header: FETCH (Code=0.05)
   Uri-Path: "ps"
   Uri-Path: "h9392"
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / conf-filter / 11: ["topic-data", "media-type"]
   }

   Response:

   Header: Content (Code=2.05)
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-data /            1: "ps/data/1bd0d6d",
      / topic-content-format /  3: 112,
   }
~~~~

### Updating the topic-configuration {#topic-update-resource}

<!--
PUT to /topic-conf
override the whole configuration
request is cbor
response is cbor
-->

A client can update a topic's configuration by submitting the updated topic representation in a PUT request to the topic URI. However, the parameters "topic-name", "topic-data", and "resource-type" are immutable post-creation, and any request attempting to change them will be deemed invalid by the broker.

On success, the topic-configuration is overwritten and broker returns a 2.04 (Changed) response and the current full resource representation. The broker may choose not to overwrite parameters that are not explicitly modified in the request.

Note that updating the "topic-data" path will automatically cancel all existing observations on it and thus will unsubscribe all subscribers. Updating the "topic-data" may happen also after it being deleted, as described on {{delete-topic-data}}, this will in turn create a new "topic-data" path for that topic-configuration.

Similarly, decreasing max-subscribers will also cause that some subscribers get unsubscribed. Unsubscribed endpoints receive a final 4.04 (Not Found) response as per {{Section 3.2 of RFC7641}}. The specific queue management for unsubscribing is left for implementors.

Please note that when using PUT the topic-configuration is being overwritten, thus some of the optional parameters (e.g., "max-subscribers", "observer-check") not included in the PUT message will be reset to their default values.

Example:

~~~~
   Request:

   Header: PUT (Code=0.03)
   Uri-Path: "ps"
   Uri-Path: "h9392"
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-name /            0: "living-room-sensor",
      / topic-data /            1: "ps/data/1bd0d6d",
      / resource-type /         2: "core.ps.data",
      / topic-content-format /  3: 112,
      / topic-type /            4: "temperature",
      / expiration-date /       5: "2023-04-28T23:59:59Z"
   }

   Response:

   Header: Changed (Code=2.04)
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-name /            0: "living-room-sensor",
      / topic-data /            1: "ps/data/1bd0d6d",
      / resource-type /         2: "core.ps.data",
      / topic-content-format /  3: 112,
      / topic-type /            4: "temperature",
      / expiration-date /       5: "2023-04-28T23:59:59Z",
      / max-subscribers /       6: 100,
      / observer-check /        7: 86400
   }
~~~~

Note that when a topic-configuration changes, it may result in disruptions for the subscribers. Some potential issues that may arise include:

* Limiting the number of subscribers will cause cancellation of ongoing subscriptions until max-subscribers has been reached.
* Changing the topic-data value will cancel all ongoing subscriptions.
* Changing of the expiration-date may cause cancellation of ongoing subscriptions if the topic expires at an earlier data.

### Updating the topic-configuration with iPATCH {#topic-update-resource-patch}

A client can partially update a topic's configuration by submitting a partial topic representation in an iPATCH request to the topic URI. The iPATCH request allows for updating only specific fields of the topic-configuration while leaving the others unchanged. As with the PUT method, the parameters "topic-name", "topic-data", and "resource-type" are immutable post-creation, and any request attempting to change them will be deemed invalid by the broker.

On success, the broker returns a 2.04 (Changed) response and the current full resource representation. The broker only updates parameters that are explicitly mentioned in the request.

As with the PUT method, updating the "topic-data" path will automatically cancel all existing observations on it and thus will unsubscribe all subscribers. Decreasing max-subscribers will also cause some subscribers to get unsubscribed. Unsubscribed endpoints receive a final 4.04 (Not Found) response as per {{Section 3.2 of RFC7641}}.

Contrary to PUT, iPATCH operations will explicitly update some parameters, leaving others unmodified.

~~~~
   Request:

   Header: iPATCH (Code=0.07)
   Uri-Path: "ps"
   Uri-Path: "h9392"
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / expiration-date /  5: "2024-02-28T23:59:59Z",
      / max-subscribers /  6: 5
   }

   Response:

   Header: Changed (Code=2.04)
   Content-Format: TBD2 (application/core-pubsub+cbor)
   Payload (in CBOR diagnostic notation):
   {
      / topic-name /            0: "living-room-sensor",
      / topic-data /            1: "ps/data/1bd0d6d",
      / resource-type /         2: "core.ps.data",
      / topic-content-format /  3: 112,
      / topic-type /            4: "temperature",
      / expiration-date /       5: "2024-02-28T23:59:59Z",
      / max-subscribers /       6: 5,
      / observer-check /        7: 86400
   }
~~~~

Note that when a topic-configuration changes through an iPATCH request, it may result in disruptions for the subscribers. For example, limiting the number of subscribers will cause cancellation of ongoing subscriptions until max-subscribers has been reached.

### Deleting a topic-configuration {#topic-delete}

A client can delete a topic by making a CoAP DELETE request on the topic resource URI.

On success, the broker returns a 2.02 (Deleted) response.

When a topic-configuration resource is deleted, the broker MUST also delete the topic-data resource, unsubscribe all subscribers by removing them from the list of observers and returning a final 4.04 (Not Found) response as per {{Section 3.2 of RFC7641}}.

Example:

~~~~
   Request:

   Header: DELETE (Code=0.04)
   Uri-Path: "ps"
   Uri-Path: "h9392"

   Response:

   Header: Deleted (Code=2.02)
~~~~

# Publish and Subscribe {#pubsub}

The overview of the publish/subscribe mechanism over CoAP is as follows: a publisher publishes to a topic by submitting the data in a PUT request to a topic-data resource and subscribers subscribe to a topic by submitting a GET request with Observe option set to 0 (register) to a topic-data resource. When resource state changes, subscribers observing the resource {{RFC7641}} at that time will receive a notification.

A topic-data resource does not exist until some initial data has been published to it. Before initial data publication, a GET request to the topic-data resource URI results in a 4.04 (Not Found) response. If such a "half created" topic is undesired, the creator of the topic can simply immediately publish some initial placeholder data to make the topic "fully created" (see {{topic-lifecycle}}).

<!--
* "URIs for topic-data MAY be broker-generated or client-generated."

   See a comment above. I think that only the host of the topic-data resource should be in control of generating this URI (to be provided to the broker if that host is not the broker already).
-->

URIs for topic resources are broker-generated (see {{topic-create}}). There is no necessary URI pattern dependence between the URI where the topic-data exists and the URI of the topic-configuration resource.

## Topic Lifecycle {#topic-lifecycle}

When a topic is newly created, it is first placed by the broker into the HALF CREATED state (see {{fig-life}}). In this state, a client can read and update the configuration of the topic and delete the topic. A publisher can publish to the topic-data resource.  However, a subscriber cannot yet subscribe to the topic-data resource nor read the latest data.

<!--
TODO I got a comment that mqtt folks my want to pre-subscribe to topics, so they'd like to be able to place an observation even if the resource is not "fully created"

Also, we might want to restrict the discovery part ONLY for FULLY created topics. If so, let's mention it.
-->

~~~~ aasvg
                HALF                          FULLY
               CREATED       Delete          CREATED
                ____        topic-data        ____     Publish
------------>  |    |  <-------------------  |    |  ------------.
   Create      |    |                        |    |               |
               |____|  ------------------->  |____|  <-----------'
                      \      Publish      /            Subscribe
               |   ^   \       ___       /   |   ^
         Read/ |   |    '-->  |   |  <--'    |   | Read/
        Update |   |  Delete  |___|  Delete  |   | Update
  topic-config  '-'   Topic          Topic    '-'  topic-data
                             DELETED
~~~~
{: #fig-life title='Lifecycle of a Topic' artwork-align="center"}

After a publisher publishes to the topic-data for the first time, the topic is placed into the FULLY CREATED state. In this state, a client can read data by means of a GET request without observe. A publisher can publish to the topic-data resource and a subscriber can observe the topic-data resource.


When a client deletes a topic-configuration resource, the topic is placed into the DELETED state and shortly after removed from the server. In this state, all subscribers are removed from the list of observers of the topic-data resource and no further interactions with the topic are possible.

When a client deletes a topic-data, the topic is placed into the HALF CREATED state, where clients can read, update and delete the topic-configuration and await for a publisher to begin publication.

## Topic-Data Interactions {#topic-data-interactions}

<!--
TODO: Should we remove this
   See comments above. I'm not sure whether the client should have any say on where the topic-data resource is hosted.

   It'd already be difficult to have some sort of coordination between the broker and the separate server hosting the topic-data resource, let alone involving the client as yet another actor in the process.

JJ: Also note that the broker has no way to know anything about a topic-data hosted elsewhere.
-->

Interactions with the topic-data resource are covered in this section.

### Publish {#publish}

A topic-configuration with a topic-data resource must have been created in order to publish data to it (See {{topic-create}}) and be in the half-created or fully-created state in order to the publish operation to work (see {{topic-lifecycle}}).

A client can publish data to a topic by submitting the data in a PUT request to the topic-data URI as indicated in its topic resource property. Please note that the topic-data URI is not the same as the topic-configuration URI used for configuring the topic (see {{topic-resource-representation}}).

On success, the broker returns a 2.04 (Changed) response. However, when data is published to the topic for the first time, the broker instead MUST return a 2.01 (Created) response and set the topic in the fully-created state (see {{topic-lifecycle}}).

If the request does not have an acceptable content-format, the broker returns a 4.15 (Unsupported Content-Format) response.

If the client is sending publications too fast, the broker returns a
4.29 (Too Many Requests) response {{RFC8516}}.

Example of first publication:

~~~~
   Request:

   Header: PUT (Code=0.03)
   Uri-Path: "ps"
   Uri-Path: "data"
   Uri-Path: "1bd0d6d"
   Content-Format: 110
   Payload:
   {
      "n": "coap://dev1.example.com/temperature",
      "u": "Cel",
      "t": 1621452122,
      "v": 23.5
   }

   Response:

   Header: Created (Code=2.01)
~~~~

Example of subsequent publication:

~~~~
   Request:

   Header: PUT (Code=0.03)
   Uri-Path: "ps"
   Uri-Path: "data"
   Uri-Path: "1bd0d6d"
   Content-Format: 110
   Payload:
   {
      "n": "coap://dev1.example.com/temperature",
      "u": "Cel",
      "t": 1621452149,
      "v": 22.5
   }

   Response:

   Header: Changed (Code=2.04)
~~~~

### Subscribe {#subscribe}

A client can subscribe to a topic-data by sending a CoAP GET request with the CoAP Observe Option set to 0 to subscribe to resource updates {{RFC7641}}.

On success, the server hosting the topic-data resource MUST return 2.05 (Content) notifications with the data and the Observe Option. Otherwise, if no Observe Option is present the client should assume that the subscription was not successful.

If the topic is not yet in the fully created state (see {{topic-lifecycle}}) the broker MUST return a response code 4.04 (Not Found).

<!--
TODO: After a publisher publishes to the topic-data for the first time, the topic is placed into the FULLY CREATED state.

This is a problem if the topic-data is hosted elsewhere and not in the broker, how does the broker know when to put it in fully created state if the pub/sub mechanism is happening directly btw pub and sub?

Shall I add: The topic-data URI may link to resources that are not hosted directly by the broker as shown in {{fig-external-server}}.
Thus, subscribers would use the broker for discovery only.
-->

The following response codes are defined for the Subscribe operation:

Success:
: 2.05 "Content". Successful subscribe with observe response, current value included in the response.

Failure:
: 4.04 "Not Found". The topic-data does not exist.

If the 'max-subscribers' parameter has been reached, the broker must treat that as specified in {{Section 4.1 of RFC7641}}. The response MUST NOT include an Observe Option, the absence of which signals to the subscriber that the subscription failed.

<!--
TODO Right. However, how can this work when the server hosting the topic-data resource is not the broker? The broker knows the maximum number of subscribers, but that separate server does not. Is it just up to a not-specified-here synchronization protocol between the broker and that server?
-->

Example of a successful subscription followed by one update:

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: "ps"
   Uri-Path: "data"
   Uri-Path: "1bd0d6d"
   Observe: 0

   Response:

   Header: Content (Code=2.05)
   Content-Format: 110
   Observe: 10001
   Max-Age: 15
   Payload:
   {
      "n": "urn:dev:os:32473-123456",
      "u": "Cel",
      "t": 1696341182,
      "v": 19.87
   }

   Response:

   Header: Content (Code=2.05)
   Content-Format: 110
   Observe: 10002
   Max-Age: 15
   Payload:
   {
      "n": "urn:dev:os:32473-123456",
      "u": "Cel",
      "t": 1696340184,
      "v": 21.87
   }
~~~~

### Unsubscribe {#unsubscribe}

A CoAP client can unsubscribe simply by cancelling the observation as described in {{Section 3.6 of RFC7641}}. The client MUST either use CoAP GET with the Observe Option set to 1 or send a CoAP Reset message in response to a notification. Also on {{Section 3.6 of RFC7641}} the client can simply "forget" the observation and the broker will remove it from the list of observers after the next notification.

As per {{RFC7641}} a server that transmits notifications mostly in non-confirmable messages, but it MUST send a notification in a confirmable message instead of a non-confirmable message at least every 24 hours.

This value can be modified at the broker by the administrator of a topic by modifying the parameter "observer-check" on {{topic-resource-representation}}. This would allow changing the rate at which different implementations verify that a subscriber is still interested in observing a topic-data resource.

<!--
TODO: another item that points to make topic-data a broker thing only.

   Yes, and again, what if the topic-data resource is not hosted at the broker but at a different server? Is it just up to a not-specified-here synchronization protocol between the broker and that server?
-->

### Delete topic-data {#delete-topic-data}

A publisher MAY delete a topic by making a CoAP DELETE request on the topic-data URI.

On success, the broker returns a 2.02 (Deleted) response.


<!--
Q: Same question here, why is this a SHOULD (see comment above).
A: Changed to MUST but I think we could discuss it. Could the broker have reasons to keep the uri of the topic-data path for later reuse in some cases? for example the broker could also implement a different behaviour for the topic-data deletion, sending back 2.02 but keeping the resource in fully created state without returning a final 4.04 to cancel existing observations BUT still having the resource addressable to allow normal GET on it, for example for retrieving the last published/historical value/s. I am ambivalent here and would welcome guidance from others. I think MUST should not be used if there are no interoperability issues cause by using SHOULD.
-->

When a topic-data resource is deleted, the broker MUST also delete the topic-data parameter in the topic resource, unsubscribe all subscribers by removing them from the list of observers and return a final 4.04 (Not Found) response as per {{Section 3.2 of RFC7641}}. The topic is then set back to the half created state as per {{topic-lifecycle}}.

Example of a successful deletion:


~~~~
   Request:

   Header: DELETE (Code=0.04)
   Uri-Path: "ps"
   Uri-Path: "data"
   Uri-Path: "1bd0d6d"

   Response:

   Header: Deleted (Code=2.02)
~~~~

## Read the latest data {#read-data}

A client can get the latest published topic-data by making a GET request to the topic-data URI in the broker. Please note that discovery of the topic-data parameter is a required previous step (see {{topic-get-resource}}).

On success, the server MUST return 2.05 (Content) response with the data.

If the target URI does not match an existing resource or the topic is not in the fully created state (see {{topic-lifecycle}}), the broker MUST return a response code 4.04 (Not Found).

Example:

~~~~
   Request:

   Header: GET (Code=0.01)
   Uri-Path: "ps"
   Uri-Path: "data"
   Uri-Path: "1bd0d6d"

   Response:

   Header: Content (Code=2.05)
   Content-Format: 110
   Max-Age: 15
   Payload:
   {
      "n": "coap://dev1.example.com/temperature",
      "u": "Cel",
      "t": 1621452122,
      "v": 23.5
   }
~~~~

<!--
TODO: Do we add wildcards here?
https://github.com/core-wg/coap-pubsub/issues/42

### Subscribe to a subset of topic-data resources  {#wildcard}

Some implementations may want to subscribe to multiple topic-data resources with one single request. That is possible by using FETCH with

-->

## Rate Limiting {#rate-limit}

The server hosting the topic-data may have to handle a potentially large number of publishers and subscribers at the same time. This means it could become overwhelmed if it receives too many publications in a short period of time.

In this situation, if a publisher is sending publications too fast, the server SHOULD return a 4.29 (Too Many Requests) response {{RFC8516}}.  As described in {{RFC8516}}, the Max-Age option {{RFC7252}} in this response indicates the number of seconds after which the client may retry. The broker MAY also stop publishing messages from that publisher for the indicated time.

When a publisher receives a 4.29 (Too Many Requests) response, it MUST NOT send any new publication requests to the same topic-data resource before the time indicated by the Max-Age option has passed.

# CoAP Pubsub Parameters {#pubsub-parameters}

This document defines parameters used in the messages exchanged between a client and the broker during the topic creation and configuration process (see {{topic-resource-representation}}). The table below summarizes them and specifies the CBOR key to use instead of the full descriptive name.

Note that the media type application/core-pubsub+cbor MUST be used when these parameters are transported in the respective message fields.

~~~~~~~~~~~
+----------------------+----------+-----------+------------+
| Name                 | CBOR Key | CBOR Type | Reference  |
| -------------------- | -------- | --------- | ---------- |
| topic-name           | TBD1     | tstr      | [RFC-XXXX] |
| topic-data           | TBD2     | tstr      | [RFC-XXXX] |
| resource-type        | TBD3     | tstr      | [RFC-XXXX] |
| topic-content-format | TBD4     | uint      | [RFC-XXXX] |
| topic-type           | TBD5     | tstr      | [RFC-XXXX] |
| expiration-date      | TBD6     | tstr      | [RFC-XXXX] |
| max-subscribers      | TBD7     | uint      | [RFC-XXXX] |
| observer-check       | TBD8     | uint      | [RFC-XXXX] |
| topic-history        | TBD9     | uint      | [RFC-XXXX] |
| initialize           | TBD10    | bool      | [RFC-XXXX] |
| conf-filter          | TBD11    | array     | [RFC-XXXX] |
+----------------------+----------+-----------+------------+
~~~~~~~~~~~
{: #fig-CoAP-Pubsub-Parameters title="CoAP Pubsub Parameters" artwork-align="center"}

# Security Considerations {#seccons}

The architecture presented in this document inherits the security considerations from CoAP {{RFC7252}} and Observe {{RFC7641}}, as well as from Web Linking {{RFC8288}}, Link-Format {{RFC6690}}, and the CoRE Resource Directory {{RFC9176}}.

Communications between each client and the broker are RECOMMENDED to be secured, e.g., by using OSCORE {{RFC8613}} or DTLS {{RFC9147}}. Security considerations for the used secure communication protocols apply too.

The content published on a topic by a publisher client SHOULD be protected end-to-end between the publisher and all the subscribers to that topic. In such a case, it MUST be possible to assert source authentication of the published data. This can be achieved at the application layer, e.g., by using COSE {{STD96}}, {{RFC9053}}.

Access control of clients at the broker MAY be enforced for performing discovery operation, and SHOULD be enforced in a fine-grained fashion for operations related to the creation, update, and deletion of topic resources, as well as for operations on topic-data resources such as publication on and subscription to topics. This prevents rogue clients to, among other things, repeatedly create topics at the broker or publish (large) contents, which may result in Denial of Service against the broker and the active subscribers.

Building on {{RFC9594}}, its application profile for publish-subscribe communication with CoAP {{I-D.ietf-ace-pubsub-profile}} provides a security model that can be used in the architecture presented in this document, in order to enable secure communication between the different parties as well as secure, authorized operations of publishers and subscribers that fulfill the requirements above.

In particular, the application profile above relies on the ACE framework for Authentication and Authorization in Constrained Environments (ACE) {{RFC9200}} and defines a method to: authorize publishers and subscribers to perform operations at the broker, with fine-grained access control; authorize publishers and subscribers to obtain the keying material required to take part to a topic managed by the broker; protect published data end-to-end between its publisher and all the subscribers to the targeted topic, ensuring confidentiality, integrity, and source authentication of the published content end-to-end. That approach can be extended to enforce authorization and fine-grained access control for administrator clients that are intended to create, update, and delete topic-configurations at the broker.

# IANA Considerations {#iana}

<!-- To be registerd in IANA:
media type: application/core-pubsub+cbor
content format/type: application/core-pubsub+cbor
subregistry for the topic config
-->

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

## Media Type Registrations {#media-type}

This specification registers the 'application/core-pubsub+cbor' media type for messages of the protocols defined in this document and carrying parameters encoded in CBOR. This registration follows the procedures specified in {{BCP13}}.

      Type name: application

      Subtype name: core-pubsub+cbor

      Required parameters: N/A

      Optional parameters: N/A

      Encoding considerations: Must be encoded as a CBOR map containing the parameters defined in {{&SELF}}.

      Security considerations: See {{sec-cons}} of {{&SELF}}.

      Interoperability considerations: none

      Published specification: {{&SELF}}

      Applications that use this media type:  This type is used by clients that create, retrieve, and update topic-configurations at servers acting as a broker.

      Fragment identifier considerations: N/A

      Additional information: N/A

      Person & email address to contact for further information: CoRE WG mailing list (core@ietf.org),
      or IETF Web and Internet Transport (WIT) Area (wit@ietf.org)

      Intended usage: COMMON

      Restrictions on usage: none

      Author/Change controller: IETF

      Provisional registration: no

## CoAP Content-Formats {#content-type}

IANA is asked to register the following entry to the "CoAP Content-Formats" registry within the "CoRE Parameters" registry group.

Content Type: application/core-pubsub+cbor

Content Coding: -

ID: TBD

Reference: {{&SELF}}

## Resource Types {#iana-rt}

IANA is asked to enter the following values in the "Resource Type (rt=) Link Target Attribute Values" registry within the "Constrained Restful Environments (CoRE) Parameters" registry group.

~~~
Value: core.ps
Description: Publish-Subscribe Broker
Reference: [RFC-XXXX]

Value: core.ps.coll
Description: Topic-collection resource of a Publish-Subscribe Broker
Reference: [RFC-XXXX]

Value: core.ps.conf
Description: Topic-configuration resource of a Publish-Subscribe Broker
Reference: [RFC-XXXX]

Value: core.ps.data
Description: Topic-data resource of a broker
Reference: [RFC-XXXX]
~~~

## CoAP Pubsub Parameters {#iana-coap-pubsub-parameters}

This specification establishes the "CoAP Pubsub topic-configuration Parameters" IANA subregistry within the "Constrained RESTful Environments (CoRE) Parameters" registry group.

The registration policy is either "Private Use", "Standards Action with Expert Review", or "Specification Required" or "Expert Review" per {{RFC8126}}. "Expert Review" guidelines are provided in {{review}}.

All assignments according to "Standards Action with Expert Review" are made on a "Standards Action" basis per Section 4.9 of {{RFC8126}} with "Expert Review" additionally required per Section 4.5 of {{RFC8126}}. The procedure for early IANA allocation of "standards track code points" defined in {{RFC7120}} also applies. When such a procedure is used, IANA will ask the designated expert(s) to approve the early allocation before registration. In addition, working group chairs are encouraged to consult the expert(s) early during the process outlined in Section 3.1 of {{RFC7120}}.

The columns of this registry are:

* Name: This is a descriptive name that enables easier reference to the item. The name MUST be unique. It is not used in the encoding.

* CBOR Key: This is the value used as the CBOR key of the item. These values MUST be unique. The value can be a positive integer, a negative integer, or a text string. Different ranges of values use different registration policies {{RFC8126}}. Integer values from -256 to 255 as well as text strings of length 1 are designated as "Standards Action With Expert Review". Integer values from -65536 to -257 and from 256 to 65535, as well as text strings of length 2 are designated as "Specification Required". Integer values greater than 65535 as well as text strings of length greater than 2 are designated as "Expert Review". Integer values less than -65536 are marked as "Private Use".

* CBOR Type: This contains the CBOR type of the item, or a pointer to the registry that defines its type, when that depends on another item.

* Reference: This contains a pointer to the public specification for the item.

This subregistry has been initially populated with the values in {{fig-CoAP-Pubsub-Parameters}}.

## Expert Review Instructions {#review}

The IANA registry established in this document is defined as "Standards Action with Expert Review", "Specification Required", and "Epert Review" are three of the registration policies defined for the IANA subregistry established in {{iana-coap-pubsub-parameters}}. This section gives some general guidelines for what the experts should be looking for; however, they are being designated as experts for a reason, so they should be given substantial latitude.

Expert reviewers should take into consideration the following points:

* Clarity and correctness of registrations. Experts are expected to check the clarity of purpose and use of the requested entries. Experts need to make sure that registered parameters are clearly defined in the corresponding specification. Parameters that do not meet these objectives of clarity and completeness must not be registered.

* Point squatting should be discouraged. Reviewers are encouraged to get sufficient information for registration requests to ensure that the usage is not going to duplicate one that is already registered and that the point is likely to be used in deployments. The zones tagged as "Private Use" are intended for testing purposes and closed environments. Code points in other ranges should not be assigned for testing.

* Specifications are required for the "Standards Action With Expert Review" range of point assignment. Specifications should exist for "Specification Required" ranges, but early assignment before a specification is available is considered to be permissible. When specifications are not provided, the description provided needs to have sufficient information to identify what the point is being used for.

* Experts should take into account the expected usage of fields when approving point assignment. Documents published via Standards Action can also register points outside the Standards Action range. The length of the encoded value should be weighed against how many code points of that length are left, the size of device it will be used on, and the number of code points left that encode to that size.

--- back

# Document Updates # {#sec-document-updates}
{:removeinrfc}

## Version -13 to -14

* Section restructuring for better readability.
* Updated topic configuration interactions.
* Introduced iPATCH section.
* Various clarifications of default values for parameters.
* New examples for several interactions.
* Updated topic discovery section.
* Other editorial changes

## Version -14 to -15

* Code bug fix https://github.com/jaimejim/aiocoap-pubsub-broker/commit/f32ce4866a81319238d6e905de439c9410cce175
* Added two new optional topic configuration parameters; ‘initialize,’ and ‘topic-history’.
* Modified all examples to conform to RFC9594.
* Added the explicit cancelation of ongoing subscriptions when topic configuration parameters are changed.
* Added editorial changes based on feedback.
* Clarifications on Topic Configuration creation.
* Other editorial changes

## Version -15 to -16

* Various updates throughout the document based on AD review.
* IANA clarifications

# Acknowledgements
{: numbered='no'}

The current version of this document contains a substantial contribution by Klaus Hartke's proposal {{I-D.hartke-t2trg-coral-pubsub}}, which defines the topic resource model and structure as well as the topic lifecycle and interactions. It also follows a similar architectural design as that provided by Marco Tiloca's {{I-D.ietf-ace-oscore-gm-admin}}.

The authors would like to also thank {{{Marco Tiloca}}}, {{{Francesca Palombini}}}, {{{Carsten Bormann}}}, {{{Hannes Tschofenig}}}, {{{Zach Shelby}}}, {{{Mohit Sethi}}}, Peter van der Stok, Tim Kellogg, Anders Eriksson, {{{Goran Selander}}}, Mikko Majanen, {{{Olaf Bergmann}}}, {{{David Navarro}}}, Oscar Novo and Lorenzo Corneo for their valuable contributions and reviews.

---
stand_alone: true
ipr: trust200902
docname: draft-ietf-core-coap-pubsub-latest
title: A publish-subscribe architecture for the Constrained Application Protocol (CoAP)
area: Applications and Real-Time Area (art)
wg: Internet Engineering Task Force
kw: CoRE
cat: std
consensus: true
submissiontype: IETF
pi:
  strict: 'yes'
  toc: 'yes'
  tocdepth: '4'
  symrefs: 'yes'
  sortrefs: 'yes'
  compact: 'yes'
  subcompact: 'no'
venue:
  mail: core@ietf.org
  github: core-wg/coap-pubsub

author:
- ins: J. Jimenez
  name: Jaime Jimenez
  org: Ericsson
  email: jaime@iki.fi
- ins: M. Koster
  name: Michael Koster
  org: Dogtiger Labs
  email: michaeljohnkoster@gmail.com
- ins: A. Keranen
  name: Ari Keranen
  org: Ericsson
  email: ari.keranen@ericsson.com

contributor:
- name: Marco Tiloca
  organization: RISE AB
  email: marco.tiloca@ri.se
  contribution: Marco provided thorough reviews and guidance on the last versions of this document.

normative:
  RFC6570:
  RFC6690:
  RFC7252:
  RFC8516:
  RFC9167:
  RFC7641:
informative:
  RFC8288:
  I-D.hartke-t2trg-coral-pubsub:
  I-D.ietf-ace-oscore-gm-admin:
  I-D.ietf-ace-pubsub-profile:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document describes a publish-subscribe architecture for the Constrained Application Protocol (CoAP), extending the capabilities of CoAP communications for supporting endpoints with long breaks in connectivity and/or up-time. CoAP clients publish on and subscribe to a topic via a corresponding topic resource at a CoAP server acting as broker.

--- middle

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) {{!RFC7252}} supports
machine-to-machine communication across networks of constrained
devices and constrained networks. CoAP uses a request/response model where clients make requests to servers in order to request actions on resources. Depending on the situation the same device may act either as a server, a client, or both.

One important class of constrained devices includes devices that are intended to run for years from a small battery, or by scavenging energy from their environment. These devices have limited up-time because they spend most of their time in a sleeping state with no network connectivity. Another important class of nodes are devices with limited reachability due to middle-boxes like Network Address Translators (NATs) and firewalls.

For these nodes, the client/server-oriented architecture of REST can be challenging when interactions are not initiated by the devices themselves. A publish/subscribe-oriented architecture where nodes exchange data via topics through a broker entity might fit these nodes better.

This document applies the idea of broker-based publish-subscribe to Constrained RESTful Environments using CoAP. It defines a broker that allows to create, discover subscribe and publish on topics.

## Terminology {#terminology}

{::boilerplate bcp14-tagged}

This specification requires readers to be familiar with all the terms and
concepts that are discussed in {{?RFC8288}} and {{!RFC6690}}. Readers
should also be familiar with the terms and concepts discussed in
{{!RFC7252}}, {{!RFC9167}} and {{!RFC7641}}. The URI template
format {{!RFC6570}} is used to describe the REST API defined in
this specification.

This specification makes use of the following terminology:

{:vspace}
publish-subscribe (pub/sub):
: A message communication model where messages associated with specific topics are sent to a broker. Interested parties, i.e. subscribers, receive these topic-based messages from the broker without the original sender knowing the recipients. The broker handles matching and delivering these messages to the appropriate subscribers.

publishers and subscribers:
: CoAP clients can act as publishers or as subscribers. Publishers send CoAP messages (publications) to the broker on specific topics. Subscribers have an ongoing observation relation (subscription) to a topic. Both roles operate without any mutual knowledge, guided by their respective topic interests.

topic collection:
: A set of topic configurations. A topic collection is hosted as one collection resource at the broker, and its representation is the list of links to the topic resources corresponding to each topic configuration.

topic-configuration:
: A set of information concerning a topic, including its configuration and other metadata. A topic configurations is hosted as one topic resource at the broker, and its representation is the set of configuration information concerning the topic. All the topic resources associated with the same topic collection share a common base URI, i.e., the URI of the collection resource. Throughout this document the word "topic" and "topic-configuration" can be used interchangeably.

topic-data resource:
: A resource where clients can publish data and/or subscribe to data for a specific topic. The representation of the topic resource corresponding to such a topic also specifies the URI to the present topic-data resource.

broker:
: A CoAP server that hosts one or more topic collections with their topic-configurations, and possibly also topic-data resources. The broker is responsible for the store-and-forward of state update representations, for the topics for which it hosts the corresponding topic-data resources. The broker is also responsible of handling the topic lifecycle as defined in {{topic-lifecycle}}. The creation, configuration, and discovery of topics at a broker is specified in {{topics}}.

## CoAP Publish-Subscribe Architecture

{{fig-arch}} shows a simple Publish/Subscribe architecture over CoAP.

Topics are created by the broker, but the initial configuration can be proposed by a client (e.g., a publisher or a dedicated administrator) over the RESTful interface of a corresponding topic resource hosted by the broker.

Publishers submit their data over the RESTful interface of a topic-data resource corresponding to the topic, which may be hosted by the broker. Subscribers to a topic are notified of new publications by using Observe {{?RFC7641}} on the corresponding topic-data resource.

The broker is responsible for the store-and-forward of state update representations between CoAP clients. Subscribers observing a resource will receive notifications, the delivery of which is done on a best-effort basis.

~~~~ aasvg
     CoAP                      CoAP                 CoAP
     clients                  server                clients
   .-----------.          .----------.  observe  .-----------.
   |           | publish  |          |<----------+           |
   | publisher +--------->+          +---------->| subscribe |
   |           |          |          +---------->|           |
   '-----------'          |          |           '-----------'
        ...               |  broker  |                ...
        ...               |          |                ...
   .-----------.          |          |  observe  .-----------.
   |           | publish  |          |<----------+           |
   | publisher +--------->|          +---------->| subscribe |
   |           |          |          +---------->|           |
   '-----------'          '----------'           '-----------'
~~~~
{: #fig-arch title='Publish-subscribe architecture over CoAP' artwork-align="center"}

This document describes two sets of interactions, interactions to configure topics and their lifecycle (see {{topic-configuration-interactions}}) and interactions about the topic-data (see {{topic-data-interactions}}).

Topic-configuration interactions are discovery, create, read configuration, update configuration, delete configuration and handle the management of the topics.

Topic-data interactions are publish, subscribe, unsubscribe, read and delete, these operations are oriented on how data is transferred from a publisher to a subscriber.

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

A topic-collection resource can have topic resources as its children resources, with resource type "core.ps.conf".

# Pub-Sub Topics {#topics}

The configuration side of a "publish/subscribe broker" consists of a collection of topics. These topics as well as the collection itself are exposed by a CoAP server as resources (see {{fig-topic}}). Each topic is associated with: a topic resource hosted by the broker and a topic-data resource that can be hosted by the broker or by a different server. The topic resource is used by a client creating or administering a topic. The topic-data resource is used by the publishers and the subscribers to a topic.

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

Each topic configuration is represented as a link, where the link target is the URI of the corresponding topic resource.

Publication and subscription to a topic occur at a link, where the link target is the URI of the corresponding topic-data resource. Such a link is specified by the topic-data entry within the topic resource (see {{topic-properties}}).

A topic resource with a topic-data link can also be simply called "topic".

The list of links to the topic resources can be retrieved from the associated topic collection resource, and represented as a Link Format document {{RFC6690}}where each such link specifies the link target attribute 'rt' (Resource Type), with value "core.ps.conf" defined in this document.

## Topic-Configuration Representation {#topic-resource-representation}

A CoAP client can create a new topic by submitting an initial configuration for the topic (see {{topic-create}}). It can also read and update the configuration of existing topics and delete them when they are no longer needed (see {{topic-configuration-interactions}}).

The configuration of a topic itself consists of a set of properties that can be set by a client or by the broker. The topic-configuration is represented as a CBOR map containing the configuration properties of the topic as top-level elements.

Unless specified otherwise, these are defined in this document and their CBOR abbreviations are defined in {{pubsub-parameters}}.

### Topic Properties {#topic-properties}

The CBOR map includes the following configuration parameters, whose CBOR abbreviations are defined in {{pubsub-parameters}} of this document.

* 'topic-name': A required field used as an application identifier. It encodes the topic name as a CBOR text string. Examples of topic names include human-readable strings (e.g., "room2"), UUIDs, or other values.

* 'topic-data': A required field (optional during creation) containing the URI of the topic-data resource for publishing/subscribing to this topic. It encodes the URI as a CBOR text string. This property may contain a fully formed URL including scheme and host or just the resource path when the topic-data resource for the topic is hosted by the broker.

* 'resource-type': A required field used to indicate the resource type of the topic-data resource for the topic. It encodes the resource type as a CBOR text string. The value should be "core.ps.conf".

* 'media-type': An optional field used to indicate the media type of the topic-data resource for the topic. It encodes the media type as a this information as the integer identifier of the CoAP content format (e.g., value is "50" for "application/json").

* 'topic-type': An optional field used to indicate the attribute or property of the topic-data resource for the topic. It encodes the attribute as a CBOR text string. Example attributes include "temperature".

* 'expiration-date': An optional field used to indicate the expiration date of the topic. It encodes the expiration date as a CBOR text string. The value should be a date string in ISO 8601 format (e.g., "2023-03-31T23:59:59Z"). The broker can use this field to automatically remove topics that are no longer valid. If this field is not present, the topic will not expire automatically.

* 'max-subscribers': An optional field used to indicate the maximum number of simultaneous subscribers allowed for the topic. It encodes the maximum number as an unsigned CBOR integer. If this field is not present, there is no limit to the number of simultaneous subscribers allowed. The broker can use this field to limit the number of subscribers for the topic.

* 'observer-check': An optional field that controls the maximum number of seconds between two consecutive Observe notifications sent as Confirmable messages to each topic subscriber. Encoded as a CBOR unsigned integer greater than 0, it ensures subscribers who have lost interest and silently forgotten the observation do not remain indefinitely on the server's observer list. This parameter applies to the server hosting the topic-data resource. If another CoAP server hosts the topic-data resource, that server is responsible for applying the observer-check value. The default value for this field is 86400, as defined in {{!RFC7641}}, which corresponds to 24 hours.

## Discovery

A client can perform a discovery of: the broker; the topic collection resources and topic resources hosted by the broker; and the topic-data resources associated with those topic resources.

### Broker Discovery {#broker-discovery}

CoAP clients MAY discover brokers by using CoAP Simple Discovery, via multicast, through a Resource Directory (RD) {{!RFC9167}} or by other means specified in extensions to {{!RFC7252}}. Brokers MAY register with a RD by following the steps on Section 5 of {{!RFC9167}} with the resource type set to "core.ps" as defined in {{iana}} of this document.

### Topic Collection Discovery

A Broker SHOULD offer a topic discovery entry point to enable clients to find topics of interest. The resource entry point is the topic collection resource collecting the topic configurations for those topics (see Section 1.2.2 of {{?RFC6690}}) and is identified by the resource type "core.ps.coll".

The specific resource path is left for implementations, examples in this document use the "/ps" path. The interactions with a topic collection are further defined in {{topic-collection-interactions}}.

Since the representation of the topic collection resource includes the links to the associated topic resources, it is not required to locate those links under "/.well-known/core", also in order to limit the size of the Link Format document returned as result of the discovery.

Example:

~~~~
=> GET
   Uri-Path: .well-known/core
   Resource-Type: core.ps.coll

   <= 2.05 Content
   Content-Format: 40 (application/link-format)
   </ps1>;rt="core.ps.coll";ct=40,
   </ps2>;rt="core.ps.coll";ct=40
~~~~

### Topic-Configuration Discovery {#topic-discovery}

Each topic collection is associated with a group of topic resources, each detailing the configuration of its respective topic (refer to Section {{topic-properties}}). Each topic resource is identified by the resource type "core.ps.conf".

Below is an example of discovery via /.well-known/core with rt=core.ps.conf that returns a list of topics, as the list of links to the corresponding topic resources.

<!--
TODO: add the ct part in IANA and add the example here:
- If you want to indicate ct= in one of this links, then it should be ct=X, where is the the Content-Format identifier for application/pubsub+cbor
-->

~~~~
=> 0.01 GET
   Uri-Path: .well-known/core
   Resource-Type: core.ps.conf

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   </ps1/h9392>;rt="core.ps.conf",
   </ps2/other/path/2e3570>;rt=core.ps.conf
~~~~

### Topic-Data Discovery

Within a topic, there is the topic-data property containing the URI of the topic-data resource that a CoAP client can subscribe and publish to. Resources exposing resources of the topic-data type are expected to use the resource type 'core.ps.data'.

Below is an example discovery via .well-known/core that returns list of all topic-data resources. This operation can be useful for subscribers to find what can they subscribe to.

~~~~
=> 0.01 GET
   Uri-Path: .well-known/core
   Resource-Type: core.ps.data

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   </ps1/data/62e4f8d>; rt=core.ps.data; obs
   </ps2/otherdata/86fe8fd>; rt=core.ps.data; obs
~~~~

## Topic Collection Interactions {#topic-collection-interactions}

These are the interactions that can happen directly with a specific topic collection.

### Retrieving all topic-configurations {#topic-get-all}

A client can request a collection of the topics present in the broker by making a GET request to the collection URI.

On success, the server returns a 2.05 (Content) response with a representation of the list of all topic-configuration resources (see Section {{topic-resource-representation}}) in the collection.

Depending on the permission set each client MAY receive a different list of topics that they are authorized to read.

If there are existing topic-data resources under the collection resource, those will also be present in the response.

Example:

~~~~
=> 0.01 GET
   Uri-Path: ps

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   </ps/h9392>;rt="core.ps.conf",
   </ps/2e3570>; ct=application/link-format; rt=core.ps.conf; obs,
~~~~

### Getting topic-configurations by Properties {#topic-get-properties}
<!--
FETCH to /topic-collection with filter
retrieve only the topics that match the filter
request is cbor
response is link format
-->

A client can filter a collection of topics by submitting the
representation of a topic filter (see Section {{topic-fetch-resource}}) in a FETCH request to the topic collection URI.

On success, the server returns a 2.05 (Content) response with a
representation of a list of topics in the collection (see
Section {{topic-discovery}}) that match the filter in CoRE link format {{!RFC6690}}.

Example:

<!--
TODO: this example and why I was using /ps/tc, tc seems redundant
update this example.
-->

~~~~
=> 0.05 FETCH
   Uri-Path: ps
   Content-Format: TBD (application/pubsub+cbor)

   {
     "resource-type" : "core.ps.conf"
   }

<= 2.05 Content
      </ps/2e3570>; ct=application/link-format; rt=core.ps.conf; obs,

~~~~

### Creating a Topic {#topic-create}
<!--
POST to /topic-collection
create new topic
request is cbor
response (created) is cbor including the link to new topic-config resource
creator proposes topic name but broker approves
-->

A client can add a new topic-configurations to a collection of topics by submitting a representation of the initial topic resource (see Section {{topic-resource-representation}}) in a POST request to the topic collection URI. The topic MUST contain at least a subset of the {{topic-properties}} , namely: topic-name and resource-type.

A CoAP endpoint creating a topic MAY specify a topic-data URI different than that used by the broker. The broker may then simply forward the observation requests to the topic-data URI as shown in {{fig-external-server}}.

If the topic-data is empty the broker will assign a resource for a publisher to publish to. Please note that the topic will NOT be fully created until a publisher has published some data to it (See {{topic-lifecycle}}).

On success, the server returns a 2.01 (Created) response indicating the topic URI of the new topic and the current representation of the topic resource.

If a topic manager is present in the broker, the topic creation  may require manager approval subject to certain conditions. If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The broker MUST respond with a 4.00 (Bad Request) error if any received parameter is specified multiple times, invalid or not recognized.

~~~~
=> 0.02 POST
   Uri-Path: ps
   Content-Format: TBD2 (application/core-pubsub+cbor)
   TBD (this should be a CBOR map with the mandatory parameters)
   {
     "topic-name" : "living-room-sensor"
     "resource-type" : "core.ps.conf"
   }

<= 2.01 Created
   Location-Path: ps/h9392
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
     "topic-name" : "living-room-sensor",
     "topic-data" : "ps/data/1bd0d6d"
     "resource-type" : "core.ps.conf"
   }
~~~~


## Topic-Configuration Interactions {#topic-configuration-interactions}

These are the interactions that can happen at the topic resource level.

### Getting a topic-configuration {#topic-get-resource}

<!--
GET to /topic-config
retrieve a topic configuration
response is cbor
-->

A client can read the configuration of a topic by making a GET request to the topic resource URI.

On success, the server returns a 2.05 (Content) response with a representation of the topic resource. The response has as payload the representation of the topic resource as specified in {{topic-resource-representation}}.

If a topic manager (TBD) is present in the broker, retrieving topic information may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The response payload is a CBOR map, whose possible entries are specified in {{topic-resource-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

For example, below is a request on the topic "ps/h9392":

~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: h9392

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
      "topic-name" : "living-room-sensor",
      "topic-data" : "ps/data/1bd0d6d",
      "resource-type": "core.ps.conf",
      "media-type": "application/senml-cbor",
      "target-attribute": "temperature",
      "expiration-date": "2023-04-00T23:59:59Z",
      "max-subscribers": 100
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

The request contains a CBOR map with a configuration filter or 'conf-filter', a CBOR array with CBOR abbreviation. Each element of the array specifies one requested configuration parameter of the current topic resource (see {{topic-resource-representation}}).

On success, the server returns a 2.05 (Content) response with a representation of the topic resource. The response has as payload the partial representation of the topic resource as specified in {{topic-resource-representation}}.

If a topic manager (TBD) is present in the broker, retrieving topic information may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error.

The response payload is a CBOR map, whose possible entries are specified in {{topic-resource-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

Both request and response MUST have Content-Format set to "application/core-pubsub+cbor".

Example:

~~~~
=> 0.05 FETCH
   Uri-Path: ps
   Uri-Path: h9392
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
     "conf-filter" : [topic-data, media-type]
   }

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
     "topic-data" : "ps/data/1bd0d6d",
     "media-type": "application/senml-cbor"
   }

~~~~

### Updating the topic-configuration {#topic-update-resource}

<!--
PUT to /topic-conf
override the whole configuration
request is cbor
response is cbor
-->

A client can update the configuration of a topic by submitting the representation of the updated topic  (see Section 3.1.3) in a PUT or POST request to the topic URI. Any existing properties in the configuration are overwritten by this update.

On success, the server returns a 2.04 (Changed) response and the current full resource representation. The broker may chose not to overwrite parameters that are not explicitly modified in the request.

Note that updating the "topic-data" path will automatically cancel all existing observations on it and thus will unsubscribe all subscribers. Similarly, decreasing max-subscribers will also cause that some subscribers get unsubscribed. Unsubscribed endpoints SHOULD receive a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2.

Example:

~~~~
=> 0.03 PUT
   Uri-Path: ps
   Uri-Path: h9392
   Content-Format: TBD2 (application/core-pubsub+cbor)

   {
      "topic-name" : "living-room-sensor",
      "topic-data" : "ps/data/1bd0d6d",
      "target-attribute": "temperature",
      "expiration-date": "2023-04-28T23:59:59Z",
      "max-subscribers": 2
   }

<= 2.04 Changed
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
      "topic-name" : "living-room-sensor",
      "topic-data" : "ps/data/1bd0d6d",
      "resource-type": "core.ps.conf",
      "media-type": "application/senml-cbor",
      "target-attribute": "temperature",
      "expiration-date": "2023-04-28T23:59:59Z",
      "max-subscribers": 2
   }
~~~~

### Deleting a topic-configuration {#topic-delete}

A client can delete a topic by making a CoAP DELETE request on the topic resource URI.

On success, the server returns a 2.02 (Deleted) response.

When a topic resource is deleted, the broker SHOULD also delete the topic-data resource, unsubscribe all subscribers by removing them from the list of observers and returning a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2.

Example:

~~~~
=> 0.04 DELETE
   Uri-Path: ps
   Uri-Path: h9392

<= 2.02 Deleted
~~~~

# Publish and Subscribe {#pubsub}

The overview of the publish/subscribe mechanism over CoAP is as follows: a publisher publishes to a topic by submitting the data in a PUT request to a topic-data resource and subscribers subscribe to a topic by submitting a GET request with the Observe option active to a topic-data resource. When resource state changes, subscribers observing the resource {{!RFC7641}} at that time will receive a notification.

As shown in {{topics}}, each topic contains two resources: topic-configuration resource and topic-data. In that section we explained the creation and configuration of the topic-configuration resources. This section will explain the management of topic-data resources.

A topic-data resource does not exist until some initial data has been published to it.  Before initial data has been published, the topic-data resource yields a 4.04 (Not Found) response. If such a "half created" topic is undesired, the creator of the topic can simply immediately publish some initial placeholder data to make the topic "fully created" (see {{topic-lifecycle}}).

URIs for topic resources are broker-generated (see {{topic-create}}). URIs for topic-data MAY be broker-generated or client-generated. There is no necessary URI pattern dependence between the URI where the data exists and the URI of the topic resource. Topic resource and data resources might even be hosted on different servers.

## Topic Lifecycle {#topic-lifecycle}

When a topic is newly created, it is first placed by the server into the HALF CREATED state (see {{fig-life}}). In this state, a client can read and update the configuration of the topic and delete the topic. A publisher can publish to the topic-data resource.  However, a subscriber cannot yet observe the topic-data resource nor read the latest data.

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
                '-'   Topic          Topic    '-'
                             DELETED
~~~~
{: #fig-life title='Lifecycle of a Topic' artwork-align="center"}

After a publisher publishes to the topic-data for the first time, the topic is placed into the FULLY CREATED state. In this state, a client can read, update and delete the topic; a publisher can publish to the topic-data resource; and a subscriber can observe the topic-data resource.

When a client deletes a topic-configuration resource, the topic is placed into the DELETED state and shortly after removed from the server. In this state, all subscribers are removed from the list of observers of the topic-data resource and no further interactions with the topic are possible.

When a client deletes a topic-data, the topic is placed into the HALF CREATED state, where clients can read, update and delete the topic-configuration and await for a publisher to begin publication.


## Topic-Data Interactions {#topic-data-interactions}

Interactions with the topic-data resource are covered in this section. The interactions with topic-data are same as that of any other CoAP resource.

One variant shown in {{fig-external-server}} is where the resource is hosted. While the broker can create a topic-data resource when the topic is created, the client can select to host the data in a different CoAP server than that of the topic resource.

~~~~ aasvg
         [central-ps.example.com]
               CoAP server 1
  .----------------------------------------.
  |            ___                         |
  |  Topic    /   \                        |      [2001:db8::2:1]
  |Collection \___/                        |       CoAP server 2
  | Resource       \                       |   .------------------.
  |                 \___________           |   |                  |
  |                  \          \          |   |                  |
  |                   \ ......   \ ......  |   |.........         |
  |                  : \___  :  : \___  :  |   |:  ___  : Topic   |
  |           Topic  : / * \ :  : / * \----(---(--/   \ : Data    |
  |        Resource  : \_|_/ :  : \___/ :  |   |: \___/ : Resource|
  |                  ....|....  .........  |   |:.......:         |
  |                  ....|....             |   |                  |
  |           Topic  :  _|_  :             |   '------------------'
  |            Data  : /   \ :             |
  |        Resource  : \___/ :             |
  |                  :.......:             |
  '----------------------------------------'
~~~~
{: #fig-external-server title="topic-data hosted externally" artwork-align="center"}

### Publish {#publish}

A topic-configuration with a topic-data resource must have been created in order to publish data to it (See Section {{topic-create}}) and be in the half-created state in order to the publish operation to work (see {{topic-lifecycle}}).

A client can publish data to a topic by submitting the data in a PUT request to the topic-data URI as indicated in its topic resource property. Please note that the topic-data URI is not the same as the topic-configuration URI used for configuring the topic (see {{topic-resource-representation}}).

On success, the server returns a 2.04 (Updated) response. However, when data is published to the topic for the first time, the server instead MUST return a 2.01 (Created) response and set the topic in the fully-created state (see {{topic-lifecycle}}).

If the request does not have an acceptable content format, the server returns a 4.15 (Unsupported Content Format) response.

If the client is sending publications too fast, the server returns a
4.29 (Too Many Requests) response {{!RFC8516}}.

Example of first publication:

~~~~
=> 0.03 PUT
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 1bd0d6d
   Content-Format: 110

   {
      "n": "temperature",
      "u": "Cel",
      "t": 1621452122,
      "v": 23.5
   }

<= 2.01 Created
~~~~

Example of subsequent publication:

~~~~
=> 0.03 PUT
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 1bd0d6d
   Content-Format: 110

   {
      "n": "temperature",
      "u": "Cel",
      "t": 182734122,
      "v": 22.5
   }

<= 2.04 Updated
~~~~

### Subscribe {#subscribe}

A client can subscribe to a topic by sending a CoAP GET request with the Observe set to '0' to subscribe to resource updates. {{!RFC7641}}.

On success, the broker MUST return 2.05 (Content) notifications with the data.

If the topic is not yet in the fully created state (see {{topic-lifecycle}}) the broker SHOULD return a response code 4.04 (Not Found).

The following response codes are defined for the Subscribe operation:

Success:
: 2.05 "Content". Successful subscribe with observe response, current value included in the response.

Failure:
: 4.04 "Not Found". Topic does not exist.

If the 'max-clients' parameter has been reached, the server must treat that as specified in section 4.1 of {{!RFC7641}}. The response MUST NOT include an Observe Option, the absence of which signals to the subscriber that the subscription failed.

Example:

~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 1bd0d6d
   Observe: 0

<= 2.05 Content
   Content-Format: 110
   Observe: 10001
   Max-Age: 15

  {
    "bn": "urn:dev:os:193-iot/sparrow/jorvas/",
    "n": "Raitis-lampotila",
    "u": "Cel",
    "t": 1696340182,
    "v": 19.87
  }

<= 2.05 Content
   Content-Format: 110
   Observe: 10002
   Max-Age: 15

  {
    "bn": "urn:dev:os:193-iot/sparrow/jorvas/",
    "n": "Raitis-lampotila",
    "u": "Cel",
    "t": 1696340182,
    "v": 21.87
  }
~~~~

### Unsubscribe {#unsubscribe}

A CoAP client can unsubscribe simply by cancelling the observation as described in Section 3.6 of {{!RFC7641}}. The client MUST either use CoAP GET with the Observe Option set to 1 or send a CoAP Reset message in response to a notification.

In some circumstances, it may be desirable to cancel an observation and release the resources allocated by the broker. In this case, a client MAY explicitly deregister by issuing a GET request that has the Token field set to the token of the observation to be cancelled and includes an Observe Option with the value set to 1 (deregister).

As per {{!RFC7641}} a server that transmits notifications mostly in non-confirmable messages MUST send a notification in a confirmable message instead of a non-confirmable message at least every 24 hours.

This value can be modified at the broker by the administrator of a topic by modifying the parameter "observer-check" on {{topic-resource-representation}}. This would allow to change the rate at which different implementations verify that a subscriber is still interested in observing a topic-data resource.

### Delete topic-data {#delete-topic-data}

A publisher MAY delete a topic by making a CoAP DELETE request on the topic-data URI.

On success, the server returns a 2.02 (Deleted) response.

When a topic-data resource is deleted, the broker SHOULD also delete the topic-data parameter in the topic resource, unsubscribe all subscribers by removing them from the list of observers and return a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2. The topic is then set back to the half created state as per {{topic-lifecycle}}.

Example:

~~~~
=> 0.04 DELETE
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 1bd0d6d

<= 2.02 Deleted
~~~~

## Read latest data {#read-data}

A client can get the latest published topic-data by making a GET request to the topic-data URI in the broker. Please note that discovery of the topic-data parameter is a required previous step (see {{topic-get-resource}}).

On success, the server MUST return 2.05 (Content) response with the data.

If the target URI does not match an existing resource or the topic is not in the fully created state (see {{topic-lifecycle}}), the broker SHOULD return a response code 4.04 (Not Found).

If the broker can not return the requested content format it MUST return a response code 4.15 (Unsupported Content Format).

Example:

~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 1bd0d6d

<= 2.05 Content
   Content-Format: 110
   Max-Age: 15

   {
      "n": "temperature",
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

In this situation, if a publisher is sending publications too fast, the server SHOULD return a 4.29 (Too Many Requests) response {{!RFC8516}}.  As described in {{!RFC8516}}, the Max-Age option {{!RFC7252}} in this response indicates the number of seconds after which the client may retry. The broker MAY also stop publishing messages from that publisher for the indicated time.

When a publisher receives a 4.29 (Too Many Requests) response, it MUST NOT send any new publication requests to the same topic-data resource before the time indicated by the Max-Age option has passed.

# CoAP Pubsub Parameters {#pubsub-parameters}

This document defines parameters used in the messages exchanged between a client and the broker during the topic creation and configuration process (see {{topic-resource-representation}}). The table below summarizes them and specifies the CBOR key to use instead of the full descriptive name.

Note that the media type application/core-pubsub+cbor MUST be used when these parameters are transported in the respective message fields.

~~~~
+-----------------+-----------+--------------+------------+
| Name            | CBOR Key  | CBOR Type    | Reference  |
|-----------------|-----------|--------------|------------|
| topic-name      | TBD1      | tstr         | [RFC-XXXX] |
| topic-data      | TBD2      | tstr         | [RFC-XXXX] |
| resource-type   | TBD3      | tstr         | [RFC-XXXX] |
| media-type      | TBD4      | tstr (opt)   | [RFC-XXXX] |
| target-attribute| TBD5      | tstr (opt)   | [RFC-XXXX] |
| expiration-date | TBD6      | tstr (opt)   | [RFC-XXXX] |
| max-subscribers | TBD7      | uint (opt)   | [RFC-XXXX] |
| observer-check  | TBD8      | uint (opt)   | [RFC-XXXX] |
+-----------------+-----------+--------------+------------+
~~~~
{: #fig-CoAP-Pubsub-Parameters title="CoAP Pubsub Parameters" artwork-align="center"}

# Security Considerations

The security considerations discussed in this document cover various aspects related to the publish-subscribe architecture and the management of topics, administrators, and the change of topic-configuration.

## Change of Topic-Configuration

When a topic configuration changes, it may result in disruptions for the subscribers. Some potential issues that may arise include:

* Limiting the number of subscribers will cause to cancel ongoing subscriptions until max-subscribers has been reached.
* Changing the topic-data value will cancel all ongoing subscriptions.
* Changing of the expiration-date may cause to cancel ongoing subscriptions if the topic expires at an earlier data.

To address these potential issues, it is vital to have an administration process in place for topic configurations, including monitoring, validation, and enforcement of security policies and procedures.

It is also recommended for subscribers to subscribe to the topic-configuration resource in order to receive notifications of topic parameter changes.

## Topic Administrators

In a publish-subscribe architecture, it is essential to ensure that topic administrators are trustworthy and authorized to perform their duties. This includes the ability to create, modify, and delete topics, enforce proper access control policies, and handle potential security issues arising from topic management.

The draft {{I-D.ietf-ace-pubsub-profile}} defines an application profile of the Authentication and Authorization for Constrained Environments (ACE) framework. The profile is designed to enable secure group communication for the architecture defined in this document "{{&SELF}}" (See {{fig-arch}}).

The profile relies on protocol-specific transport profiles of ACE for communication security, server authentication, and proof-of-possession for a key that is owned by the Client and bound to an OAuth 2.0 Access Token.

The document outlines the provisioning and enforcement of authorization information for Clients to act as Publishers and/or Subscribers. Additionally, it specifies the provisioning of keying material and security parameters that Clients use to protect their communications end-to-end through the Broker.

## Caching and Freshness

<!--
TODO fix
-->

A broker could become overloaded if it always had to provide the most recent cached resource representation of a topic-data to a subscriber. On deployments with a large number of clients and with many topic resources this would represent a big burden on the broker.

For this reason is it recommended to consider changing the default Max-Age Option, which has a value of 60 in CoAP, in order to cater to different deployment scenarios.

For example, the broker could choose not to cache anything, therefore it SHOULD explicitly include a Max-Age Option with a value of zero seconds. For more information about caching and freshness in CoAP, please check {{!RFC7641}} and {{!RFC7252}}.

# IANA Considerations {#iana}

<!-- TBD: This section is structured similarly as:
https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#name-resource-types
https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.1
https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.2
-->

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

## CoAP Pubsub Parameters ## {#iana-coap-pubsub-parameters}

IANA is asked to register the following entries in the "CoAP Pubsub Parameters" registry.

~~~
Name: topic-name
CBOR Key: TBD1
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: topic-data
CBOR Key: TBD2
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: resource-type
CBOR Key: TBD3
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: media-type
CBOR Key: TBD4
CBOR Type: tstr (opt)
Reference: [RFC-XXXX]

Name: target-attribute
CBOR Key: TBD5
CBOR Type: tstr (opt)
Reference: [RFC-XXXX]

Name: expiration-date
CBOR Key: TBD6
CBOR Type: tstr (opt)
Reference: [RFC-XXXX]

Name: max-subscribers
CBOR Key: TBD7
CBOR Type: uint (opt)
Reference: [RFC-XXXX]

Name: observer-check
CBOR Key: TBD8
CBOR Type: uint (opt)
Reference: [RFC-XXXX]

Name: core.ps
CBOR Key: TBD9
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: core.ps.coll
CBOR Key: TBD10
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: core.ps.conf
CBOR Key: TBD11
CBOR Type: tstr
Reference: [RFC-XXXX]

Name: core.ps.data
CBOR Key: TBD12
CBOR Type: tstr
Reference: [RFC-XXXX]
~~~

## Resource Types # {#iana-rt}

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
Description: Topic-data resource of a Publish-Subscribe Broker
Reference: [RFC-XXXX]
~~~

# Acknowledgements
{: numbered='no'}

The current version of this document contains a substantial contribution by Klaus Hartke's proposal {{I-D.hartke-t2trg-coral-pubsub}}, which defines the topic resource model and structure as well as the topic lifecycle and interactions. It also follows a similar architectural design as that provided by Marco Tiloca's {{I-D.ietf-ace-oscore-gm-admin}}.

The authors would like to also thank Carsten Bormann, Hannes Tschofenig, Zach Shelby, Mohit Sethi, Peter van der Stok, Tim Kellogg, Anders Eriksson, Goran Selander, Mikko Majanen, and Olaf Bergmann for their valuable contributions and reviews.

---
title: "Publish-Subscribe Broker for the Constrained Application Protocol (CoAP)"
abbrev: "Publish-Subscribe Broker for CoAP"
category: std

docname: draft-ietf-core-coap-pubsub-latest
ipr: trust200902

number:
date:
consensus: true
v: 3
area: "Applications and Real-Time (ART)"
workgroup: "Constrained RESTful Environments"
submissiontype: IETF

keyword:
 - coap
 - pubsub

author:
- ins: M. Koster
  name: Michael Koster
  organization: SmartThings
  email: Michael.Koster@smartthings.com

- ins: A. Keranen
  name: Ari Keranen
  organization: Ericsson
  email: ari.keranen@ericsson.com

- ins: J. Jimenez
  name: Jaime Jimenez
  organization: Ericsson
  email: jaime@iki.fi

normative:
  RFC7252:
  RFC7641:

informative:
  I-D.irtf-t2trg-coral-pubsub:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

The Constrained Application Protocol (CoAP), and related extensions are intended
to support machine-to-machine communication in systems where one or more
nodes are resource constrained, in particular for low power wireless sensor
networks. This document defines a publish-subscribe Broker for CoAP that
extends the capabilities of CoAP for supporting nodes with long breaks in
connectivity and/or up-time.

--- middle

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) {{!RFC7252}} supports
machine-to-machine communication across networks of constrained
devices (e.g., 8-bit microcontrollers with limited RAM and ROM) and constrained networks {{!RFC7228}}. CoAP uses a request/response model where clients make requests to servers in order to request actions on resources. Depending on the situation the same device may act either as a server, a client, or both.

One important class of constrained devices includes devices that are intended to run for years from a small battery, or by scavenging energy from their environment. These devices have limited reachability because they spend most of their time in a sleeping state with no network connectivity. Another important class of nodes are devices with limited reachability due to middle-boxes like Network Address Translators (NATs) and firewalls.

For these nodes, the client/server-oriented architecture of REST can be challenging when interactions are not initiated by the devices themselves. A publish/subscribe-oriented architecture where nodes are separated by a broker and data is exchanged via topics might fit these nodes better.

This document applies the idea of a "Publish/Subscribe Broker" to Constrained RESTful Environments.  The broker enables store-and- forward data exchange between nodes, thereby facilitating the communication of nodes with limited reachability, providing simple many-to-many communication, and easing integration with other publish/subscribe systems.

## Requirements Language

{::boilerplate bcp14}

## Terminology {#terminology}

This specification requires readers to be familiar with all the terms and
concepts that are discussed in {{?RFC5988}} and {{!RFC6690}}. Readers
should also be familiar with the terms and concepts discussed in
{{!RFC7252}} and {{!RFC9167}}. The URI template
format {{!RFC6570}} is used to describe the REST API defined in
this specification.

This specification makes use of the following terminology:

publish-subscribe (pub/sub):
: A messaging paradigm where messages are published to a Broker and potential receivers can subscribe to the Broker to receive messages. The publishers do not (need to) know where the message will be eventually sent: the publications and subscriptions are matched by a Broker and publications are delivered by the Broker to subscribed receivers.

publish-subscribe broker:
: A CoAP server capable of receiving messages (publications) from CoAP clients acting as publishers and forwarding them to CoAP clients acting as subscribers The Broker can also temporarily store publications to satisfy future subscriptions and pending notifications.

CoAP client (publisher or subscriber):
: CoAP clients can act as publishers or as subscribers. Publishers send CoAP messages to the Broker on specific topics. Subscribers have an ongoing observation relation (subscription) to a topic. Neither publishers nor subscribers need to have any knowledge each other; they just have to share the topic they are publishing and subscribing to.

topic:
: An unique identifier for a particular item being published and/or subscribed to. A Broker uses the topics to match subscriptions to publications. A reference to a Topic on a Broker is a valid CoAP URI. Topics have to be created and configured before any data can be published. Clients may propose new topics to be created; however, it is up to the broker to choose if and how a topic is created. The broker also decides the URI of each topic. The creation, configuration, and discovery of topics at a broker is specified in {{topics}}.

## CoAP Publish-Subscribe Architecture

{{fig-arch}} shows a simple Publish/Subscribe architecture over CoAP. In it, publishers submit their data over a RESTful interface a broker-managed resource (topic) and subscribers observe this resource using {{?RFC7641}}. Resource state information is updated between the CoAP clients and the Broker via topics. Topics are created by the broker but the initial configuration can be proposed by a client, normally a publisher.

The broker is responsible for the store-and-forward of state update representations between CoAP clients. Subscribers observing a resource will receive notifications, the delivery of which is done on a best-effort basis.

~~~~~~~~~~~
      CoAP                     CoAP                      CoAP
     clients                  server                   clients
   ___________              __________    observe   ____________
  |           |  publish   |          | .--------- |            |
  | publisher | ---------> |          | |--------> | subscriber |
  |___________|            |          | '--------> |____________|
        .                  |          |                   .
        .                  |  broker  |                   .
   ___________             |          |   observe   ____________
  |           |  publish   |          | .--------- |            |
  | publisher | ---------> |          | |--------> | subscriber |
  |___________|            |__________| '--------> |____________|
~~~~~~~~~~~
{: #fig-arch title='Publish-subscribe architecture over CoAP' artwork-align="center"}

# Pub-Sub Topics {#topics}

The configuration side of a "publish/subscribe broker" consists of a collection of topics. These topics as well as the collection itself are exposed by a CoAP server as resources (see {{fig-topic}}).

<!-- Consider merging fig2 and fig3 in fig 2 and deleting the former-->
~~~~~~~~~~~
               ___
       Topic  /   \
  Collection  \___/
                  \
                   \____________________
                    \___    \___        \___
            Topics  /   \   /   \  ...  /   \
                    \___/   \___/       \___/
~~~~~~~~~~~
{: #fig-topic title='Resources of a Publish-Subscribe Broker' artwork-align="center"}

## Topic Creation and Configuration

Each topic has a topic configuration. A CoAP client can create a new
topic by submitting an initial configuration for the topic. It can
also read and update the configuration of existing topics and delete
them when they are no longer needed.

The configuration of a topic itself consists of a set of properties.
These fall into one of two categories: configuration properties and
status properties. Configuration properties can be set by a client
and describe the desired configuration of a topic.  Status properties
are read-only, managed by the server, and provide information about
the actual status of a topic.

When a client submits a configuration to create a new topic or update
an existing topic, it can only submit configuration properties. When
a server returns the configuration of a topic, it returns both the
configuration properties and the status properties of the topic.

Every property has a type and a value.  The type takes the form of an
IRI {{?RFC3987}}. This IRI serves only as an identifier; it must not be
dereferenced by clients. The value can be either a Boolean value, an integer, a floating-point number, a date/time value, a byte string, a text string, or a resource reference in the form of a URI {{!RFC3986}}.

### Configuration Properties

The following configuration properties are defined:
TODO.

### Status Properties

The following status properties are defined:
TODO.

### Topic Configuration Representation {#topic-configuration-representation}

A topic configuration is represented as a CoRAL document [I-D.ietf-core-coral] containing the configuration properties and status properties of the topic as top-level elements.

Each property is represented as a link where the link relation type is the property type and the link target is the property value.

<!-- TODO: Topic configuration discovery and representation are mimmicking the pattern shown in draft-ietf-ace-oscore-gm-admin-07 and draft-ietf-ace-key-groupcomm-16. I need to look at those and port them here. Along with their IANA rt and ct registrations. We won't use CoRAL for now but leave it open for it's use in the future.

TODO (check https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#name-retrieve-a-group-configurat)

content format: application/pubsub+cbor

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html

Make a section like 5.1 or 5.2 to describe a topic
https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#section-5.1-->



## Topic Discovery {#topic-discovery}

Topics can be discovered by a client on the basis of configuration properties and status properties. For example, a client could fetch a list of all topics that have a property of type "foo" or that have a property of type "bar" with the value 42. Alternatively, topics can also be discovered simply by getting the full list of all topics.

<!-- Add ref to broker discovery-->

### Topic List Representation {#topic-list-representation}

A list of group configurations is represented as a document containing the corresponding group-configuration resources in the list. Each group-configuration is represented as a link, where the link target is the URI of the group-configuration resource.

The list can be represented as a Link Format document {{?RFC6690}} or a CoRAL document {{?I-D.ietf-core-coral}}.

### Filter Query Representation {#filter-query-representation}

   TODO.

## Interactions

### Getting All Topics {#topic-get-all}

A client can list a collection of topics by making a GET request to the collection URI.

On success, the server returns a 2.05 (Content) response with a representation of the list of all topics (see Section {{topic-configuration-representation}}) in the collection.

Example:
~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: topics

<= 2.05 Content
   Content-Format: 65536
   item </pubsub/topics/1>
   item </pubsub/topics/2>
   item </pubsub/topics/3>
~~~~~~~~~~~

### Getting Topics by Properties {#topic-get-properties}

A client can filter a collection of topics by submitting the
representation of a topic filter (see Section {{filter-query-representation}}) in a FETCH
request to the topic collection URI.

On success, the server returns a 2.05 (Content) response with a
representation of a list of topics in the collection (see
Section {{topic-list-representation}}) that match the filter.

Example:
~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: pubsub
   Uri-Path: topics
   Content-Format: TODO
      TODO

<= 2.05 Content

   Content-Format: 65536

   item </pubsub/topics/1>
   item </pubsub/topics/2>
   item </pubsub/topics/3>
~~~~~~~~~~~

### Creating a Topic {#topic-create}

A client can add a new topic to a collection of topics by submitting a representation of the initial topic configuration (see Section {{topic-configuration-representation}}) in a POST request to the topic collection URI.

If client just wants all the default configuration properties, it can simply submit an empty CoRAL document.

On success, the server returns a 2.01 (Created) response indicating the topic URI of the new topic.

Example:
~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: pubsub
   Uri-Path: topics
   Content-Format: 65536

   foo "xyz"
   bar 42

<= 2.01 Created
   Location-Path: pubsub
   Location-Path: topics
   Location-Path: 1234
~~~~~~~~~~~

### Reading the Configuration of a Topic {#topic-read-configuration}

   A client can read the configuration of a topic by making a GET
   request to the topic URI.

   On success, the server returns a 2.05 (Content) response with a
   representation of the topic configuration (see Section 3.1.3).

   Example:
~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234

<= 2.05 Content
   Content-Format: 65536
   Max-Age: 300

   foo "xyz"
   bar 42
~~~~~~~~~~~

### Updating the Configuration of a Topic {#topic-update-configuration}

   A client can update the configuration of a topic by submitting the
   representation of the updated topic configuration (see Section 3.1.3)
   in a PUT request to the topic URI.  Any existing properties in the
   configuration are replaced by this update.

   On success, the server returns a 2.04 (Updated) response.

   Example:
~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234
   Content-Format: 65536

   foo "abc"

<= 2.04 Updated
~~~~~~~~~~~

### Deleting a Topic {#topic-delete}

   A client can delete a topic by making a DELETE request on the topic
   URI.

   On success, the server returns a 2.02 (Deleted) response.

   Any subscribers to the topic are automatically unsubscribed.

   Example:
~~~~~~~~~~~
=> 0.04 DELETE
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234

<= 2.02 Deleted
~~~~~~~~~~~

## Publish/Subscribe {#pubsub}

Unless a topic is configured to use a different mechanism, publish/ subscribe is performed as follows: A publisher publishes to a topic by submitting the data in a PUT request to a broker-managed "topic data resource".  This causes a change to the state of that resources. Any subscriber observing the resource {{!RFC7641}} at that time receives a notification about the change to the resource state. Observations are maintained and terminated as specified on{{!RFC7641}}.

~~~~~~~~~~~
               ___
       Topic  /   \
  Collection  \___/
                  \
                   \___________________________
                    \          \               \
                     \ ......   \ ......        \ ......
                    : \___  :  : \___  :       : \___  :
             Topic  : /   \ :  : /   \ :       : /   \ :
     Configuration  : \___/ :  : \___/ :       : \___/ :
                    :  _|_  :  :  _|_  :  ...  :  _|_  :
             Topic  : /   \ :  : /   \ :       : /   \ :
              Data  : \___/ :  : \___/ :       : \___/ :
                    :.......:  :.......:       :.......:
~~~~~~~~~~~
F{: #fig-config title='Resources of a Publish-Subscribe Broker' artwork-align="center"}

As shown in section {{topics}}, each topic contains two resources: topic configuration and topic data. In that section we explain the creation and configuration of the topic configuration resources. This section will explain the management of topic data resources.

A topic data resource does not exist until some initial data has been published to it.  Before initial data has been published, the topic data resource yields a 4.04 (Not Found) response. If such a "half created" topic is undesired, the creator of the topic can simply immediately publish some initial placeholder data to make the topic "fully created".

All URIs for configuration and data resources are broker-generated. There does not need to be any URI pattern dependence between the URI where the data exists and the URI of the topic configuration.  Topic configuration and data resources might even be hosted on different servers.

### Topic Lifecycle {#topic-lifecycle}

When a topic is newly created, it is first placed by the server into the HALF CREATED state (see {{fig-life}}).  In this state, a client can read and update the configuration of the topic and delete the topic. A publisher can publish to the topic data resource.  However, a subscriber cannot yet observe the topic data resource.

~~~~~~~~~~~
                HALF                       FULLY
              CREATED                     CREATED
                ___                         ___     Publish/
------------>  |   |  ------------------>  |   |  ------------.
    Create     |___|        Publish        |___|  <-----------'
                     \                   /          Subscribe
                | ^   \       ___       /   | ^
          Read/ | |    '-->  |   |  <--'    | | Read/
         Update | |  Delete  |___|  Delete  | | Update
                '-'                         '-'
                            DELETED
~~~~~~~~~~~
F{: #fig-life title='Lifecycle of a Topic' artwork-align="center"}

After a publisher publishes to the topic for the first time, the topic is placed into the FULLY CREATED state. In this state, a client can read and update the configuration of the topic and delete the topic; a publisher can publish to the topic data resource; and a subscriber can observe the topic data resource.

When a client deletes a topic, the topic is placed into the DELETED state and shortly after removed from the server. In this state, all subscribers are removed from the list of observers of the topic data resource and no further interactions with the topic are possible.

### Rate Limiting {#rate-limit}

The server hosting a data resource may have to handle a potentially very large number of publishers and subscribers at the same time. This means the server can easily become overwhelmed if it receives too many publications in a short period of time.

In this situation, if a client is sending publications too fast, the server SHOULD return a 4.29 (Too Many Requests) response {{!RFC8516}}.  As described in {{!RFC 8516}}, the Max-Age option {{!RFC7252}} in this response indicates the number of seconds after which the client may retry. The Broker MAY stop publishing messages from the client for the indicated time.

When a client receives a 4.29 (Too Many Requests) response, it MUST NOT send any new publication requests to the same topic data resource before the time indicated by the Max-Age option has passed.

## Interactions {#Interactions}

### Broker Discovery {#discover}

TODO
<!-- Add ref to topic discovery-->
<!-- This section explains Broker Discovery-->

### Publish {#publish}

A client can publish data to a topic by submitting it in a PUT request to the topic data URI. The topic data URI is indicated by the status property of type <http://coreapps.org/pubsub#data> in the topic configuration.  (Note: The topic data URI is not identical to the topic URI!)

<!-- TODO: URI examples-->

The data MUST be in the content format specified by the configuration
property of type <http://coreapps.org/pubsub#accept> in the topic
configuration.

On success, the server returns a 2.04 (Updated) response.  However, when data is published to the topic for the first time, the server may instead return a 2.01 (Created) response.

If the request does not have an acceptable content format, the server returns a 4.15 (Unsupported Content Format) response.

If the client is sending publications too fast, the server returns a
4.29 (Too Many Requests) response [RFC8516].

Example:

=> 0.03 PUT
   Uri-Path: pubsub
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Content-Format: 112

   [...SenML data...]

<= 2.04 Updated

### Subscribe {#subscribe}

A client can get the latest published data and subscribe to newly published data by observing the topic data URI with a GET request that includes the Observe option [RFC7641].

On success, the server returns 2.05 (Content) notifications with the data.

Example:

=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Observe: 0

<= 2.05 Content
   Content-Format: 112
   Observe: 10001
   Max-Age: 15

   [...SenML data...]

<= 2.05 Content
   Content-Format: 112
   Observe: 10002
   Max-Age: 15

   [...SenML data...]

<= 2.05 Content
   Content-Format: 112
   Observe: 10003
   Max-Age: 15

   [...SenML data...]

### Unsubscribe {#unsubscribe}

A client can unsubscribe simply by cancelling the observation as described in Section 3.6 of {{!RFC7641}}. The client MUST either use CoAP GET with Observe using an Observe parameter of 1 or send a CoAP Reset message in response to a notification.

<!--  do we want an example or is redundant? --> 

### Getting the Latest Published Data {#get-latest-data}

A client can just get the latest published data by making a GET request to the topic data URI.

On success, the server returns 2.05 (Content) response with the data.

Example:

=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: data
   Uri-Path: 6578616d706c65


<= 2.05 Content
   Content-Format: 112
   Max-Age: 15

   [...SenML data...]

# Security Considerations

<!-- TODO: we may take content from prev versions but we have to spend some more time on the implications of the topic-config -->
TODO.

# IANA Considerations {#iana}

This document registers one attribute value in the Resource Type (rt=) registry
established with {{!RFC6690}} and appends to the definition of one CoAP Response Code in the CoRE Parameters Registry.

<!-- TODO: Redo this section. Need to add the ct and rt similar to the ones below

https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#name-resource-types

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.1

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.2 -->

## Resource Type value 'core.ps'

* Attribute Value: core.ps

* Description: {{sec-rest-api}} of [[This document]]

* Reference: [[This document]]

* Notes: None

## Resource Type value 'core.ps.discover'

* Attribute Value: core.ps.discover

* Description: {{sec-rest-api}} of [[This document]]

* Reference: [[This document]]

* Notes: None

# Acknowledgements {#acks}

The current version of this document contains a substantial contribution by Klaus Hartke's proposal {{I-D.irtf-t2trg-coral-pubsub}}, which defines the topic resource model and structure as well as the topic lifecycle and interactions.

The authors would like to also thank Hannes Tschofenig, Zach Shelby, Mohit Sethi, Peter van der Stok, Tim Kellogg, Anders Eriksson, Goran Selander, Mikko Majanen, and Olaf Bergmann for their contributions and reviews.


<!-- Old text below here-->


The clients and Broker use topics to identify a particular resource or
object in a publish-subscribe system. Topics are conventionally formed
as a hierarchy, e.g. "/sensors/weather/barometer/pressure" or
"/EP-33543/sen/3303/0/5700".  The topics are hosted by a Broker and
all the clients using the Broker share the same namespace for
topics.

Every CoAP pub/sub topic has an associated link, consisting of a reference
path on the Broker using URI path {{!RFC3986}} construction and link
attributes {{!RFC6690}}. Every topic is associated with zero or more
stored representations and a content-format specified in the link. A
CoAP pub/sub topic value may alternatively consist of a collection of one or
more sub-topics, consisting of links to the sub-topic URIs and
indicated by a link-format content-format. Sub-topics are also topics and
may have their own sub-topics, forming a tree structure of unique paths that
is implemented using URIs. The full URI of a topic includes a scheme and
authority for the Broker, for example
"coaps://192.0.2.13:5684/EP-33543/sen/3303/0/5700".

A Topic may have a lifetime defined by using the CoAP Max-Age option on topic
creation, or on publish operations to the topic. The lifetime is refreshed each
time a representation is published to the topic. If the lifetime expires, the
topic is removed from discovery responses, returns errors on subscription, and
any outstanding subscriptions are cancelled.

# CoAP pub/sub REST API {#sec-rest-api}

This section defines the REST API exposed by a CoAP pub/sub Broker to pub/sub
Clients.  The examples throughout this section assume the use of CoAP
{{!RFC7252}}. A CoAP pub/sub Broker implementing this specification SHOULD
support the DISCOVERY, CREATE, PUBLISH, SUBSCRIBE, UNSUBSCRIBE, READ,
and REMOVE operations defined in this section. Optimized implementations
MAY support a subset of the operations as required by particular constrained
use cases.

## DISCOVERY {#discover}

CoAP pub/sub Clients discover CoAP pub/sub Brokers by using CoAP Simple
Discovery or through a Resource Directory (RD)
{{!RFC9167}}. A CoAP pub/sub Broker SHOULD
indicate its presence and availability on a network by exposing a link
to the entry point of its pub/sub API at its .well-known/core location {{!RFC6690}}. A CoAP
pub/sub Broker MAY register its pub/sub REST API entry point with a Resource
Directory. {{discover-fig}} shows an example of a client discovering a
local pub/sub API using CoAP Simple Discovery. A Broker wishing to
advertise the CoAP pub/sub API for Simple Discovery or through a
Resource Directory MUST use the link relation rt=core.ps. A Broker MAY
advertise its supported content formats and other attributes in the
link to its pub/sub API.

A CoAP pub/sub Broker MAY offer a topic discovery entry point to enable Clients
to find topics of interest, either by topic name or by link attributes
which may be registered when the topic is
created. {{discover-topic-fig}} shows an example of a client looking
for a topic with a resource type (rt) of "temperature" using
Discover. The client then receives the URI of the resource and its
content-format. A pub/sub Broker wishing to advertise topic discovery
MUST use the relation rt=core.ps.discover in the link.

A CoAP pub/sub Broker MAY provide topic discovery functionality through the
.well-known/core resource. Links to topics may be exposed at
.well-known/core in addition to links to the pub/sub
API. {{discover-topic-wk-fig}} shows an example of topic discovery
through .well-known/core.

Topics in the broker may be created in hierarchies (see {{sec-create}}) with
parent topics having sub-topics. For a discovery the broker may choose
to not expose the sub-topics in order to limit amount of topic links
sent in a discovery response. The client can then perform discovery
for the parent topics it wants to discover the sub-topics.

The DISCOVER interface is specified as follows:

Interaction:
: Client -> Broker

Method:
: GET

URI Template:

: {+ps}/{+topic}{?q\*}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

: q := Query Filter (optional). MAY contain a query filter list as per
 {{!RFC6690}} Section 4.1.

Content-Format:
: application/link-format

The following response codes are defined for the DISCOVER operation:

Success:
: 2.05 "Content" with an application/link-format payload containing
  one or more matching entries for the Broker resource. A pub/sub
  Broker SHOULD use the value "/ps/" for the base URI of the pub/sub
  API wherever possible.

Failure:
: 4.04 "Not Found" is returned in case no matching entry is found for
  a unicast request.

Failure:
: 4.00 "Bad Request" is returned in case of a malformed request for a unicast
  request.

Failure:
: No error response to a multicast request.

~~~~
Client                                          Broker
  |                                               |
  | ------ GET /.well-known/core?rt=core.ps ---->>|
  | -- Content-Format: application/link-format ---|
  |                                               |
  | <<--- 2.05 Content                            |
  | </ps/>;rt=core.ps;rt=core.ps.discover;ct=40 --|
  |                                               |

~~~~
{: #discover-fig title='Example of DISCOVER pub/sub function' artwork-align="center"}


~~~~
Client                                          Broker
  |                                               |
  | ---------- GET /ps/?rt="temperature" ------->>|
  |    Content-Format: application/link-format    |
  |                                               |
  | <<-- 2.05 Content                             |
  |   </ps/currentTemp>;rt="temperature";ct=50 ---|
  |                                               |

~~~~
{: #discover-topic-fig title='Example of DISCOVER topic' artwork-align="center"}


~~~~
Client                                          Broker
  |                                               |
  | -------- GET /.well-known/core?ct=50 ------->>|
  |    Content-Format: application/link-format    |
  |                                               |
  | <<-- 2.05 Content                             |
  |   </ps/currentTemp>;rt="temperature";ct=50 ---|
  |                                               |

~~~~
{: #discover-topic-wk-fig title='Example of DISCOVER topic' artwork-align="center"}

## CREATE {#sec-create}

A CoAP pub/sub broker SHOULD allow Clients to create new topics on the
broker using CREATE. A client wishing to create a topic MUST use a CoAP POST to the pub/sub API with a payload
indicating the desired topic. The topic specification sent in the
payload MUST use a supported serialization of the CoRE link format
{{!RFC6690}}. The target of the link MUST be a URI formatted
string. The client MUST indicate the desired content format for
publishes to the topic by using the ct (Content Format) link attribute
in the link-format payload. Additional link target attributes and relation
values MAY be included in the topic specification link when a topic is created.

The client MAY indicate the lifetime of the topic by including the Max-Age
option in the CREATE request.

Topic hierarchies can be created by creating parent topics. A parent
topic is created with a POST using ct (Content Format) link attribute
value which describes a supported serialization of the CoRE link
format {{!RFC6690}}, such as application/link-format (ct=40) or its
JSON or CBOR serializations.  If a topic is created which describes a
link serialization, that topic may then have sub-topics created under
it as shown in {{create-sub-fig}}.

Ony one level in the topic hierarchy may be created as a result of a CREATE
operation, unless create on PUBLISH is supported (see {{sec-publish}}).
The topic string used in the link target MUST NOT contain the "/" character.

A topic creator MUST include exactly one content format link attribute value (ct) in the create payload. If the content format option is not included or if the option is repeated, the Broker MUST reject the operation with an error code of "4.00 Bad Request".

Only one topic may be created per request. If there is more than one link
included in a CREATE request, the Broker MUST reject the operation with an
error code of "4.00 Bad Request".

A Broker MUST return a response code of "2.01 Created" if the topic is
created and MUST return the URI path of the created topic via Location-Path
options. If a new topic can not be created, the Broker MUST return the appropriate 4.xx response code indicating the reason for failure.

A Broker SHOULD remove topics if the Max-Age of the topic is
exceeded without any publishes to the topic.  A Broker SHOULD retain a
topic indefinitely if the Max-Age option is elided or is set to zero
upon topic creation. The lifetime of a topic MUST be refreshed upon
create operations with a target of an existing topic.

A topic creator SHOULD PUBLISH an initial value to a newly-created Topic in order to enable responses to READ and SUBSCRIBE requests that may be submitted after the topic is discoverable.

The CREATE interface is specified as follows:

Interaction:
: Client -> Broker

Method:
: POST

URI Template:
: {+ps}/{+topic}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

Content-Format:
: application/link-format

Payload:
: The desired topic to CREATE

The following response codes are defined for the CREATE operation:

Success:
: 2.01 "Created". Successful Creation of the topic

Failure:
: 4.00 "Bad Request". Malformed request.

Failure:
: 4.01 "Unauthorized". Authorization failure.

{{create-fig}} shows an example of a topic called "topic1" being
successfully created.


~~~~
Client                                          Broker
  |                                               |
  | ---------- POST /ps/ "<topic1>;ct=50" ------->|
  |                                               |
  | <---------------- 2.01 Created ---------------|
  |               Location: /ps/topic1            |
  |                                               |

~~~~
{: #create-fig title='Example of CREATE topic' artwork-align="center"}


~~~~
Client                                          Broker
  |                                               |
  | ----- POST /ps/ "<parent-topic>;ct=40" ------>|
  |                                               |
  | <---------------- 2.01 Created ---------------|
  |            Location: /ps/parent-topic/        |
  |                                               |
  |-- POST /ps/parent-topic/ "<subtopic>;ct=50" ->|
  |                                               |
  | <---------------- 2.01 Created ---------------|
  |       Location: /ps/parent-topic/subtopic     |
  |                                               |
  |                                               |

~~~~
{: #create-sub-fig title='Example of CREATE of topic hierarchy' artwork-align="center"}

## PUBLISH {#sec-publish}

A CoAP pub/sub Broker MAY allow clients to PUBLISH to topics on
the Broker. A client MAY use the PUT or the POST method to publish
state updates to the CoAP pub/sub Broker. A client MUST use the content
format specified upon creation of a given topic to publish updates to
that topic. The Broker MUST reject publish operations which do not use
the specified content format.  A CoAP client publishing on a topic MAY
indicate the maximum lifetime of the value by including the Max-Age
option in the publish request. The Broker MUST return a response code
of "2.04 Changed" if the publish is accepted.  A Broker MAY return a
"4.04 Not Found" if the topic does not exist. A Broker MAY return
"4.29 Too Many Requests" if simple flow control as described in
{{sec-flow-control}} is implemented.

A Broker MUST accept PUBLISH operations using the PUT method. PUBLISH
operations using the PUT method replace any stored representation
associated with the topic, with the supplied representation. A Broker
MAY reject, or delay responses to, PUT requests to a topic while
pending resolution of notifications to subscribers from previous PUT
requests.

Create on PUBLISH: A Broker MAY accept PUBLISH operations to new topics using
the PUT method. If a Broker accepts a PUBLISH using PUT to a topic that does
not exist, the Broker MUST create the topic using the information in the
PUT operation. The Broker MUST create a topic with the URI-Path of the request,
including all of the sub-topics necessary, and create a topic link with the
ct attribute set to the content-format value from the header of the PUT request.
If topic is created, the Broker MUST return the response "2.01 Created" with
the URI of the created topic, including all of the created path segments,
returned via the Location-Path option.

{{create-publish-fig}} shows an example of a topic being created on first
PUBLISH.

A Broker MAY accept PUBLISH operations using the POST method. If a
Broker accepts PUBLISH using POST it shall respond with the 2.04 Changed
status code. If an attempt is made to PUBLISH using POST to a topic that does
not exist, the Broker SHALL return a response status indicating resource not
found, such as HTTP 404 or CoAP 4.04.

A Broker MAY perform garbage collection of stored representations
which have been delivered to all subscribers or which have timed
out. A Broker MAY retain at least one most recently published
representation to return in response to SUBSCRIBE and READ requests.

A Broker MUST make a best-effort attempt to notify all clients
subscribed on a particular topic each time it receives a publish on
that topic. An example is shown in {{subscribe-fig}}.

If a client publishes to a Broker without the Max-Age option, the Broker MUST
refresh the topic lifetime with the most recently set Max-Age value, and the
Broker MUST include the most recently set Max-Age value in the Max-Age option of
all notifications.

If a client publishes to a Broker with the Max-Age option, the Broker MUST
include the same value for the Max-Age option in all notifications.

A Broker MUST use CoAP Notification as described in {{!RFC7641}} to notify
subscribed clients.

The PUBLISH operation is specified as follows:

Interaction:
: Client -> Broker

Method:
: PUT, POST

URI Template:
: {+ps}/{+topic}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

Content-Format:
: Any valid CoAP content format

Payload:
: Representation of the topic value (CoAP resource state representation) in
  the indicated content format

The following response codes are defined for the PUBLISH operation:

Success:
: 2.01 "Created". Successful publish, topic is created

Success:
: 2.04 "Changed". Successful publish, topic is updated

Failure:
: 4.00 "Bad Request". Malformed request.

Failure:
: 4.01 "Unauthorized". Authorization failure.

Failure:
: 4.04 "Not Found". Topic does not exist.

Failure:
: 4.29 "Too Many Requests". The client should slow down the rate of publish
  messages for this topic (see {{sec-flow-control}}).

{{publish-fig}} shows an example of a new value being successfully
published to the topic "topic1". See {{subscribe-fig}} for an example
of a Broker forwarding a message from a publishing client to a
subscribed client.


~~~~
Client                                          Broker
  |                                               |
  | ---------- PUT /ps/topic1 "1033.3"  --------> |
  |                                               |
  |                                               |
  | <--------------- 2.04 Changed---------------- |
  |                                               |

~~~~
{: #publish-fig title='Example of PUBLISH' artwork-align="center"}




~~~~
Client                                          Broker
  |                                               |
  | -------- PUT /ps/exa/mpl/e "1033.3"  -------> |
  |                                               |
  |                                               |
  | <--------------- 2.01 Created---------------- |
  |             Location: /ps/exa/mpl/e           |
  |                                               |

~~~~
{: #create-publish-fig title='Example of CREATE on PUBLISH' artwork-align="center"}


## SUBSCRIBE

A CoAP pub/sub Broker MAY allow Clients to subscribe to topics on the Broker
using CoAP Observe as described in {{!RFC7641}}. A CoAP pub/sub Client wishing
to Subscribe to a topic on a Broker MUST use a CoAP GET with the Observe
option set to 0 (zero). The Broker MAY add the client to a
list of observers. The Broker MUST return a response code of "2.05 Content"
along with the most recently published value if the topic contains a valid
value and the Broker can supply the requested content format. The Broker
MUST reject Subscribe requests on a topic if the content format of the request
is not the content format the topic was created with.

If the topic was published with the Max-Age option, the Broker MUST set the Max-Age option in the valid response to the amount of time remaining for the value to be valid since the last publish operation on that topic.

The Broker MUST return a response code "4.04 Not Found" if the topic does not
exist or has been removed, or if Max-Age of a previously published
representation has expired.

If a Topic has been created but not yet published to when a SUBSCRIBE to the
topic is received, the Broker MAY acknowledge and queue the pending SUBSCRIBE
and defer the response until an initial PUBLISH occurs.

The Broker MUST return a response code "4.15 Unsupported Content Format" if it
can not return the requested content format. If a Broker is unable to accept a
new Subscription on a topic, it SHOULD return the appropriate response code
without the Observe option as per {{!RFC7641}} Section 4.1.

There is no explicit maximum lifetime of a Subscription, thus a Broker may
remove subscribers at any time. The Broker, upon removing a Subscriber, will
transmit the appropriate response code without the Observe option, as per
{{!RFC7641}} Section 4.2, to the removed Subscriber.

The SUBSCRIBE operation is specified as follows:

Interaction:
: Client -> Broker

Method:
: GET

Options:
: Observe:0

URI Template:
: {+ps}/{+topic}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

The following response codes are defined for the SUBSCRIBE operation:

Success:
: 2.05 "Content". Successful subscribe, current value included

Failure:
: 4.00 "Bad Request". Malformed request.

Failure:
: 4.01 "Unauthorized". Authorization failure.

Failure:
: 4.04 "Not Found". Topic does not exist.

Failure:
: 4.15 "Unsupported Content Format". Unsupported content format.

{{subscribe-fig}} shows an example of Client2 subscribing to "topic1"
and receiving a response from the Broker, with a subsequent
notification. The subscribe response from the Broker uses the last
stored value associated with the topic1. The notification from the
Broker is sent in response to the publish received from Client1.


~~~~
Client1   Client2                                          Broker
  |          |                   Subscribe                   |
  |          | ----- GET /ps/topic1 Observe:0 Token:XX ----> |
  |          |                                               |
  |          | <---------- 2.05 Content Observe:10---------- |
  |          |                                               |
  |          |                                               |
  |          |                    Publish                    |
  | ---------|----------- PUT /ps/topic1 "1033.3"  --------> |
  |          |                    Notify                     |
  |          | <---------- 2.05 Content Observe:11 --------- |
  |          |                                               |

~~~~
{: #subscribe-fig title='Example of SUBSCRIBE' artwork-align="center"}


## UNSUBSCRIBE

If a CoAP pub/sub Broker allows clients to SUBSCRIBE to topics on the Broker,
it MUST allow Clients to unsubscribe from topics on the Broker using the CoAP
Cancel Observation operation. A CoAP pub/sub Client wishing to unsubscribe
to a topic on a Broker MUST either use CoAP GET with Observe using an Observe
parameter of 1 or send a CoAP Reset message in response to a publish, as
per {{!RFC7641}}.

The UNSUBSCRIBE operation is specified as follows:

Interaction:
: Client -> Broker

Method:
: GET

Options:
: Observe:1

URI Template:
: {+ps}/{+topic}{?q\*}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub
REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

: q := Query Filter (optional). MAY contain a query filter list as per
 {{!RFC6690}} Section 4.1.

The following response codes are defined for the UNSUBSCRIBE operation:

Success:
: 2.05 "Content". Successful unsubscribe, current value included

Success:
: 2.07 "No Content". Successful unsubscribe, value not included

Failure:
: 4.00 "Bad Request". Malformed request.

Failure:
: 4.01 "Unauthorized". Authorization failure.

Failure:
: 4.04 "Not Found". Topic does not exist.

{{unsubscribe-fig}} shows an example of a client unsubscribe using the
Observe=1 cancellation method.

~~~~
Client                                          Broker
  |                                               |
  | ----- GET /ps/topic1 Observe:1 Token:XX ----> |
  |                                               |
  | <------------- 2.05 Content ----------------- |
  |                                               |

~~~~
{: #unsubscribe-fig title='Example of UNSUBSCRIBE' artwork-align="center"}

## READ

A CoAP pub/sub Broker MAY accept Read requests on a topic using the the CoAP
GET method if the content format of the request matches the content format the
topic was created with. The Broker MUST return a response code of "2.05
Content" along with the most recently published value if the topic contains
a valid value and the Broker can supply the requested content format.

If the topic was published with the Max-Age option, the Broker MUST set the
Max-Age option in the valid response to the amount of time remaining for the
value to be valid since the last publish operation on that topic.

The Broker MUST return a response code "4.04 Not Found" if the topic does not
exist or has been removed, or if Max-Age of a previously published
representation has expired.

If a Topic has been created but not yet published to when a READ to the topic is
received, the Broker MAY acknowledge and queue the pending READ, and defer the
response until an initial PUBLISH occurs.

The Broker MUST return a response code "4.15 Unsupported Content Format" if the
Broker can not return the requested content format.

The READ operation is specified as follows:

Interaction:
: Client -> Broker

Method:
: GET

URI Template:
: {+ps}/{+topic}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub
REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

The following response codes are defined for the READ operation:

Success:
: 2.05 "Content". Successful READ, current value included

Failure:
: 4.00 "Bad Request". Malformed request.

Failure:
: 4.01 "Unauthorized". Authorization failure.

Failure:
: 4.04 "Not Found". Topic does not exist.

Failure:
: 4.15 "Unsupported Content Format". Unsupported content-format.

{{read-fig}} shows an example of a successful READ from topic1,
followed by a Publish on the topic, followed at some time later by a
read of the updated value from the recent Publish.


~~~~
Client1   Client2                                          Broker
  |          |                     Read                      |
  |          | --------------- GET /ps/topic1 -------------> |
  |          |                                               |
  |          | <---------- 2.05 Content "1007.1"------------ |
  |          |                                               |
  |          |                                               |
  |          |                    Publish                    |
  | ---------|----------- PUT /ps/topic1 "1033.3"  --------> |
  |          |                                               |
  |          |                                               |
  |          |                     Read                      |
  |          | --------------- GET /ps/topic1 -------------> |
  |          |                                               |
  |          | <----------- 2.05 Content "1033.3" ---------- |
  |          |                                               |

~~~~
{: #read-fig title='Example of READ' artwork-align="center"}


## REMOVE

A CoAP pub/sub Broker MAY allow clients to remove topics from the Broker
using the CoAP Delete
method on the URI of the topic. The CoAP pub/sub Broker MUST return
"2.02 Deleted" if the removal is successful. The Broker MUST
return the appropriate 4.xx response code indicating the reason for
failure if the topic can not be removed.

When a topic is removed for any reason, the Broker SHOULD remove all of the
observers from the list of observers and return a final 4.04 "Not Found"
response as per {{!RFC7641}} Section 3.2. If a topic which has sub-topics is
removed, then all of its sub-topics MUST be recursively removed.

The REMOVE operation is specified as follows:

Interaction:
: Client -> Broker

Method:
: DELETE

URI Template:
: {+ps}/{+topic}

URI Template Variables:
: ps := pub/sub REST API entry point (optional). The entry point of the pub/sub REST API, as obtained from discovery, used to discover topics.

: topic := The desired topic to return links for (optional).

Content-Format:
: None

Response Payload:
: None

The following response codes are defined for the REMOVE operation:

Success:
: 2.02 "Deleted". Successful remove

Failure:
: 4.00 "Bad Request". Malformed request.

Failure:
: 4.01 "Unauthorized". Authorization failure.

Failure:
: 4.04 "Not Found". Topic does not exist.

{{remove-fig}} shows a successful remove of topic1.



~~~~
Client                                         Broker
 |                                               |
 | ------------- DELETE /ps/topic1 ------------> |
 |                                               |
 |                                               |
 | <-------------- 2.02 Deleted ---------------- |
 |                                               |

~~~~
{: #remove-fig title='Example of REMOVE' artwork-align="center"}


# CoAP pub/sub Operation with Resource Directory

A CoAP pub/sub Broker may register the base URI, which is the REST API entry point for a pub/sub service, with a Resource
Directory. A pub/sub Client may use an RD to discover a pub/sub Broker.

A CoAP pub/sub Client may register links {{!RFC6690}} with a Resource
Directory to enable discovery of created pub/sub topics. A pub/sub
Client may use an RD to discover pub/sub Topics. A client which
registers pub/sub Topics with an RD MUST use the context relation (con)
{{!RFC9167}} to indicate that the context of
the registered links is the pub/sub Broker.

A CoAP pub/sub Broker may alternatively register links to its topics to
a Resource Directory by triggering the RD to retrieve it's links from
.well-known/core.  In order to use this method, the links must first
be exposed in the .well-known/core of the pub/sub Broker. See
{{discover}} in this document.

The pub/sub Broker triggers the RD to retrieve its links by sending a
POST with an empty payload to the .well-known/core of the Resource
Directory.  The RD server will then retrieve the links from the
.well-known/core of the pub/sub Broker and incorporate them into the
Resource Directory. See {{!RFC9167}} for
further details.

# Sleep-Wake Operation

CoAP pub/sub provides a way for client nodes to sleep between operations,
conserving energy during idle periods. This is made possible by shifting
the server role to the Broker, allowing the Broker to be always-on and respond
to requests from other clients while a particular client is sleeping.

For example, the Broker will retain the last state update received from a
sleeping client, in order to supply the most recent state update to other
clients in response to read and subscribe operations.

Likewise, the Broker will retain the last state update received on the topic
such that a sleeping client, upon waking, can perform a read operation to
the Broker to update its own state from the most recent system state update.

# Simple Flow Control {#sec-flow-control}

Since the Broker node has to potentially send a large amount of
notification messages for each publish message and it may be serving a
large amount of subscribers and publishers simultaneously, the Broker
may become overwhelmed if it receives many publish messages to popular
topics in a short period of time.

If the Broker is unable to serve a certain client that is sending publish
messages too fast, the Broker SHOULD respond with Response Code 4.29, "Too Many
Requests" {{!RFC8516}} and set the Max-Age Option to indicate the number of
seconds after which the client can retry. The Broker MAY stop creating
notifications from the publish messages from this client and to this topic for
the indicated time.

If a client receives the 4.29 Response Code from the Broker for a
publish message to a topic, it MUST NOT send new publish messages to
the Broker on the same topic before the time indicated in Max-Age has
passed.

# Security Considerations {#SecurityConsiderations}

CoAP pub/sub re-uses CoAP {{!RFC7252}}, CoRE Resource Directory
{{!RFC9167}}, and Web Linking {{?RFC5988}} and
therefore the security considerations of those documents also apply to
this specification. Additionally, a CoAP pub/sub Broker and the clients
SHOULD authenticate each other and enforce access control policies. A
malicious client could subscribe to data it is not authorized to or
mount a denial of service attack against the Broker by publishing a
large number of resources.  The authentication can be performed using
the already standardized DTLS offered mechanisms, such as
certificates. DTLS also allows communication security to be
established to ensure integrity and confidentiality protection of the
data exchanged between these relevant parties. Provisioning the
necessary credentials, trust anchors and authorization policies is
non-trivial and subject of ongoing work.

The use of a CoAP pub/sub Broker introduces challenges for the use of
end-to-end security between for example a client device on a sensor
network and a client application running in a cloud-based server
infrastructure since Brokers terminate the exchange. While running
separate DTLS sessions from the client device to the Broker and from
Broker to client application protects confidentially on those paths,
the client device does not know whether the commands coming from the
Broker are actually coming from the client application. Similarly, a
client application requesting data does not know whether the data
originated on the client device. For scenarios where end-to-end
security is desirable the use of application layer security is
unavoidable. Application layer security would then provide a guarantee
to the client device that any request originated at the client
application. Similarly, integrity protected sensor data from a client
device will also provide guarantee to the client application that the
data originated on the client device itself. The protected data can
also be verified by the intermediate Broker ensuring that it
stores/caches correct request/response and no malicious
messages/requests are accepted. The Broker would still be able to
perform aggregation of data/requests collected.

Depending on the level of trust users and system designers place in
the CoAP pub/sub Broker, the use of end-to-end object security is
RECOMMENDED as described in {{?I-D.ietf-ace-pubsub-profile}}.
An example application that uses the CoAP pub/sub Broker and relies on
end-to-end object security is described in {{?RFC8387}}.
When only end-to-end encryption  is necessary and the CoAP Broker is
trusted, Payload Only Protection (Mode:PAYL) could be used.
The Publisher would wrap only the  payload before sending it to
the Broker and set the option Content-Format to application/smpayl.
Upon receival, the Broker can read the unencrypted CoAP header
to forward it to the subscribers.

# IANA Considerations {#iana}

This document registers one attribute value in the Resource Type (rt=) registry
established with {{!RFC6690}} and appends to the definition of one CoAP Response Code in the CoRE Parameters Registry.

<!-- Need to add the ct and rt similar to the ones below

https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#name-resource-types

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.1

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.2 -->

## Resource Type value 'core.ps'

* Attribute Value: core.ps

* Description: {{sec-rest-api}} of [[This document]]

* Reference: [[This document]]

* Notes: None

## Resource Type value 'core.ps.discover'

* Attribute Value: core.ps.discover

* Description: {{sec-rest-api}} of [[This document]]

* Reference: [[This document]]

* Notes: None

# Acknowledgements {#acks}

The authors would like to thank Klaus Hartke, Hannes Tschofenig, Zach Shelby, Mohit Sethi,
Peter van der Stok, Tim Kellogg, Anders Eriksson, Goran Selander, Mikko Majanen,
and Olaf Bergmann for their contributions and reviews.

--- back

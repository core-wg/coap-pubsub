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
: An unique identifier for a particular item being published and/or subscribed to. A Broker uses the topics to match subscriptions to publications. A reference to a Topic on a Broker is a valid CoAP URI. Topics have to be created and configured before any data can be published. Clients may propose new topics to be created; however, it is up to the broker to choose if and how a topic is created. The broker also decides the URI of each topic. Topics are represented as a resource collection. The creation, configuration, and discovery of topics at a broker is specified in {{topics}}. Interactions about the topic data are in {{topic-data-interactions}}.

topic configuration:
: TODO

topic data:
: TODO

## CoAP Publish-Subscribe Architecture

<!-- Some major changes are the topic configuration resource and the topic management. Interactions about topic data are separated. Topic configuration resource is now fundamental. Moving away from URI templates (/topicconfig /topicdata), using resource types instead.   -->

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

This document describes two sets of interactions, interactions to configure topics and their lifecycle (see Section {{topic-configuration-interactions}})and interactions about the topic data (see Section {{topic-data-interactions}}).

Topic configuration interactions are discovery, create, read configuration, update configuration, delete configuration and handle the management of the topics.

Topic data interactions are publish, subscribe, unsubscribe, read and are oriented on how data is transferred from a publisher to a subscriber.

# Pub-Sub Topics {#topics}

The configuration side of a "publish/subscribe broker" consists of a collection of topics. These topics as well as the collection itself are exposed by a CoAP server as resources (see {{fig-topic}}).

<!-- TODO: Consider merging fig2 and fig3 in fig 2 and deleting the former-->
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

A topic configuration is represented as a CoRAL document {{?I-D.ietf-core-coral}} containing the configuration properties and status properties of the topic as top-level elements.

Each property is represented as a link where the link relation type is the property type and the link target is the property value.

<!-- TODO:

Contents of the topic configuration resource (which mandatory?):
- content format ct
- subscription lifetime
- additional link target attributes and relation values


Topic configuration discovery and representation are mimmicking the pattern shown in draft-ietf-ace-oscore-gm-admin-07 and draft-ietf-ace-key-groupcomm-16. I need to look at those and port them here. Along with their IANA rt and ct registrations. We won't use CoRAL for now but leave it open for it's use in the future.

TODO (check https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#name-retrieve-a-group-configurat)

content format: application/pubsub+cbor

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html

Make a section like 5.1 or 5.2 to describe a topic
https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#section-5.1-->

## Topic Discovery {#topic-discovery}

<!-- section needs more work -->

A Broker can offer a topic discovery entry point to enable clients to find topics of interest on the basis of configuration properties and status properties.

Topics can be discovered by a client on the basis of configuration properties and status properties. For example, a client could fetch a list of all topics that have a property of type "foo" or that have a property of type "bar" with the value 42. Alternatively, topics can also be discovered simply by getting the full list of all topics.

For broker discovery please see {{broker-discovery}}.

### Topic List Representation {#topic-list-representation}

A list of group configurations is represented as a document containing the corresponding group-configuration resources in the list. Each group-configuration is represented as a link, where the link target is the URI of the group-configuration resource.

The list can be represented as a Link Format document {{?RFC6690}} or a CoRAL document {{?I-D.ietf-core-coral}}.

### Filter Query Representation {#filter-query-representation}

   TODO.

## Topic Configuration Interactions {#topic-configuration-interactions}

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

The topic specification sent in the payload should use a supported serialization of the CoRE link format {{!RFC6690}} but other serializations like {{?I-D.ietf-core-coral}} may be used in the future.

On success, the server returns a 2.01 (Created) response indicating the topic URI of the new topic.

<!-- error cases -->

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

A client can read the configuration of a topic by making a GET request to the topic URI.

On success, the server returns a 2.05 (Content) response with a representation of the topic configuration (see Section 3.1.3).

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

A client can update the configuration of a topic by submitting the representation of the updated topic configuration (see Section 3.1.3) in a PUT request to the topic URI. Any existing properties in the configuration are replaced by this update.

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

A client can delete a topic by making a CoAP DELETE request on the topic URI.

On success, the server returns a 2.02 (Deleted) response.

When a topic is deleted, the broker SHOULD unsubscribe all subscribers by removing them from the list of observers and returning a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2.

<!-- TODO: Document error cases -->

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

All URIs for configuration and data resources are broker-generated. There does not need to be any URI pattern dependence between the URI where the data exists and the URI of the topic configuration. Topic configuration and data resources might even be hosted on different servers.

### Topic Lifecycle {#topic-lifecycle}

When a topic is newly created, it is first placed by the server into the HALF CREATED state (see {{fig-life}}).  In this state, a client can read and update the configuration of the topic and delete the topic. A publisher can publish to the topic data resource.  However, a subscriber cannot yet observe the topic data resource nor read the latest data.

~~~~~~~~~~~
                HALF                       FULLY
              CREATED                     CREATED
                ___                         ___     Publish
------------>  |   |  ------------------>  |   |  ------------.
    Create     |___|        Publish        |___|  <-----------'
                     \                   /         Subscribe
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

### Broker Discovery {#broker-discovery}

<!-- TODO: This section explains Broker Discovery, needs more work -->

Clients MAY discover brokers by using CoAP Simple Discovery or through a Resource Directory (RD) {{!RFC9167}}. Brokers MAY register with a RD by following the steps on Section 5 of {{!RFC9167}} with the link relation rt=core.ps.

<!-- do we use the `ps` uri template as entry point or not? -->

Brokers SHOULD expose a link to the entry point of the pubsub API at their .well-known/core location {{!RFC6690}}.

<!-- is example correct? -->

Example:
~~~~~~~~~~~
=> GET
   Uri-Path: ./well-known/core
   Resource-Type: core.ps


<= 2.05 Content
    </ps/>;rt=core.ps;rt=core.ps.discover;ct=40
~~~~~~~~~~~


For details topic discovery please see {{topic-discovery}}.

## Topic Data Interactions {#topic-data-interactions}

### Publish {#publish}

A topic must have been created in order to publish data to it (See Section {{topic-create}}) and be in the fully created state in order to the publish operation to work.

A client can publish data to a topic by submitting the data in a PUT request to the topic data URI. The topic data URI is indicated by the status property of type <http://coreapps.org/pubsub#data> in the topic configuration. Please note that the topic data URI is not the same as the topic URI.

<!-- change <http://coreapps.org/pubsub#data> TBD-data  once defined in the configuration section-->

The data MUST be in the content format specified by the configuration
property <http://coreapps.org/pubsub#accept> in the topic configuration. Brokers MUST reject publish operations which do not use the specified content format.

<!-- change <http://coreapps.org/pubsub#accept> TBD-accept once defined in the configuration section  -->

On success, the server returns a 2.04 (Updated) response.  However, when data is published to the topic for the first time, the server may instead return a 2.01 (Created) response.

If the request does not have an acceptable content format, the server returns a 4.15 (Unsupported Content Format) response.

If the client is sending publications too fast, the server returns a
4.29 (Too Many Requests) response {{!RFC8516}}.

<!-- TODO: Other error cases: max_age? -->

<!-- TODO: add senml payload example below -->
<!-- TODO: add participants (publisher and broker) here and in other diagrams-->

Example:
~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: pubsub
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Content-Format: 112

   [...SenML data...]

<= 2.01 Created


=> 0.03 PUT
   Uri-Path: pubsub
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Content-Format: 112

   [...updated SenML data...]

<= 2.04 Updated
~~~~~~~~~~~

### Subscribe {#subscribe}

A client can get the latest published data and subscribe to newly published data by observing the topic data URI with a GET request that includes the Observe option set to 0 {{!RFC7641}}.

On success, the broker MUST return 2.05 (Content) notifications with the data.

If the topic is not yet in the fully created state (see {{topic-lifecycle}}) the broker SHOULD return a response code 4.04 (Not Found).

<!-- TODO There are other potential error cases based on parameters from the  configuration file (subscription lifetime, topic content format...) -->

The following response codes are defined for the Subscribe operation:

<!-- TODO: Is this the best way to represent response codes? Are they needed?  Which other potential error cases exist based on parameters from the  configuration file (subscription lifetime, topic content format...)? -->

Success:
: 2.05 "Content". Successful subscribe, current value included
Failure:
: 4.04 "Not Found". Topic does not exist.

Example:
~~~~~~~~~~~
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
~~~~~~~~~~~

### Unsubscribe {#unsubscribe}

A client can unsubscribe simply by cancelling the observation as described in Section 3.6 of {{!RFC7641}}. The client MUST either use CoAP GET with Observe using an Observe parameter of 1 or send a CoAP Reset message in response to a notification.

<!--  do we want an example or is redundant? -->

### Read Latest Data {#read-data}

A client can get the latest published topic data by making a GET request to the topic data URI in the broker.

On success, the server MUST return 2.05 (Content) response with the data.

If the target URI does not match an existing resource or the topic is not in the fully created state (see {{topic-lifecycle}}), the broker SHOULD return a response code 4.04 (Not Found).

If the Broker can not return the requested content format it MUST return a response code 4.15 (Unsupported Content Format).

<!-- TODO There are other potential error cases we need to document -->

Example:
~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: data
   Uri-Path: 6578616d706c65


<= 2.05 Content
   Content-Format: 112
   Max-Age: 15

   [...SenML data...]
~~~~~~~~~~~

# URI Templates

<!-- TODO: Maybe we want a section of the uri templates that could be used but that are not mandatory (nor recommended) to use in any case and are there just for example illustration

Also put an example in which the topic configuration is hosted on one server and the topic data on another, to illustrate why discovery is always a must-->

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
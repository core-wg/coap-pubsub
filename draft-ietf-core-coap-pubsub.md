---
v: 3

docname: draft-ietf-core-coap-pubsub-12
cat: std
consensus: yes
stream: IETF
title: A publish-subscribe architecture for the Constrained Application Protocol (CoAP)
abbrev: COREPUBSUB
area: Applications
wg: CoRE
venue:
  mail: core@ietf.org
  github: core-wg/coap-pubsub

author:
- ins: M. Koster
  name: Michael Koster
  org: Unaffiliated
  email: michaeljohnkoster@gmail.com
- ins: A. Keranen
  name: Ari Keranen
  org: Ericsson
  email: ari.keranen@ericsson.com
- ins: J. Jimenez
  name: Jaime Jimenez
  org: Ericsson
  email: jaime@iki.fi

normative:
  RFC6570:
  RFC6690:
  RFC7252:
  RFC7641:
  RFC8516:
  RFC9167:
informative:
  RFC8288:
  I-D.hartke-t2trg-coral-pubsub:
  I-D.ietf-ace-oscore-gm-admin:

--- abstract

This document describes a publish-subscribe architecture for CoAP that
extends the capabilities of CoAP for supporting nodes with long breaks in
connectivity and/or up-time. The Constrained Application Protocol (CoAP) is used by CoAP clients both to publish and to subscribe via a known topic resource.

--- middle

# Introduction {#introduction}

The Constrained Application Protocol (CoAP) {{!RFC7252}} supports
machine-to-machine communication across networks of constrained
devices and constrained networks. CoAP uses a request/response model where clients make requests to servers in order to request actions on resources. Depending on the situation the same device may act either as a server, a client, or both.

One important class of constrained devices includes devices that are intended to run for years from a small battery, or by scavenging energy from their environment. These devices have limited reachability because they spend most of their time in a sleeping state with no network connectivity. Another important class of nodes are devices with limited reachability due to middle-boxes like Network Address Translators (NATs) and firewalls.

For these nodes, the client/server-oriented architecture of REST can be challenging when interactions are not initiated by the devices themselves. A publish/subscribe-oriented architecture where nodes are separated by a broker and data is exchanged via topics might fit these nodes better.

This document applies the idea of publish-subscribe to Constrained RESTful Environments. It introduces a broker that allows to create, discover subscribe and publish on topics. The broker enables store-and-forward data exchange between CoAP endpoints, thereby facilitating the communication of nodes with limited reachability, providing simple many-to-many communication, and easing integration with other publish/subscribe systems.

## Terminology {#terminology}

This specification requires readers to be familiar with all the terms and
concepts that are discussed in {{?RFC8288}} and {{!RFC6690}}. Readers
should also be familiar with the terms and concepts discussed in
{{!RFC7252}} and {{!RFC9167}}. The URI template
format {{!RFC6570}} is used to describe the REST API defined in
this specification.

This specification makes use of the following terminology:

publish-subscribe (pub/sub):
: A messaging paradigm in which messages are published to a broker, and potential receivers can subscribe to a broker to receive messages. Message producers do not need to know where the message will be eventually sent. The broker matches publications and subscriptions, and delivers publications to subscribed receivers.

publishers and subscribers:
: CoAP clients can act as publishers or as subscribers. Publishers propose topics for creation and send CoAP messages (publications) to the broker on specific topics. Subscribers have an ongoing observation relation (subscription) to a topic. Publishers and subscribers do not need to have any knowledge of each other, but they must know the topic they are publishing and subscribing to.

topic collection:
: A resource collection is a group of related resources that share a common base URI. In this case the the topic collection contains resources of the type "topic resource". CoAP clients can discover and interact with the resources in a collection by sending CoAP requests to the URI of the collection.

topic resource:
: An entry within a topic collection in a broker. Topics are created and configured before any data can be published.  CoAP clients can propose new topics to be created, but it is up to the broker to decide whether and how a topic is created. The broker also decides the URI of each topic resource and of the topic-data resource when hosted at the broker. The creation, configuration, and discovery of topics at a broker are specified in {{topics}}. Interactions about the topic-data are defined in {{topic-data-interactions}}.

topic-data resource:
: Topic resources contain a property called "topic-data". The topic-data resource is a CoAP URI used by publishers and subscribers to publish (PUT) and subscribe (GET with Observe) to data (see {{topics}}).

broker:
: A CoAP server that hosts one or more topic collections containing topic resources. The broker is responsible for the store-and-forward of state update representations when the topic-data URI points to a resource hosted on the broker. The broker is also responsible of handling the topic lifecycle as defined in {{topic-lifecycle}}. The creation, configuration, and discovery of topics at a broker is specified in {{topics}}.

{::boilerplate bcp14-tagged}

## CoAP Publish-Subscribe Architecture

{{fig-arch}} shows a simple Publish/Subscribe architecture over CoAP. In it, publishers submit their data over a RESTful interface to a broker-managed resource (topic) and subscribers observe this resource using {{?RFC7641}}. Resource state information is updated between the CoAP clients and the broker via topics. Topics are created by the broker but the initial configuration can be proposed by a client, normally a publisher.

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

This document describes two sets of interactions, interactions to configure topics and their lifecycle (see {{topic-resource-interactions}}) and interactions about the topic data (see {{topic-data-interactions}}).

Topic resource interactions are discovery, create, read configuration, update configuration, delete configuration and handle the management of the topics.

Topic data interactions are publish, subscribe, unsubscribe, read and are oriented on how data is transferred from a publisher to a subscriber.

## Managing Topics {#managing-topics}

{{fig-api}} shows the resources of a Topic Collection that can be managed at the Broker.

~~~~~~~~~~~
             ___
   Topic    /   \
 Collection \___/
  Resource       \
                  \____________________
                   \___    \___        \___
                   /   \   /   \  ...  /   \        Topic
                   \___/   \___/       \___/      Resources
~~~~~~~~~~~
{: #fig-api title="Resources of a Broker" artwork-align="center"}

The Broker exports a topic-collection resource, with resource type "core.ps.coll" defined in {{iana}} of this document. The interfaces for the topic-collection resource is defined in {{topic-collection-interactions}}.

# Pub-Sub Topics {#topics}

The configuration side of a "publish/subscribe broker" consists of a collection of topics. These topics as well as the collection itself are exposed by a CoAP server as resources (see {{fig-topic}}). Each topic has a topic and a topic data resources. The topic resource is used by a client creating or administering a topic. The topic data resource is used by the publishers and the subscribers to a topic.

~~~~~~~~~~~
              ___
    Topic    /   \
  Collection \___/
   Resource       \
                   \___________________________
                    \          \               \
                     \ ......   \ ......        \ ......
                    : \___  :  : \___  :       : \___  :
             Topic  : / + \ :  : / + \ :       : / + \ :
          Resource  : \_|_/ :  : \_|_/ :       : \_|_/ :
                    ....|....  ....|....       ....|....
                    ....|....  ....|....       ....|....
             Topic  :  _|_  :  :  _|_  :  ...  :  _|_  :
              Data  : /   \ :  : /   \ :       : /   \ :
          Resource  : \___/ :  : \___/ :       : \___/ :
                    :.......:  :.......:       :.......:
                   \_________/\_________/ ... \_________/
                     topic 1    topic 2         topic n
~~~~~~~~~~~
{: #fig-topic title='Topic and topic-data resources of a topic' artwork-align="center"}

## Collection Representation

Each topic resource is represented as a link, where the link target is the URI of the topic resource.

Each topic-data is represented as a link, where the link target is the URI of the topic-data resource. A topic-data link is an entry within the topic resource called 'topic_data' (see {{topic-properties}}).

The list can be represented as a Link Format document {{RFC6690}}. The link to each topic resource specifies the link target attribute 'rt' (Resource Type), with value "core.pubsub.conf" defined in this document.

## Topic Creation and Configuration

A CoAP client can create a new topic by submitting an initial configuration for the topic (see {{topic-create}}). It can also read and update the configuration of existing topics and delete them when they are no longer needed (see {{topic-resource-interactions}}).

The configuration of a topic itself consists of a set of properties that can be set by a client or by the broker.

### Topic Properties {#topic-properties}

The CBOR map includes the following configuration parameters, whose CBOR abbreviations are defined in {{pubsub-parameters}} of this document.

* 'topic_name': A required field used as an application identifier. It encodes the topic name as a CBOR text string. Examples of topic names include human-readable strings (e.g., "room2"), UUIDs, or other values.

* 'topic_data': A required field containing the CoAP URI of the topic data resource for subscribing to a pubsub topic. It encodes the URI as a CBOR text string.

* 'resource_type': A required field used to indicate the resource type associated with topic resources. It encodes the resource type as a CBOR text string. The value should be "core.ps.conf".

* 'media_type': An optional field used to indicate the media type of the topic data resource. It encodes the media type as a CBOR text string (e.g.,"application/json").

* 'target_attribute': An optional field used to indicate the attribute or property of the topic_data resource. It encodes the attribute as a CBOR text string. Example attributes include "temperature".

* 'expiration_date': An optional field used to indicate the expiration date of the topic. It encodes the expiration date as a CBOR text string. The value should be a date string in ISO 8601 format (e.g., "2023-03-31T23:59:59Z"). The pubsub system can use this field to automatically remove topics that are no longer valid.

* 'max_subscribers': An optional field used to indicate the maximum number of subscribers allowed for the topic. It encodes the maximum number as a CBOR integer. If this field is not present, there is no limit to the number of subscribers allowed. The pubsub system can use this field to limit the number of subscribers for a topic.

### Topic Resource Representation {#topic-resource-representation}

A topic is represented as a CBOR map containing the configuration properties of the topic as top-level elements.

Unless specified otherwise, these are defined in this document and their CBOR abbreviations are defined in {{pubsub-parameters}}.

#### Default Values

Below are the defined default values for the topic parameters:

* 'topic_name': There is no default value. This field is required and must be specified by the client or broker.

* 'topic_data': There is no default value. This field is required and must be specified by the client or broker.

* 'resource_type': The default value is "core.ps.conf".

* 'media_type': The default value is an empty string, indicating that no media type is specified.

* 'target_attribute': The default value is an empty string, indicating that no attribute is specified.

* 'expiration_date': The default value is an empty string, indicating that no expiration date is specified. If this field is not present, the topic will not expire automatically.

* 'max_subscribers': The default value is -1, indicating that there is no limit to the number of subscribers allowed. If this field is not present, the pubsub system will not limit the number of subscribers for the topic.

## Discovery

Discovery involves that of the Broker, topic collections, topic resources and topic data.

### Broker Discovery {#broker-discovery}

<!-- TBD: This section explains Broker Discovery, needs more work -->

CoAP clients MAY discover brokers by using CoAP Simple Discovery, via multicast, through a Resource Directory (RD) {{!RFC9167}} or by other means specified in extensions to {{!RFC7252}}. Brokers MAY register with a RD by following the steps on Section 5 of {{!RFC9167}} with the resource type set to "core.ps" as defined in {{iana}} of this document.

Brokers SHOULD expose a link to the entry point of the pubsub API at their .well-known/core location {{!RFC6690}}. The specific resource path is left for implementations, examples in this document may use the "/ps" path.

Example:

~~~~~~~~~~~
=> GET
   Uri-Path: ./well-known/core
   Resource-Type: core.ps


<= 2.05 Content
    </ps>;rt=core.ps;ct=40
~~~~~~~~~~~

### Topic Discovery {#topic-discovery}

A Broker can offer a topic discovery entry point to enable clients to find topics of interest. The resource entry point thus represents a collection of related resources as specified in {{?RFC6690}} and is identified by the resource type "core.ps.coll".

The interactions with topic collections are further defined in {{topic-collection-interactions}}.

A topic collection is a group of topic resources that define the properties of the topics themselves (see Section {{topic-resource-representation}}). Each topic resource is represented as a link to its corresponding resource URI. The list can be represented as a Link Format document {{?RFC6690}}. Topic resources are identified by the resource type "core.ps.conf".

Within each topic resource there is a set of configuration properties (see Section {{topic-properties}}). The 'topic_data' property contains the URI of the topic data resource that a CoAP client can subscribe to. Resources exposing resources of the topic data type are expected to use the resource type 'core.ps.data'.

## Topic Collection Interactions {#topic-collection-interactions}

These are the interactions that can happen at the topic collection level.

### Retrieving all topics {#topic-get-all}
<!--
GET to /topic-collection
retrieve all topics
response is link format
-->

A client can request a collection of the topics present in the broker by making a GET request to the collection URI.

On success, the server returns a 2.05 (Content) response with a representation of the list of all topic resources (see Section {{topic-resource-representation}}) in the collection.

Depending on the permission set each client MAY receive a different list of topics that they are authorized to read.

Example:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: tc

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   </ps/tc/temperature>;rt="core.ps.conf",
   </ps/tc/humidity>;rt="core.ps.conf"
~~~~~~~~~~~

### Getting Topics by Properties {#topic-get-properties}
<!--
FETCH to /topic-collection with filter
retrieve only the topics that match the filter
request is cbor
response is link format
-->

A client can filter a collection of topics by submitting the
representation of a topic filter (see Section {{topic-fetch-resource}})  in a FETCH request to the topic collection URI.

On success, the server returns a 2.05 (Content) response with a
representation of a list of topics in the collection (see
Section {{topic-discovery}}) that match the filter in CoRE link format {{!RFC6690}}.

Example:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: ps
   Uri-Path: tc
   Content-Format: TBD (application/pubsub+cbor)

   {
     "resource_type" : "core.ps.conf",
     "target_attribute" : "temperature"
   }

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   </living_room_sensor>;anchor="coap://[2001:db8::2]/ps/tc/h9392",
   </kitchen_sensor>;anchor="coap://[2001:db8::2]/ps/tc/f3192"

~~~~~~~~~~~

### Creating a Topic {#topic-create}
<!--
POST to /topic-collection
create new topic
request is cbor
response (created) is cbor including the link to new topic-config resource
creator proposes topic name but broker approves
-->

A client can add a new topic to a collection of topics by submitting a representation of the initial topic resource (see Section {{topic-resource-representation}}) in a POST request to the topic collection URI.

On success, the server returns a 2.01 (Created) response indicating the topic URI of the new topic and the current representation of the topic resource.

If a topic manager is present in the broker, the topic creation  may require manager approval subject to certain conditions. If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The broker MUST respond with a 4.00 (Bad Request) error if any received parameter is specified multiple times, invalid or not recognized.

A CoAP endpoint creating a topic may specify a 'topic_data' URI different than that used by the broker. The broker may then simply forward the observation requests to the 'topic_data' URI.

If the 'topic_data' is empty the broker will assign a resource for a publisher to publish to.

~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: ps
   Uri-Path: tc
   Content-Format: TBD2 (application/core-pubsub+cbor)
   TBD (this should be a CBOR map with the mandatory parameters)
   {
     "topic_name" : "living_room_sensor"
     "resource_type" : "core.ps.conf"
   }

<= 2.01 Created
   Location-Path: ps/h9392
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
     "topic_name" : "living_room_sensor",
     "topic_data" : "coap://[2001:db8::2]/ps/data/6578616d706c65"
     "resource_type" : "core.ps.conf"
   }
~~~~~~~~~~~

## Topic Resource Interactions {#topic-resource-interactions}

These are the interactions that can happen at the topic resource level.

### Getting a topic resource  {#topic-get-resource}

<!--
GET to /topic-config
retrieve a topic configuration
response is cbor
-->

A client can read the configuration of a topic by making a GET request to the topic resource URI.

On success, the server returns a 2.05 (Content) response with a representation of the topic resource. The response has as payload the representation of the topic resource as specified in {{topic-resource-representation}}.

If a topic manager (TBD) is present in the broker, retrieving topic information may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The response payload is a CBOR map, whose possible entries are specified in {{topic-resource-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

Example:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: tc
   Uri-Path: h9392

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
      "topic_name" : "living_room_sensor",
      "topic_data" : "coap://[2001:db8::2]/ps/data/6578616d706c65",
      "resource_type": "core.ps.conf",
      "media_type": "application/senml-cbor",
      "target_attribute": "temperature",
      "expiration_date": "",
      "max_subscribers": -1
   }

~~~~~~~~~~~

### Getting part of a topic resource by filter {#topic-fetch-resource}
<!--
FETCH to /topic-conf with filter
retrieve only certain parameters from the configuration
request is cbor
response is cbor
-->

A client can read the configuration of a topic by making a FETCH request to the topic resource URI with a filter for specific parameters. This is done in order to retrieve part of the current topic resource.

The request contains a CBOR map with a configuration filter or 'conf_filter', a CBOR array with CBOR abbreviation. Each element of the array specifies one requested configuration parameter of the current topic resource (see {{topic-resource-representation}}).

On success, the server returns a 2.05 (Content) response with a representation of the topic resource. The response has as payload the partial representation of the topic resource as specified in {{topic-resource-representation}}.

If a topic manager (TBD) is present in the broker, retrieving topic information may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error.

The response payload is a CBOR map, whose possible entries are specified in {{topic-resource-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

Both request and response MUST have Content-Format set to "application/core-pubsub+cbor".

Example:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: ps
   Uri-Path: tc
   Uri-Path: h9392
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
     "conf_filter" : [topic_data, media_type]
   }

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   {
     "topic_data" : "coap://[2001:db8::2]/ps/data/6578616d706c65",
     "media_type": "application/senml-cbor"
   }

~~~~~~~~~~~

### Updating the Topic Resource {#topic-update-resource}

<!--
PUT to /topic-conf
override the whole configuration
request is cbor
response is cbor
-->

A client can update the configuration of a topic by submitting the representation of the updated topic  (see Section 3.1.3) in a PUT or POST request to the topic URI. Any existing properties in the configuration are overwritten by this update.

On success, the server returns a 2.04 (Changed) response and the current full resource representation. The broker may chose not to overwrite parameters that are not explicitly modified in the request.

Example:

~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: ps
   Uri-Path: tc
   Uri-Path: h9392
   Content-Format: TBD2 (application/core-pubsub+cbor)

   {
      "topic_name" : "living_room_sensor",
      "topic_data" : "coap://[2001:db8::2]/ps/data/6578616d706c65",
      "target_attribute": "temperature",
      "expiration_date": "2023-04-28T23:59:59Z",
      "max_subscribers": 2
   }

<= 2.04 Changed
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
      "topic_name" : "living_room_sensor",
      "topic_data" : "coap://[2001:db8::2]/ps/data/6578616d706c65",
      "resource_type": "core.ps.conf",
      "media_type": "application/senml-cbor",
      "target_attribute": "temperature",
      "expiration_date": "2023-04-28T23:59:59Z",
      "max_subscribers": 2
   }
~~~~~~~~~~~

### Deleting a Topic Resource {#topic-delete}

A client can delete a topic by making a CoAP DELETE request on the topic resource URI.

On success, the server returns a 2.02 (Deleted) response.

When a topic resource is deleted, the broker SHOULD also delete the topic data resource, unsubscribe all subscribers by removing them from the list of observers and returning a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2.
Example:

~~~~~~~~~~~
=> 0.04 DELETE
   Uri-Path: ps
   Uri-Path: tc
   Uri-Path: h9392

<= 2.02 Deleted
~~~~~~~~~~~

## Publish/Subscribe {#pubsub}

Unless a topic is configured to use a different mechanism, publish/ subscribe is performed as follows: A publisher publishes to a topic by submitting the data in a PUT request to a broker-managed "topic data resource".  This causes a change to the state of that resources. Any subscriber observing the resource {{!RFC7641}} at that time receives a notification about the change to the resource state. Observations are maintained and terminated as specified in {{!RFC7641}}.

As shown in {{topics}}, each topic contains two resources: topic resource and topic data. In that section we explained the creation and configuration of the topic resources. This section will explain the management of topic data resources.

A topic data resource does not exist until some initial data has been published to it.  Before initial data has been published, the topic data resource yields a 4.04 (Not Found) response. If such a "half created" topic is undesired, the creator of the topic can simply immediately publish some initial placeholder data to make the topic "fully created".

URIs for topic resources are broker-generated. URIs for topic-data MAY also be broker-generated or client-generated. There does not need to be any URI pattern dependence between the URI where the data exists and the URI of the topic resource. Topic resource and data resources might even be hosted on different servers.

### Topic Lifecycle {#topic-lifecycle}

When a topic is newly created, it is first placed by the server into the HALF CREATED state (see {{fig-life}}).  In this state, a client can read and update the configuration of the topic and delete the topic. A publisher can publish to the topic data resource.  However, a subscriber cannot yet observe the topic data resource nor read the latest data.

~~~~~~~~~~~
                HALF                       FULLY
              CREATED       Delete        CREATED
                ___       Topic Data        ___     Publish
------------>  |   |  <------------------  |   |  ------------.
    Create     |___|  ------------------>  |___|  <-----------'
                     \      Publish      /         Subscribe
                | ^   \       ___       /   | ^
          Read/ | |    '-->  |   |  <--'    | | Read/
         Update | |  Delete  |___|  Delete  | | Update
                '-'  Topic          Topic   '-'
                            DELETED
~~~~~~~~~~~
{: #fig-life title='Lifecycle of a Topic' artwork-align="center"}

After a publisher publishes to the topic for the first time, the topic is placed into the FULLY CREATED state. In this state, a client can read, update and delete the topic; a publisher can publish to the topic data resource; and a subscriber can observe the topic data resource.

When a client deletes a topic resource, the topic is placed into the DELETED state and shortly after removed from the server. In this state, all subscribers are removed from the list of observers of the topic data resource and no further interactions with the topic are possible.

When a client deletes a topic data, the topic is placed into the HALF CREATED state, where clients can read, update and delete the topic and awaits for a publisher to begin publication.

### Rate Limiting {#rate-limit}

The server hosting a data resource may have to handle a potentially very large number of publishers and subscribers at the same time. This means the server can easily become overwhelmed if it receives too many publications in a short period of time.

In this situation, if a client is sending publications too fast, the server SHOULD return a 4.29 (Too Many Requests) response {{!RFC8516}}.  As described in {{!RFC8516}}, the Max-Age option {{!RFC7252}} in this response indicates the number of seconds after which the client may retry. The Broker MAY stop publishing messages from the client for the indicated time.

When a client receives a 4.29 (Too Many Requests) response, it MUST NOT send any new publication requests to the same topic data resource before the time indicated by the Max-Age option has passed.

## Topic Data Interactions {#topic-data-interactions}

TBD: intro and image that shows a topic data URI hosted in a different endpoint than the broker

### Publish {#publish}

A topic must have been created in order to publish data to it (See Section {{topic-create}}) and be in the half-created state in order to the publish operation to work (see {{topic-lifecycle}}).

A client can publish data to a topic by submitting the data in a PUT request to the 'topic_data' URI as indicated in its topic resource property. Please note that the 'topic_data' URI is not the same as the topic URI used for configuring the topic (see {{topic-resource-representation}}).

On success, the server returns a 2.04 (Updated) response. However, when data is published to the topic for the first time, the server instead MUST return a 2.01 (Created) response and set the topic in the fully-created state (see {{topic-lifecycle}}).

If the request does not have an acceptable content format, the server returns a 4.15 (Unsupported Content Format) response.

If the client is sending publications too fast, the server returns a
4.29 (Too Many Requests) response {{!RFC8516}}.

Example of first publication:

~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Content-Format: 110

   {
      "n": "temperature",
      "u": "Cel",
      "t": 1621452122,
      "v": 23.5
   }

<= 2.01 Created
~~~~~~~~~~~

Example of subsequent publication:

~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Content-Format: 110

   {
      "n": "temperature",
      "u": "Cel",
      "t": 182734122,
      "v": 22.5
   }

<= 2.04 Updated
~~~~~~~~~~~

### Subscribe {#subscribe}

A client can subscribe to a topic by sending a CoAP GET request with the Observe set to '0' to subscribe to resource updates. {{!RFC7641}}.

On success, the broker MUST return 2.05 (Content) notifications with the data.

If the topic is not yet in the fully created state (see {{topic-lifecycle}}) the broker SHOULD return a response code 4.04 (Not Found).

The following response codes are defined for the Subscribe operation:

Success:
: 2.05 "Content". Successful subscribe with observe response, current value included in the response.

Failure:
: 4.04 "Not Found". Topic does not exist.

TBD: Do we want to treat max_clients as an error?

If the 'max_clients' parameter has been reached, the server must treat that as specified in section 4.1 of {{!RFC7641}}. The response MUST NOT include an Observe Option, the absence of which signals to the subscriber that the subscription failed.

Example:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 6578616d706c65
   Observe: 0

<= 2.05 Content
   Content-Format: 110
   Observe: 10001
   Max-Age: 15

   [...SenML data...]

<= 2.05 Content
   Content-Format: 110
   Observe: 10002
   Max-Age: 15

   [...SenML data...]

<= 2.05 Content
   Content-Format: 110
   Observe: 10003
   Max-Age: 15

   [...SenML data...]
~~~~~~~~~~~

### Unsubscribe {#unsubscribe}

A CoAP client can unsubscribe simply by cancelling the observation as described in Section 3.6 of {{!RFC7641}}. The client MUST either use CoAP GET with the Observe Option set to 1 or send a CoAP Reset message in response to a notification.

### Delete topic data resource {#delete-topic-data}

A publisher MAY delete a topic by making a CoAP DELETE request on the 'topic_data' URI.

On success, the server returns a 2.02 (Deleted) response.

When a topic_data resource is deleted, the broker SHOULD also delete the 'topic_data' parameter in the topic resource, unsubscribe all subscribers by removing them from the list of observers and return a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2. The topic is then set back to the half created state as per {{topic-lifecycle}}.

Example:

~~~~~~~~~~~
=> 0.04 DELETE
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 6578616d706c65

<= 2.02 Deleted
~~~~~~~~~~~

### Read latest data {#read-data}

A client can get the latest published topic data by making a GET request to the 'topic_data' URI in the broker. Please note that discovery of the 'topic_data' parameter is a required previous step (see {{topic-get-resource}}).

On success, the server MUST return 2.05 (Content) response with the data.

If the target URI does not match an existing resource or the topic is not in the fully created state (see {{topic-lifecycle}}), the broker SHOULD return a response code 4.04 (Not Found).

If the broker can not return the requested content format it MUST return a response code 4.15 (Unsupported Content Format).

Example:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: ps
   Uri-Path: data
   Uri-Path: 6578616d706c65

<= 2.05 Content
   Content-Format: 110
   Max-Age: 15

   {
      "n": "temperature",
      "u": "Cel",
      "t": 1621452122,
      "v": 23.5
   }
~~~~~~~~~~~

# CoAP Pubsub Parameters {#pubsub-parameters}

This document defines parameters used in the messages exchanged between a client and the broker during the topic creation and configuration process (see {{topic-resource-representation}}). The table below summarizes them and specifies the CBOR key to use instead of the full descriptive name.

Note that the media type application/core-pubsub+cbor MUST be used when these parameters are transported in the respective message fields.

~~~~~~~~~~~
+-----------------+-----------+--------------+------------+
| Name            | CBOR Key  | CBOR Type    | Reference  |
|-----------------|-----------|--------------|------------|
| topic_name      | TBD1      | tstr         | [RFC-XXXX] |
| topic_data      | TBD2      | tstr         | [RFC-XXXX] |
| resource_type   | TBD3      | tstr         | [RFC-XXXX] |
| media_type      | TBD4      | tstr (opt)   | [RFC-XXXX] |
| target_attribute| TBD5      | tstr (opt)   | [RFC-XXXX] |
| expiration_date | TBD6      | tstr (opt)   | [RFC-XXXX] |
| max_subscribers | TBD7      | uint (opt)   | [RFC-XXXX] |
+-----------------+-----------+--------------+------------+
~~~~~~~~~~~
{: #fig-CoAP-Pubsub-Parameters title="CoAP Pubsub Parameters" artwork-align="center"}

# Security Considerations

<!-- TBD: we may take content from prev versions but we have to spend some more time on the implications of the topic-config -->
TBD.

# IANA Considerations {#iana}

TBD.

This document registers one attribute value in the Resource Type (rt=) registry
established with {{!RFC6690}} and appends to the definition of one CoAP Response Code in the CoRE Parameters Registry.

<!-- TBD: Redo this section. Need to add the ct and rt similar to the ones below

https://www.ietf.org/archive/id/draft-ietf-ace-oscore-gm-admin-07.html#name-resource-types

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.1

https://www.ietf.org/archive/id/draft-ietf-ace-key-groupcomm-16.html#section-11.2 -->

## Resource Type value 'core.ps'

* Attribute Value: core.ps

* Description: XXX of This document

* Reference: This document

* Notes: None

## Resource Type value 'core.ps.coll'

* Attribute Value: core.ps.coll

* Description: XXX of This document

* Reference: This document

* Notes: None

## Resource Type value 'core.ps.conf'

* Attribute Value: core.ps.conf

* Description: XXX of This document

* Reference: This document

* Notes: None

## Resource Type value 'core.ps.data'

* Attribute Value: core.ps.data

* Description: XXX of This document

* Reference: This document

* Notes: None

# Acknowledgements

The current version of this document contains a substantial contribution by Klaus Hartke's proposal {{I-D.hartke-t2trg-coral-pubsub}}, which defines the topic resource model and structure as well as the topic lifecycle and interactions. It also follows a similar architectural design as that provided by Marco Tiloca's {{I-D.ietf-ace-oscore-gm-admin}}.

The authors would like to also thank Carsten Bormann, Hannes Tschofenig, Zach Shelby, Mohit Sethi, Peter van der Stok, Tim Kellogg, Anders Eriksson, Goran Selander, Mikko Majanen, and Olaf Bergmann for their valuable contributions and reviews.

---
title: "A publish-subscribe architecture for the Constrained Application Protocol (CoAP)"
abbrev: "A publish-subscribe architecture for CoAP"
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
  organization: Unaffiliated
  email: michaeljohnkoster@gmail.com

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
  I-D.hartke-t2trg-coral-pubsub:
  I-D.ietf-ace-oscore-gm-admin:

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
devices and constrained networks {{!RFC7228}}. CoAP uses a request/response model where clients make requests to servers in order to request actions on resources. Depending on the situation the same device may act either as a server, a client, or both.

One important class of constrained devices includes devices that are intended to run for years from a small battery, or by scavenging energy from their environment. These devices have limited reachability because they spend most of their time in a sleeping state with no network connectivity. Another important class of nodes are devices with limited reachability due to middle-boxes like Network Address Translators (NATs) and firewalls.

For these nodes, the client/server-oriented architecture of REST can be challenging when interactions are not initiated by the devices themselves. A publish/subscribe-oriented architecture where nodes are separated by a broker and data is exchanged via topics might fit these nodes better.

This document applies the idea of a "Publish/Subscribe Broker" to Constrained RESTful Environments.  The broker enables store-and- forward data exchange between nodes, thereby facilitating the communication of nodes with limited reachability, providing simple many-to-many communication, and easing integration with other publish/subscribe systems.

<!-- TBD once concluded the main drafting, verify every single example, specially the part of the topic configuration representation and the cbor-diag-->

## Requirements Language

{::boilerplate bcp14-tagged}

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
: A CoAP server capable of receiving messages (publications) from CoAP clients acting as publishers and forwarding them to CoAP clients acting as subscribers.

CoAP client (publisher or subscriber):
: CoAP clients can act as publishers or as subscribers. Publishers send CoAP messages to the Broker on specific topics. Subscribers have an ongoing observation relation (subscription) to a topic. Neither publishers nor subscribers need to have any knowledge of each other; they just have to share the topic they are publishing and subscribing to.

Broker:
: In this specification a broker is a CoAP server that is hosting one or more topic collections. A Broker uses the topics to match subscriptions to publications. The broker is responsible for the store-and-forward of state update representations between CoAP clients acting as subscribers and those CoAP clients acting as publishers.

topic:
: A resource within a topic collection in a broker. An unique identifier for a particular item being published and/or subscribed to. A reference to a topic on a Broker is a valid CoAP URI. Topics are created and configured before any data can be published. Clients may propose new topics to be created; however, it is up to the broker to choose if and how a topic is created. The broker also decides the URI of each topic. Topics are represented as a resource collection. The creation, configuration, and discovery of topics at a broker is specified in {{topics}}. Interactions about the topic data are in {{topic-data-interactions}}.

topic configuration:
: Every topic is composed of a topic configuration resource which is used by a client to create, configure, update and delete the configuration of topics (see {#topics}).

topic data:
: topic configurations may contain a property called topic data. The topic data resource is used by publishers and subscribers to publish (POST) and subscribe (GET with Observe) to data (see {#topics}).

## CoAP Publish-Subscribe Architecture

{{fig-arch}} shows a simple Publish/Subscribe architecture over CoAP. In it, publishers submit their data over a RESTful interface to a broker-managed resource (topic) and subscribers observe this resource using {{?RFC7641}}. Resource state information is updated between the CoAP clients and the Broker via topics. Topics are created by the broker but the initial configuration can be proposed by a client, normally a publisher.

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

This document describes two sets of interactions, interactions to configure topics and their lifecycle (see {{topic-configuration-interactions}}) and interactions about the topic data (see {{topic-data-interactions}}).

Topic configuration interactions are discovery, create, read configuration, update configuration, delete configuration and handle the management of the topics.

Topic data interactions are publish, subscribe, unsubscribe, read and are oriented on how data is transferred from a publisher to a subscriber.

## Managing Topics {#managing-topics}

{{fig-api}} shows the resources of a Topic Collection that can be managed at the Broker.

~~~~~~~~~~~
             ___
   Topic    /   \
Collection  \___/
                 \
                  \____________________
                   \___    \___        \___
                   /   \   /   \  ...  /   \        Topic
                   \___/   \___/       \___/   Configurations
~~~~~~~~~~~
{: #fig-api title="Resources of a Broker" artwork-align="center"}

The Broker exports a topic-collection resource, with resource type "core.ps.coll" defined in {{iana}} of this document. The interfaces for the topic-collection resource is defined in {{topic-collection-interactions}}.

# Pub-Sub Topics {#topics}

The configuration side of a "publish/subscribe broker" consists of a collection of topics. These topics as well as the collection itself are exposed by a CoAP server as resources (see {{fig-topic}}). Each topic has a topic configuration and a topic data resources. Topic configuration is used by a client creating or administering the topic and topic data is used by the publishers and subscribers to the topic.

~~~~~~~~~~~
               ___
       Topic  /   \
  Collection  \___/
                  \
                   \___________________________
                    \          \               \
                     \ ......   \ ......        \ ......
                    : \___  :  : \___  :       : \___  :
             Topic  : / + \ :  : / + \ :       : / + \ :
     Configuration  : \_|_/ :  : \_|_/ :       : \_|_/ :
                    ....|....  ....|....       ....|....
                    ....|....  ....|....       ....|....
                    :  _|_  :  :  _|_  :  ...  :  _|_  :
             Topic  : /   \ :  : /   \ :       : /   \ :
              Data  : \___/ :  : \___/ :       : \___/ :
                    :.......:  :.......:       :.......:
                     topic 1    topic 2         topic n
~~~~~~~~~~~
{: #fig-topic title='Configuration and Data resources of a topic' artwork-align="center"}

## Collection Representation

Each topic-configuration is represented as a link, where the link target is the URI of the topic-configuration resource.

Each topic-data is represented as a link, where the link target is the URI of the topic-data resource. A topic-data link is an entry within the topic-configuration called 'topic_data_uri' (see {{configuration-properties}}).

The list can be represented as a Link Format document {{RFC6690}}. The link to each topic-configuration resource specifies the link target attribute 'rt' (Resource Type), with value "core.pubsub.conf" defined in this document.

## Topic Creation and Configuration

A CoAP client can create a new topic by submitting an initial configuration for the topic (see {{topic-create}}). It can also read and update the configuration of existing topics and delete them when they are no longer needed (see {{topic-configuration-interactions}}).

The configuration of a topic itself consists of a set of properties. These fall into one of two categories: configuration properties and status properties.

Configuration properties can be set by a client and describe the desired configuration of a topic. Status properties are read-only, managed by the server, and provide information about the actual status of a topic.

When a client submits a configuration to create a new topic or update an existing topic, it can only submit configuration properties. When a server returns the configuration of a topic, it returns both the configuration properties and the status properties of the topic.

<!-- TBD New properties
data persistence: do we want the broker to cache some of the last resource representations? how long? Do we have opinions on message size too?
qos: do we want to reflect similar qos properties as mqtt does too?
limits: do we limit the amount of subscribers? message size, memory use?
security: tls, authentication (who can subscribe or publish), access control, oscore... -->

### Configuration Properties {#configuration-properties}

The CBOR map includes the following configuration parameters, whose CBOR abbreviations are defined in {{pubsub-parameters}} of this document.

* 'topic_name', with value the topic name encoded as a CBOR text string.

* 'topic_data_uri', with value the URI of the topic data resource for subscribing to a pubsub topic encoded as a CBOR text string.

* 'rt', with value the resource type "core.ps.conf" associated with topic configuration resources, encoded as a CBOR text string

<!-- TBD ACE goes out of the doc and is added as a extensible property coming from ACE. I just add an IANA ref to that.
Discovery as a function of the authorization status of the client -->

<!-- ace goes out-->
<!-- v* 'as_uri', with value the URI of the Authorization Server associated with the Group Manager for the topic, encoded as a CBOR text string. Candidate clients that can configure topics will have to obtain an Access Token from that Authorization Server, before starting the topic configuration or creation.-->
<!-- * 'ace-pubsub-profile'?? TBD-->

### Status Properties

The CBOR map includes the following status parameters, whose CBOR abbreviations are defined in {{pubsub-parameters}} of this document.

* 'conf_filter', is a CBOR map containing a CBOR array and with CBOR abbreviation defined in {{pubsub-parameters}}. It is used with FETCH when retrieving a partial representation of a topic configuration (see {{topic-fetch-configuration}}).

### Topic Configuration Representation {#topic-configuration-representation}

A topic configuration is represented as a CBOR map containing the configuration properties and status properties of the topic as top-level elements.

Unless specified otherwise, these are defined in this document and their CBOR abbreviations are defined in {{pubsub-parameters}}.

#### Default Values

TBD

## Discovery

Discovery involves that of the Broker, topic collections, topic configurations and topic data.

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
    </ps/>;rt=core.ps;ct=40
~~~~~~~~~~~

### Topic Discovery {#topic-discovery}

<!-- section needs more work -->

A Broker can offer a topic discovery entry point to enable clients to find topics of interest. The resource entry point thus represents a collection of related resources as specified in {{?RFC6690}} and is identified by the resource type "core.ps.coll".

The interactions with topic collections are further defined in {{topic-collection-interactions}}.

A topic collection is a group of topic configuration resources that define the properties of the topics themselves (see Section {{topic-configuration-representation}}). Each topic configuration is represented as a link to its corresponding resource URI. The list can be represented as a Link Format document {{?RFC6690}}. Topic configuration resources are identified by the resource type "core.ps.conf".

Within each topic configuration resource there is a set of configuration properties (see Section {{configuration-properties}}). The 'topic_data_uri' property contains the URI of the topic data resource that a CoAP client can subscribe to. Resources exposing resources of the topic data type are expected to use the resource type 'core.ps.data'.

## Topic Collection Interactions {#topic-collection-interactions}

These are the interactions that can happen at the topic collection level.

### Retrieving all topics {#topic-get-all}
<!--
GET to /topic-collection
retrieve all topics
response is link format
-->

A client can request a collection of the topics present in the broker by making a GET request to the collection URI.

On success, the server returns a 2.05 (Content) response with a representation of the list of all topic configurations (see Section {{topic-configuration-representation}}) in the collection.

Depending on the permission set each client MAY receive a different list of topics that they are authorized to read.

Example:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: topics

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   <coap://base-url/ps/tc/a>rt="core.ps.conf",
   <coap://base-url/ps/tc/b>rt="core.ps.conf",
   <coap://base-url/ps/tc/c>rt="core.ps.conf"
~~~~~~~~~~~

### Getting Topics by Properties {#topic-get-properties}
<!--
FETCH to /topic-collection with filter
retrieve only the topics that match the filter
request is cbor
response is link format
-->

A client can filter a collection of topics by submitting the
representation of a topic filter (see Section {{topic-fetch-configuration}})  in a FETCH request to the topic collection URI.

On success, the server returns a 2.05 (Content) response with a
representation of a list of topics in the collection (see
Section {{topic-discovery}}) that match the filter.

Example:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: pubsub
   Uri-Path: topics
   Content-Format: TBD (application/pubsub+cbor)

   {
       "topic_name" : a
   }

<= 2.05 Content
   Content-Format: 40 (application/link-format)
   <coap://base-url/ps/tc/a>rt="core.ps.conf"
~~~~~~~~~~~

### Creating a Topic {#topic-create}
<!--
POST to /topic-collection
create new topic
request is cbor
response (created) is cbor including the link to new topic-config resource
creator proposes topic name but broker approves
-->

A client can add a new topic to a collection of topics by submitting a representation of the initial topic configuration (see Section {{topic-configuration-representation}}) in a POST request to the topic collection URI.

The topic specification sent in the payload should use a supported serialization of the CoRE link format {{!RFC6690}} but other serializations like {{?I-D.ietf-core-coral}} may be used in the future.

On success, the server returns a 2.01 (Created) response indicating the topic URI of the new topic.

If a topic manager (TBD) is present in the broker, the topic creation  may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The broker MUST respond with a 4.00 (Bad Request) error in the following scenarios:

- If any received parameter is specified multiple times, invalid or not recognized (malformed).

- If the Authorization Server with URI specified in the 'as_uri' parameter is not trusted and there is no alternative AS.

<!-- TBD a coap endpoint creating a topic may specify topic_data_uri different than that used by the broker. The broker may then simply forward observation requests the topic_data_uri

if the topic_data_uri is empty the broker will assign
-->

~~~~~~~~~~~
=> 0.02 POST
   Uri-Path: pubsub
   Uri-Path: topics
   Content-Format: TBD2 (application/core-pubsub+cbor)
   TBD (this should be a CBOR map)
   {
     "topic_name" : sensor23
   }

<= 2.01 Created
   Location-Path: pubsub
   Location-Path: sensor23
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
     "topic_name" : "sensor23",
     "topic_data_uri" : "coap://mybroker/td/sensor23"
   }
~~~~~~~~~~~

## Topic Configuration Interactions {#topic-configuration-interactions}

These are the interactions that can happen at the topic configuration level.

### Getting a topic configuration  {#topic-get-configuration}

<!--
GET to /topic-config
retrieve a topic configuration
response is cbor
-->

A client can read the configuration of a topic by making a GET request to the topic configuration URI.

On success, the server returns a 2.05 (Content) response with a representation of the topic configuration. The response has as payload the representation of the topic configuration as specified in {{topic-configuration-representation}}.

If a topic manager (TBD) is present in the broker, retrieving topic information may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error. The response MUST have Content-Format set to "application/core-pubsub+cbor".

The response payload is a CBOR map, whose possible entries are specified in {{topic-configuration-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

Example:

~~~~~~~~~~~
=> 0.01 GET
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   TBD (this should be a CBOR map)
   {
     "topic_name" : "1234",
     foo
     bar
   }
~~~~~~~~~~~

### Getting part of a topic configuration by filter {#topic-fetch-configuration}
<!--
FETCH to /topic-conf with filter
retrieve only certain parameters from the configuration
request is cbor
response is cbor
-->

A client can read the configuration of a topic by making a FETCH request to the topic configuration URI with a filter for specific parameters. This is done in order to retrieve part of the current topic configuration.

The request contains a CBOR map with a configuration filter or 'conf_filter', a CBOR array and with CBOR abbreviation defined in {{pubsub-parameters}}. Each element of the array specifies one requested configuration parameter or status parameter of the current topic configuration (see {{topic-configuration-representation}}).

On success, the server returns a 2.05 (Content) response with a representation of the topic configuration. The response has as payload the partial representation of the topic configuration as specified in {{topic-configuration-representation}}.

If a topic manager (TBD) is present in the broker, retrieving topic information may require manager approval subject to certain conditions (TBD). If the conditions are not fulfilled, the manager MUST respond with a 4.03 (Forbidden) error.

The response payload is a CBOR map, whose possible entries are specified in {{topic-configuration-representation}} and use the same abbreviations defined in {{pubsub-parameters}}.

Both request and response MUST have Content-Format set to "application/core-pubsub+cbor".

Example:

~~~~~~~~~~~
=> 0.05 FETCH
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234
   Content-Format: TBD2 (application/core-pubsub+cbor)
   TBD (this should be a CBOR map)
   {
     "conf_filter" : [foo,
                      bar",]
   }

<= 2.05 Content
   Content-Format: TBD2 (application/core-pubsub+cbor)
   TBD (this should be a CBOR map)
   {
     "foo" : 10,
     "bar" : 5
   }

~~~~~~~~~~~

### Updating the Topic Configuration {#topic-update-configuration}

<!--
PUT to /topic-conf
override the whole configuration
request is cbor
response is cbor
-->

A client can update the configuration of a topic by submitting the representation of the updated topic configuration (see Section 3.1.3) in a PUT or POST request to the topic URI. Any existing properties in the configuration are overwritten by this update.

<!-- TBD details need to be added once the configuration resource structure is defined-->

On success, the server returns a 2.04 (Updated) response.

Example:

~~~~~~~~~~~
=> 0.03 PUT
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
     "foo" : 11,
     "bar" : 5
   }

<= 2.04 Changed
   Content-Format: TBD2 (application/core-pubsub+cbor)

   TBD (this should be a CBOR map)
   {
     "foo" : 11,
     "bar" : 5
   }

~~~~~~~~~~~

<!--### Partial update of a topic Configuration {#topic-update-configuration} -->

<!--
PATCH to /topic-conf
rewrite few parameters
request is cbor
response is cbor
-->

### Deleting a Topic Configuration {#topic-delete}

<!--
DELETE to /topic-conf
delete configuration and data
-->

A client can delete a topic by making a CoAP DELETE request on the topic configuration URI.

On success, the server returns a 2.02 (Deleted) response.

When a topic configuration is deleted, the broker SHOULD also delete the topic data resource, unsubscribe all subscribers by removing them from the list of observers and returning a final 4.04 (Not Found) response as per {{!RFC7641}} Section 3.2.

<!-- TBD: Document error cases and add access control -->

Example:

~~~~~~~~~~~
=> 0.04 DELETE
   Uri-Path: pubsub
   Uri-Path: topics
   Uri-Path: 1234

<= 2.02 Deleted
~~~~~~~~~~~

## Publish/Subscribe {#pubsub}

Unless a topic is configured to use a different mechanism, publish/ subscribe is performed as follows: A publisher publishes to a topic by submitting the data in a PUT request to a broker-managed "topic data resource".  This causes a change to the state of that resources. Any subscriber observing the resource {{!RFC7641}} at that time receives a notification about the change to the resource state. Observations are maintained and terminated as specified in {{!RFC7641}}.

As shown in {{topics}}, each topic contains two resources: topic configuration and topic data. In that section we explained the creation and configuration of the topic configuration resources. This section will explain the management of topic data resources.

A topic data resource does not exist until some initial data has been published to it.  Before initial data has been published, the topic data resource yields a 4.04 (Not Found) response. If such a "half created" topic is undesired, the creator of the topic can simply immediately publish some initial placeholder data to make the topic "fully created".

All URIs for configuration and data resources are broker-generated. There does not need to be any URI pattern dependence between the URI where the data exists and the URI of the topic configuration. Topic configuration and data resources might even be hosted on different servers.

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
                '-'  Config         Config  '-'
                            DELETED
~~~~~~~~~~~
{: #fig-life title='Lifecycle of a Topic' artwork-align="center"}

After a publisher publishes to the topic for the first time, the topic is placed into the FULLY CREATED state. In this state, a client can read and update the configuration of the topic and delete the topic; a publisher can publish to the topic data resource; and a subscriber can observe the topic data resource.

When a client deletes a topic configuration, the topic is placed into the DELETED state and shortly after removed from the server. In this state, all subscribers are removed from the list of observers of the topic data resource and no further interactions with the topic are possible.

When a client deletes a topic data, the topic is placed into the HALF CREATED state, where clients can read, update and delete the configuration of the topic and awaits for a publisher to begin publication.

### Rate Limiting {#rate-limit}

The server hosting a data resource may have to handle a potentially very large number of publishers and subscribers at the same time. This means the server can easily become overwhelmed if it receives too many publications in a short period of time.

In this situation, if a client is sending publications too fast, the server SHOULD return a 4.29 (Too Many Requests) response {{!RFC8516}}.  As described in {{!RFC8516}}, the Max-Age option {{!RFC7252}} in this response indicates the number of seconds after which the client may retry. The Broker MAY stop publishing messages from the client for the indicated time.

When a client receives a 4.29 (Too Many Requests) response, it MUST NOT send any new publication requests to the same topic data resource before the time indicated by the Max-Age option has passed.

## Topic Data Interactions {#topic-data-interactions}

TBD: intro and image that shows a topic data URI hosted in a different endpoint than the broker

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

<!-- TBD: Other error cases: max_age? -->

<!-- TBD: add senml payload example below -->
<!-- TBD: add participants (publisher and broker) here and in other diagrams-->

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

<!-- TBD There are other potential error cases based on parameters from the  configuration file (subscription lifetime, topic content format...) -->

The following response codes are defined for the Subscribe operation:

<!-- TBD: Is this the best way to represent response codes? Are they needed?  Which other potential error cases exist based on parameters from the  configuration file (subscription lifetime, topic content format...)? -->

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

A client can unsubscribe simply by cancelling the observation as described in Section 3.6 of {{!RFC7641}}. The client MUST either use CoAP GET with the Observe Option set to 1 or send a CoAP Reset message in response to a notification.

<!--  do we want an example or is redundant? -->

### Delete topic data {#delete-topic-data}

<!--* DELETE /topic-data/TOPICNAME
   Intended for: publishers that can also make the topic half-created again
   Goal: make the topic half-created again
   Request: no payload.
-->

A client can unsubscribe simply by cancelling the observation as described in Section 3.6 of {{!RFC7641}}. The client MUST either use CoAP GET with the Observe Option set to 1 or send a CoAP Reset message in response to a notification.

### Read Latest Data {#read-data}

A client can get the latest published topic data by making a GET request to the topic data URI in the broker.

On success, the server MUST return 2.05 (Content) response with the data.

If the target URI does not match an existing resource or the topic is not in the fully created state (see {{topic-lifecycle}}), the broker SHOULD return a response code 4.04 (Not Found).

If the Broker can not return the requested content format it MUST return a response code 4.15 (Unsupported Content Format).

<!-- TBD There are other potential error cases we need to document -->

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

<!-- TBD: Maybe we want a section of the uri templates that could be used but that are not mandatory (nor recommended) to use in any case and are there just for example illustration

Also put an example in which the topic configuration is hosted on one server and the topic data on another, to illustrate why discovery is always a must-->

# CoAP Pubsub Parameters {#pubsub-parameters}

<!--
sample topics:
 'as_uri'
 'topic_name'
 'topic_data_uri'
 'kcd'
-->

This document defines parameters used in the messages exchanged between a client and the broker during the topic creation and configuration process (see {{topic-configuration-representation}}). The table below summarizes them and specifies the CBOR key to use instead of the full descriptive name.

Note that the media type application/core-pubsub+cbor MUST be used when these parameters are transported in the respective message fields.

~~~~~~~~~~~
+-----------------+-----------+--------------+------------+
| Name            | CBOR Key  | CBOR Type    | Reference  |
+-----------------+-----------+--------------+------------+
| topic_name      | TBD1      | tstr         | [RFC-XXXX] |
+-----------------+-----------+--------------+------------+
| topic_data_uri  | TBD2      | tstr         | [RFC-XXXX] |
+-----------------+-----------+--------------+------------+
| conf_filter     | TBD3      | array        | [RFC-XXXX] |
+-----------------+-----------+--------------+------------+
~~~~~~~~~~~
{: #fig-CoAP-Pubsub-Parameters title="CoAP Pubsub Parameters" artwork-align="center"}

# Security Considerations

<!-- TBD: we may take content from prev versions but we have to spend some more time on the implications of the topic-config -->
TBD.

# IANA Considerations {#iana}

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

* Attribute Value: core.ps

* Description: XXX of This document

* Reference: This document

* Notes: None

## Resource Type value 'core.ps.conf'

* Attribute Value: core.ps

* Description: XXX of This document

* Reference: This document

* Notes: None

## Resource Type value 'core.ps.data'

* Attribute Value: core.ps

* Description: XXX of This document

* Reference: This document

* Notes: None

# Acknowledgements {#acks}

The current version of this document contains a substantial contribution by Klaus Hartke's proposal {{I-D.hartke-t2trg-coral-pubsub}}, which defines the topic resource model and structure as well as the topic lifecycle and interactions. It also follows a similar architectural design as that provided by Marco Tiloca's {{I-D.ietf-ace-oscore-gm-admin}}.

The authors would like to also thank Carsten Bormann, Hannes Tschofenig, Zach Shelby, Mohit Sethi, Peter van der Stok, Tim Kellogg, Anders Eriksson, Goran Selander, Mikko Majanen, and Olaf Bergmann for their valuable contributions and reviews.

---
tags: core-wg, pubsub
---

# Pubsub discussion

Target is to merge the [klaus's draft](https://tools.ietf.org/html/draft-hartke-t2trg-coral-pubsub-00) into [core-pubsub](https://tools.ietf.org/html/draft-ietf-core-coap-pubsub-09)

## draft-ietf-core-coap-pubsub

Interactions with the broker:

- DISCOVERY: 
    - Broker discovery GET with `rt=core.ps` to EP or Broker
    - Topic discovery GET with `rt=core.ps.discover` to Broker
- CREATE topic: 
    - POST to `/ps/` with one `ct=xx` creates a topic
    - POST to `/ps/<parenttopic>/` with one `<subtopic> ct=xx` creates a subtopic
- PUBLISH on topic:
    - PUT/POST to `/ps/topic` with current representation It may create topic if new.
- SUBSCRIBE to topic:
    - GET `/ps/topic` with `observe:0`
- UNSUBSCRIBE from topic:
    - GET `/ps/topic` with `observe:1` 
- READ latest value:
    - GET `/ps/topic` 
- DELETE topic:
    - DELETE `/ps/topic` 

### Comments

- Terminology should be splitted in draft normative terminology and pubsub based one. However I'd consider simplifying this section. CoAP pub/sub service is defined and never used.
- I don't like the way the interface definitions are formatted, hard to read. Maybe tabulation could fix it. Wonder if there is a concrete format for it.
- The discovery section mixes a bit broker discovery and topic discovery...
- Publish should probably be only PUT
- There is a bit of too much text and caveats on every section with too much normative language for multiple different cases (brokerless, pre-configured brokers...). I think we should simplify and use the simple broker case. 
- Figure 10 last mesage is wrong or missing another arrow to client 1
- Given the CoAP context we work on, the READ operation seems redundant. Might be OK to keep it for general info.
- Section 5 about operation with RD could be on the discovery part

---

## draft-hartke-t2trg-coral-pubsub

The interactions with the broker on topic lifecycle are:

- DISCOVER topics:
    - Get all topics: GET `/pubsub/topics` 
    - Get some topics: GET `/pubsub/topics` with `ct=xx``
- CREATE topic: 
    - POST to `/pubsub/topics` with one `ct=xx` creates a subtopic
- READ topic Configuration:
    - GET `/pubsub/topics/topic/config`
- UPDATE topic Configuration:
    - PUT `/pubsub/topics/topic/config`
- DELETE topic:
    - DELETE `/pubsub/topics/topic` 

### Publish/Subscribe

Topics have **configuration** and **data**

The topic data does not exist until someone has published to it.
The topic configuration only *half-creates* topic.

The interactions with the broker on the topics themselves are:

- PUBLISH on topic: 
    - PUT to `/pubsub/topics/topic/data` with current representation and right `ct`.
- SUBSCRIBE to topic:
    - GET `/pubsub/topics/topic/data` with `observe:0`
- UNSUBSCRIBE from topic:
    - GET `/pubsub/topics/topic/data` with `observe:1` 

### Comments
- Could copy section 2.1 and figure 1 or 3
- I like that parts are out of scope. Maybe we should also do that.
- Is the topic configuration stored as a well-known-like resource within each topic?
    - Config and data are per topic but can be hosted anywhere. one server could host all of the configs.
- Topic Configurations properties and status properties are not defined. There is quite a bit of missing text, so it's WIP.  Maybe too much for future work? If there are CoRAL examples we can incorporate that's fine, but if it requires a whole rework of the pubsub draft that's another story.
- Discovery of the broker itself is missing.
- Topics are all subtopics of the collection URI (flat structure).
    - Who decides on the URI structure? the broker. We should reconsider how nested structure would work.
- I guess `/1234` could also be `/topic_config` or similar.
- Figure 3 should be at the begining. 
- Since in this API there are some operations that apply to `data` and others to `configuration`, I would make that clear in the text. For example `READ` in this draft is different than `READ` in the pubsub draft.
- `/ps/topic/data` is weird and people will get confused.
- Is this the right path: `/pubsub/topics/topic/data`?
    - Could be any path, it is not mandatory to use that, nor `ps/`
- Todo: Need to get the diagram with topic and config from Klaus's meeting minutes in 2019-07-23

# Discussion

So from the interaction pov these are the interactions present:

| Subject            | core draft                              | t2trg draft (examples)                   | Comments                  |
|--------------------|-----------------------------------------|------------------------------------------|---------------------------|
| Paths to broker    | `/ps/`                                  | `ps/topics`                              | samish                    |
| Paths to topic     | `/ps/topic`                             | `/config` and `/data` per `/topic`       | config and data decoupled |
| Broker discovery   | GET `rt=core.ps`                        | missing (RD)                             | -                         |
| Topic discovery    | GET `rt=core.ps.discover`               | GET `/ps/topics` or some with `ct=xx`    | samish                    |
| Create topic       | POST`/ps/` or `/ps/<parent>/` with `ct` | POST `/ps/topics`  with `ct`             | sub/sub/sub or not        |
| Delete             | DELETE `/ps/topic`                      | DELETE `/ps/topics/topic`                | same                      |
| Publish            | PUT/POST `/ps/topic` with ct            | PUT `/ps/topics/topic/data` with ct      | POST                      |
| Subscribe          | GET `/ps/topic` with `obs:0`            | GET `/ps/topics/topic/data` with `obs:0` | same                      |
| Unsubscribe        | GET `/ps/topic` with `obs:1`            | GET `/ps/topics/topic/data` with `obs:1` | same                      |
| Read config        | -                                       | GET `/ps/topics/topic/config`            | -                         |
| Update config      | -                                       | PUT `/ps/topics/topic/config`            | -                         |
| Read current value | GET `/ps/topic`                         | missing (RFC7252)                        | Do we need this?          |

Main questions are:
1.	Do we want to restructure the text and split it in Discovery, Topic Lifecycle and Interactions with Topics?
    - KH,JJ: take my draft structure and insert the core text in it.
2.	Do we want to decouple topics in topic config and data? (Easy)
    - KH,JJ,MK: YES
3.	Do we want to keep the brokerless pubsub in 3.5? (E)
        - Originally for L+G field bus.
    - YES put it somewhere else
    - Brokerless means broker+pub or broker+sub, the third role left out is the client
    - KH: Main Broker case as it is on coap-pubsub and then decide whether to have another draft or an appendix.
    - AK: Agree on that. Specially if we make an advance document.
    - MK: New draft may take some time due to aproval process, appendix might be a better way. 
    - KH: Keeping the basic simple use case first. Additional cases added later. Not only brokerless but also additional input on configuration on topics.
    - MK: Pubsub and RD could also fit into a specific use case category.
    - KH: my pubsub draft the interface for topic discovery is the same as coral rd (coral reef).
    - MK: In the original pubsub it was also trying to mimic the RD pattern. 
4.	Do we want to have multiple nested topics or do we prefer collections (flat structure)? (Difficult)
    - coap-pubsub may not have the right text exaplaining /ss/hh/hh/ hierarchies.
    - MK: In Montreal we concluded that one of the oportunities to clean up was the hierarchy concept.
    - KH: In the wg document the topic creator can choose the URL of the topic. And thus create a nesting of topics, which is only reflected in the URL structure and the discovery. If we change in the t2trg draft that the broker generates URLs then the client cannot specify that. Then what is left of the text abotu hierachies in the draft?
    - MK: If we do not provide an equivalent feature then there should be other mechanism to provide the feature. 
    - JJ, KH: Set relations between topics (child, parent) as links on the "config" file. 
    - MK: Needs another section on that. Might be an optional feature.
    - KH: The coral way of doing it would be to specify a form with the features for topic creation. Cause the wg assumes that the server knows what the client supports.
    - KH: Adding the form would be trivial.

    - Conclusion: we use the structure in t2trg, and provide the specification on how to update the relations between topics. These relations will reside in the /config of every topic.
    4.1 A client subscribing to a parent and getting notifications of children too. (D)
5. Discovery section
    - Discovery of brokers: Discovery considerations are the same to multiple applications are similar in multiple cases; maybe link to the right section in RD and add rt
    - Discovery of topics: Discovery interface can be the same as in the CoRAL-based RD ("CoRAL Reef"); maybe factor out the common structure and define a reusable component
    - KH: has a mini api for topic discovery. 
    - MK: broker discovery is now on RD.

    - Conclusion: we will use existing discovery mechanisms with a specific rt. Section has to be clear about that.
6.	Do we want any form of wildcard (+,#) type of functionality?
    - NO	
    - Maybe define a new mechanism to enable something equivalent to wildcards
    - MK: it would be interesting to have such a feature. It could offer various features that are not present atm. Agrees to keep it simple and that it should go on an extension.
    - AK: We want to keep the getting values and publishing values the same.
7.	3.1.1.Configuration Properties, 3.1.2.Status Properties are missing (D)
    - KH: need to specify the configuration properties and status properties based on discussions. 
    - KH: Similar structure in ACE Group Manager (just with different properties)
    - Check the proposal.txt , there are a list of things that could become config properties.

    - Conclusion: needs to be turned into a spec. 
8.	Read current value (E)
    - have something there as informational
    - let's have good examples but make sure we say they are examples and not requirements.
    - KH: different drafts shouldn't use same paths for applications
    - KH: we can add a section on how to get the current value but the example is just an example, not a requirement.
    - **They shouldn't assume the URL path, they should discover it in some way**.
    - When creating a topic we get back the path to /config and /data 
    - use a "this is an example path"
9.	Implementations?
    - Michael Koster, VTT, Jim Schaad, OCF? ...
    - AK: will compile a list of existing implementations.
    - JJ: CoRE to organize interop at hackathon.
    - List:
        - https://github.com/federicorossifr/coap-publish-subscribe
        - https://github.com/gotthardp/rabbitmq-coap-pubsub  https://www.rabbitmq.com/community-plugins.html#protocols
        - https://pdfs.semanticscholar.org/d09c/167140332abf0af51b4a4ca06fb4f1f230ff.pdf
        - https://github.com/PethrusG/CoAP-PubSub-Broker/tree/master
        - http://staff.ui.ac.id/system/files/users/riri/publication/aldwin-performance_evaluation_of_coap_broker_and_access_gateway_implementation_on_wireless_sensor.pdf
        - https://github.com/PethrusG/CoAP-PubSub-Broker
        - https://github.com/Com-AugustCellars/PubSub
        - https://github.com/herjulf/coap
        - https://github.com/tbwiss/CoAP_PubSub
        - https://github.com/wajd/TeamMulticast-CoAP-PubSub
    - MK: Remote plugfest has some specific VPN for IoT
10.  TODO: (KH) **Emphasize that publish means that we are updating the state of the resource.**


## Action Points

- **JJ to merge both documents**
- **KH and MK to arrange for meetings to fix the (D) parts.**

More questions:

- https://mailarchive.ietf.org/arch/msg/core/wshviFckFyOo3HbqhabAFy40Ug8/

1. Creation of topics as a long-lived resource is throughout the protocol. Coupled with other restrictions in the protocol this makes large topic trees with fine-grained topics infeasible, such as having a MAC addresses, IP addresses, UUIDs, or hash values as entries in the topic tree. A topic tree such as "{tenant-id}/{device-mac}/{event-id}" would be a valid pub-sub topic tree, but in this protocol it is all but impossible, and certainly impossible to do efficiently. Assume there are dozens of tenants, millions of device MACs and a hundred event IDs. 

2. The protocol allows for single-shot topic creation and publishing in one message, but doesn't allow subscribing to topics that don't "exist", leaving a race condition or at least additional probes by the subscribers to see if certain topics of interest exist. Presumably if a subscriber wishes to subscribe to a topic that doesn't yet have any messages, the subscriber could create the topic, but then authorization of topic creation becomes an interesting thing. The state diagram for correctly subscribing to a topic that may not exist yet is at least three states large.

3. The protocol requires the broker to cache the most recent message on all topics. This greatly expands the footprint of a broker, because it must hang on to at least one message per topic. Message retention should be optional.

4. Max-Age is only used to communicate topic time to expire. Instead, it could be used to indicate age of the message, or the time which the message should be considered still fresh, both of which would be better uses.

5. There is no way to subscribe to multiple topics with a single subscription, nor is there any way to subscribe to an entire sub-tree.

6. There is no way to subscribe to any subset of the topic tree.

7. Protocol is ambiguous regarding whether topics that are also parent topics can themselves be topics.

8. Protocol is ambiguous regarding message delivery semantics; can messages be delivered more than once? Maybe. Forcing messages to only be delivered exactly once is a higher burden of reliability than merely ensuring messages are delivered at least once, so I am not suggesting that the protocol be limited to only exactly-once semantics.
    - CoAP has no QoS as such.
        - QoS 0,1,2 ... maybe a different draft. 
        - "Gold-Bar Dispenser" discussion
    - Series Transfer Pattern
        - https://tools.ietf.org/html/draft-bormann-t2trg-stp-01
9. DTLS client certificate is the only option for authentication, and there's no profile defined for how this is done. Thus there is no defined way to have an interoperable authentication scheme.
    - OSCORE 
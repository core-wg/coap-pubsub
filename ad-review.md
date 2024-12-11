

Hi,

 

Here is the promised review of draft-ietf-core-coap-pubsub-15. It might seem long, but most comments are small and easily fixable. In general (as individual) I am supportive of this work and hope it gets published soon.

 

This is an individual review, but as responsible AD I would have flagged these (some of these, like references being blocking) during my AD review, so you can consider this an early-AD review :)


Francesca

 

----

 

# Review of draft-ietf-core-coap-pubsub-15

 

cc @fpalombini

 

## Comments

 

### Section 1.1

 

> This specification requires readers to be familiar with all the terms and concepts that are discussed in [RFC8288] and [RFC6690].

 

8288 needs to be normative rather than informative.

 

> A topic collection is hosted as one collection resource

 

"collection resource" needs a reference to where it is defined, otherwise it would be fine to say s/collection resource/resource

 

> topic-configuration

 

"topic-configuration" seems to be the same as "topic configuration", so it would be good to be consistent (unless there is a difference I don't see).

 

> Throughout this document the word "topic" and "topic-configuration" can be used interchangeably.

 

I would suggest to change into:

 

> Throughout this document the words "topic resource" and "topic configuration resource" can be used interchangeably.

 

### Section 2.1

 

> The list of links to the topic resources can be retrieved from the associated topic collection resource

 

Can you have topics associated to several topic collection resources? Is there supposed to be any commonality between topics in a same topic collection?

 

### Section 2.2

 

> Unless specified otherwise, these are defined in this document

 

This opens up the extensibility question. What other methods of specifying configuration properties is there? and how to make sure they interoperate? Does this mean an IANA registry?

 

After reading further ahead, indeed I see that the IANA registry has been set up. Then I suggest rephrasing the sentence above to remove "Unless specified otherwise" and to mention the IANA registry instead.

 

### Section 2.2.1

 

> 'topic-data': A required field (optional during creation)

 

I am confused about the "optional during creation" text.

 

> type of the topic-data resource for the topic. It encodes the resource type as a CBOR text string. The value should be "core.ps.conf".

 

What other value can this take (so why is it a "should be")? Even if none is defined, giving some additional hints to the reader on what values this could be extended to take, would help.

 

> 'topic-media-type': An optional field used to indicate the media type of the topic-data resource for the topic. It represents the media type as the integer identifier of the CoAP content-format (e.g., value is "50" for "application/json").

 

I believe you should change this to talk about content type rather than media type, as there are media types that are not registered as CoAP content-formats.

 

> 'topic-type': An optional field used to indicate the attribute or property of the topic-data resource for the topic.

 

I think this should be better specified: what are "attribute or property"? From the example give, it sounds like any sort of human readable text could fit here.

 

> 'expiration-date': An optional field used to indicate the expiration date of the topic. It encodes the expiration date as a CBOR text string.

 

Why not use any defined CBOR tagged time format? Maybe allowing any is a bad idea, but why not reuse existing time definitions (see tags for dates at  https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml#tags)

 3.4.1. {{RFC8949}}

> The broker can use this field to automatically remove topics that are no longer valid.

 

This “automatic” removal doesn’t seem to be defined anywhere in the text. There should be some text stating that the broker checks the expiration date every so often and does the removal.

 

>  The default value for this field has been set to 100 subscribers.

 

From previous text, it sounds like there is no default value, since:

 

>  If this field is not present, there is no limit to the number of simultaneous subscribers allowed.

 

So this seems contradictory. How is this default value set/used?

 

> 'observer-check': An optional field that controls the maximum number of seconds between two consecutive Observe notifications sent as Confirmable messages to each topic subscriber.

 

Not really sure how that works. After further reading: please add a fw reference to 3.2.3 that really explains this.

 

>  it defines a counter representing the number of historical resource states the broker should retain.

 

What happens if the broker does not retain the number it's supposed to? Maybe this sentence should be changed to state the “the broker retains” (rather than should)

 

General: are some of these config properties updatable? it feels like some may be (expiracy, max subscribers) while others it does not make sense (initialize). This should be made clear.

 

### Section 2.3.2

 

> The specific resource path is left for implementations, examples in this document use the "/ps" path.

 

Ok, I still think it would be good to give a default value. Also, I don't see "/ps" being used in the example below.

 

### Section 2.3.3

 

> ;ct=TBD,

 

Should it be ct=40?

 

### Section 2.4.1

 

> Depending on its granted permissions, a client MAY retrieve a different list of links, corresponding to the topics that the client is authorized to access.

 

Slightly confused by this wording. Different than what? Do you mean different from another client? Maybe rephrasing would help.

 

### Section 2.4.1

 

This seems to appear here for the first time, but it is a more general comment, and applies throughout the text. There are 44 occurrences of the term "server". In some cases, as in this section for example, it makes more sense to replace them with "broker". Yes, we know that the broker is a CoAP server, but I think what is important here for the reader is that the entity acting as broker returns a 2.05 response. Also note that otherwise the same section talks about "server" and "broker", as in the example below, which is not ideal.

 

> On success, the server returns a 2.05 (Content) response with a representation of the topic resource. 

> (...)

> If requirements are defined for the client to create the topic as requested and the broker does not successfully assess that those requirements are met, then the broker MUST respond with a 4.03 (Forbidden) error.

 

I would suggest revising the occurrences of "server" and change to "broker" when relevant.

 

To be consistent, it might be worth also checking that the term "client" is used consistently throughout the text (or might need to be changed to "publisher" or "subscriber" depending on context).

 

### Section 2.4.2

 

>    Content-Format: TBD (application/core-pubsub+cbor)

 

I suggest using the same TBD as in section 2.4.3 (and others), and adding an RFC Editor note to let them know that this value should be updated based on IANA registration.

However I now see that TBD2 is also used for something else (Figure 5), so that should be differentiated (see suggestion below).

 

### Section 2.4.3

 

> The response must include the required topic properties (see Section 2.2.1), namely: "topic-name", "resource-type" and "topic-data". It may also include a number of optional properties too.

 

I think these should be MUST and MAY (rather than must and may)

 

In the examples: key values are already given in these examples. They are not reflected in Figure 5. I suggest removing the TBDs in figure 5 and give IANA the actual values to be registered (this is fine for a newly set up registry). Also note that "resource-type" takes key 1 in the request and 2 in the response, which I expect is an error and should be fixed.

 

### Section 2.5.1

 

> The partial representation includes only the configuration parameters such that they are present and have the same value in both the current topic configuration and in the FETCH request.

 

What FETCH request? You probably meant GET request.

 

> If requirements are defined for the client to create the topic as requested

 

Copy paste error, I suspect? s/create/read

 

Same comment as in Section 2.4.3 for the example's key values.

 

### Section 2.5.2

 

The conf-filter in the example has key 0, which is also the key for topic-name in previous examples. Should be fixed.

 

> The request contains a CBOR map with a configuration filter or 'conf-filter', a CBOR array with CBOR abbreviation.

 

Please add "of configuration parameters, as defined in Section 4" (otherwise "CBOR abbreviation" is too vague).

 

> If requirements are defined for the client to create the topic as requested and the broker does not successfully assess that those

 

Same comment as in 2.5.1, should be s/create/read (or access).

 

### Section 2.5.3

 

> Similarly, decreasing max-subscribers will also cause that some subscribers get unsubscribed.

 

It is not clear which subscribers would get unsubscribed. If this is left to the implementers (LIFO? FIFO?) then a sentence saying that should be added. In general if I was implementing I would appreciate guidance.

 

>  Unsubscribed endpoints SHOULD receive a final 4.04 (Not Found) response as per Section 3.2 of [RFC7641].

 

RFC 7641 states:

 

>    In the event that the resource changes in a way that would cause a

>   normal GET request at that time to return a non-2.xx response (for

>   example, when the resource is deleted), the server sends a

>   notification with an appropriate response code (such as 4.04 Not

>   Found) and removes the client's entry from the list of observers of

>   the resource.

 

RFC 7641 does not RECOMMEND that the final error response is sent, but mandates it. I suggest you comply with it, remove the SHOULD and just leave it descriptive: "Unsubscribed endpoints receive a final 4.04 (Not Found) response as per Section 3.2 of [RFC7641]."

 

(Note: same comment for the same text in Section 2.5.4).

 

### Section 2.5.4

 

In the example, I note that the same topic is used, however not the full list of parameters is returned (in particular, "observer-check"). This made me wonder if these optional parameters are always returned as a result to any change made that does not affect them, or if they can be omitted. If I don't see anything specified (and I might have missed it), I would expect that all parameters should always be returned. So in this case, the example should be fixed. If I missed it, please point me to the part in the text that is relevant and disregard this comment.

 

### Section 3.1

 

Figure 4: I was initially confused by the Read/Update tag, I think it would be good to specify what is for the topic-config and what is for the topic-data. So for example, say "Read/Update config" for the Half created state, and "Read data (Subscribe), Read/Update config". Also I am not sure I understand the meaning behind the arrow from "Fully created" to itself, having both publish and subscribe labels. Maybe it was meant to be "Publish / Subscribe"?

 

### Section 3.2.1

 

> On success, the server returns a 2.04 (Updated) response. However

 

>    Header: Updated (Code=2.04)

 

s/Updated/Changed

 

### Section 3.2.2

 

> with the Observe set to 0 to subscribe to resource updates

 

"with the CoAP Observe Option" or "with Observe"

 

> If the topic is not yet in the fully created state (see Section 3.1) the broker SHOULD return a response code 4.04 (Not Found).

 

This begs the question: why is this a SHOULD and not a MUST - if SHOULD is used, then it must be accompanied by at least one of:

 

(1) A general description of the character of the exceptions and/or in what areas exceptions are likely to arise.  Examples are fine but, except in plausible and rare cases, not enumerated lists.

 

(2) A statement about what should be done, or what the considerations are, if the "SHOULD" requirement is not met.

 

(3) A statement about why it is not a MUST.

 

We need to give the reader enough context to make an informed decision. If you believe the context is there, and I just missed it, please do let me know.

 

> Example:

 

It would be good to expand on what sort of example this is - a successful subscription followed by one update.

 

### Section 3.2.4

 

> When a topic-data resource is deleted, the broker SHOULD also delete the topic-data parameter in the topic resource, unsubscribe all subscribers by removing them from the list of observers and return a final 4.04 (Not Found) response as per Section 3.2 of [RFC7641].

 

Same question here, why is this a SHOULD (see comment above).

 

> Example:

 

Same here, it would be good to characterize the example.

 

### Section 3.4

 

> In this situation, if a publisher is sending publications too fast, the server SHOULD return a 4.29 (Too Many Requests) response [RFC8516].

 

I am less concerned about this SHOULD (because the reasoning is sort of implied in my mind), but the IESG might bring it up, so I would suggest you also add some text around this SHOULD.

 

### Section 5

 

I was surprised that "Communication MUST be secured" - I would expect that to be at least a RECOMMENDED, but that it is not realistic (and rather idealistic) to expect _all_ pubsub scenarios to need protection.

 

### Section 6.1

 

Don't forget to send this to the media-types mailing list for review, when the document is ready to leave the working group.

 

### Section 6.3

 

Since part of the registry is under Expert Review, there needs to be some Expert Guidelines (see Section 5 and specifically 5.3 of RFC 8126):

 

>   When a designated expert is used, the documentation should give clear

>   guidance to the designated expert, laying out criteria for performing

>   an evaluation and reasons for rejecting a request.

 

### Section 7

 

[I-D.ietf-ace-key-groupcomm] can be updated to RFC 9594

 

[RFC8126] and [RFC8288] need to be normative references

 

[RFC8613] can be informative (same as [RFC9147])

 

## Notes

 

This review is in the ["IETF Comments" Markdown format][ICMF], You can use the

[`ietf-comments` tool][ICT] to automatically convert this review into

individual GitHub issues.

 

[ICMF]: https://github.com/mnot/ietf-comments/blob/main/format.md

[ICT]: https://github.com/mnot/ietf-comments

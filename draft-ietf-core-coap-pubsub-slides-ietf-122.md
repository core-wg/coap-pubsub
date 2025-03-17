footer: Jaime Jiménez • IETF121 2024-11-05
slidenumbers: true
autoscale: true
theme: Next, 9
slide-dividers: #

# [fit] A pubsub architecture for CoAP 
## [fit] draft-ietf-core-coap-pubsub

Jaime Jiménez, IETF121, 2024-11-05

---

# Recap: Architecture Overview

![inline 55%](/Users/ejajimn/Desktop/image-1.png)

---

# Recap: Interface Overview

![inline 60%](/Users/ejajimn/Desktop/image-2.png)

---

# Updates v -13 to -14

- Restructured sections for readability.
- Topic Configuration clarification of parameter values.
- Revised interactions for topic configuration.
- Added a section on iPATCH.
- Added more examples for various interactions.
- Updated topic discovery section based on discussions.
- Editorial changes.

---

# Updates v -14 to -15

- Fixed bug in [commit f32ce486](https://github.com/jaimejim/aiocoap-pubsub-broker/commit/f32ce4866a81319238d6e905de439c9410cce175) by removing unnecesary code.
- Based on feedback Introduced two optional configuration parameters for topic configuration:
  - `initialize`: Optional boolean; if true, pre-populates the topic-data path, enabling default subscription without a 4.04 error.
  - `topic-history`: Optional unsigned CBOR integer; specifies the number of past resource states to store for a topic.

---

# Updates v -14 to -15

- Updated all examples to align with RFC9594 format.
- Implemented explicit cancellation of ongoing subscriptions upon modification of topic configuration parameters.
- Incorporated editorial revisions based on received feedback.
- Provided clarifications regarding the creation of Topic Configurations.
- Editorial changes.

---

# Next Steps

- Interop 
- **WGLC?**
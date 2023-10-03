
# Group Configurations # {#group-configurations}

A group configuration consists of a set of parameters.

## Group Configuration Representation ## {#config-repr}

The group configuration representation is a CBOR map, which includes configuration properties and status properties.

### Configuration Properties ### {#config-repr-config-properties}

The CBOR map includes the following configuration parameters, whose CBOR abbreviations are defined in {{pubsub-parameters}} of this document.

* 'xxx', which specifies yyy, encoded as a CBOR text string or a CBOR integer. Possible values are the zzz.

<!-- use case: secure pubsub, by configuring a topic to use a specific pre-shared key or oscore credentials, only subscribers using those credentials can subscribe (oscore groups related)-->

* 'topic-name', which specifies a topic identifier, encoded as a CBOR text string or a CBOR integer.

### Status Properties ### {#config-repr-status-properties}

The CBOR map includes the following status parameters. Unless specified otherwise, these are defined in this document and their CBOR abbreviations are defined in {{pubsub-parameters}}.

* 'rt', with value the resource type "core.pubsub-config" associated with pubsub topic-configuration resources, encoded as a CBOR text string.

* 'active', encoding the CBOR simple value "true" (0xf5) if the OSCORE group is currently active, or the CBOR simple value "false" (0xf4) otherwise.

* 'topic-name', with value the topic name of the topic as a CBOR text string.

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

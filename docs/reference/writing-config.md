# 写配置

This page describes how to write configuration that conforms to Istio’s schemas. All configuration schemas in Istio are defined as [protobuf messages](https://developers.google.com/protocol-buffers/docs/proto3). When in doubt, search for the protos.

## Translating to YAML

There’s an implicit mapping from protobuf to YAML using [protobuf’s mapping to JSON](https://developers.google.com/protocol-buffers/docs/proto3#json). Below are a few examples showing common mappings you’ll encounter writing configuration in Istio.

**Important things to note:**

- YAML fields are implicitly strings
- Proto `repeated` fields map to YAML lists; each element in a YAML list is prefixed by a dash (`-`)
- Proto `message`s map to objects; in YAML objects are field names all at the same indentation level
- YAML is whitespace sensitive and must use spaces; tabs are never allowed

### `map` and `message` fields

| Proto                                                        | YAML                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `message Metric {         string descriptor_name = 1;         string value = 2;         map<string, string> labels = 3; } `Copy | `descriptorName: request_count value: "1" labels:   source: origin.ip   destination: destination.service `Copy |

*Note that when numeric literals are used as strings (like value above) they must be enclosed in quotes. Quotation marks (") are optional for normal strings.*

### `repeated` fields

| Proto                                                        | YAML                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `message Metric {         string descriptor_name = 1;         string value = 2;         map<string, string> labels = 3; }  message MetricsParams {     repeated Metric metrics = 1; } `Copy | `metrics: - descriptorName: request_count   value: "1"   labels:     source: origin.ip     destination: destination.service - descriptorName: request_latency   value: response.duration   labels:     source: origin.ip     destination: destination.service `Copy |

### `enum` fields

| Proto                                                        | YAML                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `enum ValueType {     STRING = 1;     INT64 = 2;     DOUBLE = 3;     // more values omitted }  message AttributeDescriptor {     string name = 1;     string description = 2;     ValueType value_type = 3; } `Copy | `name: request.duration value_type: INT64 `Copyor`name: request.duration valueType: INT64 `Copy |

*Note that YAML parsing will handle both snake_case and lowerCamelCase field names. lowerCamelCase is the canonical version in YAML.*

### Nested `message` fields

| Proto                                                        | YAML                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `enum ValueType {     STRING = 1;     INT64 = 2;     DOUBLE = 3;     // more values omitted }  message LabelDescriptor {     string name = 1;     string description = 2;     ValueType value_type = 3; }  message MonitoredResourceDescriptor {   string name = 1;   string description = 2;   repeated LabelDescriptor labels = 3; } `Copy | `name: My Monitored Resource labels: - name: label one   valueType: STRING - name: second label   valueType: DOUBLE `Copy |

### `Timestamp`, `Duration`, and 64 bit integer fields

The protobuf spec special cases the JSON/YAML representations of a few well-known protobuf messages. 64 bit integer types are also special due to the fact that JSON numbers are implicitly doubles, which cannot represent all valid 64 bit integer values.

| Proto                                                        | YAML                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `message Quota {     string descriptor_name = 1;     map<string, string> labels = 2;     int64 max_amount = 3;     google.protobuf.Duration expiration = 4; } `Copy | `descriptorName: RequestCount labels:   label one: STRING   second label: DOUBLE maxAmount: "7" expiration: 1.000340012s `Copy |

Specifically, the [protobuf spec declares](https://developers.google.com/protocol-buffers/docs/proto3#json):

| Proto                  | JSON/YAML | Example                    | Notes                                                        |
| ---------------------- | --------- | -------------------------- | ------------------------------------------------------------ |
| Timestamp              | string    | “1972-01-01T10:00:20.021Z” | Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. |
| Duration               | string    | “1.000340012s”, “1s”       | Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision. Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision. |
| int64, fixed64, uint64 | string    | “1”, “-10”                 | JSON value will be a decimal string. Either numbers or strings are accepted. |

## What’s next

- TODO: link to overall mixer config concept guide (how the config pieces fit together)
# RTCStats.proto

Protocol Buffers schema for the [W3C WebRTC Statistics API](https://www.w3.org/TR/webrtc-stats/) — a compact, typed wire format for shipping `RTCPeerConnection.getStats()` snapshots from clients to a collector.

Why: raw `getStats()` JSON is large and untyped. Sending it as protobuf cuts payload size and gives the collector (Go, Python, Java, TypeScript, …) generated types for every stats object.

## What's inside

`rtcstat.proto` (package `webrtc.stats`) defines one message per stats type, mirroring the spec dictionaries:

| `RTCStats.type` | Message |
|---|---|
| `codec` | `RTCCodecStats` |
| `inbound-rtp` / `outbound-rtp` | `RTCInboundRTPStats` / `RTCOutboundRTPStats` |
| `remote-inbound-rtp` / `remote-outbound-rtp` | `RTCRemoteInboundRTPStats` / `RTCRemoteOutboundRTPStats` |
| `media-source` | `RTCMediaSourceAudioStats`, `RTCMediaSourceVideoStats` |
| `media-playout` | `RTCAudioPlayoutStats` |
| `peer-connection` | `RTCPeerConnectionStats` |
| `data-channel` | `RTCDataChannelStats` |
| `transport` | `RTCTransportStats` |
| `candidate-pair` | `RTCIceCandidatePairStats` |
| `local-candidate` / `remote-candidate` | `RTCIceCandidateStats` |
| `certificate` | `RTCCertificateStats` |

plus `RTCStatsEntry` (a `oneof` over the types above) and `RTCStatsReport` (a timestamped list of entries — one `getStats()` call).

Conventions:
- Field names are the spec member names in `snake_case`. The proto3 JSON mapping accepts the original `camelCase`, so objects from `getStats()` can be parsed directly.
- Enum-typed members (`RTCIceRole`, `RTCDtlsTransportState`, `qualityLimitationReason`, …) are plain `string`s, so new values added by browsers don't break decoding.
- Members are mostly `optional`, to distinguish "absent" from zero; the common `timestamp`/`type`/`id` and a few always-present members are not.

## Usage

Generate code with `protoc` (or `buf`):

```sh
protoc --python_out=gen rtcstat.proto
protoc --plugin=protoc-gen-ts_proto=./node_modules/.bin/protoc-gen-ts_proto --ts_proto_out=gen rtcstat.proto
```

Parse a `getStats()` entry (Python):

```python
from google.protobuf import json_format
import rtcstat_pb2 as pb

# `entry` is one value from RTCStatsReport, serialized to JSON by the client
msg = json_format.ParseDict(entry, pb.RTCInboundRTPStats(), ignore_unknown_fields=True)
report = pb.RTCStatsReport(timestamp=entry["timestamp"],
                           stats=[pb.RTCStatsEntry(inbound_rtp=msg)])
payload = report.SerializeToString()
```

Use `ignore_unknown_fields` — browsers expose non-standard and newly added members that are not in the schema.

## Status

Based on the W3C WebRTC Statistics specification; not every member of every dictionary is covered yet — PRs welcome. Deprecated and browser-specific members are not included, except `stalled_time` (Chromium). Field numbers may change before 1.0.

Background on using these stats for debugging and quality monitoring: [Making Friends with WebRTC Statistics](https://www.youtube.com/watch?v=Lv1iVwlnqTk) (talk, in Russian).

## License

[MIT](licence.md)

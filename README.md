# Update Prompt usage SDK specification

How an SDK reports the calls to an HTTP API to [Update Prompt](https://updateprompt.com), so you can build one for any language or framework and have it behave exactly like the official ones.

## Purpose

[Update Prompt](https://updateprompt.com) shows how often each consumer calls each endpoint of an API, so the API's owner knows what is used and by whom before changing or retiring an endpoint. A usage SDK is a small library inside that API that does two jobs:

1. **Count calls** per endpoint, consumer and minute, and send the counts to the Usage API.
2. **Apply deprecation signals** (coming): add the headers and refusals Update Prompt gives for a deprecated API.

Everything an SDK does is also possible by calling the public Usage API directly; an SDK only saves writing it. It never gets in the way of the API it runs in.

| An SDK does | An SDK does not |
|---|---|
| Count calls: method, host, route (or raw path), consumer id, query parameter names, minute | Send bodies, header values, query values, IP addresses, status codes or latency |
| Send counts in batches and retry when the API asks | Send one request per call |
| Take the route from the router and the consumer id from the application | Authenticate consumers, define endpoints, or guess ids |
| Drop counts rather than slow the application | Block, slow or fail a request, or grow memory without bound |
| Pass values as they are | Validate, clean or normalise them: Update Prompt does that |
| Expose three counters and an optional logger | Collect metrics, traces or logs |
| Use public, documented endpoints only | Ship keys or secrets, use private endpoints, or call home on start |

## Contents

| Part | What it covers |
|---|---|
| [1. Configuration](SPEC.md#1-configuration) | API key, base URL, consumer id and route, and their defaults |
| [2. One call](SPEC.md#2-one-call) | What a call becomes: method, host, route or path, consumer, query names, minute, and the one 404 rule |
| [3. Counting and sending](SPEC.md#3-counting-and-sending) | Counting per minute, sending every 10 s, batch limits, the memory limit, shutdown and serverless |
| [4. Answers](SPEC.md#4-answers) | What to do with each answer: done, retry or drop, and `Retry-After` |
| [5. Requests](SPEC.md#5-requests) | Headers and body of each request to the Usage API |
| [6. Stats and logs](SPEC.md#6-stats-and-logs) | The three counters and how often to log |
| [7. Deprecation signals](SPEC.md#7-deprecation-signals) | Coming: applying a deprecated API's headers and refusals |
| [8. Conformance](SPEC.md#8-conformance) | What every SDK must test |

## Official SDKs

| Language | Repository | Status |
|---|---|---|
| Go | [usage-sdk-go](https://github.com/Update-Prompt/usage-sdk-go) | Reference implementation: `net/http`, with chi, gin, echo and gorilla/mux coming |

## Building your own

1. Read [SPEC.md](SPEC.md) from start to end; it is short.
2. Hook into your framework's middleware so you get the route template, not the raw path, after the handler has run.
3. Let the application set the consumer id per request, where its authentication knows who is calling.
4. Count in memory, send on a timer from one background task, and never let reporting throw into a request.
5. Test the cases in [8. Conformance](SPEC.md#8-conformance) against a fake Usage API.
6. Name your SDK in its `User-Agent` (see [5. Requests](SPEC.md#5-requests)).

Building one for a language we don't cover yet? Open an issue; we'd like to hear about it.

## Contributing

Issues and pull requests that make the specification clearer are welcome. Changes to behaviour are discussed in an issue first, since every SDK follows them.

## License

[Apache License 2.0](LICENSE)

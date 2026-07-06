<div align="center">

# RestXOP

**Streaming binary attachments over REST — MTOM/XOP semantics, without the SOAP.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Maven Central](https://img.shields.io/maven-central/v/dev.restxop/restxop-spring-boot-4-starter?label=Maven%20Central)](https://central.sonatype.com/artifact/dev.restxop/restxop-spring-boot-4-starter)
[![npm](https://img.shields.io/npm/v/restxop-js?label=npm)](https://www.npmjs.com/package/restxop-js)
[![Java](https://img.shields.io/badge/Java-17%2B-orange.svg)]()
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.5%2B_%7C_4.0%2B-brightgreen.svg)]()

</div>

---

## The problem

The usual way to send a binary file alongside structured data in a REST API is to Base64-encode it and inline it in the JSON body. That works, but it comes at a cost:

- **~33% size inflation** on every payload from Base64 encoding.
- **Full buffering** — the whole attachment has to sit in memory to be encoded and decoded, which doesn't scale to large files.
- **No streaming** — you can't start processing bytes until the entire blob has been received and decoded.

The SOAP world solved this years ago with **MTOM/XOP**: keep the structured message intact, but pull binary content out into separate, streamable MIME parts referenced by ID. RestXOP brings that same idea to plain REST.

## What RestXOP does

RestXOP sends a **`multipart/related`** request whose **root part is JSON**. Binary attachments travel as separate parts, referenced from the JSON by content ID — so they stream, unencoded, instead of bloating the body.

```
POST /documents
Content-Type: multipart/related; type="application/json"; boundary=xop

--xop
Content-Type: application/json

{ "title": "Q3 Report", "file": { "$ref": "cid:doc-1" } }
--xop
Content-Type: application/pdf
Content-ID: <doc-1>
Content-Transfer-Encoding: binary

<raw PDF bytes, streamed>
--xop--
```

The JSON stays clean and readable; the bytes stay raw and streamable. Consumers that don't need the attachment can process the JSON root part without ever touching the binary.

## Features

- **No Base64 bloat** — binary parts are sent raw, streamed end to end.
- **JSON-first** — the message stays ordinary JSON, with lightweight `cid:` references to attachments.
- **Java / Spring Boot server support** — drop-in integration for producing and consuming RestXOP payloads.
- **JavaScript / TypeScript clients** — first-class support for browser and Node consumers.
- **Memory-efficient** — designed for large attachments without buffering the whole payload.
- **Apache-2.0 licensed** — open and free to use.

## Getting started

Both sides live in the main repository — **[restxop/restxop](https://github.com/restxop/restxop)** — and are published to Maven Central and npm.

### Server — Java / Spring Boot

Add the starter for your platform generation alongside your usual web starter:

```xml
<dependency>
  <groupId>dev.restxop</groupId>
  <artifactId>restxop-spring-boot-4-starter</artifactId> <!-- or -3-starter for Boot 3.5+ -->
  <version>0.1.0</version>
</dependency>
```

Give a payload class an `Attachment` field and serve it from plain Spring MVC — no extra wiring:

```java
public class Report {
    public String title;
    public Attachment document;      // dev.restxop.Attachment
}

@GetMapping(value = "/report", produces = "multipart/related")
Report report() {
    return new Report("Q3 report",
            Attachment.builder(Path.of("/data/q3.pdf"))
                    .contentType("application/pdf")
                    .build());
}
```

On the receiving side, `report.document.contentStream()` hands you the bytes as they arrive — never fully buffered.

### Client — JavaScript / TypeScript

```bash
npm install restxop-js
```

```ts
import { restxopFetch } from "restxop-js";

// The typed JSON payload resolves as soon as the root part lands…
const { payload, attachments } = await restxopFetch<Report>("/report");
console.log(payload.title);              // usable immediately

// …while the attachment streams in behind it.
const pdf = await attachments[0].blob();
```

Zero runtime dependencies, ESM-only, under 10 KB gzipped; runs in evergreen browsers and Node 20+.

## Status

RestXOP is early and actively developed. Feedback, issues, and contributions are welcome — if you've ever winced at Base64-ing a large file into a JSON body, this project is for you.

## License

Licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

---

<div align="center">
<sub>RestXOP · streaming attachments over REST · <a href="https://restxop.dev">restxop.dev</a></sub>
</div>

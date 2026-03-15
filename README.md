# HoBom ProtoBuf Definitions

Shared proto repository defining gRPC contracts between HoBom microservices.
Distributed to each service via BSR (Buf Schema Registry).

- **BSR Module**: `buf.build/hobom/hobom-buf-proto`

---

## Proto Update Flow

```
1. Modify proto files in this repo → push to main

2. GitHub Actions (push-bsr.yml) runs automatically
   ├── buf push → uploads new version to BSR
   └── repository_dispatch → sends event to 4 downstream repos

3. Each repo's proto-update.yml runs simultaneously
   ├── hobom-event-processor (Go):         buf generate → PR with .pb.go included
   ├── for-hobom-backend (NestJS):         buf export  → PR with .proto files
   ├── hobom-llm-service-backend (NestJS): buf export  → PR with .proto files
   └── hobom-space-backend (.NET):         buf export  → PR with .proto files

4. Review & merge PR in each repo → deploy via existing pipeline
```

- **Go**: Generated `.pb.go` files are included in the PR. No extra codegen needed.
- **NestJS**: `.proto` files are loaded at runtime. No codegen.
- **.NET**: `.proto` files are compiled by the SDK during `dotnet build`. No extra step.

---

## Local Usage

```sh
# Pull latest proto files (in downstream repos)
buf export buf.build/hobom/hobom-buf-proto --output proto

# Generate Go code (in hobom-event-processor)
buf generate buf.build/hobom/hobom-buf-proto
```

---

## Directory Structure

```
├── law/
│   ├── v1/save-study-material.proto              SaveStudyMaterialController
│   └── outbox/v1/find-hobom-law-outbox.proto     FindHoBomLawOutboxController
├── llm/
│   └── v1/
│       ├── generate-study-material.proto          StudyMaterialService.Generate
│       └── ask-question.proto                     StudyMaterialService.Ask
├── log/
│   └── outbox/v1/hobom-log-outbox.proto           FindHoBomLogOutboxController
├── message/
│   └── outbox/v1/
│       ├── find-hobom-message-outbox.proto        FindHoBomMessageOutboxController
│       └── patch-hobom-message-outbox.proto       PatchOutboxController
└── space/
    └── outbox/v1/
        ├── find-hobom-space-outbox.proto           FindHoBomSpaceOutboxController
        └── patch-hobom-space-outbox.proto          PatchSpaceOutboxController
```

---

## Services

| Package      | Service                            | RPC                                      | Consumer               |
| ------------ | ---------------------------------- | ---------------------------------------- | ---------------------- |
| `outbox.law` | `FindHoBomLawOutboxController`     | `FindOutboxByEventTypeAndStatusUseCase`  | hobom-event-processor  |
| `law`        | `SaveStudyMaterialController`      | `SaveStudyMaterial`                      | hobom-event-processor  |
| `llm`        | `StudyMaterialService`             | `Generate`, `Ask`                        | hobom-event-processor, for-hobom-backend |
| `outbox.log` | `FindHoBomLogOutboxController`     | `FindOutboxByEventTypeAndStatusUseCase`  | hobom-event-processor  |
| `outbox`     | `FindHoBomMessageOutboxController` | `FindOutboxByEventTypeAndStatusUseCase`  | hobom-event-processor  |
| `outbox`     | `PatchOutboxController`            | `PatchOutboxMarkAsSentUseCase`, `...Failed` | hobom-event-processor |

---

## Secrets (GitHub Actions)

Required in `hobom-buf-proto` repo settings:

| Secret | Purpose |
|--------|---------|
| `BUF_TOKEN` | BSR authentication (buf.build API token) |
| `DOWNSTREAM_PAT` | Send repository_dispatch to downstream repos (GitHub PAT with `repo` scope) |

---

## Best Practices

- Always include `option go_package` and use consistent package naming
- Avoid using `Any`, prefer strict `oneof` or wrapper types
- Use `map<string, string>` only for stringifiable structures (headers, query)
- All services should validate incoming data before trusting proto fields

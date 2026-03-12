# HoBom ProtoBuf Definitions

HoBom 마이크로서비스 간 gRPC 계약을 정의하는 공유 proto 저장소입니다.
각 서비스에서 Git 서브모듈로 참조합니다.

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

## Code Generation

```sh
# Go (hobom-event-processor)
cd hobom-buf-proto && buf generate

# TypeScript/NestJS (for-hobom-backend, hobom-llm-service-backend)
# Proto 파일을 직접 참조 (ts-proto / @grpc/proto-loader)
```

---

## Versioning Strategy

Breaking changes must follow SemVer and bump the major version.
All clients should pin a commit SHA or use a Git tag.

---

## Best Practices

- Always include `option go_package` and use consistent package naming
- Avoid using `Any`, prefer strict `oneof` or wrapper types
- Use `map<string, string>` only for stringifiable structures (headers, query)
- All services should validate incoming data before trusting proto fields

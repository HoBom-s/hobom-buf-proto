# HoBom ProtoBuf Definitions

This repository contains all shared [Protocol Buffers](https://developers.google.com/protocol-buffers) (`.proto`) definitions for the HoBom ecosystem.  
It serves as the single source of truth for all gRPC contract definitions across microservices.

---

## 🔄 Versioning Strategy
Breaking changes must follow SemVer and bump the major version. All clients should pin a commit SHA or use a Git tag. Proto compatibility is enforced using Buf (coming soon).

---

## ✅ Best Practices
Always include option go_package and use consistent package naming.  Avoid using Any, prefer strict oneof or wrapper types. Use map<string, string> only for stringifiable structures (headers, query). All services should validate incoming data before trusting proto fields.
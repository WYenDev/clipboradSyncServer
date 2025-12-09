# ClipSync Server

This is the Server component of the ClipSync project, enabling real-time clipboard synchronization across devices via gRPC.

ClipSync comprises three parts:
- Proto: https://github.com/WYenDev/clipboradSyncProto
- Server: this repository
- Client: https://github.com/WYenDev/clipboardSyncCliClient

## Architecture
- gRPC API is defined in `protobuf/clipboardSync.proto`; Go stubs are generated into `genproto/clipboardSync` (via `make gen`).
- Handler layer (`handler/clipboardSync`) exposes gRPC endpoints, manages streams (`grpcStreamWrapper.go`), and applies interceptors.
- Service layer (`service/`) implements room management and clipboard update broadcasting (`clipboardSync.go`, `room.go`).
- Shared models (`shared/streamWriter.go`) define `ClipboardContent`, `ClipboardUpdate`, `Validate*`, `UpdateEvent`, and the `StreamWriter` abstraction used across layers.
- Types (`types/`) declare service interfaces (`ClipboardSyncService`) to decouple contracts from implementation.
- Entry point is `main.go`, with gRPC setup in `grpc.go`.

## Data Flow
1. Client joins a room and subscribes to updates.
2. Server validates the join (`ValidateJoin`) and establishes a stream (`StreamWriter`).
3. Client sends `ClipboardUpdate`; server publishes `UpdateEvent` to subscribers in the room.

## Repository Structure
- `genproto/clipboardSync`: generated Go protobuf and gRPC code (do not edit).
- `handler/clipboardSync`: RPC handlers, stream wrapper, interceptor.
- `service/`: room management and clipboard synchronization logic.
- `shared/`: common payload/event models and stream interface.
- `types/`: service interface and type contracts.
- `protobuf/`: `.proto` definition and README (Git submodule).
- Root: `main.go`, `grpc.go`, `Dockerfile`, `go.mod`, `go.sum`, `makefile`.

## Build & Run
- Build: `go build ./...`
- Run server: `make run-clipboardSync`
- Generate stubs: `make gen`

## Test
- All packages: `go test ./...`
- Single package: `go test ./handler/clipboardSync`
- Single test: `go test ./handler/clipboardSync -run TestName`

## Development Notes
- Format with `gofmt -s -w .`; vet with `go vet ./...`.
- Imports grouped: stdlib, external, local.
- Errors: return `error`, wrap context via `fmt.Errorf("...: %w", err)`.
- Respect `context.Context` as the first parameter where applicable.
- Do not modify files in `genproto/`.

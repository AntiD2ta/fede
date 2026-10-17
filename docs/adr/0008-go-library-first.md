# fede is a Go library first, with a thin CLI

fede's engine is a Go library and the `fede` CLI is a thin consumer of it, so custom builds and packages exercise the same interface the CLI uses. Data crossing every seam is serializable, which keeps a future out-of-process plugin protocol cheap. We chose Go over TypeScript because fede no longer needs to share code with a harness extension, and Go gives a single static binary.

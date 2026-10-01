# Agent and MCP documentation

[Master specification](../specification/README.md)

**Status:** SPEC-LOCKED principles; PROPOSED documentation and tool contracts. **Evidence:** VALIDATION REQUIRED.

- [Agent guide](agent-guide.md): reading order, sources of truth, status, and design boundaries.
- [MCP architecture and security](mcp-architecture.md): metadata, permissions, mutation and production boundaries.
- [Django adoption guides](../django/README.md): version-aware conceptual translation requirements.

Candidate 2 adds untrusted-content labelling, a CodeExecution capability class and the development MCP binding rules to [MCP architecture](mcp-architecture.md).

Rjango is AI-native but never AI-dependent. The framework runs fully without an AI service. The full MCP protocol, schemas, transports, consent model and agent documentation system remain design work; this scaffold is not a server implementation.

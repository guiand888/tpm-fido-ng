# go_commands_and_compiling.md

Running `go` commands and compiling and code execution ALWAYS in containers.

## Guidelines

- Any compiling or code execution MUST be done is a dedicated distrobox container named "tpm-fido", NEVER on the local system.

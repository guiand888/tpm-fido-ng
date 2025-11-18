# Context

## Current Work Focus

Phase 2 (Go Version & Toolchain Modernization) completed successfully. Project now targets Go 1.22+ with updated CI/CD pipeline. Next focus:
- Phase 3: Direct Dependency Updates
- Keeping dependencies up-to-date with security patches
- Maintaining browser compatibility (Chrome/Firefox on Linux)

## Recent Changes

- **Phase 2 Completed**: Upgraded Go from 1.16 to 1.22
  - Updated `go.mod` to Go 1.22
  - Updated `.github/workflows/go.yml` to test against Go 1.22 and 1.23
  - Ran `go mod tidy` to update dependencies
  - Verified code compiles successfully with Go 1.22
  - Verified all tests pass with Go 1.22
  - Updated `Readme.md` with Go 1.22+ requirement

## Next Steps

1. **Phase 3**: Upgrade all direct dependencies to latest stable versions
   - `github.com/google/go-tpm` - Currently v0.3.2 (2021); check for latest
   - `golang.org/x/crypto` - Currently May 2021 snapshot; update to latest
   - `github.com/psanford/uhid` - Currently April 2021 snapshot; check for updates
   - `github.com/foxcpp/go-assuan` - Currently v1.0.0; check for newer versions
2. **Phase 4**: Security & Vulnerability Remediation (govulncheck scan)
3. **Phase 5**: Code Quality & Compatibility Testing
4. **Security**: Address attestation private key embedded in plaintext in [`attestation/attestation.go:14`](../../attestation/attestation.go:14) - should be externalized using SOPS

## Known Issues

- **Security**: Attestation private key is hardcoded in source (see brief.md Security Notes)
- **Dependencies**: Still using versions from 2021 (will be addressed in Phase 3)

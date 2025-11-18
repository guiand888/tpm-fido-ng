
# tpm-fido Modernization & Security Refresh Plan

## Executive Summary

This document outlines a comprehensive strategy to modernize the tpm-fido repository from its current state (Go 1.16, dependencies from 2021) to a modern, actively-maintained codebase with current security patches and best practices. The goal is to establish tpm-fido as a reliable, community-driven fork with regular security updates and compatibility with modern Linux systems.

## Current State Assessment

### Go Version
- **Current**: Go 1.16 (released Feb 2021, EOL since March 2022)
- **Target**: Go 1.22+ (latest stable LTS)
- **Impact**: Security patches, performance improvements, language features

### Dependencies (as of go.mod)
| Dependency | Current Version | Status | Action |
|---|---|---|---|
| github.com/google/go-tpm | v0.3.2 (2021) | Outdated | Upgrade to latest |
| golang.org/x/crypto | v0.0.0-20210513164829 (2021) | Outdated | Upgrade to latest |
| github.com/psanford/uhid | v0.0.0-20210426002309 (2021) | Outdated | Upgrade to latest |
| github.com/foxcpp/go-assuan | v1.0.0 (2020) | Outdated | Upgrade to latest |

### Known Security Issues
1. **Attestation private key embedded in source** (`attestation/attestation.go:14`)
   - Should be externalized using SOPS or similar secret management
   - Currently a plaintext secret in version control

2. **No vulnerability scanning in CI/CD**
   - No automated security checks in GitHub Actions
   - No dependency vulnerability scanning

3. **Outdated CI/CD tooling**
   - GitHub Actions versions are outdated
   - No SAST or code quality checks

## Modernization Strategy

### Phase 1: Dependency Audit & Security Assessment
**Objective**: Understand current security posture and identify all gaps

**Tasks**:
- [ ] Audit current Go version (1.16 from 2021) and identify upgrade path
- [ ] Audit all direct dependencies for security vulnerabilities and outdated versions
- [ ] Check for transitive dependency vulnerabilities using `go mod graph`
- [ ] Document security gaps and CVEs in current dependencies
- [ ] Create dependency upgrade strategy with compatibility testing plan

**Deliverables**:
- Security audit report documenting all vulnerabilities
- Dependency upgrade roadmap with compatibility notes
- Risk assessment for each dependency upgrade

---

### Phase 2: Go Version & Toolchain Modernization
**Objective**: Upgrade to modern Go version with latest security patches

**Tasks**:
- [ ] Update `go.mod` to Go 1.22+ (latest stable LTS)
- [ ] Update CI/CD workflow to test against Go 1.22+ and latest stable
- [ ] Verify code compiles and tests pass with new Go version
- [ ] Update build documentation in `Readme.md`

**Deliverables**:
- Updated `go.mod` with Go 1.22+
- Updated `.github/workflows/go.yml` with new Go versions
- Verified build and test success

**Compatibility Notes**:
- Go 1.22 introduces range-over-int and other features
- No breaking changes expected in tpm-fido codebase
- All existing code should compile without modification

---

### Phase 3: Direct Dependency Updates
**Objective**: Upgrade all direct dependencies to latest stable versions

**Tasks**:
- [ ] Upgrade `github.com/google/go-tpm` to latest stable version
- [ ] Upgrade `golang.org/x/crypto` to latest stable version
- [ ] Upgrade `github.com/psanford/uhid` to latest stable version
- [ ] Upgrade `github.com/foxcpp/go-assuan` to latest stable version
- [ ] Run `go mod tidy` and verify `go.sum` updates
- [ ] Test each dependency upgrade incrementally

**Deliverables**:
- Updated `go.mod` and `go.sum` with latest versions
- Compatibility testing results for each dependency
- Documentation of any API changes requiring code updates

**Dependency Details**:

#### github.com/google/go-tpm
- **Current**: v0.3.2 (2021)
- **Latest**: Check upstream for current stable
- **Impact**: TPM 2.0 operations, security patches
- **Testing**: Verify TPM backend functionality

#### golang.org/x/crypto
- **Current**: v0.0.0-20210513164829 (2021)
- **Latest**: Check upstream for current stable
- **Impact**: HKDF, ChaCha20-Poly1305, ECDSA operations
- **Testing**: Verify cryptographic operations in both backends

#### github.com/psanford/uhid
- **Current**: v0.0.0-20210426002309 (2021)
- **Latest**: Check upstream for current stable
- **Impact**: HID device emulation, USB communication
- **Testing**: Verify HID frame assembly/disassembly

#### github.com/foxcpp/go-assuan
- **Current**: v1.0.0 (2020)
- **Latest**: Check upstream for current stable
- **Impact**: Pinentry GUI communication
- **Testing**: Verify user confirmation dialogs

---

### Phase 4: Security & Vulnerability Remediation
**Objective**: Identify and remediate all known vulnerabilities

**Tasks**:
- [ ] Run `go mod verify` to check for tampering
- [ ] Scan for known vulnerabilities using `govulncheck`
- [ ] Address any identified CVEs in dependencies
- [ ] Document security patches applied
- [ ] Create `SECURITY.md` with vulnerability reporting guidelines

**Deliverables**:
- Clean `govulncheck` scan results
- `SECURITY.md` with vulnerability reporting process
- Security patch documentation

**SECURITY.md Template**:
```markdown
# Security Policy

## Reporting Vulnerabilities

Please report security vulnerabilities to [contact method].
Do not open public issues for security vulnerabilities.

## Supported Versions

- Latest release: Receives security updates
- Previous release: Receives critical security updates only
- Older releases: No longer supported

## Security Updates

All security updates are documented in CHANGELOG.md.
```

---

### Phase 5: Code Quality & Compatibility Testing
**Objective**: Ensure updated dependencies don't break functionality

**Tasks**:
- [ ] Run full test suite with updated dependencies
- [ ] Test cross-compilation for i386, arm, arm64 architectures
- [ ] Verify TPM backend functionality (if TPM available)
- [ ] Verify memory backend functionality
- [ ] Test HID device emulation
- [ ] Document any breaking changes or compatibility issues

**Deliverables**:
- Test results for all architectures
- Compatibility matrix documentation
- Breaking change documentation (if any)

**Test Coverage**:
- Unit tests: `go test ./...`
- Cross-compilation: i386, arm, arm64
- Integration tests: Registration/authentication flows
- Memory backend: In-memory key storage and signing
- TPM backend: TPM 2.0 operations (if available)

---

### Phase 6: Secret Management & Configuration
**Objective**: Implement proper secret management for sensitive data

**Tasks**:
- [ ] Create `.sops.yaml` for secret management (attestation key protection)
- [ ] Document how to externalize attestation private key
- [ ] Add guidance for CI/CD secret handling
- [ ] Update `.gitignore` to prevent accidental secret commits

**Deliverables**:
- `.sops.yaml` configuration file
- Secret management documentation
- Updated `.gitignore`

**.sops.yaml Template**:
```yaml
creation_rules:
  - path_regex: '.*\.secrets\.ya?ml$'
    encrypted_regex: '.*'
    age: 
      - REPLACE_WITH_AGE_PUBLIC_RECIPIENT_1
  - path_regex: 'attestation/.*\.key$'
    encrypted_regex: '.*'
    age:
      - REPLACE_WITH_AGE_PUBLIC_RECIPIENT_1
```

**Documentation**:
- How to generate age keypair locally
- Where to store private keys (CI secrets, local keychain)
- How to encrypt/decrypt secrets with sops
- CI/CD integration for secret decryption

---

### Phase 7: CI/CD Pipeline Enhancement
**Objective**: Modernize and secure the CI/CD pipeline

**Tasks**:
- [ ] Update GitHub Actions workflow with latest action versions
- [ ] Add security scanning (SAST/dependency scanning)
- [ ] Add code quality checks (linting, formatting)
- [ ] Add vulnerability scanning in CI pipeline
- [ ] Document CI/CD process and security gates

**Deliverables**:
- Updated `.github/workflows/go.yml`
- New security scanning workflows
- CI/CD documentation

**Workflow Enhancements**:
1. **Dependency Scanning**: `github/dependency-review-action`
2. **Vulnerability Scanning**: `github/super-linter` or similar
3. **Code Quality**: `golangci-lint` integration
4. **Security**: `trivy` for container/dependency scanning
5. **SAST**: `semgrep` or similar for code analysis

---

### Phase 8: Documentation & Release Preparation
**Objective**: Update all documentation to reflect modernization

**Tasks**:
- [ ] Update `Readme.md` with new Go version requirements
- [ ] Create `CHANGELOG.md` documenting all updates
- [ ] Update `dev/rules/memory-bank/tech.md` with new dependency versions
- [ ] Create migration guide for users upgrading from old version
- [ ] Document known issues and compatibility notes

**Deliverables**:
- Updated `Readme.md`
- `CHANGELOG.md` with all changes
- Migration guide for users
- Updated memory bank documentation

**CHANGELOG.md Template**:
```markdown
# Changelog

## [1.0.0] - 2025-11-18

### Changed
- Upgraded Go from 1.16 to 1.22
- Updated all dependencies to latest stable versions
- Enhanced CI/CD pipeline with security scanning

### Security
- Fixed [CVE-XXXX-XXXXX] in dependency X
- Added vulnerability scanning to CI/CD
- Implemented secret management with SOPS

### Added
- SECURITY.md with vulnerability reporting guidelines
- Automated dependency updates via Dependabot
```

---

### Phase 9: Testing & Validation
**Objective**: Comprehensive testing of modernized codebase

**Tasks**:
- [ ] Perform end-to-end testing with Chrome/Firefox (if possible)
- [ ] Validate registration and authentication flows
- [ ] Test on multiple Linux distributions
- [ ] Verify udev rules and permissions still work
- [ ] Create test plan documentation

**Deliverables**:
- Test plan documentation
- Test results for multiple distributions
- End-to-end test results

**Test Plan**:
1. **Unit Tests**: All existing tests pass
2. **Integration Tests**: Registration and authentication flows
3. **Cross-Platform**: Ubuntu, Fedora, Arch Linux
4. **Browser Compatibility**: Chrome, Firefox
5. **TPM Compatibility**: TPM 2.0 devices
6. **Permissions**: udev rules and group membership

---

### Phase 10: Release & Communication
**Objective**: Release modernized version and communicate changes

**Tasks**:
- [ ] Create release branch and tag
- [ ] Generate release notes highlighting security updates
- [ ] Update repository description and badges
- [ ] Communicate fork status and maintenance commitment
- [ ] Set up automated dependency update process (Dependabot)

**Deliverables**:
- Release tag and branch
- Release notes
- Updated repository metadata
- Dependabot configuration

**Release Notes Template**:
```markdown
# Release v1.0.0 - Modernization & Security Refresh

This release modernizes tpm-fido with current dependencies and security patches.

## Highlights
- Upgraded Go from 1.16 to 1.22
- Updated all dependencies to latest stable versions
- Added comprehensive security scanning to CI/CD
- Implemented secret management with SOPS
- Enhanced documentation and testing

## Security Updates
- Fixed [CVE-XXXX-XXXXX] in dependency X
- Added vulnerability scanning to CI/CD pipeline
- Implemented SECURITY.md with vulnerability reporting

## Breaking Changes
None - all existing functionality preserved

## Migration Guide
See MIGRATION.md for upgrade instructions
```

---

## Implementation Timeline

| Phase | Duration | Priority | Dependencies |
|---|---|---|---|
| Phase 1: Audit | 2-3 days | Critical | None |
| Phase 2: Go Upgrade | 1-2 days | Critical | Phase 1 |
| Phase 3: Dependencies | 2-3 days | Critical | Phase 2 |
| Phase 4: Security | 2-3 days | Critical | Phase 3 |
| Phase 5: Testing | 3-5 days | Critical | Phase 4 |
| Phase 6: Secrets | 1-2 days | High | Phase 5 |
| Phase 7: CI/CD | 2-3 days | High | Phase 6 |
| Phase 8: Docs | 2-3 days | High | Phase 7 |
| Phase 9: Validation | 3-5 days | High | Phase 8 |
| Phase 10: Release | 1-2 days | High | Phase 9 |

**Total Estimated Duration**: 3-4 weeks

---

## Success Criteria

1. ✅ All dependencies upgraded to latest stable versions
2. ✅ Go version upgraded to 1.22+
3. ✅ Zero known vulnerabilities in `govulncheck` scan
4. ✅ All tests passing on all supported architectures
5. ✅ CI/CD pipeline includes security scanning
6. ✅ Secret management implemented with SOPS
7. ✅ Comprehensive documentation updated
8. ✅ Release notes published
9. ✅ Dependabot configured for automated updates
10. ✅ Community communication about fork status

---

## Risk Assessment

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Dependency API changes | Medium | High | Incremental testing, compatibility matrix |
| TPM compatibility issues | Low | High | Thorough testing on multiple TPM devices |
| Breaking changes in go-tpm | Low | High | Review upstream changes, test thoroughly |
| CI/CD complexity | Medium | Medium | Incremental workflow updates, documentation |
| User migration issues | Medium | Medium | Clear migration guide, backward compatibility |

---

## Next Steps

1. **Immediate**: Begin Phase 1 (Dependency Audit)
2. **Week 1**: Complete Phases 1-3 (Audit, Go Upgrade, Dependencies)
3. **Week 2**: Complete Phases 4-5 (Security, Testing)
4. **Week 3**: Complete Phases 6-8 (Secrets, CI/CD, Documentation)
5. **Week 4**: Complete Phases 9-10 (Validation, Release)

---

## References

- [Go Release Policy](https://go.dev/doc/devel/release)
- [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)
- [SOPS Documentation](https://github.com/mozilla/sops)
- [GitHub Security Best Practices](https://docs.github.com/en/code-security)
- [Go Security Best Practices](https://golang.org/doc/security)

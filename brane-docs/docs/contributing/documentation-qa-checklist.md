# Documentation QA Checklist

Use this checklist when reviewing any documentation change against
brane-deployment. The implementation source of truth is brane-deployment.

## Before merging a documentation change

### CLI commands
- [ ] Every shell command has been run against the documented brane-deployment revision
- [ ] brane certs add invocations pass all three files: ca.pem, client.pem, client-key.pem
- [ ] brane instance add invocations include --unchecked for TLS-only gRPC nodes
- [ ] brane workflow run invocations include a use-case argument and a .bs file path
- [ ] No command uses a flag or subcommand removed in the documented revision

### Ports and services
- [ ] Central brane-api is documented at port 50051
- [ ] Central brane-drv is documented at port 50053
- [ ] Worker brane-reg is documented at port 50051
- [ ] Worker brane-chk is documented at port 50052
- [ ] No removed or renamed service appears as current

### Architecture
- [ ] brane-job is documented as sharing brane-chk network namespace
- [ ] brane-job reaches the checker through localhost, not a hostname
- [ ] Policy deliberation and policy store paths are documented as file mounts, not network endpoints

### Role capabilities
- [ ] Administrator guide covers: deploy, health check, certificate generation, token generation
- [ ] Policy Manager guide covers: upload, validate, activate, inspect history
- [ ] User guide covers: package build, package list, instance add, certs add, workflow run
- [ ] No role guide documents capabilities belonging to another role as primary actions

### Frontend navigation
- [ ] Page labels match homepage.py PAGES dict labels exactly
- [ ] No removed page appears as a primary sidebar destination
- [ ] Workstation setup is documented as internally accessible, not a primary sidebar item
- [ ] CLI cross-links are present for every documented UI action

### Sensitive material
- [ ] No documentation example contains a real private key, JWT, or certificate bundle
- [ ] Policy token generation examples use placeholder values

### Known platform limitation
- [ ] Remote workflow execution is documented as requiring a server node (not the Mac control shell)
  when the Mac CLI config path divergence is unresolved

## Release checklist

When a new brane-deployment revision is deployed:

1. Record the new revision SHA in docs/audits/YYYY-MM-DD-baseline.md
2. Run the infrastructure health check and attach the output
3. Re-run the end-to-end validation (hello_world workflow)
4. Update the revision reference in docs/brane-spec/02-architecture.md
5. Update the revision reference in docs/brane-user-guide/06-administrators.md
6. Check this QA checklist against any changed CLI flags or service ports

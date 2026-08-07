# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.6.0] - 2026-08-07

Fixes from an adversarial review of the whole repo (16 confirmed findings).

### Security

- GitHub Actions script injection closed in the Armbian build workflow: values derived from the upstream `armbian/build` VERSION file and build metadata now reach `run:` scripts via `env:` instead of inline `${{ }}` interpolation, the fetched version must match a version-shaped regex, and the fetches use `curl -f` (#45).
- The k3s agent config at `/etc/rancher/k3s/config.yaml` embeds the cluster join token; it is now written `0600` instead of world-readable `0644` (#45).
- `prepare-armbian-image.sh` no longer ships a hardcoded default root password: `ROOT_PASSWORD` is required, is not echoed to the terminal, and the first-boot autoconfig file carrying it is `0600` (#45).
- Third-party actions running with write permissions or secret access are pinned to commit SHAs: `anchore/sbom-action` (v0.24.0), `softprops/action-gh-release` (v2.6.2), `docker/setup-qemu-action` (v3.7.0) (#45).

### Added

- k3s upgrades: bumping `k3s_version` now reinstalls the binary on provisioned nodes instead of failing the post-install version check (the installer was previously skipped whenever the binary existed) (#45).
- `k3s_prereq` ships the previously missing `registries.yaml.j2`, so defining `private_registries` renders `/etc/rancher/k3s/registries.yaml` instead of failing with a template-not-found error (#45).
- Dependabot covers the Terraform provider pin via a `terraform` ecosystem entry; the invalid `ansible` ecosystem entry, which invalidated the entire config file, is gone (#45).

### Fixed

- `recover-node.yml` reworked. Node selection moved from `--limit` (which skipped the control-plane cleanup/Longhorn/verify plays without error and, when omitted together with `force_reinstall`, targeted every worker at once) to a required, validated `-e recover_target=<node>` variable. `target_nodes` was previously always empty, so the stale-state cleanup and Longhorn recovery never did anything; they now operate on the selected nodes, and a cross-play guard aborts if the cleanup play was skipped. The stale-disk removal referenced `item.item.item` on a loop whose items only nest one level, so it crashed exactly when a disk needed cleanup; it now uses `item.item`, and an unused companion task is gone (#45).
- `addons.yml` fresh-install deadlock: the Prometheus PVCs need Longhorn's StorageClass while Longhorn's ServiceMonitor needs Prometheus's CRDs, so the prometheus deploy no longer blocks (`wait: false`) and the play verifies monitoring readiness after all addons land (#45).
- `kernel_modules` / `sysctl_settings` in group_vars renamed to `base_kernel_modules` / `base_sysctl_settings` so the base role actually applies them; previously the `dm_crypt` preload and the `fs.inotify.max_user_instances: 8192` override were silently discarded (#45).
- The k3s agent role reads the join token from the control plane via delegation instead of relying on facts set by a same-run server play, so `--limit workers` and `recover-node.yml` no longer fail on an undefined variable (#45).
- `download-armbian-image.sh` uses `curl -f` on all downloads (an HTTP error page can no longer be saved as the image) and warns when no checksum is available; `prepare-armbian-image.sh` preserves the image's original `resolv.conf` (the host's copy was previously baked into every prepared image) and its cross-arch QEMU detection actually fires (`grep -o` returned multi-line output that never matched the comparison) (#45).
- The base role's `Reload sysctl` handler is gated for Molecule, matching the repo's rule that live systemd/kernel operations skip container runs; the pre-commit shellcheck hook covers `ansible/scripts/` (#45).

## [1.5.0] - 2026-06-23

### Added

- **Brand identity & README revamp** — a portable brand guide (`docs/BRAND.md`), an Ansible-logo banner, a cluster icon, and brand-matched repo buttons (Terraform provider, Terraform modules, and a cluster back-link), with the README realigned to the Turing Pi family convention and the Architecture/Overview sections reordered (#28, #29, #30, #31, #32, #33, #34, #35, #36, #37, #38, #39).
- **Molecule coverage extended to the `base` role** — every active systemd/D-Bus/service-start task is gated behind the `molecule_test` flag so the role tests cleanly under Docker; `verify.yml` asserts durable config artifacts (packages, written config, enabled units) rather than live runtime (#42).
- Armbian R2 lifecycle: a prune step keeps only the newest 3 image versions and the manifests cap history to match; README gains live Armbian-version, build-status, and release badges (#25).
- CI lints `.github/` and repo-root YAML (yamllint runs from the repo root) and pins its linters to the pre-commit versions (#24).

### Changed

- The `k3s_server` Molecule scenario now installs k3s without starting it (`k3s_server_skip_start` → `INSTALL_K3S_SKIP_START`), avoiding flaky systemd-in-Docker failures while still verifying the install and config (#41).
- Flaky Molecule tests moved into a dedicated `paths: ansible/**`-filtered workflow wrapped in a 3-attempt retry, so docs/asset PRs skip them and "Ansible Lint" stays the only required check (#40).
- `ansible/inventories/server/hosts.yml.example` k3s_version bumped from v1.31.3 to v1.31.4+k3s1 (pass-5 missed this template file; live `hosts.yml` was already on v1.31.4) (#13).
- Armbian image metadata bumped to `26.08.0-trunk` (build 2026-06-06, kernel 6.1.115) in `images.json` (`eb1d3f0`).
- Hardened Ansible network operations against flaky CI — get.k3s.io downloads, Helm repo fetches, git/pip installs, apt, and the Helm installer now retry; Kubernetes readiness waits are index-safe (#26).
- Pinned the `jfreed-dev/turingpi` Terraform provider to `~> 1.4` with a committed multi-platform `.terraform.lock.hcl`; scoped the owner auto-approve workflow to `main` with a concurrency guard (#23, #24).
- `make install-deps` pins ansible-lint/yamllint to the CI versions and `make lint` no longer swallows yamllint failures; `.ansible-lint` now enforces `no-handler` (#27).
- Hardened CI plumbing — the Release workflow's changelog-generation step, the owner auto-approve workflow, and Dependabot's migration to the org reusable auto-merge workflow (#14, #15, #17, #20).

### Fixed

- `k3s_server` kubeconfig rewrite converted to a handler, resolving the ansible-lint `no-handler` warning (production profile now passes clean); the `rknn` model download's retries now take effect via a missing `until` (#26).
- Stale version references, add-on ordering, and NPU/vault details corrected across the docs (#16).

### Removed

- Dead `rknn_version: "2.3.2"` variable from `ansible/inventories/server/hosts.yml` + `.example`. The role at `ansible/roles/rknn/tasks/main.yml` clones HEAD of `airockchip/rknn-llm`; this variable was never read. INSTALL.md and ARCHITECTURE.md correctly cite v1.2.1 (#13).
- Unused `geerlingguy.docker` / `geerlingguy.pip` Galaxy role dependencies (never referenced by any play or role) and their now-vestigial lint excludes in `.yamllint`, `.ansible-lint`, and `.pre-commit-config.yaml` (#27).

## [1.4.0] - 2026-05-19

> **Scope note:** This entry was originally written to cover PRs #7–#12 (today's pre-release cleanup) but the full v1.3.6 → v1.4.0 commit range contains ~36 commits of un-changelogged work that accumulated on `main` between 2025-12-27 (v1.3.6) and 2026-05-19 (this release). The list below has since been backfilled to reflect the full scope; the commits cited as `(commit XXX)` are the source-of-truth merges that introduced the change.

### Added

- **Pre-commit hooks + Ansible Vault tooling + Molecule tests** (`4777527`). The repo now has `.pre-commit-config.yaml`, an opt-in vault workflow (see `docs/VAULT-SETUP.md`), and Molecule scenarios for `base` and `k3s_server` roles.
- **Architecture documentation with Mermaid diagrams** (`fe3f1e4`, `f64e26b`, refined in #8 to use the `neutral` theme).
- **`docs/ARMBIAN-BUILD.md`** + nightly `Build Armbian Image` GitHub Actions workflow that pushes to the `armbian-builds` R2 bucket under `turing-rk1/armbian/<version>/`.
- **SBOM generation in CI** (`e8a5086`) + Dependabot auto-merge for minor/patch updates (`c5bd023`).
- **GitHub Actions self-hosted runner migration** (`ac9caa8`, `3372fda`) — workflows now use `runs-on: [self-hosted, linux]` against the Spark fleet.
- **YAML-format issue templates** replacing the previous markdown templates (`b837332`, `1159338`, `860f1ec`, `37915ba`, `2c40ebe`).
- `CODE_OF_CONDUCT.md` (Contributor Covenant, copied from sister repo) (#11).
- `.editorconfig` for cross-editor formatting consistency (#11).

### Changed

- **License: MIT → Apache 2.0** at the LICENSE file level (`ddc0ec7`) + README badge + License section updated to match (#9).
- **Org migration: `jfreed-dev` → `freed-dev-llc`.** Repo was silently transferred; this release fixes all surviving `jfreed-dev` references in README + 21 CHANGELOG compare-links + docs (#9). GitHub URL redirects still work but the canonical org is now `freed-dev-llc`.
- **K3s version: v1.31.3 → v1.31.4+k3s1** across inventory (first bumped in `be9d9a6`), docs, the `k3s_server` role's default, and Molecule tests (#9, this release).
- **Monitoring stack tweaks** — Aria scrape config added, Grafana LoadBalancer IP wired up, `node-exporter` disabled (`9ef10f6`). README + ARCHITECTURE.md updated accordingly (#9).
- **Armbian image storage moved** under the `armbian/` subdirectory on R2 (`3151f1d`); metadata files at `turing-rk1/armbian/<version>/` rather than at the root.
- **Armbian metadata bumped to 26.05.0-trunk in the local images.json** (`5fbd3e6`). Note: the live R2 unified manifest at `armbian-builds.techki.to/turing-rk1/images.json` may lag — the publish step pushes per-build but the unified file is refreshed only when a new build actually runs.
- Architecture docs (`docs/ARCHITECTURE.md`) — storage diagram redrawn with NVMe on all 4 nodes (matches inventory `has_nvme: true` since v1.1.4); `Node Exporter` removed from monitoring stack; `inventories/vm/` + `inventories/laptop/` references removed (those inventories never existed) (#9).
- Implementation guide (`docs/IMPLEMENTATION.md`) — deleted Phase 7 (VM Cluster Deployment) and Phase 8 (Laptop/Workstation Setup); both referenced playbooks (`vm-provision.yml`, `workstation.yml`) and inventories that don't exist in this repo (#9, this release). Phase 0.4 firmware download switched from a stale Ubuntu 22.04 URL to the Armbian image published nightly to R2 — resolved dynamically from `images.json` (#9). Target Environments table trimmed to just Server; summary table dropped Phase 7/8 and the orphan "Total (VM)" row.
- Hardcoded `~/Code/turing-ansible-cluster` paths in INSTALL.md + IMPLEMENTATION.md replaced with `$REPO_ROOT` (#10).
- Hardcoded `~/Code/turing-rk1-cluster` (sibling repo) path in IMPLEMENTATION.md updated to `~/Repos/turing-rk1-cluster` (this release).
- `docs/ARCHITECTURE.md` `rknn-llm` runtime label corrected from v1.2.3 → v1.2.1 to match INSTALL.md (the role clones HEAD, INSTALL.md is canonical) (#10).
- `docs/VAULT-SETUP.md` clarified that vault is opt-in — the default inventory keeps secrets in plain text (gitignored) (#10).
- Mermaid diagrams in `docs/ARCHITECTURE.md` locked to the `neutral` theme for cross-mode legibility (#8).
- README badge owner corrected (`jfreed-dev` → `freed-dev-llc`) (#3).

### Fixed

- **Armbian build workflow** `no_check_bucket = true` for bucket-scoped R2 token (#5); pinned to `ubuntu-latest` to avoid Self-Hosted ARM Docker breakage (#4).
- **`base` role** — skip `modules-load.d` write when `base_kernel_modules` is empty (#6).
- **Molecule** — Ansible 2.18+ compatibility (`b52462a`); `ALLOW_BROKEN_CONDITIONALS` env var work (`6007dad`, `d5fd87b`).
- **CI workflow failures** unrelated to this repo (`b3c6313`).
- **`maximize-build-space` action** pinned to commit SHA for supply-chain safety (`3694891`).
- Node20-deprecated GitHub Actions bumped ahead of the 2026-06-02 forced upgrade (#7).

### Compatibility

The K3s version bump (v1.31.3 → v1.31.4+k3s1) is a patch-level upgrade and is non-breaking. The role-level default change matches what the live inventory has been overriding to since `be9d9a6`; no operator action required. The Apache 2.0 relicense (from MIT) is intentionally permissive — no action required for consumers.

## [1.3.6] - 2025-12-27

### Fixed

- `k3s_prereq` role: Symlink for `/var/lib/rancher` now created for all nodes with NVMe
  - Previously only created for agent nodes, causing control plane K3s data loss on re-runs
- `k3s_agent` role: Fixed template variable names in `config.yaml.j2`
  - Changed `k3s_server_url` → `k3s_agent_server_url`
  - Changed `k3s_server_token` → `k3s_agent_server_token`
- `rknn` role: Removed `/opt/rkllama` from directory creation list
  - Directory is now created by git clone to avoid "directory not empty" errors
- `longhorn` role: Added node labeling for automatic disk detection
  - Nodes are now labeled with `node.longhorn.io/create-default-disk=true`
  - Required for `createDefaultDiskLabeledNodes: true` helm setting

### Changed

- Updated README to show node1 has NVMe (matches inventory)
- Updated INSTALL.md hardware table to reflect NVMe boot on all nodes

## [1.3.5] - 2025-12-27

### Added

- Cluster reset script (`scripts/reset-cluster.sh`) for clean reinstall
  - Stops and uninstalls K3s on all nodes
  - Wipes NVMe data partitions and Longhorn storage
  - Clears container, CNI, and iptables configuration
  - Power cycles nodes via BMC
  - Supports dry-run mode and selective node targeting

## [1.3.4] - 2025-12-27

### Changed

- Made yamllint a hard failure in CI (was continue-on-error)
- Made terraform fmt a hard failure in CI (was continue-on-error)
- Fixed terraform formatting in `environments/server/main.tf`

## [1.3.3] - 2025-12-27

### Changed

- Renamed Ansible roles to use underscores for ansible-lint compliance
  - `k3s-agent` → `k3s_agent`
  - `k3s-prereq` → `k3s_prereq`
  - `k3s-server` → `k3s_server`
  - `nginx-ingress` → `nginx_ingress`
  - `prometheus-stack` → `prometheus_stack`
- Fixed variable naming to use role prefixes
  - `k3s_server_token` → `k3s_agent_server_token`
  - `k3s_server_url` → `k3s_agent_server_url`
  - `ingress_controller` → `nginx_ingress_controller`
- Removed `role-name` skip from `.ansible-lint` (now fully compliant)

## [1.3.2] - 2025-12-27

### Changed

- Removed default Grafana password from playbook output (use secrets file)
- Added security warnings for `TURINGPI_INSECURE` flag (TLS bypass)
- Added security comments for git clone operations documenting repo trust
- Pinned Ansible collection versions for reproducible builds
  - kubernetes.core: 6.2.0
  - community.general: 12.1.0
  - ansible.posix: 2.1.0

## [1.3.1] - 2025-12-27

### Fixed

- Security: Kubeconfig permissions changed from 0644 to 0600 (owner-only access)
- Security: SSH StrictHostKeyChecking now uses `accept-new` instead of disabled
  - Accepts new host keys on first connection
  - Detects MITM attacks on subsequent connections
- Security: Added k3s binary version verification after installation
- Ansible role variable naming to follow `role_prefix_` convention
  - `base` role: `kernel_modules` → `base_kernel_modules`, etc.
  - `rknn` role: `rkllama_*` → `rknn_*`
- Shell tasks with pipes now use `set -o pipefail` for proper error handling
- YAML line length violations in playbooks
- Broken Armbian download link (armbian.com/turing-rk1 → armbian-builds.techki.to)
- Broken rkllama GitHub link in NPU-API docs (notpunhnox → jfreed-dev)

### Changed

- Replaced `systemctl` command with `ansible.builtin.systemd` module in bootstrap.yml
- Pip upgrade task uses `state: present` with `--upgrade` instead of `state: latest`
- Added `# noqa: no-handler` for appropriate debug tasks

## [1.3.0] - 2025-12-26

### Added

- NPU LLM API service with systemd integration
  - rkllama Flask server runs on each node (port 8080)
  - Auto-starts on boot, restarts on failure
  - OpenAI-compatible `/generate` endpoint
- DeepSeek 1.5B model auto-download (~1.9GB)
  - Pre-configured Modelfile for API server
  - Symlinked from `/opt/rkllama/models/` to `~/RKLLAMA/models/`
- Python virtual environment at `/opt/rkllama/venv`
  - Bypasses Debian externally-managed-environment restrictions
  - Includes transformers, flask, huggingface_hub
- NPU API documentation (`docs/NPU-API.md`)
  - Full endpoint reference with examples
  - Python client examples
  - Load balancing guide
  - Troubleshooting section

### Changed

- Updated README NPU section with API quick start
- Enhanced rknn role with service deployment tasks
- Added handlers for service restart on config changes

## [1.2.1] - 2025-12-26

### Changed

- Migrated image storage from GitHub Releases to Cloudflare R2
  - Custom domain: `armbian-builds.techki.to`
  - 10GB free storage with no egress fees
  - S3-compatible API via rclone
- Updated workflow to use PAT token for branch protection bypass
- Download script now fetches from Cloudflare R2

## [1.2.0] - 2025-12-26

### Added

- Automated Armbian image build workflow (`.github/workflows/armbian-build.yml`)
  - Daily check for upstream armbian/build version changes
  - Automatic build with QEMU ARM64 cross-compilation
  - Auto-updates `images.json` with new image metadata
- Image metadata registry (`images.json`)
  - Tracks latest image version, checksum, download URL
  - Build history with release links
- `--latest` flag for download script
  - Fetches image info from `images.json`
  - Automatic SHA256 verification
- Upload/download scripts for Google Drive distribution

### Changed

- Updated `docs/ARMBIAN-BUILD.md` with "Latest Pre-built Image" section
- Enhanced download script to support direct URLs and `--latest` flag

## [1.1.9] - 2025-12-26

### Changed

- Enhanced `prepare-armbian-image.sh` script
  - Installs required packages via chroot (open-iscsi, nfs-common, curl, etc.)
  - Enables iSCSI services for Longhorn compatibility
  - SSH keys injected directly (no manual provisioning needed)
  - Hostnames updated to match Ansible inventory (node1-4)
  - Auto-detects cross-architecture and uses QEMU if available
  - SKIP_PACKAGES=true option to skip package installation

## [1.1.8] - 2025-12-26

### Added

- Node recovery playbook (`playbooks/recover-node.yml`)
  - Cleans up stale K3s node password secrets
  - Removes stale Longhorn disk entries
  - Re-registers nodes with cluster
  - Configures Longhorn storage automatically
  - Usage: `ansible-playbook playbooks/recover-node.yml --limit node3`

### Changed

- Improved `base` role
  - Added role defaults for packages and kernel modules
  - Enhanced iSCSI configuration with socket and initiator setup
  - Fixed hostname persistence in /etc/hostname

- Improved `k3s-prereq` role
  - Auto-detects NVMe boot vs eMMC boot scenarios
  - Handles post-migration partition layout (nvme0n1p1=root, nvme0n1p2=longhorn)
  - Better idempotency for storage configuration

- Improved `k3s-agent` role
  - Added service existence check before restart
  - Waits for node to reach Ready state
  - Creates node password directory

- Improved `bootstrap.yml` playbook
  - Added pre-flight checks (memory, architecture, NVMe)
  - Post-bootstrap verification of iSCSI and storage
  - Better logging of configuration state

## [1.1.7] - 2025-12-26

### Changed

- Completed node3 NVMe boot migration
  - Reflashed with fresh Armbian image via BMC
  - Migrated to NVMe boot (50G root + 415G Longhorn)
  - Re-registered with K3s cluster
  - All 4 nodes now fully operational on NVMe

### Fixed

- K3s server crash on node1 caused by empty node-passwd file
- Longhorn manager crash on node3 (missing open-iscsi package)

## [1.1.6] - 2025-12-26

### Added

- Google Drive image distribution scripts
  - `scripts/upload-armbian-image.sh` - Upload images with rclone, auto-compression, checksums
  - `scripts/download-armbian-image.sh` - Download with gdown/rclone/curl, checksum verification
- Image Distribution section in `docs/ARMBIAN-BUILD.md`
- Image Distribution quick reference in README

## [1.1.5] - 2025-12-26

### Changed

- Migrated worker nodes (node2, node4) from eMMC to NVMe boot
  - 50G root partition (ext4)
  - 415G Longhorn storage partition (xfs)
- Node3 NVMe migration prepared (pending BMC power cycle)
- All nodes now configured for NVMe boot with dual-partition layout

### Added

- Worker node considerations section in NVMe migration guide
  - K3s agent directory structure fix
  - Longhorn disk UUID mismatch resolution

## [1.1.4] - 2025-12-26

### Changed

- Migrated node1 (control plane) from eMMC to NVMe boot
  - 50G root partition (ext4)
  - 415G Longhorn storage partition (xfs)
- Updated `has_nvme: true` for node1 in server inventory

## [1.1.3] - 2025-12-26

### Added

- CHANGELOG.md with full release history

## [1.1.2] - 2025-12-26

### Added

- Comprehensive Armbian build documentation (`docs/ARMBIAN-BUILD.md`)
  - Build prerequisites and system requirements
  - Quick start build commands
  - Customization options (SSH keys, static IPs, packages)
  - Multi-node automated build script
  - Troubleshooting guide
- Building Armbian section in README

## [1.1.1] - 2025-12-26

### Added

- License badge in README

## [1.1.0] - 2025-12-26

### Added

- GitHub issue templates (bug report, feature request)
- Pull request template with checklist
- Dependabot configuration for Ansible collections and GitHub Actions
- CODEOWNERS file for automatic review requests

## [1.0.0] - 2025-12-26

### Added

- MIT LICENSE file
- SECURITY.md with vulnerability reporting policy
- CONTRIBUTING.md with contribution guidelines
- CI status badge in README
- Example inventory with placeholder IPs (`hosts.yml.example`)
- Templated Grafana password in Prometheus Helm values

### Changed

- Moved `prometheus.yml` to `templates/prometheus-values.yml.j2` for variable substitution
- Sanitized password examples in INSTALL.md documentation
- Updated .gitignore patterns

### Fixed

- Release workflow for repositories with few commits

## [0.1.0] - 2025-12-24

### Added

- Initial infrastructure-as-code for K3s on Turing Pi RK1
- Terraform modules for BMC flashing operations
- Ansible playbooks for cluster deployment:
  - `bootstrap.yml` - OS preparation
  - `kubernetes.yml` - K3s installation
  - `addons.yml` - Helm chart deployment
  - `npu-setup.yml` - RKNN runtime installation
- Ansible roles:
  - `base` - System configuration
  - `k3s-prereq` - K3s prerequisites and NVMe setup
  - `k3s-server` - Control plane installation
  - `k3s-agent` - Worker node installation
  - `metallb` - L2 load balancer
  - `nginx-ingress` - Ingress controller
  - `prometheus-stack` - Monitoring
  - `longhorn` - Distributed storage
  - `portainer` - Container management
  - `rknn` - NPU runtime
- GitHub Actions CI workflow (lint, syntax check, terraform validate)
- GitHub Actions release workflow
- Comprehensive installation guide (INSTALL.md)
- Implementation documentation (docs/IMPLEMENTATION.md)

[Unreleased]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.6.0...HEAD
[1.6.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.5.0...v1.6.0
[1.5.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.4.0...v1.5.0
[1.4.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.6...v1.4.0
[1.3.6]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.5...v1.3.6
[1.3.5]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.4...v1.3.5
[1.3.4]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.3...v1.3.4
[1.3.3]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.2...v1.3.3
[1.3.2]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.1...v1.3.2
[1.3.1]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.3.0...v1.3.1
[1.3.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.2.1...v1.3.0
[1.2.1]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.2.0...v1.2.1
[1.2.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.9...v1.2.0
[1.1.9]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.8...v1.1.9
[1.1.8]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.7...v1.1.8
[1.1.7]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.6...v1.1.7
[1.1.6]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.5...v1.1.6
[1.1.5]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.4...v1.1.5
[1.1.4]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.3...v1.1.4
[1.1.3]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.2...v1.1.3
[1.1.2]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.1...v1.1.2
[1.1.1]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.1.0...v1.1.1
[1.1.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/compare/v0.1.0...v1.0.0
[0.1.0]: https://github.com/freed-dev-llc/turing-ansible-cluster/releases/tag/v0.1.0

# Potential Contributions

Generated from a comparison of the `kubernetes-sigs/agent-sandbox` roadmap with the current repository state and live GitHub issues/PRs on 2026-07-21.

## Snapshot

- Open issues: 114 total - P0: 2, P1: 20, P2: 44, P3: 38, P4: 10.
- Open PRs: 100 total, so several roadmap items already have active implementation work.
- Main takeaway: the roadmap is directionally accurate, but several items are only partially implemented, many are already covered by open PRs, and at least one "Completed" item still has an implementation gap.

## Roadmap vs current state

| Roadmap area | Current state | Gap |
| --- | --- | --- |
| Portable backend / decouple API from runtime | KEP and `packages/sandboxd/spec/process/v1/process.proto` exist. | No fully wired runtime backend abstraction or reference daemon path yet. Mostly protocol/design groundwork. |
| WarmPool rolling updates / smart selection | `SandboxWarmPool.spec.updateStrategy` supports `Recreate` / `OnReplenish`; claim selection has NodeSpread-like logic. | Still fragile: adoption, race, over-create, and stale-template bugs remain open (#418, #478, #764, #1215, #1229). |
| First-class router | Go `sandbox-router/` exists with TLS, mTLS, metrics, and cache docs. | Not obviously first-class in Helm/k8s install; official image, packaging, and routing gaps remain (#1058, #883, #1109). |
| Auto suspend/resume / scale-to-zero | Manual `spec.operatingMode: Suspended` and `shutdownTime` / TTL exist. | No completed automatic idle detection or traffic/API-triggered resume. Active work: #968, #849, PRs #970, #972, #1160. |
| TypeScript SDK | No TypeScript SDK in the local tree. | Active PR #976; contribution should likely be review, tests, or docs rather than starting a duplicate SDK. |
| MCP server | Only `examples/mcp-server-sandbox/` exists locally. | First-class MCP server is active in PR #1141; not merged locally. |
| Networking, storage, tenancy | Template-level NetworkPolicy, claim env injection, and claim `volumeClaimTemplates` exist. | Claim-time dynamic NetworkPolicy attach, identity association, and polished SDK storage management remain open (#818, #643, #554). |
| Performance / observability | Controller metrics and load/stress infrastructure exist. | Continuous benchmark automation, lifecycle/stage metrics, ServiceMonitor, and PrometheusRule support are incomplete or in-flight (#403, #624, #686, #1140, #1155, #1016, #1138). |
| v1beta1 migration | v1beta1 exists; v1alpha1 is still registered for migration. | Some examples still use `extensions.agents.x-k8s.io/v1alpha1`, especially `examples/latebind-storage-gke-sandbox`. |
| Docs, UI, community | Strong docs/site coverage. | UI is absent (#697); API default/runtime behavior docs remain open (#1104, #696, PR #1106). |

## High-signal contribution opportunities

| Issue | Why it is worth considering | Notes |
| --- | --- | --- |
| #154 Headless Service should set ports when `containerPort` is set | Clear P1 bug, and the roadmap marks "Headless Service Port Handling" as completed while `controllers/sandbox_controller.go::reconcileService` still creates Services without `spec.ports`. | Add service port derivation, drift reconciliation, and unit tests. |
| #1229 Conflict during annotation-path adoption completion falls through and adopts a different sandbox | Fresh, narrow correctness bug in the `SandboxClaim` adoption path. | Likely a surgical controller fix plus a regression test. |
| #1201 Python SDK multi-cluster config clobbering | High-impact P1 SDK bug for multi-cluster consumers. | Assigned; coordinate before taking. Likely fix is avoiding global Kubernetes config state. |
| #403 Continuous benchmarking and tracking performance | P1, `help wanted`, unassigned, and directly aligned with the performance roadmap. | Good slice: automate reporting around existing load/stress recipes. |
| #320 Python client package missing comprehensive type annotations | Unassigned `help wanted`, contained SDK quality work. | Add annotations and a `py.typed` marker carefully, without formatter churn. |
| #554 Support for managing persistent volumes via SDK | Roadmap-aligned storage gap; API support is partially present already. | Assigned; coordinate. Opportunity is SDK models/docs/parity, not CRD basics. |
| #1140 / #1155 Prometheus metrics gaps | Good observability roadmap work. | Coordinate with active metrics PRs #1076, #1110, #1111, #1143, and #1247. |
| #1058 Publish official sandbox-router image | Helps make the Go router truly first-class. | Packaging/release/Helm-oriented contribution. |

## Avoid duplicating active PR work

Check ownership before starting work on these areas:

- TypeScript SDK (#977) - active PR #976.
- MCP server - active PR #1141.
- Go client `WarmPoolName` validation bug (#1119) - active PR #1179.
- Router NXDOMAIN / warm-pool routing issue (#883) - active PRs #1239 and #1246.
- ServiceMonitor support (#1016) - active PR #1017.
- Idle lifecycle / auto suspend-resume (#968, #849) - active PRs #970, #972, and #1160.

## Concrete mismatch to verify first

The roadmap marks "Headless Service Port Handling" as completed, but the current `reconcileService` path creates a headless Service with `ClusterIP: None` and a selector only; it does not derive `spec.ports` from container ports. Issue #154 remains open and looks like the cleanest first contribution.

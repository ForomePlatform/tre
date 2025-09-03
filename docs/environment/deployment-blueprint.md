# Deployment Blueprint

The TRE can be deployed as a pilot for a single project, as an institutional service, or as a federated arrangement across organisations.

**Reference components**
- Compute (on-demand and batch), secure storage tiers, and network isolation.
- Identity federation and single sign-on; role-based permissions.
- Logging and monitoring; backup and disaster recovery.
- Cost and capacity management.

**Decision table**

| Scenario | Footprint | Notes |
|---|---|---|
| Pilot / small team | Single-project | Fast start; limited operations overhead. |
| Many teams / shared service | Institutional multi-tenant | Central governance; quotas; standard toolchain. |
| Cross-institution collaboration | Federated TREs | Policy alignment; shared governance; no raw data movement. |

??? info "Operations summary"
    - Monitoring, incident response, and security patching.
    - Data lifecycle management: ingest, curate, archive; retention policies.
    - Performance tuning and capacity planning.

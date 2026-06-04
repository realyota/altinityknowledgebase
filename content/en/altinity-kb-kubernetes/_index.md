---
title: "Using the Altinity Kubernetes Operator for ClickHouse®"
linkTitle: "Using the Altinity Kubernetes Operator for ClickHouse®"
keywords:
- clickhouse in kubernetes
- kubernetes issues
- ALtinity Kubernetes operator for ClickHouse
description: >
    Run ClickHouse® in Kubernetes without any issues.
weight: 8
aliases: 
  /altinity-kb-kubernetes/altinity-kb-possible-issues-with-running-clickhouse-in-k8s/
---

## Useful links

The Altinity Kubernetes Operator for ClickHouse® is a mature, production-tested operator for running ClickHouse® clusters on Kubernetes. It is used in managed and self-managed deployments, including large installations with hundreds of ClickHouse® nodes, and helps automate day-2 operations such as configuration rendering, rolling updates, scaling, storage lifecycle, monitoring integration, and Keeper/ZooKeeper-aware schema maintenance.

The Altinity Kubernetes Operator for ClickHouse® repo has very useful documentation: 

- [Installing the Operator](https://docs.altinity.com/altinitykubernetesoperator/quickstartinstallation/)
- [Quick Start Guide](https://github.com/Altinity/clickhouse-operator/blob/master/docs/quick_start.md)
- [Operator Custom Resource Definition explained](https://github.com/Altinity/clickhouse-operator/blob/master/docs/custom_resource_explained.md)
- [Examples - YAML files to deploy the operator in many common configurations](https://github.com/Altinity/clickhouse-operator/tree/master/docs/chi-examples)
- [Main documentation](https://github.com/Altinity/clickhouse-operator/tree/master/docs#table-of-contents)

### Official ClickHouse® Inc. Operator vs Altinity Operator

The Altinity Kubernetes Operator for ClickHouse® is the more mature, production-proven, out-of-the-box operator with a rich and well-tested feature set. It is widely used to run and manage ClickHouse® clusters on Kubernetes, including large enterprise deployments. Altinity maintains this operator, and it is also the foundation of Altinity.Cloud, so the same operator model is battle-tested in production.

The official ClickHouse(R) Kubernetes Operator is a different project maintained by ClickHouse Inc. It is not a drop-in replacement for the Altinity operator: the APIs, CRDs, object names, feature set, and operational model are different. The official operator was announced on January 29, 2026, uses the `clickhouse.com/v1alpha1` API, and its public release history is still young. The ClickHouse GitHub releases page currently shows `v0.0.5` as the latest release as of June 2, 2026. By comparison, the Altinity operator repository was created on January 10, 2019, and currently has 9,078 commits and 116 GitHub contributors.

Because the official ClickHouse® Inc. operator is not managed by Altinity, Altinity Support cannot provide full support coverage or commit to fixing bugs in that operator. Use the Altinity operator when you need Altinity-supported production operation.

Taking the above into account, the Altinity operator is the recommended choice for production deployments that require:

- long-term maturity and stability with an established release history;
- full Altinity Support coverage, including troubleshooting and bug fixes;
- support for any ClickHouse and ClickHouse Keeper build or version that is compatible with the selected operator release, including Altinity Stable Builds, official ClickHouse images, custom/private images, and older ClickHouse versions when paired with the appropriate operator version;
- a richer CRD model, including CHI, CHIT, CHOP, and CHK resources;
- a broader production feature set that is not exposed by the official operator's current `clickhouse.com/v1alpha1` API as of June 3, 2026, including reusable CHI templates and auto-templates, shard/replica/host-level overrides, service templates at CHI/cluster/shard/replica levels, pod distribution policies, schema maintenance during scale-out, storage-management controls, PDB management, custom macros, hooks, operator runtime configuration through CHOP, metrics-exporter integration, bundled Prometheus/Grafana dashboards and alerts, and Keeper/ZooKeeper auto-binding or explicit configuration;
- proven use in large-scale enterprise environments and as the foundation of Altinity.Cloud.

See the [Altinity operator repository](https://github.com/Altinity/clickhouse-operator), the [Altinity operator releases](https://github.com/Altinity/clickhouse-operator/releases), and the [Altinity operator feature overview](https://altinity.com/kubernetes-operator/).

## Installing the operator

Use this KB section only as an entry point. The operator repository is the source of truth for installation commands, supported Kubernetes versions, Helm values, runbooks, and full YAML examples.

The operator uses standard Kubernetes APIs and is not tied to one Kubernetes vendor or distribution. It can run on managed Kubernetes services such as AKS, EKS, and GKE, and on self-managed or lightweight Kubernetes distributions such as k3s, as long as the cluster provides the supported Kubernetes API version plus the required CRDs, RBAC, StatefulSets, Services, persistent storage, and networking behavior.

The operator is not tied to a specific ClickHouse® server binary or image; choose the ClickHouse® version that matches your compatibility and upgrade requirements. For production deployments, Altinity recommends [Altinity Stable Builds for ClickHouse®](https://docs.altinity.com/altinitystablebuilds/). See also [Why Every ClickHouse® User Should Appreciate Altinity Stable Builds](https://altinity.com/blog/why-every-clickhouse-user-should-appreciate-altinity-stable-builds).

Suggested path:

1. Check the Kubernetes version before selecting manifests or a Helm chart. The current operator line documents Kubernetes `1.25+` as the supported baseline; operator versions `0.16.0` and newer are documented for Kubernetes `1.25` and newer. For older Kubernetes clusters, use the compatibility table in the repo [installation details](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_installation_details.md).
2. Start with the [Quick Start Guide](https://github.com/Altinity/clickhouse-operator/blob/master/docs/quick_start.md) if you need a minimal `kubectl apply` installation and a first test `ClickHouseInstallation`.
3. Use the Altinity docs [Installing the Operator](https://docs.altinity.com/altinitykubernetesoperator/quickstartinstallation/) page when choosing between Helm and `kubectl`, and the [Upgrading the Operator](https://docs.altinity.com/altinitykubernetesoperator/upgrade/) page for upgrade paths. Use the repo [installation details](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_installation_details.md) when checking version-specific manifest URLs and lower-level Kubernetes resources.
4. Review the [operator configuration documentation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_configuration.md) before changing operator-level behavior such as watched namespaces, reconcile settings, metrics collection, and access credentials. In Kubernetes deployments, change them through operator Helm values, the install manifest that renders `/etc/clickhouse-operator/config.yaml`, or `ClickHouseOperatorConfiguration`, then restart the operator when the change is not handled by `watch.configuration.onChange: restart`.
5. Decide which namespaces the operator should watch. The operator [watch namespace configuration](https://github.com/Altinity/clickhouse-operator/blob/ff1f18a8e1950eb48cd2595c5ce4a1e6653ca878/config/config.yaml#L21) supports `include` and `exclude` namespace lists or regexp patterns. Empty `include` watches the operator's own namespace, or all namespaces when the operator runs in `kube-system`. If you run multiple operator instances in the same Kubernetes cluster, configure them to watch different namespaces so they do not reconcile the same resources.
6. Keep operator runtime configuration separate from CHI/CHK default ClickHouse/Keeper configuration. Operator behavior is configured through Helm values or install manifests that render `/etc/clickhouse-operator/config.yaml`, plus optional `ClickHouseOperatorConfiguration` resources; see [Operator configuration](#operator-configuration) below for the merge order and restart behavior.
7. Review the default CHI and CHK configuration shipped with the operator before overriding it. The defaults live in [`config/chi`](https://github.com/Altinity/clickhouse-operator/tree/master/config/chi) and [`config/chk`](https://github.com/Altinity/clickhouse-operator/tree/master/config/chk). The operator [`config.yaml` ClickHouse paths](https://github.com/Altinity/clickhouse-operator/blob/master/config/config.yaml#L48) and [Keeper paths](https://github.com/Altinity/clickhouse-operator/blob/master/config/config.yaml#L277) map those folders as common, host-specific, and user configuration paths. In [`ClickHouseOperatorConfiguration`](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/70-chop-config.yaml#L30), `spec.clickhouse.configuration.file.path` changes which folders the operator reads. To change the actual default XML/YAML content, use the operator Helm [`configs.*` values](https://github.com/Altinity/clickhouse-operator/blob/master/deploy/helm/clickhouse-operator/values.yaml#L210) such as `configs.configdFiles`, `configs.confdFiles`, and `configs.usersdFiles`, or override per installation with CHI/CHK settings and templates. Do not edit generated files inside ClickHouse® pods.
8. For Helm-based installs, choose the chart that matches the ownership model:
   - Use the [operator Helm chart](https://github.com/Altinity/clickhouse-operator/tree/master/deploy/helm/clickhouse-operator) when you want to install and upgrade only the operator. The chart installs and updates CRDs by default through the `crdHook`; if `crdHook.enabled: false` is used, apply the CRDs manually from the chart's [`crds/`](https://github.com/Altinity/clickhouse-operator/tree/master/deploy/helm/clickhouse-operator/crds) directory.
   - Use the [Altinity Helm charts](https://github.com/Altinity/helm-charts) when you want to manage ClickHouse, optional Keeper, and optionally the operator from a higher-level chart. The `clickhouse` chart includes the operator Helm chart as a dependency/subchart, so the operator can be configured separately through the parent chart values. `clickhouse-eks` is an EKS-specific high-availability chart.
9. Read the [Custom Resource explanation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/custom_resource_explained.md) before writing production CHI YAML. Use the [CHI examples](https://github.com/Altinity/clickhouse-operator/tree/master/docs/chi-examples) as validated starting points, then adapt storage, services, security, templates, and topology to the target cluster.
10. If the cluster needs replication, prefer [ClickHouse® Keeper](https://kb.altinity.com/altinity-kb-setup-and-maintenance/altinity-kb-zookeeper/clickhouse-keeper/) for new deployments. The operator supports `ClickHouseKeeperInstallation` (CHK), and CHI can reference CHK directly through the [Keeper reference](https://github.com/Altinity/clickhouse-operator/blob/master/docs/keeper_reference.md). Use the older [ZooKeeper setup](https://github.com/Altinity/clickhouse-operator/blob/master/docs/zookeeper_setup.md) only when you intentionally operate an external ZooKeeper ensemble.
11. If you already run CHK on operator `0.23.x`, do not upgrade directly to `0.24.0` or newer without the documented [Keeper migration procedure](https://github.com/Altinity/clickhouse-operator/blob/master/docs/keeper_migration_from_23_to_24.md).
12. For high availability, design the ClickHouse® cluster before applying YAML. Use the Altinity KB [ClickHouse® High Availability Architecture](https://docs.altinity.com/operationsguide/availability-and-recovery/availability-architecture/) guidance for replication, redundancy, failover, and replica placement across failure domains. In operator terms, this usually means replicated tables, more than one replica per shard, persistent volumes, Keeper/CHK, and `podDistribution` or zone rules so replicas do not land on the same node or availability zone. Useful operator examples: [Keeper reference with replication](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/04-replication-zookeeper-07-keeper-ref.yaml), [persistent-volume replication](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/04-replication-zookeeper-03-minimal-AWS-persistent-volume.yaml), [pod per host](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/10-zones-01-simple-02-aws-pod-per-host.yaml), and [zones distribution](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/14-zones-distribution-01.yaml).
13. For day-2 operations, follow the dedicated operator docs for the full procedure and version-specific details: [operator upgrade](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_upgrade.md) and [storage](https://github.com/Altinity/clickhouse-operator/blob/master/docs/storage.md).

### Operator configuration

Use the operator [configuration documentation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_configuration.md) when changing operator behavior.

The operator builds its runtime settings from these inputs, in order:

1. File-based config. In Kubernetes this is normally `/etc/clickhouse-operator/config.yaml`, mounted from the operator `*-files` ConfigMap rendered by the install bundle or Helm chart.
2. `ClickHouseOperatorConfiguration` resources, for example the repo [70-chop-config.yaml](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/70-chop-config.yaml).

Later inputs merge with earlier inputs. The operator reads the file-based config and `ClickHouseOperatorConfiguration` resources on startup. Changes to `ClickHouseOperatorConfiguration` can trigger an operator restart when `watch.configuration.onChange: restart` is set; otherwise restart the operator pod manually after changing runtime configuration.

Do not confuse this with ClickHouse/Keeper server ConfigMaps. Those are generated during CHI/CHK reconciliation from the default files, CHI/CHK settings, files, and templates, then mounted into ClickHouse/Keeper pods.

Restart examples:

```bash
kubectl -n <operator-namespace> rollout restart deployment/clickhouse-operator
kubectl -n <operator-namespace> rollout status deployment/clickhouse-operator

# Deleting the operator pod is also fine; in the standard install the Deployment/ReplicaSet recreates it.
kubectl -n <operator-namespace> delete pod -l app=clickhouse-operator
kubectl -n <operator-namespace> rollout status deployment/clickhouse-operator
```

Example:

```yaml
apiVersion: clickhouse.altinity.com/v1
kind: ClickHouseOperatorConfiguration
spec:
  clickhouse:
    metrics:
      timeouts:
        collect: 9
  reconcile:
    runtime:
      reconcileCHIsThreadsNumber: 10
      reconcileShardsThreadsNumber: 5
```

### Security

Review the operator [security hardening guide](https://github.com/Altinity/clickhouse-operator/blob/master/docs/security_hardening.md) before using a CHI in production. Do this during installation design, not after exposing services.

- ClickHouse® has a built-in `default` user, and the operator keeps it available for ClickHouse® and distributed-query compatibility unless you override or remove it. The operator adds restrictive network rules for that user. For production, disable the `default` user when possible, or set a password and configure inter-cluster communication with a cluster secret so distributed queries do not depend on exposed user credentials.
- Keep the operator's own `clickhouse_operator` credentials in a Kubernetes Secret. Do not put the username and password directly in the operator config unless there is a specific reason.
- For ClickHouse® users, prefer `valueFrom.secretKeyRef` for `password`, `password_sha256_hex`, or `password_double_sha1_hex`, or provide password hashes explicitly. The operator accepts a plaintext `password` and normalizes it to `password_sha256_hex` before rendering ClickHouse® config, but the plaintext value still remains in the submitted CHI YAML and Kubernetes object. The older `k8s_secret_*` and `k8s_secret_env_*` user settings are still handled by the normalizer but are deprecated; use `valueFrom.secretKeyRef` in new manifests.
- Store sensitive server settings, certificates, and external-system credentials in Kubernetes Secrets. The operator can map them into ClickHouse® configuration through environment variables or secret-backed files.

  Example:

  ```yaml
  spec:
    configuration:
      settings:
        s3/my_bucket/secret_access_key:
          valueFrom:
            secretKeyRef:
              name: s3-credentials
              key: AWS_SECRET_ACCESS_KEY
      files:
        server.crt:
          valueFrom:
            secretKeyRef:
              name: ssl-files
              key: server.crt
  ```

- For network hardening, plan TLS and service exposure explicitly. `secure: "yes"` enables secure inter-node configuration for the cluster, `insecure: "no"` disables insecure pod-service ports, but load balancer services still need explicit `serviceTemplate` ports.

### Monitoring

Use the operator [monitoring](https://github.com/Altinity/clickhouse-operator/blob/master/docs/monitoring_setup.md), [Prometheus setup](https://github.com/Altinity/clickhouse-operator/blob/master/docs/prometheus_setup.md), and [Grafana setup](https://github.com/Altinity/clickhouse-operator/blob/master/docs/grafana_setup.md) docs when adding observability. The operator exposes metrics through the `clickhouse-operator-metrics` ClusterIP service in the operator namespace. The Prometheus setup doc covers integrating an existing Prometheus or deploying one for the operator, and the Grafana setup doc covers dashboards and the Prometheus datasource. The operator also ships ready-to-use [Grafana dashboards](https://github.com/Altinity/clickhouse-operator/tree/master/deploy/helm/clickhouse-operator/files), [Grafana Operator dashboard manifests](https://github.com/Altinity/clickhouse-operator/tree/master/deploy/grafana/grafana-with-grafana-operator), and [Prometheus alert rules](https://github.com/Altinity/clickhouse-operator/tree/master/deploy/prometheus). Plan this during installation so scrape endpoints, namespaces, ServiceMonitor settings, alert rules, and dashboards match how the operator is deployed.

The standard install runs a `metrics-exporter` container next to the operator. It reads the operator configuration, connects to watched ClickHouse® hosts with the configured operator access credentials, queries selected `system` tables, and exposes ClickHouse® metrics on the `ch-metrics` port `8888`. The same `clickhouse-operator-metrics` service also exposes operator process metrics on `op-metrics` port `9999`.

Metrics collection is configurable in the operator config under `clickhouse.metrics` or in `ClickHouseOperatorConfiguration` under `spec.clickhouse.metrics`. Use it to tune the collection timeout, select which `system` tables are scraped with `tablesRegexp`, and exclude noisy metrics with `excludeRegexp`; see the [operator configuration example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/70-chop-config.yaml#L91).

The metrics exporter also supports custom ClickHouse® metrics through `system.custom_metrics`. By default, `tablesRegexp: "^(metrics|custom_metrics)$"` makes the exporter read both `system.metrics` and `system.custom_metrics`. A `system.custom_metrics` view or table must expose at least `metric` and `value` columns; each row is exported as a Prometheus gauge named `chi_clickhouse_metric_<metric>`. For example, `SELECT 'OrdersPending' AS metric, toFloat64(count()) AS value FROM orders WHERE status = 'pending'`. Keep custom metrics cheap and deterministic because the exporter queries them on every watched ClickHouse® host during metrics collection.

## Cluster lifecycle

Use this KB section as an operational checklist for common day-2 actions. Follow the [Altinity Kubernetes Operator documentation](https://docs.altinity.com/altinitykubernetesoperator/) and the linked repo docs for full procedures, manifests, and version-specific behavior.

- Manage ClickHouse® clusters through `ClickHouseInstallation` and `ClickHouseKeeperInstallation` resources. Do not edit generated StatefulSets, Services, or ConfigMaps as the primary workflow; the operator reconciles them from the custom resources.
- For scaling, change the CHI layout, templates, or settings intentionally and let the operator reconcile the cluster. Before scaling down, check replication health, Keeper/ZooKeeper health, where pods are scheduled, and the schema maintenance behavior below.
- For upgrades, plan the operator upgrade, CRD/manifests, and ClickHouse® server version separately. Use the operator [upgrade documentation](https://docs.altinity.com/altinitykubernetesoperator/upgrade/) for the operator path and the ClickHouse® release notes for server compatibility.
- For ClickHouse® server upgrades, the operator makes the mechanical part straightforward: update the CHI pod template image and let reconciliation apply the change. Review [Altinity Stable Builds release notes](https://docs.altinity.com/releasenotes/altinity-stable-release-notes/) and any backward-incompatible changes before upgrading. By default, server changes are applied through rolling reconciliation rather than replacing the whole cluster at once. The first shard is always reconciled alone, which gives an early canary signal; concurrency starts from the second shard and is limited by `reconcileShardsThreadsNumber` and `reconcileShardsMaxConcurrencyPercent`. Remember that higher concurrency means more shards or hosts are being changed at the same time. See the [ClickHouse® version upgrade example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi_update_clickhouse_version.md) for the pod-template update pattern.
- For configuration changes, check the operator [configuration restart policy](https://github.com/Altinity/clickhouse-operator/blob/master/config/config.yaml#L100). It controls which ClickHouse® configuration changes require a server restart during reconciliation and which changes can be applied without restarting ClickHouse®.
- For large clusters, reconciliation can take a long time. Tune [reconciliation runtime and StatefulSet update settings](https://github.com/Altinity/clickhouse-operator/blob/master/config/config.yaml#L354) if reconciliation takes too long or hits update timeouts.
- To explicitly trigger reconciliation for a specific CHI/CHK, update `spec.taskID` to any new value. This is useful after changes that are accepted by the operator but do not automatically enqueue every affected custom resource, for example shared template changes.
- For storage changes, treat PVCs and PVs as part of the cluster design. Production ClickHouse® and Keeper/CHK deployments need persistent `volumeClaimTemplates`, and the StorageClass/PV reclaim policy must be set to the required data-retention behavior, for example `Retain` when volumes must not be deleted with the claim. Use the operator [storage documentation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/storage.md) for ClickHouse® PVC templates, the [CHK examples](https://github.com/Altinity/clickhouse-operator/tree/master/docs/chk-examples) for Keeper PVC templates, the Altinity blog on [preventing ClickHouse® storage deletion with reclaimPolicy](https://altinity.com/blog/preventing-clickhouse-storage-deletion-with-the-altinity-kubernetes-operator-reclaimpolicy), and the CSI driver's storage class rules before resizing volumes or changing storage behavior.
- For storage expansion, first verify that the StorageClass and CSI driver allow PVC expansion, for example `allowVolumeExpansion: true`. For operator-managed PVC lifecycle operations such as volume rescale, set `spec.defaults.storageManagement.provisioner: Operator` instead of relying on the default `StatefulSet` provisioner. Then increase `resources.requests.storage` in the CHI or CHK `volumeClaimTemplates` and let Kubernetes and the operator reconcile the PVCs. Expansion is one-way in Kubernetes; shrinking PVCs is not supported. See the operator [resizable volume example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/03-persistent-volume-05-resizeable-volume-1.yaml) and [storage-management rescale manifest](https://github.com/Altinity/clickhouse-operator/blob/master/tests/e2e/manifests/chi/test-021-2-rescale-volume-02-enlarge-disk.yaml) for the pattern.

### Error handling during reconciliation

Use the operator [ClickHouse configuration errors handling](https://github.com/Altinity/clickhouse-operator/blob/master/docs/clickhouse_config_errors_handling.md) doc when a create or rolling update fails because ClickHouse cannot start, an image is invalid, settings are invalid, or a pod template prevents pods from becoming ready.

During create and rolling update operations, the operator waits for the affected StatefulSet to reach `Ready`. If it does not become ready within `statefulSetUpdateTimeout`, the operator uses its failure-action settings:

- `onStatefulSetCreateFailureAction`: `abort` or `delete` for a newly created StatefulSet.
- `onStatefulSetUpdateFailureAction`: `abort` or `rollback` for an updated StatefulSet.

The operator aborts the current rolling process after a failed StatefulSet and leaves the next action to the administrator. Check the ClickHouse logs, pod events, generated ConfigMaps, and the CHI/CHK diff before retrying the reconcile.

Examples:

```bash
# Trigger a new reconcile for a CHI by changing taskID.
kubectl -n databases patch chi <name> --type=merge -p '{"spec":{"taskID":"random-uuid-or-name"}}'

# Or edit the CHI directly and change spec.taskID.
kubectl -n <namespace> edit chi <name>

# Force a rolling ClickHouse® restart during reconcile.
kubectl -n <namespace> patch chi <name> --type=merge -p '{"spec":{"restart":"RollingUpdate"}}'
```

Use `spec.restart: RollingUpdate` only when a forced rolling restart is required. The operator treats it as a restart policy during reconcile, so remove it after the intended restart to avoid unnecessary restarts on later reconciliations.

### Backups

The operator does not replace a backup system. Plan backups as a separate workflow and make it observable, scheduled, and restore-tested. Common operator deployments use [`clickhouse-backup`](https://github.com/Altinity/clickhouse-backup) as a sidecar in every ClickHouse® pod, then trigger backup commands with Kubernetes `Job` or `CronJob` resources. Track the backup tool version separately from the ClickHouse® server version; see [Altinity Backup release notes](https://docs.altinity.com/releasenotes/altinity-backup-release-notes/) before upgrades.

Use a CHI/CHIT `podTemplate` when the backup process must run next to ClickHouse® and share the same pod lifecycle. The operator repository has a ready example in [`tpl-clickhouse-backups.yaml`](https://github.com/Altinity/clickhouse-operator/blob/master/tests/e2e/manifests/chit/tpl-clickhouse-backups.yaml) and a matching CHI in [`test-cluster-for-backups.yaml`](https://github.com/Altinity/clickhouse-operator/blob/master/tests/e2e/manifests/chi/test-cluster-for-backups.yaml). The sidecar example runs `clickhouse-backup server`, enables the API on port `7171`, enables `/metrics`, creates integration tables, and configures S3-compatible remote storage.

Minimal sidecar shape:

```yaml
spec:
  templates:
    podTemplates:
      - name: clickhouse-with-backup
        metadata:
          annotations:
            clickhouse.backup/scrape: "true"
            clickhouse.backup/port: "7171"
            clickhouse.backup/path: /metrics
        spec:
          containers:
            - name: clickhouse-pod
              image: altinity/clickhouse-server:25.8.16.10001.altinitystable
            - name: clickhouse-backup
              image: altinity/clickhouse-backup:2.4.15
              command:
                - bash
                - -xc
                - /bin/clickhouse-backup server
              env:
                - name: API_LISTEN
                  value: "0.0.0.0:7171"
                - name: API_ENABLE_METRICS
                  value: "true"
                - name: API_CREATE_INTEGRATION_TABLES
                  value: "true"
                - name: REMOTE_STORAGE
                  value: s3
                - name: S3_BUCKET
                  value: clickhouse-backup
              ports:
                - name: backup-rest
                  containerPort: 7171
```

Put remote storage credentials, encryption keys, and backup user passwords in Kubernetes Secrets and mount or reference them from the pod template. Do not store S3 keys or backup passwords as plaintext values in reusable CHI/CHIT manifests.

Use `CronJob` for scheduled backups and one-off `Job` resources for manual or migration backups. The job should create a unique backup name, submit the command through the sidecar API or the `clickhouse-backup` integration tables such as `system.backup_actions`, wait for a terminal success/error state, and exit non-zero on failure. For remote backups, trigger a full remote operation such as `create_remote`, or run `create` followed by `upload`; do not treat a local-only backup under `/var/lib/clickhouse/backup` as a durable backup. With integration tables enabled, a Job can use `clickhouse-client` to submit and check work through ClickHouse®:

```sql
INSERT INTO system.backup_actions(command)
VALUES ('create_remote backup_20260602_010000');

SELECT command, status, error
FROM system.backup_actions
WHERE command = 'create_remote backup_20260602_010000';
```

Monitor the backup sidecar separately from ClickHouse®. The repository Prometheus example has a dedicated `kubernetes-clickhouse-backup-pods` scrape job based on the `clickhouse.backup/*` pod annotations, and the backup alert rules cover a down sidecar, recent restart, failed backup, abnormal duration, backup size changes, and unexpected local backup leftovers. See [`prometheus-template.yaml`](https://github.com/Altinity/clickhouse-operator/blob/master/deploy/prometheus/prometheus-template.yaml) and [`prometheus-alert-rules-backup.yaml`](https://github.com/Altinity/clickhouse-operator/blob/master/deploy/prometheus/prometheus-alert-rules-backup.yaml).

For replicated clusters, define the backup topology deliberately. Usually this means backing up at least one healthy replica per shard, not every replica blindly, but the exact policy depends on local tables, replicated tables, distributed tables, RBAC, dictionaries, and object-storage layout. Always test restore into a separate namespace or cluster before relying on the schedule.

### Managing CHI and CHK resources

Manage ClickHouse® and Keeper through `ClickHouseInstallation` and `ClickHouseKeeperInstallation` specs. The operator renders Kubernetes resources and ClickHouse®/Keeper XML from those custom resources, so changes should be made in CHI/CHK YAML, templates, or operator configuration.

For XML settings generated from CHI/CHK paths, the special scalar values `_remove_` and `_removed_` are rendered as `remove="1"`. Use this when a setting inherited from the default config has to be removed rather than overridden with another value.

Example:

```yaml
spec:
  configuration:
    settings:
      logger/level: _remove_
```

The operator renders this as:

```xml
<logger>
  <level remove="1"/>
</logger>
```

### CHI and CHK templates

Use templates when the same pod, service, storage, or host settings must be reused across many CHI or CHK resources. CHI supports inline templates under `spec.templates` and reusable `ClickHouseInstallationTemplate` resources. CHK supports the same pattern for Keeper pod and volume templates.

Common template types:

- `podTemplates` define ClickHouse®/Keeper pod shape: image, resources, node selectors, tolerations, affinity, security context, sidecars, and custom volume mounts.
- `volumeClaimTemplates` define persistent data or log PVCs.
- `serviceTemplates` define Kubernetes Services and load balancer settings.
- `hostTemplates` define ClickHouse® host-level settings such as ports.

Use `spec.defaults.templates` to select defaults for the whole CHI/CHK, and override them at cluster, shard, or replica level when needed. Use `spec.useTemplates` to merge reusable `ClickHouseInstallationTemplate` resources into a CHI. Manual templates are referenced explicitly; auto templates can be selected by template policy and selectors.

Example:

```yaml
apiVersion: clickhouse.altinity.com/v1
kind: ClickHouseInstallation
spec:
  useTemplates:
    - name: clickhouse-version
  defaults:
    templates:
      podTemplate: clickhouse-pod
      dataVolumeClaimTemplate: data
  templates:
    podTemplates:
      - name: clickhouse-pod
        spec:
          containers:
            - name: clickhouse-pod
              image: altinity/clickhouse-server:25.8.16.10001.altinitystable
    volumeClaimTemplates:
      - name: data
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 500Gi
```

Template changes do not by themselves mean every existing CHI/CHK is immediately reconciled. The operator template policy in [`config.yaml`](https://github.com/Altinity/clickhouse-operator/blob/master/config/config.yaml#L292) is `ApplyOnNextReconcile` by default: template updates are accepted, but applied on the next regular reconcile of the matching CHI/CHK. If you need the change now, manually trigger reconciliation by updating the CHI/CHK object, for example by changing an annotation, or by applying the CHI/CHK manifest again with the intended spec. With `ReadOnStart`, template updates are only read when the operator starts.

See the operator [template documentation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_configuration.md#defaults-for-clickhouseinstallation), [applying template changes](https://github.com/Altinity/clickhouse-operator/blob/master/docs/operator_configuration.md#applying-changes-from-clickhouseinstallationtemplates), and [CHIT examples](https://github.com/Altinity/clickhouse-operator/tree/master/docs/chit-examples).

### Operator pod distribution and affinities

Use `spec.templates.podTemplates[].podDistribution` to ask the operator to generate Kubernetes pod affinity or anti-affinity rules for ClickHouse® pods. This belongs to the pod template, so it can be reused through CHI inline templates or `ClickHouseInstallationTemplate` resources. It is not a global operator setting.

Each rule has:

- `type`: distribution rule type.
- `scope`: boundary for matching pods. The CRD accepts `Shard`, `Replica`, `Cluster`, `ClickHouseInstallation`, and `Namespace`; empty scope is normalized to `Cluster` for the main ClickHouse®/shard/replica anti-affinity rules.
- `topologyKey`: Kubernetes topology label used by scheduler affinity rules. If omitted, the operator uses `kubernetes.io/hostname`, so the rule applies per node. Use zone labels such as `topology.kubernetes.io/zone` when the placement rule should apply per availability zone instead.
- `number`: only for `MaxNumberPerNode`; negative values are normalized to `0`.

Common anti-affinity types:

- `ClickHouseAntiAffinity`: pushes ClickHouse® pods away from other ClickHouse® pods, usually to avoid more than one ClickHouse® pod per selected topology unit.
- `ShardAntiAffinity`: pushes replicas of the same shard away from each other. This is the usual data-loss avoidance rule because replicas of one shard should not share one node or zone.
- `ReplicaAntiAffinity`: pushes pods with the same replica name away from each other across shards. This helps spread load for queries that scan the same replica index across shards.
- `AnotherNamespaceAntiAffinity`, `AnotherClickHouseInstallationAntiAffinity`, and `AnotherClusterAntiAffinity`: push pods away from other namespaces, CHIs, or clusters.

Affinity types do the opposite and attract pods to the same topology unit: `NamespaceAffinity`, `ClickHouseInstallationAffinity`, `ClusterAffinity`, `ShardAffinity`, `ReplicaAffinity`, and `PreviousTailAffinity`. Use them only when co-location is intentional, for example to build a circular replication layout or to keep a cluster inside a selected topology boundary.

`MaxNumberPerNode` limits how many ClickHouse® pods can be placed in the selected topology unit. `CircularReplication` is a shortcut expanded by the operator into `ShardAntiAffinity`, `ReplicaAntiAffinity`, `MaxNumberPerNode` with `number` equal to the replica count, `PreviousTailAffinity`, `NamespaceAffinity`, `ClickHouseInstallationAffinity`, and `ClusterAffinity`.

Example for spreading replicas of the same shard across nodes:

```yaml
spec:
  defaults:
    templates:
      podTemplate: clickhouse-pod
  templates:
    podTemplates:
      - name: clickhouse-pod
        podDistribution:
          - type: ShardAntiAffinity
            scope: Cluster
            topologyKey: kubernetes.io/hostname
        spec:
          containers:
            - name: clickhouse-pod
              image: altinity/clickhouse-server:25.8.16.10001.altinitystable
```

Example for a compact circular replication policy:

```yaml
spec:
  templates:
    podTemplates:
      - name: circular-replication
        podDistribution:
          - type: CircularReplication
```

See the distribution examples for [scope](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/13-distribution-01-distribution-scope.yaml), [CircularReplication](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/13-distribution-02-3x3-circular-replication.yaml), [expanded distribution rules](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/13-distribution-03-3x3-distribution-detailed.yaml), and [zone distribution](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/14-zones-distribution-01.yaml). Also check the [CRD explanation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/custom_resource_explained.md) before combining `podDistribution` with explicit Kubernetes `affinity`, `nodeSelector`, tolerations, or storage topology constraints.

### Replication

Use the operator [replication setup documentation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/replication_setup.md) when building replicated ClickHouse® clusters. Replication requires Keeper/ZooKeeper connectivity, a CHI layout with replicas, and ClickHouse® replicated table engines such as `ReplicatedMergeTree`. The operator provides macros such as `{installation}`, `{cluster}`, `{shard}`, and `{replica}` so replicated table paths and replica names can be generated consistently across pods. For new deployments, prefer CHK/Keeper references over hand-maintained ZooKeeper node lists where possible. See also [using clickhouse-keeper](https://kb.altinity.com/altinity-kb-setup-and-maintenance/altinity-kb-zookeeper/clickhouse-keeper/).

{{% alert title="Warning" color="warning" %}}
If you do not use CHK/Keeper auto-binding and configure Keeper/ZooKeeper hosts manually, do not configure a single Keeper service host as the only endpoint. List each Keeper/ZooKeeper node separately so ClickHouse® can balance requests and fail over across the coordination ensemble. With CHK `keeper` references, prefer the default `serviceType: replicas`; `serviceType: service` exposes only one service endpoint.
{{% /alert %}}

If you still need ZooKeeper, the operator repository ships ready-to-use [ZooKeeper manifests](https://github.com/Altinity/clickhouse-operator/tree/master/deploy/zookeeper) and a [ZooKeeper setup guide](https://github.com/Altinity/clickhouse-operator/blob/master/docs/zookeeper_setup.md), including quick-start and advanced examples. Treat them as a ZooKeeper option for existing or explicitly ZooKeeper-based deployments; CHK/Keeper is the preferred path for new operator-managed ClickHouse® clusters.

### Schema maintenance during scaling

When a CHI is scaled up, the operator automatically [creates schema](https://github.com/Altinity/clickhouse-operator/blob/master/config/config.yaml#L156) on the new ClickHouse® instances. For an added shard, it analyzes other shards and creates the matching databases, local tables, and distributed tables. For an added replica, it analyzes replicas in the same shard and creates the replicated databases and tables before applying the shard-level logic. Non-replicated tables are schema-only in this process: the operator can create matching table definitions on new hosts, but it does not copy existing rows, move parts, or perform resharding across shards.

When a CHI is scaled down, the operator drops replicated tables for deleted shards or replicas so stale metadata is not left in Keeper/ZooKeeper. Before changing shard or replica counts, review the operator [schema maintenance documentation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/schema_migration.md) and verify that the existing tables follow the expected local/distributed or replicated table patterns.

### Stop/start a ClickHouseInstallation

Use `spec.stop` on the `ClickHouseInstallation` when you need to stop ClickHouse® pods for planned maintenance. Do this through the CHI resource, not by manually scaling generated StatefulSets.

When `spec.stop` is enabled, the operator sets ClickHouse® StatefulSet replicas to `0`; Pods and Services are removed, while PVCs are kept. When `spec.stop` is disabled again, the operator recreates Pods and Services and reattaches the retained PVCs. Before stopping a cluster, check that the related PVs use the expected reclaim policy, especially before node, namespace, or storage maintenance.

Example:

```bash
# Stop the CHI.
kubectl -n <namespace> patch chi <name> --type=merge -p '{"spec":{"stop":"yes"}}'

# Start the CHI again.
kubectl -n <namespace> patch chi <name> --type=merge -p '{"spec":{"stop":"no"}}'
```

## User network access

Do not treat ClickHouse® user network access and the operator's own access account as the same setting.

- Configure ClickHouse® users in `spec.configuration.users`. Network restrictions are regular ClickHouse® user settings, for example `<user>/networks/ip`.
- The operator configuration has a default user template for users it creates. In the Helm chart defaults, that template allows only `::1` and `127.0.0.1`, unless a user overrides `networks/ip`.
- The operator's internal ClickHouse® account is separate. It uses `clickhouse.access.*` or the configured Kubernetes Secret for credentials, and the operator restricts that account to the operator pod IP.

Example:

```yaml
spec:
  configuration:
    users:
      admin/networks/ip:
        - 10.0.0.0/8
      admin/password_sha256_hex: <sha256-password>
      admin/profile: default
      admin/quota: default
```

For complete examples, use the operator [network user example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/05-settings-07-network.yaml) and the [operator configuration example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/70-chop-config.yaml).

## Service Templates explained with use case for load balancing between replicas with labels

Service templates customize Kubernetes `Service` objects generated by the operator. They are defined under `spec.templates.serviceTemplates` and then referenced from `spec.defaults.templates` or from lower-level cluster/shard/replica templates.

There are four common service-template levels:

- `serviceTemplate`: one CHI-level Service for the whole `ClickHouseInstallation`.
- `clusterServiceTemplate`: one Service per cluster in the CHI.
- `shardServiceTemplate`: one Service per shard.
- `replicaServiceTemplate`: one Service per replica host.

Current CRDs also expose `serviceTemplates` as a CHI-level list, so multiple CHI-level Services can be created. The cluster, shard, and replica template references are single names; you cannot attach two different `clusterServiceTemplate` values to the same cluster level and get two cluster-level Services from that field alone.

Use case: a CHI has two shards and two replicas. The second replica uses a pod template with an extra pod label `role: insert`. The goal is to expose:

- one cluster-level LoadBalancer for all ready ClickHouse® pods in the cluster;
- one insert-only LoadBalancer that targets only pods from the second replica.

Do not try to put both Services into `clusterServiceTemplate`; it accepts one template name. For a single-cluster CHI, use a cluster-level service for the full cluster and a CHI-level service for the filtered insert endpoint.

Example:

```yaml
apiVersion: clickhouse.altinity.com/v1
kind: ClickHouseInstallation
metadata:
  name: exp3
  namespace: clickhouse-dev
spec:
  defaults:
    templates:
      serviceTemplate: insert-service
      clusterServiceTemplate: cluster-service
  configuration:
    clusters:
      - name: exp3
        secret:
          auto: "True"
        layout:
          shardsCount: 2
          replicasCount: 2
          replicas:
            - templates:
                podTemplate: ch-pod
            - templates:
                podTemplate: ch-pod-insert
  templates:
    podTemplates:
      - name: ch-pod
        spec:
          containers:
            - name: clickhouse
              image: altinity/clickhouse-server:23.3.13.7.altinitystable
      - name: ch-pod-insert
        metadata:
          labels:
            role: insert
        spec:
          containers:
            - name: clickhouse
              image: altinity/clickhouse-server:23.3.13.7.altinitystable
    serviceTemplates:
      - name: cluster-service
        generateName: cluster-{chi}-{cluster}
        spec:
          ports:
            - name: http
              port: 8123
            - name: tcp
              port: 9000
          type: LoadBalancer
      - name: insert-service
        generateName: cluster-{chi}-insert
        spec:
          selector:
            role: insert
          ports:
            - name: http
              port: 8123
            - name: tcp
              port: 9000
          type: LoadBalancer
```

The generated cluster-level Service uses the operator's cluster selector, so it targets ready pods from cluster `exp3`. The generated insert Service is CHI-level and has the template selector merged with the operator's CHI-level ready selector. Its selector includes `role: insert`, so it targets only pods created from `ch-pod-insert`.

Expected Services:

```text
cluster-exp3-exp3     LoadBalancer   <cluster-ip>   <external-ip>   8123/TCP,9000/TCP
cluster-exp3-insert   LoadBalancer   <cluster-ip>   <external-ip>   8123/TCP,9000/TCP
```

Expected insert-service selector:

```yaml
clickhouse.altinity.com/app: chop
clickhouse.altinity.com/chi: exp3
clickhouse.altinity.com/namespace: clickhouse-dev
clickhouse.altinity.com/ready: "yes"
role: insert
```

Because `insert-service` is CHI-level, its `generateName` can use CHI-level macros such as `{chi}`, but not `{cluster}`. Keep one cluster per CHI for this pattern; multiple clusters in one CHI make service selection and operational ownership harder, and a CHI-level insert service would select matching pods across all clusters in that CHI.

See the operator [service template example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/02-templates-02-service-template.yaml), the [maximal CHI service template section](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/99-clickhouseinstallation-max.yaml), and the [CRD explanation](https://github.com/Altinity/clickhouse-operator/blob/master/docs/custom_resource_explained.md) for the full set of service-template fields and macros.

## PostgreSQL and MySQL interfaces with operator

ClickHouse® can expose PostgreSQL and MySQL wire-protocol interfaces. In Kubernetes, enabling the server setting is not enough: the CHI or CHIT must also expose the container port in a pod template and publish it through a service template.

PostgreSQL interface:

```yaml
apiVersion: clickhouse.altinity.com/v1
kind: ClickHouseInstallationTemplate
metadata:
  name: postgresql-port
spec:
  templating:
    policy: auto
  configuration:
    settings:
      postgresql_port: 9005
  templates:
    podTemplates:
      - name: pod-template-with-postgresql
        spec:
          containers:
            - name: clickhouse-pod
              ports:
                - containerPort: 9005
                  name: postgresql
                  protocol: TCP
    serviceTemplates:
      - name: chi-service-template
        spec:
          ports:
            - name: postgresql
              port: 19005
              protocol: TCP
              targetPort: 9005
```

MySQL interface:

```yaml
apiVersion: clickhouse.altinity.com/v1
kind: ClickHouseInstallationTemplate
metadata:
  name: mysql-port
spec:
  templating:
    policy: auto
  configuration:
    settings:
      mysql_port: 9004
  templates:
    podTemplates:
      - name: pod-template-with-mysql
        spec:
          containers:
            - name: clickhouse-pod
              ports:
                - containerPort: 9004
                  name: mysql
                  protocol: TCP
    serviceTemplates:
      - name: chi-service-template
        spec:
          ports:
            - name: mysql
              port: 19004
              protocol: TCP
              targetPort: 9004
```

The operator repository has matching template examples for [PostgreSQL port](https://github.com/Altinity/clickhouse-operator/blob/master/tests/e2e/manifests/chit/tpl-postgresql-port.yaml) and [MySQL port](https://github.com/Altinity/clickhouse-operator/blob/master/tests/e2e/manifests/chit/tpl-mysql.yaml).

For PostgreSQL clients, create users with an authentication method supported by the ClickHouse® PostgreSQL interface for your server version. Older guidance and the current PostgreSQL interface docs note plain-text password requirements for `psql` compatibility, for example:

```sql
CREATE USER pg_user IDENTIFIED WITH plaintext_password BY 'qwerty';
GRANT SELECT ON default.* TO pg_user;
```

For MySQL clients, use a MySQL-compatible password type such as `double_sha1_password`:

```sql
CREATE USER mysql_user IDENTIFIED WITH double_sha1_password BY 'qwerty';
GRANT SELECT ON default.* TO mysql_user;
```

For PostgreSQL SSL, `postgresql_port` uses the ClickHouse® server TLS settings when TLS is configured. Kubernetes load balancers still need to expose the intended ports explicitly. If the LB terminates TLS, keep that path separate from a plain TCP path. If TLS is passed through to ClickHouse®, validate the actual port with `openssl s_client`; a result with no peer certificate usually means the tested endpoint is not serving TLS.

```bash
openssl s_client -connect <host>:<port>
```

See the ClickHouse® [PostgreSQL interface](https://clickhouse.com/docs/interfaces/postgresql), [MySQL interface](https://clickhouse.com/docs/interfaces/mysql), and [CREATE USER identification](https://clickhouse.com/docs/sql-reference/statements/create/user#identification) documentation for version-specific authentication behavior and client limitations.

## Start/Stop cluster

- Don't delete the operator using:

```bash
kubectl delete -f https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install-bundle.yaml
```

- kubectl delete chi cluster-name # chi is the name of the CRD clickhouseInstallation

## Possible issues with running ClickHouse® in K8s

When `clickhouse-server` cannot start, the pod can enter `CrashLoopBackOff`. Start with the previous container logs and Kubernetes events before changing the CHI or touching data on disk:

```bash
kubectl logs <pod> -c clickhouse-pod -n <namespace> --previous
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Check these cases first:

1. Bad generated ClickHouse® configuration. Inspect the CHI, templates, generated ConfigMaps, and ClickHouse® logs. Generated config files are mounted into the pod as read-only files, so fix the CHI or templates and let the operator reconcile.
2. Backward-incompatible ClickHouse® server upgrade. Roll back the image if the cluster must be restored quickly, then review release notes for every skipped version and adjust configuration or schema before retrying the upgrade. Refer to [Altinity Stable® Builds for ClickHouse® Release Notes](https://docs.altinity.com/releasenotes/altinity-stable-release-notes/) before upgrading.
3. DNS cache mismatch after pod or service IP changes. For dynamic backends such as Kafka, run `SYSTEM DROP DNS CACHE` when the server is running, and set `disable_internal_dns_cache: 1` in CHI settings when stale DNS is a recurring issue.
4. ClickHouse® init timeout. If container logs show the ClickHouse® image entrypoint timed out during init, increase `CLICKHOUSE_INIT_TIMEOUT` in the ClickHouse® pod template.
5. Wrong ownership or permissions on mounted volumes. Fix the pod `securityContext`, `fsGroup`, or run a controlled one-time ownership repair. For persistent volumes, use the operator [security context example](https://github.com/Altinity/clickhouse-operator/blob/master/docs/chi-examples/03-persistent-volume-07-security-context.yaml) rather than relying on repeated manual `chown` operations.
6. Broken local state or replica metadata divergence after an unclean restart. `force_restore_data` is a recovery action, not a generic startup fix. Use it only after checking replication state and accepting the possible data-loss or detached-parts outcome; see [Recovery after complete data loss](/altinity-kb-setup-and-maintenance/recovery-after-complete-data-loss/).
7. Table metadata prevents startup. Only rename or move the specific problematic table metadata file after confirming it in ClickHouse® logs and preserving a copy, for example `table.sql` to `table.sql.bak`; then repair the schema intentionally.
8. Emergency access when ClickHouse® cannot stay up. Use a temporary debug pod with the PVC attached, or a controlled entrypoint override that keeps the container running long enough to inspect files. Remove the emergency override after the recovery work.

### S3 table function and AWS session tokens

Some ClickHouse® versions have a known issue where the S3 table function path does not correctly initialize the AWS session token even though session-token handling exists in the S3 client code. See the ClickHouse® discussion in [PR #57850](https://github.com/ClickHouse/ClickHouse/pull/57850#issuecomment-1966404710) and the referenced [S3 client code](https://github.com/ClickHouse/ClickHouse/blob/63b445799136fa9fc4be362df35181064692fb48/src/IO/S3/Client.cpp#L877C1-L877C34).

For affected versions, avoid relying on temporary session-token parameters in the S3 table function. A practical workaround is to configure S3 storage with `use_environment_credentials` and pass AWS credentials, including `AWS_SESSION_TOKEN`, to the ClickHouse® container through Kubernetes Secrets.

Example S3 disk setting:

```yaml
spec:
  configuration:
    settings:
      storage_configuration/disks/s3/use_environment_credentials: "1"
```

Example pod template environment:

```yaml
spec:
  templates:
    podTemplates:
      - name: clickhouse-pod
        spec:
          containers:
            - name: clickhouse-pod
              env:
                - name: AWS_ACCESS_KEY_ID
                  valueFrom:
                    secretKeyRef:
                      name: s3-credentials
                      key: AWS_ACCESS_KEY_ID
                - name: AWS_SECRET_ACCESS_KEY
                  valueFrom:
                    secretKeyRef:
                      name: s3-credentials
                      key: AWS_SECRET_ACCESS_KEY
                - name: AWS_SESSION_TOKEN
                  valueFrom:
                    secretKeyRef:
                      name: s3-credentials
                      key: AWS_SESSION_TOKEN
```

Caveats:

- Ensure all ClickHouse® and Keeper/CHK state that must persist is backed by PVCs, and that reclaim policy is set intentionally as described in the storage lifecycle checklist above.
- Page cache is node-local. Kubernetes can move pods between nodes, but cached pages are not moved with them; durability must come from persistent volumes, replication, and the storage backend, not from node-local cache.
- Large ClickHouse® schemas take time to load at server startup, especially with many databases, tables, and parts. `async_load_databases` was added in ClickHouse® 23.11 and enabled by default in 24.6; in 25.2 it became enabled by default even when an older `config.xml` does not set it. For slow startup, tune the current server-level loading pools such as `max_active_parts_loading_thread_pool_size`, `tables_loader_foreground_pool_size`, and `tables_loader_background_pool_size`. On older ClickHouse® versions before 23.5, the table-level `max_part_loading_threads` setting controlled part loading and is now obsolete.
- On some cloud storage backends, file deletion or unlink operations can be slow and affect part cleanup. Confirm the storage bottleneck before tuning ClickHouse® settings such as `max_part_removal_threads`.

## Runbooks

### Mount / Inspect PVC in Kubernetes

Use a temporary debug pod when ClickHouse® cannot start and you need to inspect files on a PVC. Prefer read-only inspection first. If the PVC is `ReadWriteOnce`, the debug pod usually must run on the same node as the existing pod or the original pod must be stopped first; otherwise Kubernetes may block the attach with a multi-attach error. Do not modify ClickHouse® data files unless you have identified the exact recovery action.

See also [Inspect a Kubernetes Persistent Volume Claim](https://frank.sauerburger.io/2021/12/01/inspect-k8s-pvc.html).

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-inspector
spec:
  containers:
    - name: pvc-inspector
      image: busybox
      command:
        - tail
      args:
        - -f
        - /dev/null
      resources:
        requests:
          cpu: 500m
          memory: 512Mi
        limits:
          cpu: 500m
          memory: 512Mi
      volumeMounts:
        - name: pvc-mount
          mountPath: /var/lib/clickhouse
          readOnly: true
  volumes:
    - name: pvc-mount
      persistentVolumeClaim:
        claimName: default-1-1-chi-dev-trd-dev-trd-0-0-0
```

Useful commands:

```bash
kubectl -n <namespace> apply -f pvc-inspector.yaml
kubectl -n <namespace> exec -it pvc-inspector -- sh
kubectl -n <namespace> delete pod pvc-inspector
```

### How to check if a node was replaced

Start with the ClickHouse® pod description and check the `Node` and `Start Time` fields:

```bash
kubectl -n evolution-prod describe pod/chi-pr-ach-pr-ach-11-1-0
```

Example output:

```text
Name:             chi-pr-ach-pr-ach-11-1-0
Namespace:        evolution-prod
Priority:         0
Service Account:  default
Node:             ip-10-129-210-177.eu-central-1.compute.internal/10.129.210.177
Start Time:       Mon, 15 Jul 2024 11:07:59 +0200
```

`describe pod` shows the current node assignment. To prove that a node was replaced or the pod moved, compare it with the previous pod events, operator logs, StatefulSet history, cloud-node lifecycle events, or monitoring data. Useful checks:

```bash
kubectl -n <namespace> describe pod/<pod>
kubectl -n <namespace> get events --sort-by=.lastTimestamp
kubectl get node <node-name> -o wide
kubectl describe node <node-name>
```

For ClickHouse® pods with PVCs, also check whether the move was blocked or delayed by volume attach/detach:

```bash
kubectl -n <namespace> describe pvc <pvc-name>
kubectl get volumeattachment
```

Then describe the node from the pod output:

```bash
kubectl describe node/ip-10-129-210-177.eu-central-1.compute.internal
```

For Karpenter-managed nodes, node events can give a quick signal whether this is an old node or a recently replaced node. Compare the event age range:

```text
# Node running for 16d.
Events:
  Type    Reason            Age                   From       Message
  ----    ------            ----                  ----       -------
  Normal  Unconsolidatable  14m (x1613 over 16d)  karpenter  Provisioner "dedicated-for-clickhouse-c6a-4xlarge" has consolidation disabled

# Node likely substituted around 11 minutes ago.
Events:
  Type    Reason            Age                 From       Message
  ----    ------            ----                ----       -------
  Normal  Unconsolidatable  14m (x3 over 11m)   karpenter  Provisioner "dedicated-for-clickhouse-c6a-4xlarge" has consolidation disabled
```

The exact reason still depends on cloud autoscaler events and node lifecycle data, but a short event history on the node is a strong hint that the node object is new.

## DELETE PVCs

https://altinity.com/blog/preventing-clickhouse-storage-deletion-with-the-altinity-kubernetes-operator-reclaimpolicy

## Scaling

Best way is to scale down the deployments to 0 replicas, after that reboot the node and scale up again:

1. first check that all your PVCs have the retain policy:

```bash
kubectl get pv -o=custom-columns=PV:.metadata.name,NAME:.spec.claimRef.name,POLICY:.spec.persistentVolumeReclaimPolicy
# Patch it if you need
kubectl patch pv <pv_id> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

```yaml
spec:
 templates:
    volumeClaimTemplates:
       - name: XXX
         reclaimPolicy: Retain
```

2. After that just create a stop.yaml and `kubectl apply -f stop.yaml`

```yaml
kind: ClickHouseInstallation
spec:
 stop: yes
```

3. Reboot kubernetes node
4. Scale up deployment changing the stop property to no and do an `kubectl apply -f stop.yml`

```yaml
kind: ClickHouseInstallation
spec:
 stop: no
```

## Check where pods are executing

```bash
kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName -n zk
# Check which hosts in which AZs
kubectl get node -o=custom-columns=NODE:.metadata.name,ZONE:.metadata.labels.'failure-domain\.beta\.kubernetes\.io/zone'
```

## Check node instance types:

```sql
kubectl get nodes -o json|jq -Cjr '.items[] | .metadata.name," ",.metadata.labels."beta.kubernetes.io/instance-type"," ",.metadata.labels."beta.kubernetes.io/arch", "\n"'|sort -k3 -r

ip-10-3-9-2.eu-central-1.compute.internal t4g.large arm64
ip-10-3-9-236.eu-central-1.compute.internal t4g.large arm64
ip-10-3-9-190.eu-central-1.compute.internal t4g.large arm64
ip-10-3-9-138.eu-central-1.compute.internal t4g.large arm64
ip-10-3-9-110.eu-central-1.compute.internal t4g.large arm64
ip-10-3-8-39.eu-central-1.compute.internal t4g.large arm64
ip-10-3-8-219.eu-central-1.compute.internal t4g.large arm64
ip-10-3-8-189.eu-central-1.compute.internal t4g.large arm64
ip-10-3-13-40.eu-central-1.compute.internal t4g.large arm64
ip-10-3-12-248.eu-central-1.compute.internal t4g.large arm64
ip-10-3-12-216.eu-central-1.compute.internal t4g.large arm64
ip-10-3-12-170.eu-central-1.compute.internal t4g.large arm64
ip-10-3-11-229.eu-central-1.compute.internal t4g.large arm64
ip-10-3-11-188.eu-central-1.compute.internal t4g.large arm64
ip-10-3-11-175.eu-central-1.compute.internal t4g.large arm64
ip-10-3-10-218.eu-central-1.compute.internal t4g.large arm64
ip-10-3-10-160.eu-central-1.compute.internal t4g.large arm64
ip-10-3-10-145.eu-central-1.compute.internal t4g.large arm64
ip-10-3-9-57.eu-central-1.compute.internal m5.large amd64
ip-10-3-8-146.eu-central-1.compute.internal m5.large amd64
ip-10-3-13-1.eu-central-1.compute.internal m5.xlarge amd64
ip-10-3-11-52.eu-central-1.compute.internal m5.xlarge amd64
ip-10-3-11-187.eu-central-1.compute.internal m5.xlarge amd64
ip-10-3-10-217.eu-central-1.compute.internal m5.xlarge amd64
```

## Search for missing affinity rules:

```bash
kubectl get pods -o json -n zk |\
jq -r "[.items[] | {name: .metadata.name,\
 affinity: .spec.affinity}]"
[
  {
    "name": "zookeeper-0",
    "affinity": null
  },
  . . .
]
```

## Storage classes

```bash
kubectl get pvc -o=custom-columns=NAME:.metadata.name,SIZE:.spec.resources.requests.storage,CLASS:.spec.storageClassName,VOLUME:.spec.volumeName
...
NAME                         SIZE   CLASS   VOLUME
datadir-volume-zookeeper-0   25Gi   gp2     pvc-9a3...9ee

kubectl get storageclass/gp2 
...
NAME            PROVISIONER       RECLAIMPOLICY...   
gp2 (default)   ebs.csi.aws.com   Delete
```

## Using CSI driver to protect storage:

```yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp2-protected
parameters:
  encrypted: "true"
  type: gp2
provisioner: ebs.csi.aws.com
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

## Enable Resize of Volumes

Operator does not delete volumes, so those were probably deleted by Kubernetes. In some new versions there is a feature flag that deletes PVCs attached to STS when STS is deleted.

Please try do the following: Use operator 0.20.3. Add the following to the defaults:
``

```yaml
 defaults:
    storageManagement:
      provisioner: Operator
```

That enables storage management by operator, instead of STS. It allows to extend volumes without re-creating STS, and us increase Volume size without restart of ClickHouse® statefulset pods for CSI drivers which support `allowVolumeExpansion` in storage classes because statefulset template don't change and we don't need delete/create statefulset

## Change server settings:

https://github.com/Altinity/clickhouse-operator/issues/828

```yaml
kind: ClickHouseInstallation
spec:
  configuration:
    settings:
        max_concurrent_queries: 150
```

Or **edit ClickHouseInstallation:**

```bash
kubectl -n <namespace> get chi

NAME              CLUSTERS   HOSTS   STATUS      HOSTS-COMPLETED   AGE
dnieto-test       1          4       Completed                     211d
mbak-test         1          1       Completed                     44d
rory-backupmar8   1          4       Completed                     42h

kubectl -n <namespace> edit ClickHouseInstallation dnieto-test
```

## Configurations:

How to modify yaml configs:

https://github.com/Altinity/clickhouse-operator/blob/dc6cdc6f2f61fc333248bb78a8f8efe792d14ca2/tests/e2e/manifests/chi/test-016-settings-04.yaml#L26

## clickhouse-operator install Example:

use latest release if possible
https://github.com/Altinity/clickhouse-operator/releases

- No. Nodes/replicas: 2 to 3 nodes with 500GB per node minimum
- Zookeeper: 3 node ensemble
- Type of instances: m6i.x4large to start with and you can go up to m6i.16xlarge
- Persistent Storage/volumes: EBS gp2 for data and logs and gp3 for zookeeper

### Install operator in namespace

```bash
#!/bin/bash

# Namespace to install operator into
OPERATOR_NAMESPACE="${OPERATOR_NAMESPACE:-dnieto-test-chop}"
# Namespace to install metrics-exporter into
METRICS_EXPORTER_NAMESPACE="${OPERATOR_NAMESPACE}"
# Operator's docker image
OPERATOR_IMAGE="${OPERATOR_IMAGE:-altinity/clickhouse-operator:latest}"
# Metrics exporter's docker image
METRICS_EXPORTER_IMAGE="${METRICS_EXPORTER_IMAGE:-altinity/metrics-exporter:latest}"

# Setup clickhouse-operator into specified namespace
kubectl apply --namespace="${OPERATOR_NAMESPACE}" -f <( \
    curl -s https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install-template.yaml | \
        OPERATOR_IMAGE="${OPERATOR_IMAGE}" \
        OPERATOR_NAMESPACE="${OPERATOR_NAMESPACE}" \
        METRICS_EXPORTER_IMAGE="${METRICS_EXPORTER_IMAGE}" \
        METRICS_EXPORTER_NAMESPACE="${METRICS_EXPORTER_NAMESPACE}" \
        envsubst \
)
```

### Install zookeeper ensemble

zookeepers will be named like zookeeper-0.zoons

```bash
> kubectl create ns zoo3ns
> kubectl -n zoo3ns apply -f https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/zookeeper/quick-start-persistent-volume/zookeeper-3-nodes-1GB-for-tests-only.yaml

# check names they should be like:
# zookeeper.zoo3ns if using a new namespace
# If using the same namespace zookeeper.<localnamespace>
# zookeeper must be accessed using the service like service_name.namespace
```

### Deploy test cluster

```bash
> kubectl -n dnieto-test-chop apply -f dnieto-test-chop.yaml
```

```yaml
# dnieto-test-chop.yaml
apiVersion: "clickhouse.altinity.com/v1"
kind: "ClickHouseInstallation"
metadata:
  name: "dnieto-dev"
spec:
  configuration:
    settings:
	    max_concurrent_queries: "200"
			merge_tree/ttl_only_drop_parts: "1"
		profiles:
	    default/queue_max_wait_ms: "10000"
			readonly/readonly: "1"
		users:
      admin/networks/ip:
        - 0.0.0.0/0
        - '::/0'
			admin/password_sha256_hex: ""
      admin/profile: default
      admin/access_management: 1
	  zookeeper:  
			nodes:
        - host: zookeeper.dnieto-test-chop
          port: 2181
		clusters:
      - name: dnieto-dev
        templates:
          podTemplate: pod-template-with-volumes
          serviceTemplate: chi-service-template
        layout:
          shardsCount: 1
          # put the number of desired nodes 3 by default
          replicasCount: 2
  templates:
    podTemplates:
      - name: pod-template-with-volumes
        spec:
          containers:
            - name: clickhouse
              image: clickhouse/clickhouse-server:22.3
              # separate data from logs 
              volumeMounts:
                - name: data-storage-vc-template
                  mountPath: /var/lib/clickhouse
                - name: log-storage-vc-template
                  mountPath: /var/log/clickhouse-server
    serviceTemplates:
      - name: chi-service-template
        generateName: "service-{chi}"
        # type ObjectMeta struct from k8s.io/meta/v1
        metadata:
          annotations:
						# https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
						# this tags for elb load balancer
            #service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
				    #service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
						#https://kubernetes.io/docs/concepts/services-networking/service/#aws-nlb-support
				    service.beta.kubernetes.io/aws-load-balancer-internal: "true"
					  service.beta.kubernetes.io/aws-load-balancer-type: nlb
				spec:
          ports:
            - name: http
              port: 8123
            - name: client
              port: 9000
          type: LoadBalancer
    volumeClaimTemplates:
      - name: data-storage-vc-template
        spec:
        # no storageClassName - means use default storageClassName
        # storageClassName: default
        # here if you have a storageClassName defined for gp2 you can use it.
        # kubectl get storageclass
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 50Gi
        reclaimPolicy: Retain
      - name: log-storage-vc-template
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 2Gi
```

### Install monitoring:

In order to setup prometheus as a backend for all the asynchronous_metric_log / metric_log tables and also set up grafana dashboards:

- https://github.com/Altinity/clickhouse-operator/blob/master/docs/prometheus_setup.md
- https://github.com/Altinity/clickhouse-operator/blob/master/docs/grafana_setup.md
- [clickhouse-operator/monitoring_setup.md at master · Altinity/clickhouse-operator](https://github.com/Altinity/clickhouse-operator/blob/master/docs/monitoring_setup.md)

## Extra configs

There is an admin user by default in the deployment that is used to admin stuff

## KUBECTL chi basic comands:

```bash
*> kubectl get crd*

NAME                                                       CREATED AT
clickhouseinstallations.clickhouse.altinity.com            2021-10-11T13:46:43Z
clickhouseinstallationtemplates.clickhouse.altinity.com    2021-10-11T13:46:44Z
clickhouseoperatorconfigurations.clickhouse.altinity.com   2021-10-11T13:46:44Z
eniconfigs.crd.k8s.amazonaws.com                           2021-10-11T13:41:23Z
grafanadashboards.integreatly.org                          2021-10-11T13:54:37Z
grafanadatasources.integreatly.org                         2021-10-11T13:54:38Z
grafananotificationchannels.integreatly.org                2022-05-17T14:27:48Z
grafanas.integreatly.org                                   2021-10-11T13:54:37Z
provisioners.karpenter.sh                                  2022-05-17T14:27:49Z
securitygrouppolicies.vpcresources.k8s.aws                 2021-10-11T13:41:27Z
volumesnapshotclasses.snapshot.storage.k8s.io              2022-04-22T13:34:20Z
volumesnapshotcontents.snapshot.storage.k8s.io             2022-04-22T13:34:20Z
volumesnapshots.snapshot.storage.k8s.io                    2022-04-22T13:34:20Z

> *kubectl -n test-clickhouse-operator-dnieto2 get chi*
NAME        CLUSTERS   HOSTS   STATUS   HOSTS-COMPLETED   AGE
simple-01                                                 70m

> *kubectl -n test-clickhouse-operator-dnieto2 describe chi simple-01*
Name:         simple-01
Namespace:    test-clickhouse-operator-dnieto2
Labels:       <none>
Annotations:  <none>
API Version:  clickhouse.altinity.com/v1
Kind:         ClickHouseInstallation
Metadata:
  Creation Timestamp:  2023-01-09T20:38:06Z
  Generation:          1
  Managed Fields:
    API Version:  clickhouse.altinity.com/v1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:configuration:
          .:
          f:clusters:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2023-01-09T20:38:06Z
  Resource Version:  267483138
  UID:               d7018efa-2b13-42fd-b1c5-b798fc6d0098
Spec:
  Configuration:
    Clusters:
      Name:  simple
Events:      <none>

> *kubectl get chi --all-namespaces*

NAMESPACE                          NAME                             CLUSTERS   HOSTS   STATUS      HOSTS-COMPLETED   AGE
andrey-dev                         source                           1          1       Completed                     38d
eu                                 chi-dnieto-test-common-configd   1          1       Completed                     161d
eu                                 dnieto-test                      1          4       Completed                     151d
laszlo-dev                         node-rescale-2                   1          4       Completed                     5d13h
laszlo-dev                         single                           1          1       Completed                     5d13h
laszlo-dev2                        zk2                              1          1       Completed                     52d
test-clickhouse-operator-dnieto2   simple-01

> *kubectl -n test-clickhouse-operator-dnieto2 edit clickhouseinstallations.clickhouse.altinity.com simple-01

# Troubleshoot operator stuff
> kubectl -n test-clickhouse-operator-ns edit chi
> kubectl -n test-clickhouse-operator describe chi
> kubectl -n test-clickhouse-operator get chi -o yaml

# Check operator logs usually located in kube-system or specific namespace
> kubectl -n test-ns logs chi-operator-pod -f

# Check output to yaml
> kubectl -n test-ns get services -o yaml*
```

## Problem with DELETE finalizers:

https://github.com/Altinity/clickhouse-operator/issues/830

There's a problem with stuck finalizers that can cause old CHI installations to hang. The sequence of operations looks like this.

1. You delete the existing ClickHouse® operator using `kubectl delete -f operator-installation.yaml` with running CHI clusters.
2. You then drop the namespace where the CHI clusters are running, e.g., `kubectl delete ns my-namespace`
3. This hangs. You run `kubectl get ns my-namespace -o yaml` and you'll see a message like the following: "message: 'Some content in the namespace has finalizers remaining: [finalizer.clickhouseinstallation.altinity.com](http://finalizer.clickhouseinstallation.altinity.com/)"

That means the CHI can't delete because its finalizer was deleted out from under it.

The fix is to figure out the chi name which should still be visible and edit it to remove the finalizer reference.

1. `kubectl -n my-namespace get chi`
2. `kubectl -n my-namespace edit [clickhouseinstallations.clickhouse.altinity.com](http://clickhouseinstallations.clickhouse.altinity.com/) my-clickhouse-cluster`

Remove the finalizer from the spec, save it, and everything will delete properly.

**`TIP: if you delete the ns too and there is no ns just create it and apply the above method`**

## Karpenter scaler

```sql
> kubectl -n karpenter get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/karpenter-75c8b7667b-vbmj4   1/1     Running   0          16d
pod/karpenter-75c8b7667b-wszxt   1/1     Running   0          16d

NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)            AGE
service/karpenter   ClusterIP   172.20.129.188   <none>        8080/TCP,443/TCP   16d

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/karpenter   2/2     2            2           16d

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/karpenter-75c8b7667b   2         2         2       16d

> kubectl -n karpenter logs pod/karpenter-75c8b7667b-vbmj4

2023-02-06T06:33:44.269Z	DEBUG	Successfully created the logger.
2023-02-06T06:33:44.269Z	DEBUG	Logging level set to: debug
{"level":"info","ts":1675665224.2755454,"logger":"fallback","caller":"injection/injection.go:63","msg":"Starting informers..."}
2023-02-06T06:33:44.376Z	DEBUG	controller	waiting for configmaps	{"commit": "f60dacd", "configmaps": ["karpenter-global-settings"]}
2023-02-06T06:33:44.881Z	DEBUG	controller	karpenter-global-settings config "karpenter-global-settings" config was added or updated: settings.Settings{BatchMaxDuration:v1.Duration{Duration:10000000000}, BatchIdleDuration:v1.Duration{Duration:1000000000}}	{"commit": "f60dacd"}
2023-02-06T06:33:44.881Z	DEBUG	controller	karpenter-global-settings config "karpenter-global-settings" config was added or updated: settings.Settings{ClusterName:"eu", ClusterEndpoint:"https://79974769E264251E43B18AF4CA31CE8C.gr7.eu-central-1.eks.amazonaws.com", DefaultInstanceProfile:"KarpenterNodeInstanceProfile-eu", EnablePodENI:false, EnableENILimitedPodDensity:true, IsolatedVPC:false, NodeNameConvention:"ip-name", VMMemoryOverheadPercent:0.075, InterruptionQueueName:"Karpenter-eu", Tags:map[string]string{}}	{"commit": "f60dacd"}
2023-02-06T06:33:45.001Z	DEBUG	controller.aws	discovered region	{"commit": "f60dacd", "region": "eu-central-1"}
2023-02-06T06:33:45.003Z	DEBUG	controller.aws	unable to detect the IP of the kube-dns service, services "kube-dns" is forbidden: User "system:serviceaccount:karpenter:karpenter" cannot get resource "services" in API group "" in the namespace "kube-system"	{"commit": "f60dacd"}
2023/02/06 06:33:45 Registering 2 clients
2023/02/06 06:33:45 Registering 2 informer factories
2023/02/06 06:33:45 Registering 3 informers
2023/02/06 06:33:45 Registering 6 controllers
2023-02-06T06:33:45.080Z	DEBUG	controller.aws	discovered version	{"commit": "f60dacd", "version": "v0.20.0"}
2023-02-06T06:33:45.082Z	INFO	controller	Starting server	{"commit": "f60dacd", "path": "/metrics", "kind": "metrics", "addr": "[::]:8080"}
2023-02-06T06:33:45.082Z	INFO	controller	Starting server	{"commit": "f60dacd", "kind": "health probe", "addr": "[::]:8081"}
I0206 06:33:45.182600       1 leaderelection.go:248] attempting to acquire leader lease karpenter/karpenter-leader-election...
2023-02-06T06:33:45.226Z	INFO	controller	Starting informers...	{"commit": "f60dacd"}
2023-02-06T06:33:45.417Z	INFO	controller.aws.pricing	updated spot pricing with instance types and offerings	{"commit": "f60dacd", "instance-type-count": 607, "offering-count": 1400}
2023-02-06T06:33:47.670Z	INFO	controller.aws.pricing	updated on-demand pricing	{"commit": "f60dacd", "instance-type-count": 505}
```

## Operator Affinities:

![Screenshot from 2023-02-21 11-26-36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/90052686-7c87-413f-95f7-41c12d233190/Screenshot_from_2023-02-21_11-26-36.png)

## Deploy operator with clickhouse-keeper

https://github.com/Altinity/clickhouse-operator/issues/959
[setup-example.yaml](https://github.com/Altinity/clickhouse-operator/blob/eb3fc4e28514d0d6ea25a40698205b02949bcf9d/docs/chi-examples/03-persistent-volume-07-do-not-chown.yaml)

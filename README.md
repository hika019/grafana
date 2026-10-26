# Grafana / InfluxDB on Kubernetes

## InfluxDB 3 admin token

`influxdb-token-job.yml` creates the `influxdb-admin-token` Secret in the `grafana` namespace when it does not already exist.

- Generates the token in memory using `/dev/urandom`; no temporary token file is written.
- Writes the Secret key `admin-token.json` directly through the Kubernetes API.
- Preserves any existing Secret. Pod restarts and Argo CD syncs do not rotate the token.
- The InfluxDB container still mounts the Secret as `/etc/influxdb/admin-token.json`: its `--admin-token-file` parameter requires a file.
- The Job manifest includes a dedicated ServiceAccount, Role, and RoleBinding.
- Back up the Secret securely. If deleted after InfluxDB has initialized, do **not** blindly regenerate it; the replacement token may not be recognized by the existing database.
- Do not apply `influxdb.yml` (InfluxDB 2) and `influxdb3.yml` (InfluxDB 3) together: they declare the same Deployment and Service names.

### Manual deployment

```bash
kubectl apply -f namespace.yml
kubectl apply -f influxdb-token-job.yml
kubectl wait -n grafana --for=condition=complete job/influxdb-token-init --timeout=120s
kubectl apply -f influxdb3.yml
```

### Argo CD

Include `influxdb-token-job.yml` in the Argo CD Application source. Its Job has sync wave `-1`, ahead of the InfluxDB workload (default wave `0`). Ensure the `grafana` namespace exists before syncing; neither the Job nor its ServiceAccount can be created before the namespace.

If using Kustomize, explicitly select `influxdb3.yml` rather than `influxdb.yml`.

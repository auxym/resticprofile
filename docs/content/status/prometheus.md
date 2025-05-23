---
title: "Prometheus"
weight: 5
---



resticprofile can generate a prometheus file, or send the report to a push gateway. For now, only a `backup` command will generate a report.
Here's a configuration example with both options to generate a file and send to a push gateway:

{{< tabs groupid="config-with-json" >}}
{{% tab title="toml" %}}

```toml
version = "1"

[root]
  prometheus-save-to-file = "root.prom"
  prometheus-push = "http://localhost:9091/"

  [root.backup]
    extended-status = true
    no-error-on-warning = true
    source = [ "/" ]
```

{{% /tab %}}
{{% tab title="yaml" %}}

```yaml
version: "1"

root:
  prometheus-save-to-file: "root.prom"
  prometheus-push: "http://localhost:9091/"
  backup:
    extended-status: true
    no-error-on-warning: true
    source:
      - /
```

{{% /tab %}}
{{% tab title="hcl" %}}

```hcl
"root" = {
  "prometheus-save-to-file" = "root.prom"
  "prometheus-push" = "http://localhost:9091/"

  "backup" = {
    "extended-status" = true
    "no-error-on-warning" = true
    "source" = ["/"]
  }
}
```

{{% /tab %}}
{{% tab title="json" %}}

```json
{
  "version": "1",
  "root": {
    "prometheus-save-to-file": "root.prom",
    "prometheus-push": "http://localhost:9091/",
    "backup": {
      "extended-status": true,
      "no-error-on-warning": true,
      "source": [
        "/"
      ]
    }
  }
}
```

{{% /tab %}}
{{< /tabs >}}

{{% notice style="note" %}}
Please note you need to set `extended-status` to `true` if you want all the available metrics. See [Extended status]({{% relref "/status/index.html#-extended-status" %}}) for more information.
{{% /notice %}}

Here's an example of the generated prometheus file:

```
# HELP resticprofile_backup_added_bytes Total number of bytes added to the repository.
# TYPE resticprofile_backup_added_bytes gauge
resticprofile_backup_added_bytes{profile="prom"} 9.83610983e+08
# HELP resticprofile_backup_dir_changed Number of directories with changes.
# TYPE resticprofile_backup_dir_changed gauge
resticprofile_backup_dir_changed{profile="prom"} 0
# HELP resticprofile_backup_dir_new Number of new directories added to the backup.
# TYPE resticprofile_backup_dir_new gauge
resticprofile_backup_dir_new{profile="prom"} 847
# HELP resticprofile_backup_dir_unmodified Number of directories unmodified since last backup.
# TYPE resticprofile_backup_dir_unmodified gauge
resticprofile_backup_dir_unmodified{profile="prom"} 0
# HELP resticprofile_backup_duration_seconds The backup duration (in seconds).
# TYPE resticprofile_backup_duration_seconds gauge
resticprofile_backup_duration_seconds{profile="prom"} 4.453124672
# HELP resticprofile_backup_files_changed Number of files with changes.
# TYPE resticprofile_backup_files_changed gauge
resticprofile_backup_files_changed{profile="prom"} 0
# HELP resticprofile_backup_files_new Number of new files added to the backup.
# TYPE resticprofile_backup_files_new gauge
resticprofile_backup_files_new{profile="prom"} 6006
# HELP resticprofile_backup_files_processed Total number of files scanned by the backup for changes.
# TYPE resticprofile_backup_files_processed gauge
resticprofile_backup_files_processed{profile="prom"} 6006
# HELP resticprofile_backup_files_unmodified Number of files unmodified since last backup.
# TYPE resticprofile_backup_files_unmodified gauge
resticprofile_backup_files_unmodified{profile="prom"} 0
# HELP resticprofile_backup_processed_bytes Total number of bytes scanned for changes.
# TYPE resticprofile_backup_processed_bytes gauge
resticprofile_backup_processed_bytes{profile="prom"} 1.016520315e+09
# HELP resticprofile_backup_status Backup status: 0=fail, 1=warning, 2=success.
# TYPE resticprofile_backup_status gauge
resticprofile_backup_status{profile="prom"} 2
# HELP resticprofile_backup_time_seconds Last backup run (unixtime).
# TYPE resticprofile_backup_time_seconds gauge
resticprofile_backup_time_seconds{profile="prom"} 1.707863748e+09
# HELP resticprofile_build_info resticprofile build information.
# TYPE resticprofile_build_info gauge
resticprofile_build_info{goversion="go1.22.0",profile="prom",version="0.26.0-dev"} 1

```

## Prometheus Pushgateway

Prometheus Pushgateway uses the job label as a grouping key. All metrics with the same grouping key get replaced when pushed. To prevent metrics from multiple profiles getting overwritten by each other, the default job label is set to `<profile_name>.<command>` (e.g. `root.backup`).

If you need more control over the job label, you can use the `prometheus-push-job` property. This property can contain the `$command` placeholder, which is replaced with the name of the executed command.

Additionally, the request format can be specified with `prometheus-push-format`. The default is `text`, but it can also be set to `protobuf` (see [compatibility with Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/#exposition-formats)).

## User defined labels

You can add your own prometheus labels. Please note they will be applied to **all** the metrics.
Here's an example:

{{< tabs groupid="config-with-json" >}}
{{% tab title="toml" %}}

```toml
version = "1"

[root]
  prometheus-save-to-file = "root.prom"
  prometheus-push = "http://localhost:9091/"

  [[root.prometheus-labels]]
    host = "{{ .Hostname }}"

  [root.backup]
    extended-status = true
    no-error-on-warning = true
    source = [ "/" ]
```

{{% /tab %}}
{{% tab title="yaml" %}}

```yaml
version: "1"

root:
  prometheus-save-to-file: "root.prom"
  prometheus-push: "http://localhost:9091/"
  prometheus-labels:
    - host: {{ .Hostname }}
  backup:
    extended-status: true
    no-error-on-warning: true
    source:
      - /
```

{{% /tab %}}
{{% tab title="hcl" %}}

```hcl
"root" = {
  "prometheus-save-to-file" = "root.prom"
  "prometheus-push" = "http://localhost:9091/"

  "prometheus-labels" = {
    "host" = "{{ .Hostname }}"
  }

  "backup" = {
    "extended-status" = true
    "no-error-on-warning" = true
    "source" = ["/"]
  }
}
```

{{% /tab %}}
{{% tab title="json" %}}

```json
{
  "version": "1",
  "root": {
    "prometheus-save-to-file": "root.prom",
    "prometheus-push": "http://localhost:9091/",
    "prometheus-labels": [
      {
        "host": "{{ .Hostname }}"
      }
    ],
    "backup": {
      "extended-status": true,
      "no-error-on-warning": true,
      "source": [
        "/"
      ]
    }
  }
}
```

{{% /tab %}}
{{< /tabs >}}


which will add the `host` label to all your metrics.

<p><img src="https://grafana.com/blog/assets/img/blog/timeshift/grafana_release_icon.png" alt="grafana logo" title="grafana" align="right" height="60" /></p>

# Ansible Role: grafana

[![Build Status](https://travis-ci.org/cloudalchemy/ansible-grafana.svg?branch=master)](https://travis-ci.org/cloudalchemy/ansible-grafana)
[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Ansible Role](https://img.shields.io/badge/ansible%20role-cloudalchemy.grafana-blue.svg)](https://galaxy.ansible.com/cloudalchemy/grafana/)
[![GitHub tag](https://img.shields.io/github/tag/cloudalchemy/ansible-grafana.svg)](https://github.com/cloudalchemy/ansible-grafana/tags)

Provision and manage [grafana](https://github.com/grafana/grafana) - platform for analytics and monitoring

## Requirements

- Ansible >= 2.7 (It might work on previous versions, but we cannot guarantee it)
- libselinux-python on deployer host (only when deployer machine has SELinux)
- grafana >= 5.1 (for older grafana versions use this role in version 0.10.1 or earlier)
- jmespath on deployer machine. If you are using Ansible from a Python virtualenv, install *jmespath* to the same virtualenv via pip.

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `monitoring_grafana_use_provisioning` | true | Use Grafana provisioning capability when possible (**monitoring_grafana_version=latest will assume >= 5.0**). |
| `monitoring_grafana_provisioning_synced` | false | Ensure no previously provisioned dashboards are kept if not referenced anymore. |
| `monitoring_grafana_version` | latest | Grafana package version |
| `monitoring_grafana_yum_repo_template` | etc/yum.repos.d/grafana.repo.j2 | Yum template to use |
| `monitoring_grafana_manage_repo` | true | Manage package repo (or don't) |
| `monitoring_grafana_instance` | {{ ansible_fqdn \| default(ansible_host) \| default(inventory_hostname) }} | Grafana instance name |
| `monitoring_grafana_logs_dir` | /var/log/grafana | Path to logs directory |
| `monitoring_grafana_data_dir` | /var/lib/grafana | Path to database directory |
| `monitoring_grafana_address` | 0.0.0.0 | Address on which grafana listens |
| `monitoring_grafana_port` | 3000 | port on which grafana listens |
| `monitoring_grafana_cap_net_bind_service` | false | Enables the use of ports below 1024 without root privileges by leveraging the 'capabilities' of the linux kernel. read: http://man7.org/linux/man-pages/man7/capabilities.7.html |
| `monitoring_grafana_url` | "http://{{ monitoring_grafana_address }}:{{ monitoring_grafana_port }}" | Full URL used to access Grafana from a web browser |
| `monitoring_grafana_api_url` | "{{ monitoring_grafana_url }}" | URL used for API calls in provisioning if different from public URL. See [this issue](https://github.com/cloudalchemy/ansible-grafana/issues/70). |
| `monitoring_grafana_domain` | "{{ ansible_fqdn \| default(ansible_host) \| default('localhost') }}" | setting is only used in as a part of the `root_url` option. Useful when using GitHub or Google OAuth |
| `monitoring_grafana_server` | { protocol: http, enforce_domain: false, socket: "", cert_key: "", cert_file: "", enable_gzip: false, static_root_path: public, router_logging: false } | [server](http://docs.grafana.org/installation/configuration/#server) configuration section |
| `monitoring_grafana_security` | { admin_user: admin, admin_password: "" } | [security](http://docs.grafana.org/installation/configuration/#security) configuration section |
| `monitoring_grafana_database` | { type: sqlite3 } | [database](http://docs.grafana.org/installation/configuration/#database) configuration section |
| `monitoring_grafana_welcome_email_on_sign_up` | false | Send welcome email after signing up |
| `monitoring_grafana_users` | { allow_sign_up: false, auto_assign_org_role: Viewer, default_theme: dark } | [users](http://docs.grafana.org/installation/configuration/#users) configuration section |
| `monitoring_grafana_auth` | {} | [authorization](http://docs.grafana.org/installation/configuration/#auth) configuration section |
| `monitoring_grafana_ldap` | {} | [ldap](http://docs.grafana.org/installation/ldap/) configuration section. group_mappings are expanded, see defaults for example |
| `monitoring_grafana_session` | {} | [session](http://docs.grafana.org/installation/configuration/#session) management configuration section |
| `monitoring_grafana_analytics` | {} | Google [analytics](http://docs.grafana.org/installation/configuration/#analytics) configuration section |
| `monitoring_grafana_smtp` | {} | [smtp](http://docs.grafana.org/installation/configuration/#smtp) configuration section |
| `monitoring_grafana_alerting` | {} | [alerting](http://docs.grafana.org/installation/configuration/#alerting) configuration section |
| `monitoring_grafana_log` | {} | [log](http://docs.grafana.org/installation/configuration/#log) configuration section |
| `monitoring_grafana_metrics` | {} | [metrics](http://docs.grafana.org/installation/configuration/#metrics) configuration section |
| `monitoring_grafana_tracing` | {} | [tracing](http://docs.grafana.org/installation/configuration/#tracing) configuration section |
| `monitoring_grafana_snapshots` | {} | [snapshots](http://docs.grafana.org/installation/configuration/#snapshots) configuration section |
| `monitoring_grafana_image_storage` | {} | [image storage](http://docs.grafana.org/installation/configuration/#external-image-storage) configuration section |
| `monitoring_grafana_dashboards` | [] | List of dashboards which should be imported |
| `monitoring_grafana_dashboards_dir` | "dashboards" | Path to a local directory containing dashboards files in `json` format |
| `monitoring_grafana_datasources` | [] | List of datasources which should be configured |
| `monitoring_grafana_environment` | {} | Optional Environment param for Grafana installation, useful ie for setting http_proxy |
| `monitoring_grafana_plugins` | [] |  List of Grafana plugins which should be installed |
| `monitoring_grafana_alert_notifications` | [] | List of alert notification channels to be created, updated, or deleted |

Datasource example:

```yaml
monitoring_grafana_datasources:
  - name: prometheus
    type: prometheus
    access: proxy
    url: 'http://{{ prometheus_web_listen_address }}'
    basicAuth: false
```

Dashboard example:

```yaml
monitoring_grafana_dashboards:
  - dashboard_id: 111
    revision_id: 1
    datasource: prometheus
```

Alert notification channel example:

**NOTE**: setting the variable `monitoring_grafana_alert_notifications` will only come into
effect when `monitoring_grafana_use_provisioning` is `true`. That means the new
provisioning system using config files, which is available starting from Grafana
v5.0, needs to be in use.

```yaml
monitoring_grafana_alert_notifications:
  notifiers:
    - name: Channel 1
      type: email
      uid: channel1
      is_default: false
      send_reminder: false
      settings:
        addresses: "example@example.com"
        autoResolve: true
  delete_notifiers:
    - name: Channel 2
      uid: channel2
```

## Supported CPU Architectures

Historically packages were taken from different channels according to CPU architecture. Specifically, armv6/armv7 and aarch64/arm64 packages were via [unofficial packages distributed by fg2it](https://github.com/fg2it/grafana-on-raspberry). Now that Grafana publishes official ARM builds, all packages are taken from the official [Debian/Ubuntu](http://docs.grafana.org/installation/debian/#installing-on-debian-ubuntu) or [RPM](http://docs.grafana.org/installation/rpm/) packages.

## Example

### Playbook

Fill in the admin password field with your choice, the Grafana web page won't ask to change it at the first login.

```yaml
- hosts: all
  roles:
    - role: cloudalchemy.grafana
      vars:
        monitoring_grafana_security:
          admin_user: admin
          admin_password: enter_your_secure_password
```

## License

This project is licensed under MIT License. See [LICENSE](/LICENSE) for more details.

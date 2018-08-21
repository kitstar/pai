OpenPai leverage [Prometheus Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) to do
alert. Alertmanager can send administrator an email to indicate cluster failure or problem with
pre-configured rules.

# Configuration

To enable alert manager, you should configure it correctly:
* select a node to deploy alert manager, it can be a master node of kubernetes or any other node. Once you selected, you should mark this node with `alert-manager: "true"` in `cluster-configuration` file, as [example](../../cluster-configuration/cluster-configuration.yaml)
* configure alert manager by adding `alerting` fields under `prometheus` to services-configuration file. See [example configuration](../../cluster-configuration/services-configuration.yaml).

`alerting` fields has following subfield:

| Field Name | Description |
| --- | --- |
| alert_manager_port | port for alert manager to listen, make sure this port is not used in node |
| alert_receiver | which email should receive alert email |
| smtp_url | smtp server url for alert manager to connect |
| smtp_from | this email address is where alerting email sent from |
| smtp_auth_username | use this user name to login to smtp server. This user should be able to send email as `smtp_from`, can be same with `smtp_from` |
| smtp_auth_password | use this password to login to smtp server |

More advanced configurations for alert manager is not supported in pai, see official alert manager
[document](https://prometheus.io/docs/alerting/configuration/) for more options.

# Alerting rule

Checkout [rule directory](../prometheus-alert) to see rules we already used for alerting.
Following are these rule's triggering condition:

| Rule name | Triggering condition |
| --- | --- |
| k8sApiServerNotOk | response from api server's healthz page is not ok or connection error |
| k8sEtcdNotOk | response from etcd server's healthz page is not ok or connection error |
| k8sKubeletNotOk | response from each kubelet's healthz page is not ok or connection error |
| k8sDockerDaemonNotOk | docker daemon's status is not ok |
| NodeFilesystemUsage | One of node's free space in file system is less than 20% |
| NodeMemoryUsage | One of node's free memory is less than 20% |
| NodeCPUUsage | One of node's cpu idle time is less than 20% |
| NodeDiskPressure | kubernetes indicate one of node is under disk pressure |
| PaiServicePodNotRunning | kubernetes indicate one of pai service pod is not in running status |
| PaiServicePodNotReady | kubernetes indicate one of pai service pod is not in ready status |

If you want to add more rules, please reference syntax
[here](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/).
After adding rules, you should stop and start prometheus by using paictl

```
cd $pai-management
./paictl.py service stop -p $pai-config -n prometheus
./paictl.py service start -p $pai-config -n prometheus
```

Please fire a pull request if you find any rule useful.

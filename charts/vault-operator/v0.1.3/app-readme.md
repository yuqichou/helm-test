# Vault Operator
[vault-operator](https://github.com/coreos/vault-operator) Simplify vault cluster configuration and management.

__DISCLAIMER:__ While this chart has been well-tested, the vault-operator is still currently in beta. Current project status is available [here](https://github.com/coreos/vault-operator).

### Using the Vault cluster
See the [Vault usage guide](https://github.com/coreos/vault-operator/blob/master/doc/user/vault.md) on how to initialize, unseal, and use the deployed Vault cluster.

Consult the [monitoring guide](https://github.com/coreos/vault-operator/blob/master/doc/user/monitoring.md) on how to monitor and alert on a Vault cluster with Prometheus.

See the [recovery guide](https://github.com/coreos/vault-operator/blob/master/doc/user/recovery.md) on how to backup and restore Vault cluster data using the etcd opeartor

For an overview of the default TLS configuration or how to specify custom TLS assets for a Vault cluster see the [TLS setup guide](https://github.com/coreos/vault-operator/blob/master/doc/user/tls_setup.md).

**Warning:**
Upgrade `vault-operator` from 0.1.2 to 0.1.3 is not supported, if you wish to use the newest version you will need to re-deploy the `vault-operator 0.1.3`.

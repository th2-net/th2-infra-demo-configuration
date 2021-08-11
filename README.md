## Version 1.5.4 - scenario with interacting with MiniFixUI via WinAppDriver via th2-hand ##

This version based on [infra 1.5.4](https://github.com/th2-net/th2-infra/tree/release-v1.5.4)

### Additional preconditions and requirements:
1. Box with Windows 10 installed
1. [MiniFix](http://elato.se/minifix/download.html) application installed
1. [WinAppDriver](https://github.com/microsoft/WinAppDriver) installed and configured
1. Administrator privilidges to let WinAppDriver listen external address.


### Execution steps:
ℹ️  **Instructions to launch applications outside the cluster(ExternalBox functionality):** https://github.com/th2-net/th2-documentation/wiki/Connecting-external-box-to-cluster-using-kubectl
1. **Create new namespace using CRs from this branch** (please find general information about namespace creation below).
2. **Configure session in hand:**\
    An exaple:
```yaml
  custom-config:
    driversMapping:
      session-name:
        type: "windows"
        url: "http://<EXTERNAL_IP>:<PORT>"
```
where EXTERNAL_IP - Windows 10 machine IP with WinAppDriver opened and configured and MiniFix installed.
Port - Is opened (or forwareded) port which are listened by WinAppDriver
 
3. **Launch the test script.**

    **Test script**: https://github.com/th2-net/th2-demo-script/tree/ver-1.5.4-hand-win-scenario 


## Environment schema
![alt text](schema_win_154.png)


# General information about configuring Schema #

Schema can be configured to be deployed to kubernetes and managed by infra manager.
Behaviour is controlled by `k8s-propagation` property in the `infra-mgr-config.yml` file.
These are the possible values for this property:

- `off`  - No synchronization will be done
- `deny` - No synchronization will be done and associated namespace will be removed from the kubernetes
- `sync` - Synchronizes repository changes with kubernetes
- `rule` - Synchronizes repository changes with kubernetes. Also monitors resource changes in kubernetes and 
         brings them back to repository state
  
## Creating new namespace
1) Create a new branch based on master
2) Make all the wanted changes in the `CRs`.
3) If you want to add new component make sure to include it in `links-live.yml`, `dictionary-links.yml`, `codec-links.yml` (if needed) link files are placed in `links` directory.
4) If you are going to have several namespaces together, make sure to assign each component in each namespace with unique `nodePort`. (nodePorts must be unique across the namespaces too)components that require `nodePort` are: `rpt-data-provider`, `rpt-data-viewer`, `act`, `check1`. Following ports are reserved by th2-infra: `rmq ampq protocol: 32000`, `cassandra cql: 32010`, `ingress: 30000`.
5) Make sure that `k8s-propagation` property in `infra-mgr-config.yml` file is set to `sync` (only branches that have this property set to `sync` or `rule` will be deployed by infra manager).
6) commit all new branch to `git`. (After committing new namespace will be created automatically, it might take 20-40 seconds)

## Restarting existing namespace
There are two methods to restart the namespace

**Repository only method**
1) set `k8s-propagation` property to `deny` in `infra-mgr-config.yml`. Namespace will be deleted by infra manager during 30-60 secs.
2) set `k8s-propagation` property to `sync` or `rule` in `infra-mgr-config.yml`. Schema will be deployed by infra manager during 30-60 secs.

**Involving kubernetes**
1) set `k8s-propagation` property in `infra-mgr-config.yml` to `rule` and commit this change.
2) delete an existing namespace using `kubectl delete namespace NAMESPACE_NAME` command. Schema will be redeployed automatically after 30-60 secs

## Restarting single component
in order to restart single component just delete `pod` of that specific component using `kubectl delete pod POD_NAME -n POD_NAMESPACE` command or using kubernetes dashboard (if you have necessary privileges). After deleting, `pod` will be recreated automatically.  

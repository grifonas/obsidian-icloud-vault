#superset #telekom

# Superset Argo Setup
Controlled by two repos:
- [mpathic-cockpit-argo-root](https://gitlab.devops.telekom.de/eliza/mpathic-cockpit-argo-root)
	- Responsible for the "Argo manifest"
	- Can be tracked in Argo under its own ["parent" app](https://argocd.dev.oc.telekom.net/applications/mpathic-cockpit-argo-root).
- [mpathic-cockpit-deployment-configurations](https://gitlab.devops.telekom.de/eliza/mpathic-cockpit-deployment-configurations).
	- Responsible for the actual charts
# Upgrade Process

Concerns the [mpathic-cockpit-deployment-configurations](https://gitlab.devops.telekom.de/eliza/mpathic-cockpit-deployment-configurations) repo.

1. Get the new release:
```
helm repo update
helm pull superset/superset 
```

2. Backup curr. superset folder in `mpathic-cockpit-deployment-configurations` repo:
```
CURR_VERSION=$(cat superset/Chart.yaml | grep appVersion | awk '{print $NF}')
mv superset superset-
```

3. Extract the downloaded version into a new `superset` folder

4. Copy values.[ENV].yaml from previous version to the new one:

```
cp superset-${CURR_VERSION}/values.*.yaml superset/
```

5. Compare `superset-${CURR_VERSION}/values.yaml` with the new `superset/values.yaml`. Ensure the values are in order.
6. Compare
	1. `superset/values.[ENV].yaml` with the old `superset-${CURR_VERSION}/values.[ENV].yaml`
	2. `superset/secrets-[ENV].yaml` with the old `superset${CURR_VERSION}/secrets-[ENV].yaml`
7. If it's `PROD`  get someone from Telekom to backup the DB. We've been asking **Ralf Becker**.
8. Commit and push
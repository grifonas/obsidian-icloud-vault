#superset #telekom

# Superset Upgrade
# Superset Argo Setup Description
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


2. Backup current superset folder in `mpathic-cockpit-deployment-configurations` repo:
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


7. Copy `extra_config.py` from the previous Helm chart folder into the new one.


8. If it's `PROD`  get someone from Telekom to backup the DB. We've been asking **Ralf Becker**.


9. Commit and push


# To Do on 18.10.2023:
- [x] Backup
- [x] Export all data in the UI
	- [x] Dasboards
	- [x] Datasets
	- [x] Databases
	- [x] Charts
- [x] Push argo root and deploy configs (root will point to existing version in `superset-2.0.1/superset-0.8.5`)
- [x] Check
	- [x]  Argo Root: https://argocd.oc.telekom.net/applications/mpathic-cockpit-argo-root?resource=
	- [x] Argo Superset: https://argocd.oc.telekom.net/applications/superset?resource=

- [ ] Change `root` to point to `superset` - the new version
- [ ] Check here: https://superset.mpathic-cockpit.oc.telekom.net/superset/welcome/




# Prod Connection to Athena:
```
awsathena+rest://athena.eu-central-1.amazonaws.com/analytics_dwh_prod_public?s3_staging_dir=s3%3A%2F%2Fprod-ci-mpathic-analytics-dwh%2Fsuperset
```

```
GUEST_ROLE_NAME="embedded"
WTF_CSRF_ENABLED=False
FEATURE_FLAGS = {
  'TAGGING_SYSTEM': True,
  'EMBEDDED_SUPERSET': True,
}


extraEnv:
  TALISMAN_ENABLED : False
```
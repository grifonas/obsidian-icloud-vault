
# Datahub Upgrade Issues

### missing `datahub-auth-secrets` on fresh helm install
**An upgrade job fails with**:

	```
	Error: secret "datahub-auth-secrets" not found
	```
**To Fix**:
- Option 1:
	Values.yaml. Described [here](https://github.com/acryldata/datahub-helm/issues/295). Tested on a fresh install.  Worked 👇
	```yaml
	global:
	  datahub:
	    metadata_service_authentication:
	      enabled: true
	      systemClientId: "__datahub_system"
	      systemClientSecret:
	        secretRef: "datahub-auth-secrets"
	        secretKey: "system_client_secret"
	      tokenService:
	        signingKey:
	          secretRef: "datahub-auth-secrets"
	          secretKey: "token_service_signing_key"
	        salt:
	          secretRef: "datahub-auth-secrets"
	          secretKey: "token_service_salt"
	      provisionSecrets:
	        enabled: true
	        autoGenerate: true
	        annotations:
	          helm.sh/hook: "pre-install,pre-upgrade"
	          helm.sh/hook-weight: "-6"
	```
- Option 2. From Datahub Slack:
```
I basically set global.datahub.systemUpdate to false and tried to install. Secrets got created. Then I set the value to true and did upgrade.
This may not be the best solution .This might get fixed in later releases I hope.
```
NOT TESTED ☝
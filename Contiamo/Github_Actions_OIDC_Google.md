#tech-tip #oidc #gcp

# Github Actions > GCP OIDC

Taken from https://gist.githubusercontent.com/palewire/12c4b2b974ef735d22da7493cf7f4d37/raw/ee0126af9848e89658085e227de531b480726014/README.md

## Create a Workload Identity Federation

The first step is to create a [Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation) that will allow your GitHub Action to log in to your Google Cloud account. The instructions below are cribbed from the documentation for the [google-github-actions/auth](https://github.com/google-github-actions/auth#setting-up-workload-identity-federation) Action. You should follow along in your terminal.

The first command creates a [service account](https://cloud.google.com/iam/docs/service-accounts) with Google. I will save the name I make up, as well as my Google project id, as environment variables for reuse. You should adapt the variables here, and others as we continue, to fit your project and preferred naming conventions.

```bash
export PROJECT_ID=my-project-id
export SERVICE_ACCOUNT=my-service-account

gcloud iam service-accounts create "${SERVICE_ACCOUNT}" \
  --project "${PROJECT_ID}"
```

Enable Google's IAM API for use.

```bash
gcloud services enable iamcredentials.googleapis.com \
  --project "${PROJECT_ID}"
```

Create a workload identity pool that will manage the GitHub Action's roles in Google Cloud's permission system.

```bash
export WORKLOAD_IDENTITY_POOL=my-pool

gcloud iam workload-identity-pools create "${WORKLOAD_IDENTITY_POOL}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="${WORKLOAD_IDENTITY_POOL}"
```

Get the unique identifier of that pool.

```bash
gcloud iam workload-identity-pools describe "${WORKLOAD_IDENTITY_POOL}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --format="value(name)"
```

Export the returned value to a new variable.

```bash
export WORKLOAD_IDENTITY_POOL_ID=whatever-you-got-back
```

Create a provider within the pool for GitHub to access.

```bash
export WORKLOAD_PROVIDER=my-provider

gcloud iam workload-identity-pools providers create-oidc "${WORKLOAD_PROVIDER}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="${WORKLOAD_IDENTITY_POOL}" \
  --display-name="${WORKLOAD_PROVIDER}" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```

Allow a GitHub Action based in your repository to login to the service account via the provider.

```bash
export REPO=my-username/my-repo

gcloud iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"
```

Ask Google to return the identifier of that provider.

```bash
gcloud iam workload-identity-pools providers describe "${WORKLOAD_PROVIDER}" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="${WORKLOAD_IDENTITY_POOL}" \
  --format="value(name)"
```

That will return a string that you should save for later. We'll use it in our GitHub Action.

Finally, we need to make sure that the service account we created at the start has permission to muck around with Google Artifact Registry.

```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/artifactregistry.admin"
```

To verify that worked, you can ask Google print out the permissions assigned to the service account.

```
gcloud projects get-iam-policy $PROJECT_ID \
    --flatten="bindings[].members" \
    --format='table(bindings.role)' \
    --filter="bindings.members:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com"
```

## Create a Google Artifact Registry repository

Go to the [Google Artifact Registry](https://console.cloud.google.com/artifacts) interface within your project. Create a new repository by hitting the buttona at the top. Tell Google it will be in the Docker format and then select a region. It doesn't matter which region. Save the name you give the repo and the region's abbreviation, which will be something like `us-west1`.


## Prerequisites
 * GCP Project created 
 `gcloud projects create example-foo-bar-1 --name="Happy project" --labels=type=happy` 
 * Workload identity [pools and providers](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers) have been created 
 * The roles/iam.workloadIdentityUser role has been granted to the service account that will authenticate with workload identity federation. See [documentation](https://cloud.google.com/blog/products/identity-security/secure-your-use-of-third-party-tools-with-identity-federation) for guidance on provisioning workload identity with GitHub Actions.
 * The service account has the following roles. 
    * roles/artifactregistry.writer
    * roles/run.developer
 * All required APIs for Google Cloud services have been enabled.
 * [Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images) repository must be created. 
```
gcloud projects create example-foo-bar-1 --name="Happy project" --labels=type=happy

```

enable services

```

 gcloud services enable artifactregistry.googleapis.com iam.googleapis.com run.googleapis.com sts.googleapis.com cloudresourcemanager.googleapis.com

```

create wif

```
gcloud iam workload-identity-pools create "my-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="Demo pool"

gcloud iam workload-identity-pools providers create-oidc "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --display-name="Demo provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.aud=assertion.aud" \
  --issuer-uri="https://token.actions.githubusercontent.com"

```
create service account:

```
gcloud iam service-accounts create cloudrun-deploy --display-name="Cloud run deploy SA"
```

get the workload identity pool Project number

```
Get the project number by running gcloud projects list --filter="PROJECT_ID=${PROJECT_ID}" --format="value(PROJECT_NUMBER)"

```
assign SA proper roles
```
gcloud iam service-accounts add-iam-policy-binding "my-service-account@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/1234567890/locations/global/workloadIdentityPools/my-pool/attribute.repository/my-org/my-repo"
```


create repository
```
gcloud artifacts repositories create my-cloudrun-repo \
    --repository-format=docker \
    --location=us \
    --description="store cloud run images" 
    
```

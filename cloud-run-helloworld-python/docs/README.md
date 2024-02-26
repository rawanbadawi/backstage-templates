

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
export PROJECT_ID=example-foo-bar-1

```

```
gcloud projects create ${PROJECT_ID}--name="Happy project" --labels=type=happy

```

enable services

```
gcloud config set project ${PROJECT_ID}

 gcloud services enable artifactregistry.googleapis.com iam.googleapis.com run.googleapis.com sts.googleapis.com cloudresourcemanager.googleapis.com

```

create wif

```
export MY_ORG=rawanbadawi
gcloud iam workload-identity-pools create "my-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="Demo pool"

gcloud iam workload-identity-pools providers create-oidc "my-provider1" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --display-name="Demo provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.aud=assertion.aud,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com" \ 
  --attribute-condition=attribute.repository.contains('${MY_ORG}')
```
create service account:

```
export SA_NAME="cloudrun-deploy"
gcloud iam service-accounts create ${SA_NAME} --display-name="Cloud run deploy SA"
```

get the workload identity pool Project number

```

export PROJECT_NUMBER=`gcloud projects list --filter="PROJECT_ID=${PROJECT_ID}" --format="value(PROJECT_NUMBER)"`

```
assign SA proper roles
```
export MY_ORG="rawanbadawi"
export MY_REPO="cloud-run-deploy"
gcloud iam service-accounts add-iam-policy-binding "${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/my-pool/attribute.repository/${MY_ORG}/${MY_REPO}"
```
    gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/artifactregistry.writer

     gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/iam.serviceAccountUser

     gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/run.developer

create repository
```
gcloud artifacts repositories create my-cloudrun-repo \
    --repository-format=docker \
    --location=us \
    --description="store cloud run images" 
    
```

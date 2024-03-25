

## Prerequisites
 * GCP Project created 
 `gcloud projects create example-foo-bar-1 --name="Happy project" --labels=type=happy` 
 * Workload identity [pools and providers](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers) have been created 
 * The roles/iam.workloadIdentityUser role has been granted to the service account that will authenticate with workload identity federation. See [documentation](https://cloud.google.com/blog/products/identity-security/secure-your-use-of-third-party-tools-with-identity-federation) for guidance on provisioning workload identity with GitHub Actions.
 * The service account has the following roles. 
    * roles/artifactregistry.writer
    * roles/run.admin
 * All required APIs for Google Cloud services have been enabled.
 * [Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images) repository must be created. 
```
# authorize gcloud to access the Cloud Platform with Google user credentials
export PROJECT_ID=example-foo-bar-1
gcloud auth login
```

```
# create a new Google project. Skip if you are using an existing project
gcloud projects create ${PROJECT_ID}--name="Happy project" --labels=type=happy

```

enable services

```
gcloud config set project ${PROJECT_ID}

 gcloud services enable artifactregistry.googleapis.com iam.googleapis.com run.googleapis.com sts.googleapis.com cloudresourcemanager.googleapis.com

```

Create Workload Identity Federation pool and provider for [keyless authentication from Github Actions](https://cloud.google.com/blog/products/identity-security/enabling-keyless-authentication-from-github-actions)

```
export MY_ORG=MY_ORG # the github org or username
gcloud iam workload-identity-pools create "my-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="Demo pool"

gcloud iam workload-identity-pools providers create-oidc "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --display-name="Demo provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.aud=assertion.aud,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com" 
```
create service account used to deploy cloud run via Github Actions:

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
export MY_REPO="cloud-run-deploy"
gcloud iam service-accounts add-iam-policy-binding "${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/my-pool/attribute.repository/${MY_ORG}/${MY_REPO}"

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/artifactregistry.writer

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/iam.serviceAccountUser

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role=roles/run.developer

# WARNING DO NOT USE IN PROD Only for demo purposes. 

gcloud projects add-iam-policy-binding ${PROJECT_ID} --member="serviceAccount:${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --roles/run.services.setIamPolicy
```

Create Artifact Registry Docker repository
```
gcloud artifacts repositories create my-cloudrun-repo \
    --repository-format=docker \
    --location=us \
    --description="store cloud run images" 
    
```
Run the template and remember to save the names of the project number, project ID, Artifact registry name, location etc... to input as steps in the template
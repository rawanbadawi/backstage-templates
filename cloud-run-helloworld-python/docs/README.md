

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

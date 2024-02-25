# Continuous Integration & Continuous Deployment for Cloud Run
This repository is designed to lint code, scan code, build docker container, push to artifact registry and deploy packaged code to Cloud Run.

## Prerequisites
 * GCP Project created
 * Workload identity [pools and providers](https://cloud.google.com/iam/docs/manage-workload-identity-pools-providers) have been created and added to [GitHub Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).
 * The roles/iam.workloadIdentityUser role has been granted to the service account that will authenticate with workload identity federation. See [documentation](https://cloud.google.com/blog/products/identity-security/secure-your-use-of-third-party-tools-with-identity-federation) for guidance on provisioning workload identity with GitHub Actions.
 * The service account has been added to GitHub Secrets and has the following roles. 
    * roles/artifactregistry.writer
    * roles/run.developer
 * All required APIs for Google Cloud services have been enabled.
 * [Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images) repository must be created. 



## Deploying Cloud Run
1. Create a new feature branch from main, make necessary changes to your code. 
2. Raise a pull request from your new feature branch to main, 
3. When the pull request is raised, the workflow will lint the code to ensure quality and CodeQL will scan the code to ensure there are no vulnerabilities in the code. 
4. If there are no linting issues or CodeQL vulnerabilities, the pull request can be merged after the workflow completes and approvals are received. 
5. Once merged, the image would be built and pushed to Artifact Registry in the Google Cloud project used for development.
6. In main, once the image is built, it will immediately be deployed to Cloud Run as a new revision in the development project. 

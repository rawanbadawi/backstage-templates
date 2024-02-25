# Continuous Integration & Continuous Deployment for ${{values.serviceName}} on Cloud Run
This repository is designed to lint code, scan code, build docker container, push to artifact registry and deploy packaged code to Cloud Run.

## Deploying Cloud Run
1. Create a new feature branch from main, make necessary changes to your code. 
2. Raise a pull request from your new feature branch to main, 
3. When the pull request is raised, the workflow will lint the code to ensure quality and CodeQL will scan the code to ensure there are no vulnerabilities in the code. 
4. If there are no linting issues or CodeQL vulnerabilities, the pull request can be merged after the workflow completes and approvals are received. 
5. Once merged, the image would be built and pushed to Artifact Registry in the Google Cloud project used for development.
6. In main, once the image is built, it will immediately be deployed to Cloud Run as a new revision in the development project. 

# Backstage Templates for GCP

This repo will host templates to deploy and publish artifacts within backstage.

## What is Backstage?



## How can I get started?

Install and configure backstage, if you are new, here is a [step by step tutorial you can follow](https://backstage.spotify.com/learn/standing-up-backstage/standing-up-backstage/1-intro/)

Once you reach add components stop and change the - allow array from this

```yaml
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location]

```

to 

```yaml
catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, API, Resource, Location, Template]  # < add Template


```

restart the application cntl-C then yarn dev .

Then proceed with adding components and use this link to register

`https://github.com/rawanbadawi/backstage-templates/blob/main/catalog-info.yaml`


## Running the Template

Click Create > Then click Choose on the desired template to run. Fill in the steps and run. 


Please make sure to run the pre-reqs for the templates, for example our starter Cloud Run template's pre-reqs are [documented here](https://github.com/rawanbadawi/backstage-templates/tree/main/cloud-run-helloworld-python) 

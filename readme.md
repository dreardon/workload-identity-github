# Github Actions Workload Identity Federation

This is the Github Actions portion of a multi project set of examples for configuring Google Cloud Workload Identity Federation. For further information on this set-up please refer to Google's documentation here: https://cloud.google.com/iam/docs/workload-identity-federation-with-deployment-pipelines#github-actions

## Google Disclaimer
This is not an officially supported Google product

## Table of Contents
1. [Prerequisites](https://github.com/dreardon/workload-identity-github#prerequisites)
1. [Google Service Account and Identity Pool](https://github.com/dreardon/workload-identity-github#create-a-google-service-account-and-identity-pool)
1. [Connect Identity Pool to Github](https://github.com/dreardon/workload-identity-github#connect-identity-pool-to-github)

## Prerequisites
<ul type="square"><li>An existing Google Project, you'll need to reference PROJECT_ID later in this setup</li>
<li>Enabled services</li>

```
gcloud services enable iamcredentials.googleapis.com
```
</ul>

## Create a Google Service Account and Identity Pool
```
export PROJECT_ID=[Google Project ID]
export PROJECT_NUMBER=[Google Project Number]
export WORKLOAD_IDENTITY_POOL=[Workload Identity Pool] #New Workload Identity Pool Name

gcloud config set project $PROJECT_ID

gcloud iam workload-identity-pools create $WORKLOAD_IDENTITY_POOL \
    --location="global" \
--description="Workload Identity Pool for Github Actions" \
--display-name="Github Actions Workload Pool"

```

## Connect Identity Pool to Github

gcloud iam workload-identity-pools describe $WORKLOAD_IDENTITY_POOL \
  --project="${PROJECT_ID}" \
  --location="global" \
  --format="value(name)"

gcloud iam workload-identity-pools providers create-oidc $WORKLOAD_IDENTITY_PROVIDER \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool=$WORKLOAD_IDENTITY_POOL \
  --display-name="My GitHub Organization" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository,attribute.repository_owner=assertion.repository_owner" \
  --attribute-condition="assertion.repository_owner == '${GITHUB_ORG}'" \
  --issuer-uri="https://token.actions.githubusercontent.com"

gcloud iam workload-identity-pools providers describe $WORKLOAD_PROVIDER \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool=$WORKLOAD_IDENTITY_POOL \
  --format="value(name)"


gcloud secrets add-iam-policy-binding github-actions-wif-secret \
  --project="${PROJECT_ID}" \
  --role="roles/secretmanager.secretAccessor" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/${WORKLOAD_IDENTITY_POOL}/*"

```
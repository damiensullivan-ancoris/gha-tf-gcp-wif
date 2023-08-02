# gha-tf-gcp-wif
## Github Actions + Terraform + GCP + Workload Identity Federation 
###USING WIF Google Authentication for GITHub Actions:

## Setup GCP
1.	export PROJECT_ID="ci-cd-terraform-cloudbuild" # update with your value

2.	gcloud iam service-accounts create "gha-test" \
  --project "${PROJECT_ID}"

3. (Optional) Grant the Google Cloud Service Account permissions to access Google Cloud resources. This step varies by use case. For demonstration purposes, you could grant access to a Google Secret Manager secret or Google Cloud Storage object.

4. gcloud services enable iamcredentials.googleapis.com \
  --project "${PROJECT_ID}"

export WORKLOAD_IDENTITY_POOL_ID="projects/1048815135427/locations/global/workloadIdentityPools/my-pool"

5. gcloud iam workload-identity-pools create "my-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="Demo pool"

6. gcloud iam workload-identity-pools describe "my-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --format="value(name)"

6.1 export WORKLOAD_IDENTITY_POOL_ID="projects/1048815135427/locations/global/workloadIdentityPools/my-pool" # value from above

7. gcloud iam workload-identity-pools providers create-oidc "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --display-name="Demo provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"

8. export REPO="damiensullivan-ancoris/gha-tf-gcp-wif" 

8.1 gcloud iam service-accounts add-iam-policy-binding "gha-test@ci-cd-terraform-cloudbuild.iam.gserviceaccount.com" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"

9. gcloud iam workload-identity-pools providers describe "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --format="value(name)"

projects/1048815135427/locations/global/workloadIdentityPools/my-pool/providers/my-provider

## Setup Github Action Workflow: 
1. In Github GUI - Actions - New Workflow - setup a workflow yourself
2. Paste in the following code and Commit Changes.

   
  name: wif-test
  on: push
  jobs:
    my-job: 
      runs-on: ubuntu-latest
      
      # Add "id-token" with the intended permissions.
      permissions:
        contents: 'read'
        id-token: 'write'
  
      steps:
      # actions/checkout MUST come before auth
      - uses: 'actions/checkout@v3'
  
      - id: 'auth'
        name: 'Authenticate to Google Cloud'
        uses: 'google-github-actions/auth@v1'
        with:
          workload_identity_provider: 'projects/1048815135427/locations/global/workloadIdentityPools/my-pool/providers/my-provider'
          service_account: 'gha-test@ci-cd-terraform-cloudbuild.iam.gserviceaccount.com'
  
      # ... further steps are automatically authenticated
      - name: my-step
        run: echo "Hello World!"
        
      # Install GCLOUD
      - name: Install GCLOUD
        uses: 'google-github-actions/setup-gcloud@v1'
        
      # Get a GCP Secret
      - id: 'gcloud'
        name: 'gcloud'
        run: |-
          gcloud secrets versions access "latest" --secret "test-secret"

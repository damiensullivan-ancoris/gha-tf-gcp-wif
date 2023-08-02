# gha-tf-gcp-wif

### Github Actions + Terraform + GCP + Workload Identity Federation 

USING WIF Google Authentication for GITHub Actions:

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

# This should look like:
#
#   projects/123456789/locations/global/workloadIdentityPools/my-pool
#

7. gcloud iam workload-identity-pools providers create-oidc "my-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="my-pool" \
  --display-name="Demo provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"

8. export REPO="damiensullivan-ancoris/gha-tf-gcp-wif" 

** STEP 8 I messed up and got a similar error to yours 
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

# test-helm

-> Firstly We need to create one repo in GitHub and then add the app.py, Dockerfile, deployment.yaml, service.yaml and chart.yaml

-> Enable API's in GCP CLI 
gcloud services enable container.googleapis.com
gcloud services enable artifactregistry.googleapis.com
gcloud services enable iamcredentials.googleapis.com

-> Create Artifact repo from GCP CLI 
gcloud artifacts repositories create repo2 \
  --repository-format=docker \
  --location=us-central1 \
  --description="Python app repo"

-> Create GKE cluster
gcloud container clusters create gke \
  --zone us-central1-a \
  --num-nodes 2 \
  --disk-size 25

-> Grant roles to service account (Change the project id and service account name)
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:github-actions@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/container.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:github-actions@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/artifactregistry.admin"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:github-actions@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/iam.workloadIdentityUser"

-> Create Workload Identity Federation Pool
gcloud iam workload-identity-pools create pool1 \
  --project=$PROJECT_ID \
  --location="global" \
  --display-name="GitHub Pool"

-> Create WIF provider for pool created above (Change the pool name,Github username and Github reponame)
gcloud iam workload-identity-pools providers create-oidc github-provider1 \
    --location="global" \
    --workload-identity-pool=“pool1” \
    --issuer-uri="https://token.actions.githubusercontent.com" \
    --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
    --attribute-condition="attribute.repository=='<YOUR_GITHUB_USER>/<YOUR_REPO_NAME>'"

-> Bind Service account to WIF (Change the service account name, project id, project number, workload identity pool name, github username and github repo name)
gcloud iam service-accounts add-iam-policy-binding \
  github-actions@$PROJECT_ID.iam.gserviceaccount.com \
  --project=$PROJECT_ID \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/$PROJECT_NUMBER/locations/global/workloadIdentityPools/$POOL_ID/attribute.repository/'git-username/repo-name'”

-> Now add the values.yaml file (Change the project id and Artifact repo name)

-> In the next step we need to add deploy.yaml file
.github/workflows/deploy.yaml (Change the project id, region, zone, cluster name, artifact repo name, service account name, project id, project number, workload identity pool name, workload identity provider name)

-> After adding the above file then click on the commit changes. Then now go to the actions tab. The Actions will be automatically running.

-> Once after the successful deployment of actions go to the GCP CLI and run below commands
kubectl get pods -n default
Kubectl get svc 

-> You'll be getting the External IP. Copy and paste that external in the new browser tab and change the https to http. You'll be getting the output as what we mentioned in the app.py file. 

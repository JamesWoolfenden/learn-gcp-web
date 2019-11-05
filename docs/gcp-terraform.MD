https://cloud.google.com/community/tutorials/managing-gcp-projects-with-terraform

# First the environment
export TF_VAR_org_id=YOUR_ORG_ID
export TF_VAR_billing_account=YOUR_BILLING_ACCOUNT_ID
export TF_ADMIN=${USER}-terraform-admin
export TF_CREDS=~/.config/gcloud/${USER}-terraform-admin.json

gcloud organizations list
gcloud beta billing accounts list

#create terraform project and link to main
`gcloud projects create ${TF_ADMIN} --organization ${TF_VAR_org_id}  --set-as-default `

` gcloud beta billing projects link ${TF_ADMIN} --billing-account ${TF_VAR_billing_account}`

on windows:
` gcloud projects create $env:TF_ADMIN --organization $env:TF_VAR_org_id --set-as-default `

` gcloud beta billing projects link $env:TF_ADMIN --billing-account $env:TF_VAR_billing_account `

#sort a service account out
gcloud iam service-accounts create terraform \
  --display-name "Terraform admin account"

gcloud iam service-accounts keys create ${TF_CREDS} \
  --iam-account terraform@${TF_ADMIN}.iam.gserviceaccount.com

on Windows
` gcloud iam service-accounts create terraform   --display-name "Terraform admin account" `

`gcloud iam service-accounts keys create $env:TF_CREDS --iam-account "terraform@$env:TF_ADMIN.iam.gserviceaccount.com" `


gcloud projects add-iam-policy-binding ${TF_ADMIN} \
  --member serviceAccount:terraform@${TF_ADMIN}.iam.gserviceaccount.com \
  --role roles/viewer

gcloud projects add-iam-policy-binding ${TF_ADMIN} \
  --member serviceAccount:terraform@${TF_ADMIN}.iam.gserviceaccount.com \
  --role roles/storage.admin

on windows:
  gcloud projects add-iam-policy-binding $env:TF_ADMIN --member "serviceAccount:terraform@$env:TF_ADMIN.iam.gserviceaccount.com" --role roles/viewer

gcloud projects add-iam-policy-binding $env:TF_ADMIN  --member "serviceAccount:terraform@$env:TF_ADMIN.iam.gserviceaccount.com" --role roles/storage.admin

# enable some apis
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable cloudbilling.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable compute.googleapis.com

gcloud organizations add-iam-policy-binding ${TF_VAR_org_id} \
  --member serviceAccount:terraform@${TF_ADMIN}.iam.gserviceaccount.com \
  --role roles/resourcemanager.projectCreator

gcloud organizations add-iam-policy-binding ${TF_VAR_org_id} \
  --member serviceAccount:terraform@${TF_ADMIN}.iam.gserviceaccount.com \
  --role roles/billing.user

  on Windows:
gcloud organizations add-iam-policy-binding $env:TF_VAR_org_id --member "serviceAccount:terraform@$env:TF_ADMIN.iam.gserviceaccount.com"  --role roles/resourcemanager.projectCreator

gcloud organizations add-iam-policy-binding $env:TF_VAR_org_id --member "serviceAccount:terraform@$env:TF_ADMIN.iam.gserviceaccount.com"  --role roles/billing.user

# make a bucket
gsutil mb -p ${TF_ADMIN} gs://${TF_ADMIN}

cat > remote_state.tf <<EOF
terraform {
 backend "gcs" {
   bucket  = "${TF_ADMIN}"
   prefix  = "terraform/state"
   project = "${TF_ADMIN}"
 }
}
EOF

or on WIndows:
gsutil mb -p $env:TF_ADMIN "gs://$env:TF_ADMIN"

$content=@"
terraform {
    backend "gcs" {
           bucket  = "$env:TF_ADMIN"
           prefix  = "terraform/state"
           project = "$env:TF_ADMIN" 
           }
}
"@           
$content|set-content remote_state.tf


#versioning on bucket
` gsutil versioning set on gs://${TF_ADMIN} `
on WIndows:
`gsutil versioning set on gs://$env:TF_ADMIN `


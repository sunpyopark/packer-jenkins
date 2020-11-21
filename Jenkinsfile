pipeline { //Opening Pipeline Statement

agent any //Keeping it simple

environment { //Initialize important variables

REGION = 'us-east-1' //alt can be a param

ENVIRONMENT = 'environment' //alt can be a param

APPLICATION = 'server' //alt can be a param

PACKER_ACCESS_KEY="\$(aws ssm get-parameters --region \"${REGION}\" --name \"/packer/akey\" --query 'Parameters[0].Value' | tr -d '\"\')"

PACKER_SECRET_KEY="\$(aws ssm get-parameters --region \"${REGION}\" --name \"/packer/skey\" --query 'Parameters[0].Value' | tr -d '\"\')"

TERRAFORM_ROLE = "\$(aws ssm get-parameters --region \"${REGION}\" --name \"/terraform/role\" --query 'Parameters[0].Value' | tr -d '\"\')"

TERRAFORM_BUCKET = "\$(aws ssm get-parameters --region \"${REGION}\" --name \"/terraform/bucket\" --query 'Parameters[0].Value' | tr -d '\"\')"

}

stages {

stage ('Fetch') { //Clone Github repo to workspace

steps {

checkout scm

}

}

stage('Build') { //Build AMI based on Packer templates

steps {

dir('packer') {

sh “rm manifest.json -f”

sh "packer.io validate -var \"region=${REGION}\" -var \"environment=${ENVIRONMENT}\" -var \"aws_access_key=${ACCESS_KEY}\" -var \"aws_secret_key=${SECRET_KEY}\" template.json"

sh "packer.io build -var \"region=${REGION}\" -var \"environment=${ENVIRONMENT}\" -var \"aws_access_key=${ACCESS_KEY}\" -var \"aws_secret_key=${SECRET_KEY}\" template.json"

sh "cat manifest.json | jq .builds[0].artifact_id | tr -d '\"' | cut -b 11- > .ami"

script {

AMI_ID = readFile('.ami').trim()

}}}}

stage('Deploy') {//Deploy newly created AMI

steps {

dir('terraform') {

sh "terraform init -backend-config=\"role_arn=${TERRAFORM_ROLE}\" -backend-config=\"bucket=${TERRAFORM_BUCKET}\""

sh "terraform plan -var \"region=${REGION}\" -var \"ami_id=${AMI_ID}\" -var \"role_arn=${TERRAFORM_ROLE}\" -var \"environment=${ENVIRONMENT}\""

sh "terraform apply -var \"region=${REGION}\" -var \"ami_id=${AMI_ID}\" -var \"role_arn=${TERRAFORM_ROLE}\"  -var \"environment=${ENVIRONMENT}\" --auto-approve"

}}}}}

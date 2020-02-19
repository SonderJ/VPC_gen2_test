# VPC_GEN2_test
just to do some vpc tests in IBM Cloud
## preparation
### things to read and watch
1. About Terraform
   - [Terraform introduction video](https://www.ibm.com/cloud/blog/new-builders/terraform)
   - [O'Reilly Book: Terraform Up & Running](https://learning.oreilly.com/library/view/terraform-up-and/9781491977071/cover.html)
   - [HashiCorp Terraform reference](https://www.terraform.io/intro/index.html)
   - [IBM Cloud Terraform provider download from Github](https://github.com/IBM-Cloud/terraform-provider-ibm)
   - [IBM CLoud Provider documentation](https://ibm-cloud.github.io/tf-ibm-docs/)
   - [Terraformer: Automate creation of Terraform for IBM Virtual Private Cloud.](https://github.com/ibm-cloud-architecture/terraformer)

3. About Schematics
   - [General Availability of IBM Cloud Schematics, a fully open, managed Terraform solution for deploying cloud infrastructure](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-schematics-enabling-infrastructure-as-code)
   - [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started)
   
1. other related links

   - [Announcing IBM Cloud Collection for Ansible](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-collection-for-ansible)

Actually the [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started) explains the flow how to setup a a Schematics workspace and how to prepare your environment. However there are a lot of links left and right that fork quickly and might get you off the track and more confusing 

the essential steps are:
1. you have to have access to a fully functional account. **Don't try to create resources trought schematics with a Lite Account!** 
Even if `terraform plan` command completes successfully the `terraform apply` command will only make it half way through. It successfuly creates a VPC and security groups and maybe other "free" resources but it gets stuck when creating the actual VSI. It leaves you with a VSI-Zombie that remains in starting state and you will not be able to delete this VM manually through the GUI. The only way to get rid of this guy is to open a ticket and let the 3rd Level support kill him.

2. [Installing the stand-alone IBM Cloud CLI on your local system](https://cloud.ibm.com/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

For the first time use you IBM ID to log in with specifying the region 
´ibmcloud login --sso -r eu-de´

Creating an API key : https://cloud.ibm.com/docs/iam?topic=iam-userapikey#create_user_key
ibmcloud iam api-key-create MyKey -d "this is my API key" --file key_file
ibmcloud iam api-key-create jsonder -d "Joergs API key" --file js_api_key


logging in with an API-key
https://cloud.ibm.com/docs/iam?topic=iam-federated_id#federated_id
ibmcloud login --apikey @key_file_name -r eu-de

Install or update the VPC infrastructure service plug-in.
´ibmcloud plugin install vpc-infrastructure´

3. create ssh-key

    If you haven't done already in the past you have to create an ssh-key-pair and the public part of it has to be uploaded to your account
    
´ssh-keygen -t rsa -C "jsonderorhoweveryouwanttocallme" -b 2048´

the "-b 2048" is very important because in my ubuntu system the standard key length is 3072 which is not accepted by IBM Cloud (standard should be 2048)
 


You can upload the public key using the CLI:

´ibmcloud is key-create jsonder @js_ssh_key2048.pub --resource-group-name default´

... or via the IBM Cloud Portal 

 
   this page links to a couple of subtask you have to complete if you start on a green field
   - [ ] setting up the playground 
   - [x] setting up CLI for VPC
- create and publish you ssh key

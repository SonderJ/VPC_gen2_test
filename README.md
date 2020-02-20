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

Actually the [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started) explains the flow how to setup a Schematics workspace and how to prepare your environment. However there are a lot of links left and right that fork quickly and might get you off the track and might confuse more than help. 

So just a few escorting hints and essential steps ...

1. Create a [Github account](https://github.com/) to maintain and store your Terraform code
   If you want to use Terraform together with Schematics you have to use Github to link to your Terraform code templates
1. You a fully functional IBM Cloud account. 

   So either upgrade your free Lite Account or someone who geives you access to his account  

   **Don't try to create resources through schematics with a Lite Account!** 

   Even if the `terraform plan` command completes successfully the `terraform apply` command will only make it half way through. 
       It successfuly creates a VPC and security groups and maybe other "free" resources but it gets stuck when creating the actual VSI. It leaves you with a Zombie-VSI that remains in starting state and you will not be able to delete this VM (not with ´terraform destroy' nor manually through the GUI). The only way to get rid of this undead is to open a ticket and let the 3rd Level support kill him for you.

2. [Install the stand-alone IBM Cloud CLI on your local system](https://cloud.ibm.com/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

   You usually install the CLI directly on your Laptop but with Windows it comes with a Container environment that conflicts with other Hypervisors you may have installed so I finally decided to install everything on an Ubuntu-based VM (not all Unix derivates are supported by the CLI installer! - Ubuntu works fine , MINT doesn't)  
      
   For the first time use you IBM Cloud ID to log in with specifying the region (you don't need the ´-r´ option)
   ´ibmcloud login --sso -r us-south´
   This involves your web-browser in the log in process to supply you with an access token which needs to be copied back to your CLI. 

   If you want to avoid this web-browser-detour you better [create an API-key](https://cloud.ibm.com/docs/iam?topic=iam-userapikey#create_user_key) for future logins

   ´ibmcloud iam api-key-create MyKey -d "this is my API key" --file key_file´
   
   example: ´ibmcloud iam api-key-create jsonder -d "Joergs API key" --file js_api_key´

   - [logging in with an API-key](https://cloud.ibm.com/docs/iam?topic=iam-federated_id#federated_id)
   ´ibmcloud login --apikey @key_file_name´

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

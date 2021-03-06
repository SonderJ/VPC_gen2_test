# Terraform with Schematics on VPC Gen2

I just did some very simple testing to provision resources on IBM Cloud with Terraform and Schematices.

This is not another Schematics or Terraform guide (you better follow the links below) just some notes that may help others ...

### Things to read and watch
1. About Terraform
   - [Terraform introduction video](https://www.ibm.com/cloud/blog/new-builders/terraform)
   - [O'Reilly Book: Terraform Up & Running](https://learning.oreilly.com/library/view/terraform-up-and/9781491977071/cover.html)
   - [HashiCorp Terraform reference](https://www.terraform.io/intro/index.html)
   - [IBM Cloud Terraform provider download from Github](https://github.com/IBM-Cloud/terraform-provider-ibm)
   - [IBM CLoud Provider documentation](https://ibm-cloud.github.io/tf-ibm-docs/)
   - [Terraformer: Automate creation of Terraform for IBM Virtual Private Cloud.](https://github.com/ibm-cloud-architecture/terraformer)

3. About Schematics
   - [General Availability of IBM Cloud Schematics, a fully open, managed Terraform solution for deploying cloud infrastructure](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-schematics-enabling-infrastructure-as-code)
   - [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started) <= start here
   
1. other related links

   - [Announcing IBM Cloud Collection for Ansible](https://www.ibm.com/cloud/blog/announcements/ibm-cloud-collection-for-ansible)

Actually the [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started) explains the flow of setting up a Schematics workspace and how to prepare your environment. However there are a lot of links left and right that fork quickly and might get you off the track and might confuse more than help. 

#### So just a few escorting hints and essential steps ...

1. Create a [Github account](https://github.com/) to maintain and store your Terraform code
   If you want to use Terraform together with Schematics you have to use Github to link to your Terraform code templates
1. You need a fully functional IBM Cloud account. 

   So either upgrade your free Lite Account or someone who geives you access to his account  

   **Don't try to create resources with Terraform/Schematics on a Lite Account!** 

   Even if the `terraform plan` command completes successfully the `terraform apply` command will only make it half way through. 
   It successfuly creates a VPC and security groups and maybe other "free" resources but it gets stuck when creating the actual VSI. It leaves you with a Zombie-VSI that remains in starting state and you will not be able to delete this VM (not with ´terraform destroy' nor manually through the GUI). The only way to get rid of this undead is to open a ticket and let the 3rd Level support kill him for you.

3. [Install the stand-alone IBM Cloud CLI on your local system](https://cloud.ibm.com/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

   You usually install the CLI directly on your Laptop but with Windows it comes with a Container environment that conflicts with other Hypervisors you may have installed so I finally decided to install everything on an Ubuntu-based VM (not all Unix derivates are supported by the CLI installer! - Ubuntu works fine , MINT doesn't)  
      
   For the first time use you IBM Cloud ID to log in with specifying the region (you don't need the `-r` option it will ask you for hte region when you login the first time and you can of course change the current region in the CLI)
   `ibmcloud login --sso -r us-south`
   This involves your web-browser in the log in process to supply you with an access token which needs to be copied back to your CLI. 

   If you want to avoid this web-browser-detour you better [create an API-key](https://cloud.ibm.com/docs/iam?topic=iam-userapikey#create_user_key) for future logins

   `ibmcloud iam api-key-create MyKey -d "this is my API key" --file key_file`
 
   example: `ibmcloud iam api-key-create jsonder -d "Joergs API key" --file js_api_key`

   [logging in with an API-key](https://cloud.ibm.com/docs/iam?topic=iam-federated_id#federated_id)
   `ibmcloud login --apikey @key_file_name`

   Once you are logged in don't forget to install or update the VPC infrastructure service plug-in.
    `ibmcloud plugin install vpc-infrastructure`
    
3. Babysteps to get started with the CLI

   All CLI commands to deal with VPCs in IBM Cloud start with `ibmcloud is` (is = infrastructure service)
   
   So the most important command for you in the beginning will be `ibmcloud is --help` 
   
   ... end of baby steps guide :grinning: ... ok, the real CLI guide is [here](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started)

3. Create your ssh-key pair

    If you haven't done already in the past you have to create an ssh-key-pair and the public part of it has to be uploaded to your account
    
    `ssh-keygen -t rsa -C "jsonderorhoweveryouwanttocallme" -b 2048`

    the "-b 2048" is very important because in my ubuntu system the standard key length is 3072 (standard should be 2048) which is not accepted by IBM Cloud (IBM Cloud accepts the key-lengths 2048 and 4096)
 
    You can upload the public key using the CLI:

    `ibmcloud is key-create jsonder @js_ssh_key2048.pub --resource-group-name default`

    ... or via the IBM Cloud Portal
    
    The [variables.tf](./variables.tf) file in this set contains the name of the default ssh-key. So if you dont want to name your public ssh-key `jsonder` in your Cloud setup you have to edit this file accordingly and replace by your key-name.

#### Why do you actually need the Command line Interface?

Actually I don't remember that I really needed the CLI to just execute an existing Terraform template with Schematics and Github.

Only if you want to change something in the terraform-files you might need to pull some information out of the IBM Cloud. For example I changed the terraform-file which is actually based on the sample code of the [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started) guide to provision on a Gen2 based VPC instead of Gen1. 
In Gen2 the image names are slightly different (I wouldn't mind but Terraform does :wink:) so I had to change the image name (or image ID) in [vpc.tf](./vpc.tf). ~~You can not~~ I could not find this image name in the GUI but the CLI command `ibmcloud is images` lists all images in the region so I was able to pick the one I needed.

```terraform
data ibm_is_image "os" {
  name = "ibm-ubuntu-18-04-1-minimal-amd64-1"
}

resource ibm_is_instance "vsi1" {
  name    = "${local.BASENAME}-vsi1"
  resource_group = "${data.ibm_resource_group.group.id}"
  vpc     = "${ibm_is_vpc.vpc.id}"
  zone    = "${local.ZONE}"
  keys    = ["${data.ibm_is_ssh_key.ssh_key_id.id}"]
  image   = "${data.ibm_is_image.os.id}"
  profile = "bx2-2x8"
 ... 
```
You might have spotted in the above code snippet that you actually need the _image-id_ (which can be found in the first column in the image below) but using this in a terraform file doesn't make it easier for human beeings to read it.

result of `ibmcloud is images`

![ibmcloud is images](./ibmcloud_is_images.gif)

tbc.

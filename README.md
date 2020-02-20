# Terraform on VPC Gen2 Test
Just a very simple test to provision to do some vpc tests in IBM Cloud
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

   **Don't try to create resources with Terraform/Schematics on a Lite Account!** 

   Even if the `terraform plan` command completes successfully the `terraform apply` command will only make it half way through. 
       It successfuly creates a VPC and security groups and maybe other "free" resources but it gets stuck when creating the actual VSI. It leaves you with a Zombie-VSI that remains in starting state and you will not be able to delete this VM (not with ´terraform destroy' nor manually through the GUI). The only way to get rid of this undead is to open a ticket and let the 3rd Level support kill him for you.

3. [Install the stand-alone IBM Cloud CLI on your local system](https://cloud.ibm.com/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

   You usually install the CLI directly on your Laptop but with Windows it comes with a Container environment that conflicts with other Hypervisors you may have installed so I finally decided to install everything on an Ubuntu-based VM (not all Unix derivates are supported by the CLI installer! - Ubuntu works fine , MINT doesn't)  
      
   For the first time use you IBM Cloud ID to log in with specifying the region (you don't need the `-r` option)
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
   All CLI commands to deal with VPCs in IBM Cloud start with `ibmcloud is` (infrastructure service)
   ... so the most important command for you in the beginning will be `ibmcloud is --help` :grinning:



3. Create your ssh-key pair

    If you haven't done already in the past you have to create an ssh-key-pair and the public part of it has to be uploaded to your account
    
    `ssh-keygen -t rsa -C "jsonderorhoweveryouwanttocallme" -b 2048`

    the "-b 2048" is very important because in my ubuntu system the standard key length is 3072 (standard should be 2048) which is not accepted by IBM Cloud (IBM Cloud accepts the key-lengths 2048 and 4096)
 
    You can upload the public key using the CLI:

    `ibmcloud is key-create jsonder @js_ssh_key2048.pub --resource-group-name default`

    ... or via the IBM Cloud Portal
    
    The [variables.tf](./variables.tf) file in this set contains the name of the default ssh-key. So if you dont want to name your public ssh-key `jsonder` in your Cloud setup you have to edit this file accordingly and replace by your key-name.

**When do you actually need the Command line Interface?**

Actually I don't remember that I really needed the CLI to just execute an existing Terraform templates with Schematics and Github.

Only if you want to change something in the terraform-files you might need to pull some information out of the IBM Cloud. For example I changed the terraform-file which is actually based on the sample code of the [Getting started with Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-getting-started) guide to provision on a Gen2 based VPC instead of Gen1. 
In Gen2 the image names are slightly different (I wouldn't mind but Terraform does :wink:) so I had to change the image name (or image ID) in [vpc.tf](./vpc.tf). ~~You can not~~ I could not find this imgage name in the GUI but the CLI command `ibmcloud is images` lists all images in the region so that I was able to pick the one I needed   

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
```
(as you see ... actually you need the image-id but adding this in the terraform file doesn't make it easier for human beeing to read)

`ibmcloud is images`
```shell
Listing images for generation 2 compute in all resource groups and region us-south under account Jörg Sonder's Account as user Joerg.Sonder@de.ibm.com...
ID                                          Name                                               Status       Arch      OS                                                              File size(GB)   Visibility   Resource group   
r006-e0039ab2-fcc8-11e9-8a36-6ffb6501dd33   ibm-centos-7-6-minimal-amd64-1                     available    amd64     centos-7-amd64(7.x - Minimal Install)                           2               public       -   
r006-d4aec81e-fcc6-11e9-9149-870ebf69fd8d   ibm-debian-9-9-minimal-amd64-1                     available    amd64     debian-9-amd64(9.x Stretch/Stable - Minimal Install)            4               public       -   
r006-931515d2-fcc3-11e9-896d-3baa2797200f   ibm-redhat-7-6-minimal-amd64-1                     available    amd64     red-7-amd64(7.x - Minimal Install)                              6               public       -   
r006-34ceeafe-fcc6-11e9-893a-57dde2f48a21   ibm-ubuntu-16-04-5-minimal-amd64-1                 available    amd64     ubuntu-16-04-amd64(16.04 LTS Xenial Xerus Minimal Install)      2               public       -   
r006-14140f94-fcc4-11e9-96e7-a72723715315   ibm-ubuntu-18-04-1-minimal-amd64-1                 available    amd64     ubuntu-18-04-amd64(18.04 LTS Bionic Beaver Minimal Install)     2               public       -   
r006-5f9568ae-792e-47e1-a710-5538b2bdfca7   ibm-windows-server-2012-full-standard-amd64-3      available    amd64     windows-2012-amd64(2012 Standard Edition)                       22              public       -   
r006-8bb3e8aa-b789-4292-8679-3564b3a9366a   ibm-windows-server-2012-r2-full-standard-amd64-3   available    amd64     windows-2012-r2-amd64(2012 R2 Standard Edition)                 27              public       -   
r006-54e9238a-ffdd-4f90-9742-7424eb2b9ff1   ibm-windows-server-2016-full-standard-amd64-3      available    amd64     windows-2016-amd64(2016 Standard Edition)                       45              public       -   
r006-a5636224-fcce-11e9-8542-cf9657fdcaa3   ibm-centos-7-7-minimal-ppc64le-1                   available    ppc64le   centos-7-ppc64le(7.x - Minimal Install)                         2               public       -   
r006-67ca3f5a-fcce-11e9-a809-73839369f0fc   ibm-debian-9-11-minimal-ppc64le-1                  available    ppc64le   debian-9-ppc64le(9.x Stretch/Stable - Minimal Install)          2               public       -   
r006-d2f5be47-f7fb-4e6e-b4ab-87734fd8d12b   ibm-ubuntu-18-04-3-minimal-ppc64le-2               available    ppc64le   ubuntu-18-04-ppc64le(18.04 LTS Bionic Beaver Minimal Install)   4               public       -   
99edcc54-c513-4d46-9f5b-36243a1e50e2        ibm-centos-7-0-64                                  deprecated   amd64     centos-7-amd64(7.x - Minimal Install)                           2               public       -   
f8ae058e-4273-42e9-b549-68d09219e9e5        ibm-centos-7-0-minimal-amd64-1                     deprecated   amd64     centos-7-amd64(7.x - Minimal Install)                           2               public       -   
82c23b8c-6039-461e-a2b0-e909ce286f8d        ibm-debian-9-0-64-minimal-amd64-1                  deprecated   amd64     debian-9-amd64(9.x Stretch/Stable - Minimal Install)            4               public       -   
3f323501-667e-4583-b081-19cca9ac55fd        ibm-debian-9-0-64-minimal-for-vsi                  deprecated   amd64     debian-9-amd64(9.x Stretch/Stable - Minimal Install)            4               public       -   
a573bfb9-4e55-481f-9060-31d757155ded        ibm-redhat-7-0-64-minimal-for-vsi                  deprecated   amd64     red-7-amd64(7.x - Minimal Install)                              6               public       -   
04f4c424-a90d-4c2b-a77f-db67ff9b1629        ibm-ubuntu-16-04-05-64-minimal-for-vsi             deprecated   amd64     ubuntu-16-04-amd64(16.04 LTS Xenial Xerus Minimal Install)      2               public       -   
bf962ae4-4140-462b-8fa3-56fa1b49b06a        ibm-ubuntu-18-04-64                                deprecated   amd64     ubuntu-18-04-amd64(18.04 LTS Bionic Beaver Minimal Install)     2               public       -   
a63864aa-c766-42d3-8bd0-999fdb87fb42        ibm-windows-2012-full-std-64                       deprecated   amd64     windows-2012-amd64(2012 Standard Edition)                       22              public       -   
7bc64e71-c287-4a2c-85b1-e7822b4bd8f6        ibm-windows-2012-full-std-amd64-1                  deprecated   amd64     windows-2012-amd64(2012 Standard Edition)                       22              public       -   
987cb0ec-4f2b-4733-a232-f1a3efc727d8        ibm-windows-2012-r2-full-std-64                    deprecated   amd64     windows-2012-r2-amd64(2012 R2 Standard Edition)                 27              public       -   
b3442350-a35d-4b33-b995-05c5f1155668        ibm-windows-2012-r2-full-std-amd64-1               deprecated   amd64     windows-2012-r2-amd64(2012 R2 Standard Edition)                 27              public       -   
6e9208e4-b757-4530-a22e-66722464c334        ibm-windows-2016-full-std-64                       deprecated   amd64     windows-2016-amd64(2016 Standard Edition)                       32              public       -   
9de244af-e231-4aae-a958-aa60d735c826        ibm-windows-2016-full-std-amd64-1                  deprecated   amd64     windows-2016-amd64(2016 Standard Edition)                       45              public       -   
86120242-0148-4c41-8882-c6eb9d0c396e        ibm-windows-2016-full-std-amd64-2                  deprecated   amd64     windows-2016-amd64(2016 Standard Edition)                       45              public       -   
r006-eb1c47cc-fcc9-11e9-8de1-33db5afbbd14   ibm-windows-server-2012-full-standard-amd64-1      deprecated   amd64     windows-2012-amd64(2012 Standard Edition)                       22              public       -   
r006-a219157a-9862-4360-be63-b912320a399a   ibm-windows-server-2012-full-standard-amd64-2      deprecated   amd64     windows-2012-amd64(2012 Standard Edition)                       22              public       -   
r006-158e7a2c-fccd-11e9-80fb-4f991aaa30f9   ibm-windows-server-2012-r2-full-standard-amd64-1   deprecated   amd64     windows-2012-r2-amd64(2012 R2 Standard Edition)                 27              public       -   
r006-9d8b336f-c203-4860-8234-38fcf8334793   ibm-windows-server-2012-r2-full-standard-amd64-2   deprecated   amd64     windows-2012-r2-amd64(2012 R2 Standard Edition)                 27              public       -   
r006-6b74bf1e-fccd-11e9-80ba-1b59914ac95b   ibm-windows-server-2016-full-standard-amd64-1      deprecated   amd64     windows-2016-amd64(2016 Standard Edition)                       45              public       -   
r006-253c540d-1b96-4e55-a1c3-7341616c7b19   ibm-windows-server-2016-full-standard-amd64-2      deprecated   amd64     windows-2016-amd64(2016 Standard Edition)                       45              public       -   
fb9706f2-bedb-4ce8-bfae-036c5f917d9f        ibm-centos-7-0-64-minimal-ppc64le-1                deprecated   ppc64le   centos-7-ppc64le(7.x - Minimal Install)                         2               public       -   
63efa246-2113-48f4-a7eb-844d865021c7        ibm-centos-7-0-minimal-ppc64le-2                   deprecated   ppc64le   centos-7-ppc64le(7.x - Minimal Install)                         2               public       -   
0dbe1dce-217a-44d2-b56e-68c4dc298e22        ibm-debian-9-0-64-minimal-ppc64le-1                deprecated   ppc64le   debian-9-ppc64le(9.x Stretch/Stable - Minimal Install)          2               public       -   
c1ca9c90-33a4-4c47-816b-0bca1011b5e6        ibm-debian-9-0-minimal-ppc64le-2                   deprecated   ppc64le   debian-9-ppc64le(9.x Stretch/Stable - Minimal Install)          2               public       -   
r006-22414c6c-fcce-11e9-8729-a354f4054d61   ibm-ubuntu-18-04-3-minimal-ppc64le-1               deprecated   ppc64le   ubuntu-18-04-ppc64le(18.04 LTS Bionic Beaver Minimal Install)   4               public       -   
6e1a3e7b-69b5-41fe-839b-9f4a84b0faae        ibm-ubuntu-18-04-64-minimal-ppc64le-1              deprecated   ppc64le   ubuntu-18-04-ppc64le(18.04 LTS Bionic Beaver Minimal Install)   4               public       -   
a49598fa-47f4-47f1-b404-9d69a70ea7f6        ibm-ubuntu-18-04-64-ppc64le-minimal-for-vsi        deprecated   ppc64le   ubuntu-18-04-ppc64le(18.04 LTS Bionic Beaver Minimal Install)   6               public       -   
```

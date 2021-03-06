# oci-mesosphere

This is a Terraform module that deploys [Mesosphere](https://www.mesosphere.com/) on [Oracle Cloud Infrastructure (OCI)](https://cloud.oracle.com/en_US/cloud-infrastructure). It is developed jointly by Oracle and Mesosphere.

This repo is under active development. Building open source software is a community effort. We're excited to engage with the community building this.

## Deployment Architecture
![mesos architecture](https://i.imgur.com/rhui8LG.png)

## Prerequisites
First off you'll need to do some pre deploy setup. That's all detailed [here](https://github.com/oracle/oci-quickstart-prerequisites).
1.	Install the latest version of terraform and packer
2.	Sign up for OCI, retrieve the account data, store the necessary credentials in the x.tfvars files as well as the vars.json files and create .aws and/or .oci directory for cli and s3 access
3.	Create a bucket in your tenant to store the state-file using the scripts in /global/statebucket

## Clone the Module
Now, you'll want a local copy of this repo. You can make that with the commands:

    git clone https://github.com/oracle-quickstart/oci-mesosphere.git
    cd oci-mesosphere/terraform
    ls

That should give you this:

    .
    ├── LICENSE
    ├── README.md
    ├── global
    │   ├── bastion
    │   │   ├── backend.tf
    │   │   ├── bootscript.tpl
    │   │   ├── datasources.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   ├── provider.tf
    │   │   └── vars.tf
    │   ├── bootnode
    │   │   ├── backend.tf
    │   │   ├── bootscript.tpl
    │   │   ├── datasources.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   ├── provider.tf
    │   │   └── vars.tf
    │   ├── proxy
    │   │   ├── backend.tf
    │   │   ├── bootscript.tpl
    │   │   ├── datasources.tf
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   ├── provider.tf
    │   │   └── vars.tf
    │   └── statebucket
    │       ├── bucket.tf
    │       ├── datasources.tf
    │       ├── outputs.tf
    │       ├── provider.tf
    │       └── vars.tf
    ├── modules
    │   └── packer
    │       ├── WebServerImage.json
    │       ├── bastion.json
    │       ├── bootnode.json
    │       ├── genconf
    │       │   ├── config.yaml
    │       │   └── ip-detect
    │       ├── haproxy.json
    │       ├── master.json
    │       ├── scripts
    │       │   ├── diskmount.sh
    │       │   └── nvmemount.sh
    │       ├── slave.json
    │       └── webserver.json
    ├── network
    │   ├── backend.tf
    │   ├── network.tf
    │   ├── outputs.tf
    │   ├── provider.tf
    │   └── vars.tf
    └── services
        └── prod
            ├── master
            │   ├── backend.tf
            │   ├── bootscript.tpl
            │   ├── datasources.tf
            │   ├── main.tf
            │   ├── outputs.tf
            │   ├── provider.tf
            │   └── vars.tf
            ├── public
            │   ├── backend.tf
            │   ├── bootscript.tpl
            │   ├── datasources.tf
            │   ├── main.tf
            │   ├── outputs.tf
            │   ├── provider.tf
            │   └── vars.tf
            └── slave
                ├── backend.tf
                ├── bootscript.tpl
                ├── datasources.tf
                ├── main.tf
                ├── outputs.tf
                ├── provider.tf
                └── vars.tf


![](./images/git-clone.png)

## Installation sequence

1.	Deploy the network topology using the scripts in /network
2.	Build the custom images using packer and the scripts in /module/packer
3.	Provision the boot node, for a commercial DC/OS complete the installation steps according to </link>
4.	Released the bastion and Proxy for access protection
5.	Provision the master nodes, slaves and public nodes using the scripts at /services/prod/… . Wait for the masters to complete before provisioning the slaves and the public nodes. This will take a while, because the HDD needs to be formatted as part of the installation process.
6.	Redirect the HAProxy, the existing config file is using Round Robin between three public nodes

## Initialize the deployment
We now need to initialize the directory with the module in it.  This makes the module aware of the OCI provider.  You can do this by running:

    terraform init

This gives the following output:

![](./images/terraform-init.png)

## Deploy the module
Now for the main attraction.  Let's make sure the plan looks good:

    terraform plan

That gives:

![](./images/terraform-plan.png)

If that's good, we can go ahead and apply the deploy:

    terraform apply

You'll need to enter `yes` when prompted.  Once complete, you'll see something like this:

![](./images/terraform-apply.png)

When the apply is complete, the infrastructure will be deployed, but cloud-init scripts will still be running.  Those will wrap up asynchronously.  So, it'll be a few more minutes before your cluster is accessible.  Now is a good time to get a coffee.

When the deployment is completed, it will show you the public IP of one of the instances created on Oracle Cloud Infrastructure (OCI).

`http://<public IP of the instance>:8080`

![](./images/app.png)

## View the instance in the Console
You can also login to the web console [here](https://console.us-phoenix-1.oraclecloud.com/a/compute/instances) to view the IaaS that is running the cluster.

![](./images/console.png)

## Post processing
1.	In order to support HTTPS, the cluster requires a domain and a certificate. The standard settings rely on one domain, services are separated using directories. Changing this structure will need wildcard certificates
2.	Services on Mesos communicate via ports. Building new services, usually the port range for the public needs to be adjusted accordingly. The initial port range is very narrow to keep exposure limited, when exposing services to the public internet

## Destroy the Deployment
When you no longer need the deployment, you can run this command to destroy it:

    terraform destroy

You'll need to enter `yes` when prompted.

![](./images/terraform-destroy.png)

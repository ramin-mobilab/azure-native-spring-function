# Running Spring Boot + GraalVM native images on Azure Functions

This sample application shows how to:

- Compile a Spring Boot application using GraalVM
- Deploy and run that application on [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/?WT.mc_id=github-social-judubois)

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/en-us/free/?WT.mc_id=github-social-judubois).
- The Azure CLI must be installed. [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli/?WT.mc_id=github-social-judubois).
- Terraform must be installed. [Install Terraform](https://www.terraform.io/).

To check if Azure is correctly set up, login using the CLI, by running `az login`.

## Fork this repository

All compilation and deployment will be done using GitHub Actions, so you need your own repository for this.
The easiest way is to fork this repository to your own GitHub account, using the `fork` button on the top right-hand corner of this page.

## Configure environment variables

You need to configure the following environment variables:

```bash
AZ_LOCATION=westeurope
AZ_RESOURCE_GROUP=azure-native-spring-function
AZ_FUNCTION_NAME_APP=<your-unique-name>
AZ_STORAGE_NAME=<your-unique-name>
```

- `AZ_LOCATION` : The name of the Azure location to use. Use `az account list-locations` to list available locations. Common values are `eastus`, `westus`, `westeurope`.
- `AZ_RESOURCE_GROUP` : The resource group in which you will work. It should be unique in your subscription, so you can probably keep the default `azure-native-spring-function`.
- `AZ_FUNCTION_NAME_APP` : Functions will run into an application, and its name should be unique across Azure.
- `AZ_STORAGE_NAME` : Functions binaries and configuration will be stored inside a storage account. Its name should also be unique across Azure. It must be between 3 and 24 characters in length and may contain numbers and lowercase letters only.

In order not to type those values again, you can store them in a `.env` file at the root of this project:

- This `.env` file will be ignored by Git (so it will remain on your local machine and won't be shared).
- You will be able to configure those environment variables by running `source .env`.

## Create the cloud infrastructure using Terraform

Pass the environment variables we created earlier to Terraform:

```bash
export TF_VAR_AZ_LOCATION=${AZ_LOCATION}
export TF_VAR_AZ_RESOURCE_GROUP=${AZ_RESOURCE_GROUP}
export TF_VAR_AZ_FUNCTION_NAME_APP=${AZ_FUNCTION_NAME_APP}
export TF_VAR_AZ_STORAGE_NAME=${AZ_STORAGE_NAME}
```

Go to the `src/main/terraform` directory and run:

- `terraform init` to initialize your Terraform environment
- `terraform apply --auto-approve` to create all the necessary Azure resources

This will create the following Azure resources:

- A resource group that will store all resources (just delete this resource group to remove everything)
- An Azure Functions plan. This is a consumption plan, running on Linux: you will only be paid for your usage, with a generous free tier.
[Here is the full pricing documentation](https://azure.microsoft.com/en-us/pricing/details/functions/?WT.mc_id=github-social-judubois).
- An Azure Functions application, that will use the plan described in the point above.
- An Azure Storage account, which will be used to store your function's data (the binary and the configuration files).

## Configure GitHub Actions to build and deploy the application

The GitHub Actions workflow that we will use is available in [.github/workflows/build-and-deploy.yml](.github/workflows/build-and-deploy.yml).

```bash
az ad sp create-for-rbac --name http://azure-native-spring-function --role contributor --scopes /subscriptions/10494bac-4dc9-4f66-9563-996f688d9c6c/resourceGroups/azure-native-spring-function --sdk-auth
```

# How to create Azure ML resources using different methods

This project shows how to train a Fashion MNIST model using an Azure ML pipeline, and how to deploy it using an online managed endpoint. It demonstrates how to create Azure ML resources using the following three methods: the Azure ML Studio UI, the Azure ML Python SDK, and the Azure ML CLI. It uses MLflow for tracking and model representation.

## Blog post

To learn more about the code in this repo, check out the accompanying blog post: https://bea.stollnitz.com/blog/aml-pipeline-mixed/

## Setup

- You need to have an Azure subscription. You can get a [free subscription](https://azure.microsoft.com/en-us/free) to try it out.
- Create a [resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).
- Create a new machine learning workspace by following the "Create the workspace" section of the [documentation](https://docs.microsoft.com/en-us/azure/machine-learning/quickstart-create-resources). Keep in mind that you'll be creating a "machine learning workspace" Azure resource, not a "workspace" Azure resource, which is entirely different!
- Install the Azure CLI by following the instructions in the [documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
- Install the ML extension to the Azure CLI by following the "Installation" section of the [documentation](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-configure-cli).
- Install and activate the conda environment by executing the following commands:

```
conda env create -f environment.yml
conda activate aml_pipeline_mixed
```

- Within VS Code, go to the Command Palette clicking "Ctrl + Shift + P," type "Python: Select Interpreter," and select the environment that matches the name of this project.
- In a terminal window, log in to Azure by executing `az login --use-device-code`.
- Set your default subscription by executing `az account set -s "<YOUR_SUBSCRIPTION_NAME_OR_ID>"`. You can verify your default subscription by executing `az account show`, or by looking at `~/.azure/azureProfile.json`.
- Set your default resource group and workspace by executing `az configure --defaults group="<YOUR_RESOURCE_GROUP>" workspace="<YOUR_WORKSPACE>"`. You can verify your defaults by executing `az configure --list-defaults` or by looking at `~/.azure/config`.
- Add a `config.json` file to the root of your project (or somewhere in the parent folder hierarchy) containing your Azure subscription ID, resource group, and workspace:
  {
  "subscription_id": "<YOUR_SUBSCRIPTION_ID>",
  "resource_group": "<YOUR_RESOURCE_GROUP>",
  "workspace_name": "<YOUR_WORKSPACE>"
  }
- You can now open the [Azure Machine Learning studio](https://ml.azure.com/), where you'll be able to see and manage all the machine learning resources we'll be creating.
- Install the [Azure Machine Learning extension for VS Code](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.vscode-ai), and log in to it by clicking on "Azure" in the left-hand menu, and then clicking on "Sign in to Azure."

## Training and inference on your development machine

- Under "Run and Debug" on VS Code's left navigation, choose the "Train locally" run configuration and press F5.
- Do the same for the "Test locally" run configuration.
- You can analyze the metrics logged in the "mlruns" directory with the following command:

```
mlflow ui
```

## Training and deployment in the cloud

### Create the compute cluster

Create the compute cluster using the [Azure ML Studio UI](https://ml.azure.com/). Go to "Compute", "Compute clusters", "New."

- Select a Location, such as "West US 2."
- Select a Virtual machine tier, such as "Dedicated."
- Select a Virtual machine type, such as "CPU."
- Select a Virtual machine size, such as "Standard_DS4_v2."
- Click Next.
- Enter a Compute name, such as "cluster-cpu."
- Select a Minimum number of nodes, such as 0.
- Select a Maximum number of nodes, such as 4.
- Click Create.

Look at the "Compute clusters" page. Your newly created cluster should be listed there.

### Create the dataset

Create the dataset using the [Azure ML Studio UI](https://ml.azure.com/). Go to "Data", "Create", "From local files."

- Enter a Name, such as "data-fashion-mnist."
- Change the "Dataset type" to "File."
- Click Next.
- Click "Browse," "Browse folder," and select the "data" folder generated when you ran the "train.py" file locally.
- Click Upload.
- Click Next.
- Click Create.

### Create and run the pipeline

Select the "Train in the cloud" run configuration and press F5.
Go to the Azure ML Studio and wait until the Job completes.

### Register the model

When the pipeline is done running, it prints the run id at the end. For example:

```
Execution Summary
=================
RunId: blue_soca_jkf490bx8y
```

Set the shell variable run_id to that run id. For example:

```
run_id=blue_soca_jkf490bx8y
```

Create the Azure ML model using the CLI.

```
cd aml_pipeline_mixed
az ml model create --name model-pipeline-mixed --version 1 --path "azureml://jobs/$run_id/outputs/model_dir" --type mlflow_model
```

You don't need to download the trained model, but here's how you would do it if you wanted to:

```
az ml job download --name $run_id --output-name "model_dir"
```

### Create and invoke the endpoint

Create the deployment and endpoint using the CLI:

```
az ml online-endpoint create -f cloud/endpoint.yml
az ml online-deployment create -f cloud/deployment.yml --all-traffic
```

Invoke the endpoint.

```
az ml online-endpoint invoke --name endpoint-pipeline-mixed --request-file test_data/images_azureml.json
```

Clean up the endpoint to avoid getting charged.

```
az ml online-endpoint delete --name endpoint-pipeline-mixed -y
```

## Related resources

- [Train with the SDK](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-train-sdk?WT.mc_id=aiml-44165-bstollnitz)
- [Train with the CLI](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-train-cli?WT.mc_id=aiml-44165-bstollnitz)

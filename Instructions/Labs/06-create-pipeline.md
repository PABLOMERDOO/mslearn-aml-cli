---
lab:
    title: 'Lab: Run a pipeline with components'
    module: 'Module: Run pipelines in Azure Machine Learning with CLI (v2)'
---

# Run a pipeline with components

In this exercise, you will build a pipeline with components. The pipeline will be submitted with the CLI (v2). First, you'll run a pipeline. Next, you'll create components in the Azure Machine Learning workspace so that they can be reused. Finally, you'll create a pipeline with the Designer in the Azure Machine Learning Studio to experience how you can reuse components to create new pipelines.

## Prerequisites

Before you continue, complete the [Create an Azure Machine Learning Workspace and assets with the CLI (v2)](01-create-workspace.md) lab to set up your Azure Machine Learning environment.

You'll run all commands in this lab from the Azure Cloud Shell. If this is your first time using the cloud shell, complete the [Create an Azure Machine Learning Workspace and assets with the CLI (v2)](Instructions/Labs/01-create-workspace.md) lab to set up the cloud shell environment.

1. Open the Cloud Shell by navigating to [http://shell.azure.com](https://shell.azure.com/?azure-portal=true) and signing in with your Microsoft account.
1. The repo [https://github.com/MicrosoftLearning/mslearn-aml-cli](https://github.com/MicrosoftLearning/mslearn-aml-cli) should be cloned. You can explore the repo and its contents by using the `code .` command in the Cloud Shell.
1. If your compute instance is stopped. Start the instance again by using the following command. Change <your-compute-instance-name> to your compute instance name before running the code:
    ```azurecli
    az ml compute start --name "<your-compute-instance-name>"
    ```

## Run a pipeline

You can train a model by running a job that refers to one training script. To train a model as part of a pipeline, you can use Azure Machine Learning to run multiple scripts. The configuration of the pipeline is defined in a YAML file, similar to a command job.

In this exercise, you'll start by preprocessing the data and training a Decision Tree model. You can explore the pipeline job definition **job.yml** by navigating to **mslearn-aml-cli/Allfiles\Labs\05\job.yml**. The dataset used is the **diabetes-data** dataset registered to the Azure Machine Learning workspace in the set-up. 

1. Run the following command in the Cloud Shell to open the files of the cloned repo.
    ```azurecli
    code .
    ```
1. Navigate to **mslearn-aml-cli/Allfiles/Labs/05/** and open **job.yml** by selecting the file.
1. The **compute** value is **aml-cluster** to specify that the compute cluster you created in the set-up should be used. If you named your compute cluster differently, update the value.
1. Run the job by using the following command:
    ```azurecli
    az ml job create --file ./mslearn-aml-cli/Allfiles/Labs/05/job.yml
    ```
1. Open another tab in your browser and open the Azure Machine Learning Studio. Go to the **Experiments** page and locate the **diabetes-pipeline-example** experiment. Open the run to monitor the job. Refresh the view if necessary. Once completed, you can explore the details of the job and of each component by expanding the **Child runs**.

## Create components

To reuse the pipeline's components, you can create the component in the Azure Machine Learning workspace. Next to the components that were part of the pipeline you just ran, you'll create another new component you haven't used before. You'll use the new component in the next part.

1. Each component is created separately. Run the following code to create the components:
    ```azurecli
    az ml component create --file summary-stats.yml
    az ml component create --file fix-missing-data.yml
    az ml component create --file normalize-data.yml
    az ml component create --file train-decision-tree.yml
    az ml component create --file train-logistic-regression.yml
1. Navigate to the **Components** page in the Azure Machine Learning Studio. All created components should show in the list here. 

## Create a new pipeline with the Designer

You can reuse the components by creating a pipeline with the Designer. You can recreate the same pipeline, or change the algorithm you use to train a model by replacing the component used to train the model.

1. Navigate to the **Designer** page in the Azure Machine Learning Studio.
1. Create a new pipeline.
1. Rename the pipeline to *Train-Diabetes-Classifier*.
1. Change the default compute target to use the compute cluster (*aml-cluster*) you created.
1. In the left menu, expand the **Datasets** section.
1. Drag and drop the **diabetes-data** component to the canvas.
1. In the left menu, expand the **Custom Components** section.
1. Drag and drop the **Remove Empty Rows** component on to the canvas, below the **diabetes-data**. Connect the output of the data to the input of the new component.
1. Drag and drop the **Normalize numerical columns** component on to the canvas, below the **Remove empty rows**. Connect the output of the previous component to the input of the new component.
1. Drag and drop the **Train a Decision Tree Classifier Model** component on to the canvas, below the **Remove empty rows**. Connect the output of the previous component to the input of the new component.
1. Your pipeline should look like this:
![Decision Tree Pipeline in Designer](media/designer-pipeline-decision.png)
1. Submit your pipeline and wait until all components have successfully completed.

You have now trained the model with a similar pipeline as before (only omitting the calculation of the summary statistics). You can change the algorithm you use to train the model by replacing the last component:

1. Remove the **Train a Decision Tree Classifier Model** component from the pipeline. 
1. Drag and drop the **Train a Logistic Regression Model** component on to the canvas, below the **Remove empty rows**. Connect the output of the previous component to the input of the new component.

The new model training component expects a numeric input, namely the regularization rate. 

1. Select the **Train a Logistic Regression Model** component and enter **1** for the **regularization_rate**. 
1. Submit the pipeline. Once completed, you can review the metrics and compare it with the previous pipeline to see if the model's performance has improved.

## Clean up resources

When you're finished exploring Azure Machine Learning, the compute cluster will automatically scale down to 0 nodes, so there is no need to stop the cluster.

> **Note:** If you have finished exploring Azure Machine Learning, you can delete the Azure Machine Learning workspace and associated resources. However, if you plan to complete any other labs in this series, you will need to repeat the set-up to create the workspace and prepare the environment first.

To delete the Azure Machine Learning workspace, you can use the following command in the CLI:

```azurecli
az ml workspace delete
```
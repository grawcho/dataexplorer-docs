---
title: 'Azure DevOps task for Azure Data Explorer'
description: 'In this topic, you learn to create a release pipeline and deploy'
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: how-to
ms.date: 05/05/2019

#Customer intent: I want to use Azure DevOps to create a release pipeline and deploy
---

# Azure DevOps Task for Azure Data Explorer

[Azure DevOps Services](https://azure.microsoft.com/services/devops/) provides development collaboration tools such as high-performance pipelines, free private Git repositories, configurable Kanban boards, and extensive automated and continuous testing capabilities. [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) is an Azure DevOps capability that enables you to manage CI/CD to deploy your code with high-performance pipelines that work with any language, platform, and cloud.
[Azure Data Explorer - Pipeline Tools](https://marketplace.visualstudio.com/items?itemName=Azure-Kusto.PublishToADX) is the Azure Pipelines task that enables you to create release pipelines and deploy your database changes to your Azure Data Explorer databases. It's available for free in the [Visual Studio Marketplace](https://marketplace.visualstudio.com/).
this extension includes 3 basic tasks:
   * Azure Data Explorer Command - Run Admin Commands against ADX cluster
   * Azure Data Explorer Query - Run Queries against ADX cluster and parse the results
   * Azure Data Explorer Query Server Gate - Agentless task to Gate releases depending on the query outcome

   ![Task Types](media/devops/ExtensionTaskTypes.png)
* Internal Microsoft users or early adopters can submit a request to share the [internal version of the extension](https://marketplace.visualstudio.com/items?itemName=Azure-Kusto.ADXAdminCommands) 

This document describes a simple example on the use of the **Azure Data Explorer - Pipeline Tools** task to deploy your schema changes to your database. For complete CI/CD pipelines, refer to [Azure DevOps documentation](/azure/devops/user-guide/what-is-azure-devops#vsts).

## Prerequisites

* If you don't have an Azure subscription, create a [free Azure account](https://azure.microsoft.com/free/) before you begin.
* Azure Data Explorer Cluster setup:
    * An [Azure Data Explorer cluster and database](create-cluster-database-portal.md).
    * Create Azure Active Directory (Azure AD) app by [provisioning an Azure AD application](./provision-azure-ad-app.md).
    * Grant access to your Azure AD App on your Azure Data Explorer database by [managing Azure Data Explorer database permissions](manage-database-permissions.md).
* Azure DevOps setup:
    * [Sign up for a free organization](/azure/devops/user-guide/sign-up-invite-teammates)
    * [Create an organization](/azure/devops/organizations/accounts/create-organization)
    * [Create a project in Azure DevOps](/azure/devops/organizations/projects/create-project)
    * [Code with Git](/azure/devops/user-guide/code-with-git)
* Extension Installation:
    * if you are the Azure DevOps instance owner install the extension from the [extension page](https://marketplace.visualstudio.com/items?itemName=Azure-Kusto.PublishToADX)
    * ![Create folders](media/devops/GetExtention.PNG)   ![Create folders](media/devops/ExtensionInstall.PNG)
    * if you are NOT the Azure DevOps instance owner, Please refer / request your ADO instance admin to install it.

## Create folders

Create the following sample folders (*Functions*, *Policies*, *Tables*) in your Git repository. Copy the files from [here](https://github.com/Azure/azure-kusto-docs-samples/tree/master/DevOps_release_pipeline) into the respective folders as seen below and commit the changes. The sample files are provided to execute the following workflow.

![Create folders](media/devops/create-folders.png)

> [!TIP]
> When creating your own workflow, we recommend making your code idempotent. For example, use [`.create-merge table`](kusto/management/create-merge-table-command.md) instead of [`.create table`](kusto/management/create-table-command.md), and use [`.create-or-alter`](kusto/management/create-alter-function.md) function instead of [`.create`](kusto/management/create-function.md) function.

## Create a release pipeline

1. Sign in to your [Azure DevOps organization](https://dev.azure.com/).
1. Select **Pipelines** > **Releases** from left-hand menu and select **New pipeline**.

    ![New pipeline](media/devops/new-pipeline.png)

1. The **New release pipeline** window opens. In the **Pipelines** tab, in the **Select a template** pane, select **Empty job**.

     ![Select a template](media/devops/select-template.png)

1. Select **Stage** button. In **Stage** pane, add the **Stage name**. Select **Save** to save your pipeline.

    ![Name the stage](media/devops/stage-name.png)

1. Select **Add an artifact** button. In the **Add an artifact** pane, select the repository where your code exists, fill out relevant information, and click **Add**. Select **Save** to save your pipeline.

    ![Add an artifact](media/devops/add-artifact.png)

1. In the **Variables** tab, select **+ Add** to create a variable for **Endpoint URL** that will be used in the task. Write the **Name** and the **Value** of the endpoint. Select **Save** to save your pipeline. 

    ![Create variable](media/devops/create-variable.png)

    To find your Endpoint_URL, the overview page of your **Azure Data Explorer Cluster** in the Azure portal contains the Azure Data Explorer cluster URI. Construct the URI in the following format `https://<Azure Data Explorer cluster URI>?DatabaseName=<DBName>`.  For example, https:\//kustodocs.westus.kusto.windows.net?DatabaseName=SampleDB

    ![Azure Data Explorer cluster URI](media/devops/adx-cluster-uri.png)

## Create tasks to deploy

1. In the **Pipeline** tab, click on **1 job, 0 task** to add tasks. 

    ![Add tasks](media/devops/add-task.png)

1. Create three tasks to deploy **Tables**, **Functions**, and **Policies**, in this order. 

1. In the **Tasks** tab, select **+** by **Agent job**. Search for **Azure Data Explorer**. In **Marketplace**, install the **Azure Data Explorer – Pipeline Tools** extension. Then, select **Add** in **Run Azure Data Explorer Command**.

     ![Add admin commands](media/devops/add-admin-commands.png)

1. Click on **Kusto Command** on the left and update the task with the following information:
    * **Display name**: Name of the task
    * **File path**: In the **Tables** task, specify */Tables/*.csl since the table creation files are in the *Table* folder.
    * **Endpoint URL**: enter the `EndPoint URL`variable created in previous step.
    * Select **Use Service Endpoint** and pick the service endpoint in the selection box (if none exist create a new endpoint **+ New**.)

    ![Update Kusto command task](media/devops/kusto-command-task.png)

1. Complete the following information in the **Add Azure Data Explorer service connection** window:

    |Setting  |Suggested value  |
    |---------|---------|
    |**Connection name**     |    Enter a name to identify this service endpoint     |
    |**Cluster Url**    |    Value can be found in the overview section of your Azure Data Explorer Cluster in the Azure portal | 
    |**Service Principal Id**    |    Enter the AAD App ID (created as prerequisite)     |
    |**Service Principal App Key**     |    Enter the AAD App Key (created as prerequisite)    |
    |**AAD tenant Id**    |      Enter your AAD tenant (such as microsoft.com, contoso.com...)    |

    Select **Allow all pipelines to use this connection** checkbox. Select **OK**.

    ![Add service connection](media/devops/add-service-connection.png)

1. Repeat steps 1-5 another two times to deploy files from the *Functions* and *Policies* folders. Select **Save**. In the **Tasks** tab, see the three tasks created: **Deploy Tables**, **Deploy Functions**, and **Deploy Policies**.

    ![Deploy all folders](media/devops/deploy-all-folders.png)

1. Query Task - Run A query against a cluster

    ![Query Task](media/devops/QueryTask.png)

    Set Task Results based on the Query Row Count or single Value
    
    ![Query Task](media/devops/RowCount.png)  ![Query Task](media/devops/SingleValue.png) 

1. Query Gate Task - Run A query against a cluster and Gate Release Progress pending on Query Results Row Count

    How To Add the Gate:

    ![Query Gate Task](media/devops/QueryGateAdd.png)

    How To Configure the Task:
   
    ![Query Gate Task](media/devops/QueryGate.png)

    How it looks when running a release:

    ![Query Gate Task](media/devops/QueryGateLook.png)




1. Select **+ Release** > **Create release** to create a release.

    ![Create a release](media/devops/create-release.png)

1. In the **Logs** tab, check the deployment status is successful.

    ![Deployment is successful](media/devops/deployment-successful.png)

You have now completed creation of a release pipeline for deployment of three tasks to pre-production.

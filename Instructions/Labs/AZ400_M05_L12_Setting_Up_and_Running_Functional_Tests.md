---
lab:
    title: 'Setting Up and Running Functional Tests'
    module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# Setting Up and Running Functional Tests

# Student lab manual

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identify an existing Azure subscription 'CECH_SoIT_Bootcamp'assigned to you by university, you will need to use this for completing all your labs in the course.

- Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

## Lab overview

[Selenium](http://www.seleniumhq.org/) is a portable open source software-testing framework for web applications. It can operate on almost every operating system. It supports all modern browsers and multiple languages, including .NET (C#) and Java.

This lab will teach you how to execute Selenium test cases on a C# web application as part of the Azure DevOps Release pipeline.

## Objectives

After you complete this lab, you will be able to:

- Configure a self-hosted Azure DevOps agent.
- Configure the release pipeline.
- Trigger build and release.
- Run tests in Chrome and Firefox.

## Estimated timing: 60 minutes

## Instructions

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which include the preconfigured Parts Unlimited team project based on an Azure DevOps Demo Generator template and Azure resources.

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **Selenium** template.

1. On your lab computer, start a web browser and navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab.

    > **Note**: For more information on the site, see [What is the Azure DevOps Services Demo Generator?](https://docs.microsoft.com/en-us/azure/devops/demo-gen).

1. Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1. If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1. On the **Create New Project** page, in the **New Project Name** textbox, type **Setting Up and Running Functional Tests**, in the **Select organization** dropdown list, select your Azure DevOps organization, and then click **Choose template**.
1. In the list of templates, in the toolbar, click **DevOps Labs**, select the **Selenium** template and click **Select Template**.
1. Back on the **Create New Project** page, click **Create Project**

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1. On the **Create New Project** page, click **Navigate to project**.

#### Task 2: Create Azure resources

In this task, you will provision an Azure VM running Windows Server 2016 along with SQL Express 2017, Chrome, and Firefox.

1. Click here in **[Deploy to Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json)** link. This will automatically redirect you to the **Custom deployment** blade in the Azure portal.
1. If prompted, sign in with the UC user account.
1. On the **Custom deployment** blade, select **Edit template**.
1. On the **Edit template** blade, locate the line `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"`, replace it with `"https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1"`, and click **Save**.
1. Back on the **Custom deployment** blade, specify the following settings:

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | the name of a new resource group **az400m11l02-RG-<6+2>** |
    | Region | the name of the Azure region in which you want to deploy the Azure resources in this lab |
    | Virtual Machine Name | **azvm<6+2>** |

1. Click **Review + create** and then click **Create**.

    > **Note**: Wait for the process to complete. This should take about 15 minutes.

### Exercise 1: Implement Selenium tests by using a self-hosted Azure DevOps agent

In this exercise, you will implement Selenium tests by using a self-hosted Azure DevOps agent.

#### Task 1: Configure a self-hosted Azure DevOps agent

In this task, you will configure a self-hosted agent by using the VM you deployed in the previous exercise. Selenium requires the agent to be run in the interactive mode to execute the UI tests.

1. In the web browser window displaying the Azure portal, search for and select **Virtual machines** and, from the **Virtual machines** blade, select **azvm<6+2>**.
1. On the **azvm<6+2>** blade, select **Connect**, in the drop-down menu, select **RDP**, on the **RDP** tab of the **azvm<6+2> \| Connect** blade, select **Download RDP File** and open the downloaded file.
1. When prompted, sign in with the following credentials:

    | Setting | Value |
    | --- | --- |
    | User Name | **vmadmin** |
    | Password | **P2ssw0rd@123** |

1. Within the Remote Desktop session to **azvm<6+2>**, open a Chrome web browser window, navigate to **<https://dev.azure.com>** and sign in to your Azure DevOps organization.
1. In the lower left corner of the **Azure DevOps** portal, click **Organization settings**.
1. In the vertical menu on the left hand side of the page, in the **Pipelines** section, click **Agent pools**.
1. On the **Agent pools** pane, click **Default**.
1. On the **Default** pane, click **New agent**.
1. On  the **Get the agent** panel, ensure that the **Windows** tab and the **x64** section are selected and then click **Download**.
1. Start File Explorer, create a directory **C:\\AzAgent** and extract content of the downloaded agent zip file residing in the **Downloads** folder into this directory.
1. Within the Remote Desktop session to **azvm<6+2>**, right-click the **Start** menu and click **Command Prompt (Admin)**.
1. Within the **Administrator: Command Prompt** window, run the following to start the installation of the agent binaries:

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

1. In the **Administrator: Command Prompt** window, when prompted to **Enter server URL**, type **https://dev.azure.com/\<your-DevOps-organization-name\>**, where **\<your-DevOps-organization-name\>** represents the name of your Azure DevOps Organization, and press the **Enter** key.
1. In the **Administrator: Command Prompt** window, when prompted **Enter Authentication type (press enter for PAT)**, press the **Enter key**.
1. In the **Administrator: Command Prompt** window, when prompted **Enter personal access token**, switch to the Azure DevOps portal, close the **Get the agent** panel, in the upper right corner of the Azure DevOps page, click the **User settings** icon, in the dropdown menu, click **Personal access tokens**, on the **Personal Access Tokens** pane, and click **+ New Token**.
1. On the **Create a new personal access token** pane, specify the following settings and click **Create** (leave all others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **Setting Up and Running Functional Tests lab** |
    | Scopes | **Custom Defined** |
    | Scopes | Click **Show all scopes** (at the bottom of the window) |
    | Scopes | **Agent Pools** - **Read & Manage** |

1. On the **Success** pane, copy the value of the personal access token to Clipboard.

    > **Note**: Make sure you copy the token. You will not be able to retrieve it once you close this pane.

1. On the **Success** pane, click **Close**.
1. Switch back to the **Administrator: Command Prompt** window and paste the content of Clipboard and press the **Enter key**.
1. In the **Administrator: Command Prompt** window, when prompted **Enter agent pool (press enter for default)**, press the **Enter key**.
1. In the **Administrator: Command Prompt** window, when prompted **Enter agent name (press enter for azvm<6+2>)**, press the **Enter key**.
1. In the **Administrator: Command Prompt** window, when prompted **Enter work folder (press enter for _work)**, press the **Enter key**.
1. In the **Administrator: Command Prompt** window, when prompted **Enter run agent as service (Y/N) (press enter for N)**, press the **Enter key**.
1. In the **Administrator: Command Prompt** window, when prompted **Enter configure autologon and run agent on startup (Y/N) (press enter for N)**, press the **Enter key**.
1. Once the agent is registered, in the **Administrator: Command Prompt** window, type **run.cmd** and press the **Enter** to start the agent.

    > **Note**: You also need to install the Dac Framework which is used by the application you will be deploying later in the lab.

1. Within the Remote Desktop session to **azvm<6+2>**, start another instance of the web browser, navigate to the [Microsoft SQL Server Data-Tier Application Framework (18.2) download page](https://www.microsoft.com/en-us/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions) and click **Download**.
1. On the **Choose the download you want**, select the **EN\x64\DacFramework.msi** checkbox and click **Next**. This will trigger automatic download of the **DacFramework.msi** file.
1. Once the download of the **DacFramework.msi** file completes, use it to run the installation of the Microsoft SQL Server Data-Tier Application Framework with the default settings.

#### Task 2: Configure a release pipeline

In this task, you will configure the release pipeline.

> **Note**: The Azure VM has the agent configured to deploy the applications and run Selenium testcases. The release definition uses **[Phases](https://docs.microsoft.com/en-us/vsts/build-release/concepts/process/phases)** to deploy to target servers.

1. Within the Remote Desktop session to **azvm<6+2>**, in the browser window displaying the **Azure DevOps** portal, click the **Azure DevOps** symbol in the upper left corner.
1. On the pane displaying your organization projects, click the tile representing the **Setting Up and Running Functional Tests** project.
1. On the **Setting Up and Running Functional Tests** pane, in the vertical navigational pane, select **Pipelines**, within the **Pipelines** section, click **Releases** and then, on the **Selenium** pane, click **Edit**.
1. On the **All pipelines > Selenium** pane, click the **Tasks** tab header and, in the dropdown menu, click **Dev**.
1. Within the list of tasks of the **Dev** stage, review the **IIS Deployment**, **SQL Deployment**, and **Selenium test execution** deployment phases.

- **IIS Deployment phase**: In this phase, we deploy application to the VM using the following tasks:

  - **IIS Web App Manage**: This task runs on the target machine where we registered agent. It creates a *website* and an *Application Pool* locally with the name **PartsUnlimited** running under the port **82** , [**http://localhost:82**](http://localhost:82)
  - **IIS Web App Deploy**: This task deploys the application to the IIS server using **Web Deploy**.

- **Database deploy phase**: In this phase, we use [**SQL Server Database Deploy**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) task to deploy [**dacpac**](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications) file to the DB server.

- **Selenium tests execution**: Executing **UI testing** as part of the release process allows us to detect unexpected changes. Setting up automated browser based testing drives quality in your application, without having to do it manually. In this phase, we will execute Selenium tests on the deployed web application. The subsequent tasks describe using Selenium to test the websites in the release pipeline.

  - **Visual Studio Test Platform Installer**: The [Visual Studio Test Platform Installer](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts) task will acquire the Microsoft test platform from nuget.org or a specified feed, and add it to the tools cache. It satisfies the **vstest** requirements so any subsequent Visual Studio Test task in a build or release pipeline can run without needing a full Visual Studio install on the agent machine.
  - **Run Selenium UI tests**: This [task](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md) uses **vstest.console.exe** to execute the selenium testcases on the agent machines.

1. On the **All pipelines > Selenium** pane, click the **IIS Deployment** phase and, on the **Agent job** pane, verify that the **Default** Agent pool is selected.
1. Repeat the previous step for **SQL Deployment** and the **Selenium tests execution** phases. If needed, click **Save** to save the changes.

#### Task 3: Trigger Build and Release

In this task, we will trigger the **Build** to compile Selenium C# scripts along with the Web application. The resulting binaries are copied to self-hosted agent and the Selenium scripts are executed as part of the automated **Release**.

1. Within the Remote Desktop session to **azvm<6+2>**, in the browser window displaying the **Azure DevOps** portal, in the vertical navigational pane, in the **Pipelines** section, click **Pipelines** and then, on the **Pipelines** pane, click **Selenium**.
1. On the **Selenium** pane, click **Run pipeline** and, on the **Run pipeline**, click **Run**.

    > **Note**: This build will publish the test artifacts to Azure DevOps, which will be used in release.

    > **Note**: Once the build is successful, release will be triggered.

1. On the pipeline runs pane, in the **Jobs** section, click **Phase 1** and monitor the build progress until its completion.

     >**[Screenshot 1](https://github.com/mandalapu1994/AZ400-Azure_DevOps_Engineer_Expert/blob/main/Instructions/Labs/AZ400_M05_L12_Setting_Up_and_Running_Functional_Tests.md)**: Show all jobs were run sucessfully with your name or <6+2> visible in the top of the page.

1. In the browser window displaying the **Azure DevOps** portal, in the vertical navigational pane, in the **Pipelines** section, click **Releases**, click the entry representing the release, and, on the **Selenium > Release-1** pane, click **Dev**.
1. On the **Selenium > Release-1 > Dev** pane, monitor the corresponding deployment.
1. Once the **Selenium test execution** phase starts, monitor the web browser tests.
1. Once the release completes, on the **Selenium > Release-1 > Dev** pane, click on the **Tests** tab to analyze the test results. Select the required filters from the dropdown in **Outcome** section to view the tests and their status.

    >**[Screenshot 2](https://github.com/mandalapu1994/AZ400-Azure_DevOps_Engineer_Expert/blob/main/Instructions/Labs/AZ400_M05_L12_Setting_Up_and_Running_Functional_Tests.md)**: Showtests and their status with your name or <6+2> visible in the top of the page.

## Review

In this lab, you learned how to execute Selenium test cases on a C# web application, as part of the Azure DevOps Release pipeline.

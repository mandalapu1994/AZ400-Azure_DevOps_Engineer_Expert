---
lab:
    title: 'Setup lab environment'
    module: 'Module 0: Orientation and Welcome'
---

# Setup lab environment

# Student lab manual

## Instructions

1. Open a browser and navigate to [https://portal.azure.com](https://portal.azure.com), then then select Login, ther use your UC credentials to login.

1. Verify your organization it should be **University of Cincinnati** as shown in the below screenshot or select the **Switch Organization** and switch to **University of Cincinnati**

    ![Create Project](images/organization-verify.png)

1. Search for DevOps and select **Azure DevOps Organizations**.

1. Next, click on the link labelled **My Azure DevOps Organizations** or navigate directly to [https://aex.dev.azure.com](https://aex.dev.azure.com).

1. On the **We need a few more details** page, enter below details and select **Continue**.

    | Entry | Value |
    | -- | -- |
    | Your name | Give your name here |
    | We'll reach you at | Give your **UC email** |
    | From | **United States** |

1. In [https://aex.dev.azure.com](https://aex.dev.azure.com) click the blue button **Create new organization**.
1. Accept the *Terms of Service* by clicking **Continue**.
1. In prompt **"Almost done"**, use below details and select **Continue**.

    | Entry | Value |
    | -- | -- |
    | Name your Azure DevOps Organization | Give your **<6+2>**|
    | We'll host your projects in | **Central US** |
    | Captcha | Enter the characters you see |

1. Once the newly created organization opens in **Azure DevOps**, click **Organization settings** in the bottom left corner.

1. At the **Organization settings** screen click **Billing** (opening this screen takes a few seconds).

    > Note: For **AZ400 DevOps Engineer Expert** class we will be using a **free tire of Azure DevOps**, you will only have access to the limited minutes of CI/CD pipelines and since there only **1 Self-Hosted CI/CD** is available; you will be depended on the traffic or number of users using the same services, so I suggest you to work on the labs as early as possible.

1. In **Organization Settings**, go to section **Security** and click **Policies**.

1. Toggle the switch to **On** for **Third-party application access via OAuth**
    > Note: The OAuth setting helps enable tools such as the DemoDevOpsGenerator to register extensions. Without this, several labs may fail due to a lack of the required extensions.

1. Toggle the switch to **On** for **Allow public projects**
    > Note: Extensions used in some labs might require a public project to allow using the free version.

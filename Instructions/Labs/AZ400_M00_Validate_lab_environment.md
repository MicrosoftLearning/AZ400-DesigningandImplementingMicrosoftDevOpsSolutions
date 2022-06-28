---
lab:
    title: 'Lab 00: Validate lab environment'
    module: 'Module 0: Welcome'
---

# Lab 00: Validate lab environment

# Student lab manual

## Instructions

> **Note**: Start at step 4, if you do already have a **personal Microsoft Account** setup and an active Microsoft Azure Pass subscription linked to that account.

1. Get a new **Azure Pass promocode** from the instructor or other source.
2. Use a private browser session to get a new **personal Microsoft Account (MSA)** at [https://account.microsoft.com](https://account.microsoft.com).
3. Using the same browser session, go to [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com) to redeem your Azure Pass using your Microsoft Account (MSA). For details, see [Redeem a Microsoft Azure Pass](https://www.microsoftazurepass.com/Home/HowTo?Length=5). Follow the instructions for redemption. 

4. Open a browser and navigate to [https://portal.azure.com](https://portal.azure.com), then search at the top of the Azure portal screen for **Azure DevOps**. In the resulting page, click **Azure DevOps organizations**. 
5. Next, click on the link labelled **My Azure DevOps Organizations** or navigate directly to [https://aex.dev.azure.com](https://aex.dev.azure.com).
6. On the **We need a few more details** page, select **Continue**.
7. In the drop-down box on the left, choose **Default Directory**, instead of “Microsoft Account”.
8. If prompted (*"We need a few more details"*), provide your name, e-mail address, and location and click **Continue**.
9. Back at [https://aex.dev.azure.com](https://aex.dev.azure.com) with **Default Directory** selected click the blue button **Create new organization**.
10. Accept the *Terms of Service* by clicking **Continue**.
11. If prompted (*"Almost done"*), leave the name for the Azure DevOps organization at default (it needs to be a globally unique name) and pick a hosting location close to you from the list.
12. Once the newly created organization opens in **Azure DevOps**, click **Organization settings** in the bottom left corner.
13. At the **Organization settings** screen click **Billing** (opening this screen takes a few seconds).
14. Click **Setup billing** and on the right-hand side of the screen select the **Azure Pass - Sponsorship** subscription and click **Save** to link the subscription with the organization.
15. Once the screen shows the linked Azure Subscription ID at the top, change the number of **Paid parallel jobs** for **MS Hosted CI/CD** from 0 to **1**. Then click the **SAVE** button at the bottom. 
16. In **Organization Settings**, go to section **Security** and click **Policies**.
17. Toggle the switch to **On** for **Third-party application access via OAuth**
    > Note: The OAuth setting helps enable tools such as the DemoDevOpsGenerator to register extensions. Without this, several labs may fail due to a lack of the required extensions.
18. Toggle the switch to **On** for **Allow public projects**
    > Note: Extensions used in some labs might require a public project to allow using the free version.
19. **Wait at least 3 hours before using the CI/CD capabilities** so that the new settings are reflected in the backend. Otherwise, you will still see the message *"No hosted parallelism has been purchased or granted"*.
20. OPTIONAL: you can validate these new settings have been successfully applied by creating and triggering a build pipeline. To do this either talk to the instructor or create a demo project into the newly created organization with billing enabled, using [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net).

---
lab:
    title: 'Share team knowledge using Azure Project Wiki'
    module: 'Module 08: Implement continuous feedback'
---

# Share team knowledge using Azure Project Wiki

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Set up the sample eShopOnWeb Project:** If you don't already have the sample eShopOnWeb Project that you can use for this lab, create one by following the instructions available at [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

## Lab overview

In this lab, you'll create and configure a wiki in Azure DevOps, including managing markdown content and creating a Mermaid diagram.

## Objectives

After you complete this lab, you will be able to:

- Create a wiki in an Azure Project.
- Add and edit markdown.
- Create a Mermaid diagram.

## Estimated timing: 45 minutes

## Instructions

### Exercise 1: Publish code as a wiki

In this exercise, you will step through publishing an Azure DevOps repository as wiki and managing the published wiki.

> **Note**: Content that you maintain in a Git repository can be published to an Azure DevOps wiki. For example, content written to support a software development kit, product documentation, or README files can be published directly to a wiki. You have the option of publishing multiple wikis within the same Azure DevOps team project.

#### Task 1: Publish a branch of an Azure DevOps repo as wiki

In this task, you will publish a branch of an Azure DevOps repo as wiki.

> **Note**: If your published wiki corresponds to a product version, you can publish new branches as you release new versions of your product.

1. In the vertical menu on the left side, click **Repos**, in the upper section of the **Files** pane, make sure you have  the **eShopOnWeb** repo selected (choose it from the dropdown on the top with Git icon). In the branch dropdown list (on top of "Files" with the branch icon), select **main**, and review the content of the main branch.
1. To the left of the **Files** pane, in the listing of the repo folder and file hierarchy, expand the **src** folder and browse to **Web -> wwwroot -> images** subfolder. In the **Images** subfolder, locate the **brand.png** entry, hover with the mouse pointer over its right end to reveal the vertical ellipsis (three dots) symbol representing the **More** menu, click **Download** to download the **brand.png** file to the local **Downloads** folder on your lab computer.

    > **Note**: You will use this image in the next exercise of this lab.

1. We will store the Wiki source files in a separate folder within the Repos current folder structure. From within **Repos**, select **Files**. Notice the **eShopOnWeb** Repo title on top of the folder structure. **Select the ellipsis (3 dots)**, Choose **New / Folder**, and provide **`Documents`** as title for the New Folder name. As a repo doesn't allow you to create an empty folder, provide **`READ.ME`** as New File name.
1. Confirm the creation of the folder and the file by **pressing the Create button**.
1. The READ.ME file will open in the built-in view mode. Since this is stored 'as code', you need to **Commit** the changes by clicking the **Commit** button. In the Commit window, confirm once more by pressing **Commit**.
1. In the Azure DevOps vertical menu on the left side, click **Overview**, in the **Overview** section, select **Wiki**, select **Publish code as wiki*.
1. On the **Publish code as wiki** pane, specify the following settings and click **Publish**.

    | Setting | Value |
    | ------- | ----- |
    | Repository | **eShopOnWeb** |
    | Branch | **main** |
    | Folder | **/Documents** |
    | Wiki name | **`eShopOnWeb (Documents)`** |

    > **Note**: This will automatically open the Wiki section, and publish **the editor**, where you can provide a Wiki page title, as well as adding the actual content. Notice you are encouraged to use MarkDown format, but make use of the ribbon to help you with some of the MarkDown layout syntax.

1. In the Wiki Page **Title** field, enter "`Welcome to our Online Retail Store!`"

1. In the body of the Wiki Page, paste in the following text:

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. This sample text gives you an overview of several of the common MarkDown syntax features, ranging from Title and subtitles (##), bold (**), italic (*), how to create tables, and more.

1. Once finished, **press** the Save button in the upper right corner.

1. **Refresh** your browser, or select any other DevOps portal option and return to the Wiki section. Notice you are now presented with the **EshopOnWeb (Documents)** Wiki, as well as having the **Welcome to our Online Retail Store** as **HomePage** of the Wiki.

#### Task 2: Manage content of a published wiki

In this task, you will manage content of the wiki you published in the previous task.

1. In the vertical menu on the left side, click **Repos**, ensure that the dropdown menu in the upper section of the **Files** pane displays the **eShopOnWeb** repo and **main** branch, in the repo folder hierarchy, select the **Documents** folder, and select the **Welcome-to-our-Online-Retail-Store!.md** file.
1. Notice how the MarkDown format is visible here as raw text format, allowing you to continue editing the file content from here as well.

> **Note**: Since the Wiki source files are handled as source code, remember all practices from traditional source control (Clone, Pull Requests, Approvals and more), can now also be applied to Wiki pages.

### Exercise 2: Create and manage a project wiki

In this exercise, you will step through creating and managing a project wiki.

> **Note**: You can create and manage wiki independently of the existing repos.

#### Task 1: Create a project wiki including a Mermaid diagram and an image

In this task, you will create a project wiki and add to it a Mermaid diagram and an image.

1. On your lab computer, in the Azure DevOps portal displaying the **Wiki pane** of the **EShopOnweb** project,  with the content of the **eShopOnWeb (Documents)** wiki selected, at the top of the pane, click the **eShopOnWeb (Documents)** dropdown list header (the arrow down icon), and, in the drop down list, select **Create new project wiki**.
1. In the **Page title** text box, type **`Project Design`**.
1. Place the cursor in the body of the page, click the left-most icon in the toolbar representing the header setting and, in the dropdown list, click **Header 1**. This will automatically add the hash character (**#**) at the beginning of the line.
1. Directly after the newly added **#** character, type **`Authentication and Authorization`** and press the **Enter** key.
1. Click the left-most icon in the toolbar representing the header setting and, in the dropdown list, click **Header 2**. This will automatically add the hash character (**##**) at the beginning of the line.
1. Directly after the newly added **##** character, type **`Azure DevOps OAuth 2.0 Authorization Flow`** and press the **Enter** key.
1. **Copy and paste** the following code to insert a mermaid diagram on your wiki.

    ```text
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    > **Note**: For details regarding the Mermaid syntax, refer to [About Mermaid](https://mermaid-js.github.io/mermaid/#/)

1. To the right of the editor pane, in the preview pane, click **Load diagram** and review the outcome.

    > **Note**: The output should resemble the flowchart that illustrates how to [Authorize access to REST APIs with OAuth 2.0](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth)

1. In the upper right corner of the editor pane, click the down-facing caret next to the **Save** button and, in the dropdown menu, click **Save with revision message**.
1. In the **Save page** dialog box, type **`Authentication and authorization section with the OAuth 2.0 Mermaid diagram`** and click **Save**.
1. On the **Project Design** editor pane, place the cursor at the end of the Mermaid element you added earlier in this task, press the **Enter** key to add an extra line, click the left-most icon in the toolbar representing the header setting and, in the dropdown list, click **Header 2**. This will automatically add the double hash character (**##**) at the beginning of the line.
1. Directly after the newly added **##** character, type **`User Interface`** and press the **Enter** key.
1. On the **Project Design** editor pane, in the toolbar, click the paper clip icon representing the **Insert a file** action, in the **Open** dialog box, navigate to the **Downloads** folder, select the **Brand.png** file you downloaded in the previous exercise, and click **Open**.
1. Back on the **Project Design** editor pane, review the preview pane and verify that the image is properly displayed.
1. In the upper right corner of the editor pane, click the down-facing caret next to the **Save** button and, in the dropdown menu, click **Save with revision message**.
1. In the **Save page** dialog box, type **`User Interface section with the eShopOnWeb image`** and click **Save**.
1. Back on the editor pane, in the upper right corner, click **Close**.

#### Task 2: Manage a project wiki

In this task, you will manage the newly created project wiki.

> **Note**: You will start by reverting the most recent change to the wiki page.

1. On your lab computer, in the Azure DevOps portal displaying the **Wiki pane** of the **eShopOnWeb** project, with the content of the **Project Design** wiki selected, in the upper right corner, click the vertical ellipsis symbol and, in the dropdown menu, click **View revisions**.
1. On the **Revisions** pane, click the entry representing the most recent change.
1. On the resulting pane, review the comparison between the previous and the current version of the document, click **Revert**, when prompted for the confirmation, click **Revert** again, and then click **Browse Page**.
1. Back on the **Project Design** pane, verify that the change was successfully reverted.

    > **Note**: Now you will add another page to the project wiki and set it as the wiki home page.

1. On the **Project Design** pane, at the bottom left corner, click **+ New page**.
1. On the page editor pane, in the **Page title** text box, type **`Project Design Overview`**, click **Save**, and then click **Close**.
1. Back in the pane listing the pages within the **Project Design** project wiki, locate the **Project Design Overview** entry, select it with the mouse pointer, drag and drop it above the **Project Design** page entry.
1. Confirm the changes by pressing the **Move** button in the appearing window.
1. Verify that the **Project Design Overview** entry is listed as the top level page with the home icon designating it as the wiki home page.

## Review

In this lab, you created and configured Wiki in an Azure DevOps, including managing markdown content and creating a Mermaid diagram.

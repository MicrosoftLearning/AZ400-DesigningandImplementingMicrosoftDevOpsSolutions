---
lab:
    title: 'Agile planning and portfolio management with Azure Boards'
    module: 'Module 01: Implement development for enterprise DevOps'
---

# Agile planning and portfolio management with Azure Boards

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://docs.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [Create an organization or project collection](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops).

## Lab overview

In this lab, you'll learn about the agile planning and portfolio management tools and processes provided by Azure Boards and how they can help you quickly plan, manage, and track work across your entire team. You'll explore the product backlog, sprint backlog, and task boards that can track the flow of work during an iteration. We'll also look at the enhanced tools in this release to scale for larger teams and organizations.

## Objectives

After you complete this lab, you will be able to:

- Manage teams, areas, and iterations.
- Manage work items.
- Manage sprints and capacity.
- Customize Kanban boards.
- Define dashboards.
- Customize team process.

## Estimated timing: 60 minutes

## Instructions

### Exercise 0: (skip if done) Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which consist of a new Azure DevOps project with a repository based on the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Task 1: (skip if done) Create and configure the team project

In this task, you will create an **eShopOnWeb** Azure DevOps project to be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization. Click on **New Project**. Give your project the name **eShopOnWeb**. Define **Private** as Visibility option.
1. Click **Advanced** and specify **Scrum** as **Work Item Process**.
 Click on **Create**.

    ![Screenshot of the create new project panel.](images/create-project.png)

### Exercise 1: Manage Agile project

In this exercise, you will use Azure Boards to perform a number of common agile planning and portfolio management tasks, including management of teams, areas, iterations, work items, sprints and capacity, customizing Kanban boards, defining dashboards, and customizing team processes.

#### Task 1: Manage teams, areas, and iterations

In this task, you will create a new team and configure its area and iterations.

Each new project is configured with a default team, which name matches the project name. You have the option of creating additional teams. Each team can be granted access to a suite of Agile tools and team assets. The ability to create multiple teams gives you the flexibility to choose the proper balance between autonomy and collaboration across the enterprise.

1. On your lab computer, start a web browser and navigate to the Azure DevOps portal at `https://aex.dev.azure.com`.

   > **Note**: If prompted, sign in using the Microsoft account associated with your Azure DevOps subscription.

1. Open the **eShopOnWeb** project under your Azure DevOps organization.

    > **Note**: Alternatively, you can access the project page directly by navigating to the <https://dev.azure.com/YOUR-AZURE-DEVOPS-ORGANIZATION/PROJECT-NAME> URL, where the YOUR-AZURE-DEVOPS-ORGANIZATION placeholder, represents your account name, and the PROJECT-NAME placeholder represents the name of the project.

1. Click the cogwheel icon labeled **Project settings** located in the lower left corner of the page to open the **Project settings** page.

    ![Screenshot of the Azure DevOps settings page.](images/m1/project_settings_v1.png)

1. In the **General** section, select the **Teams** tab. There is already a default team in this project, **eShopOnWeb Team** but you'll create a new one for this lab. Click **New Team**.

    ![Screenshoot of the Teams project settings window.](images/m1/new_team_v1.png)

1. On the **Create a new team** pane, in the **Team name** textbox, type **`EShop-Web`**, leave other settings with their default values, and click **Create**.

    ![Screenshot of the project settings teams page.](images/m1/eshopweb-team_v1.png)

1. In the list of **Teams**, select the newly created team to view its details.

    > **Note**: By default, the new team has only you as its member. You can use this view to manage such functionality as team membership, notifications, and dashboards.

1. Click **Iterations and Area Paths** link at the top of the **EShop-Web** page to start defining the schedule and scope of the team.

    ![Screenshot of Iterations and Area Paths option."](images/m1/EShop-WEB-iterationsareas_v1.png)

1. At the top of the **Boards** pane, select the **Iterations** tab and then click **+ Select iteration(s)**.

    ![Screenshot of the iterations configuration page.](images/m1/EShop-WEB-select_iteration_v1.png)

1. Select **eShopOnWeb\Sprint 1** and click **Save and close**. Note that this first sprint will show up in the list of iterations, but the Dates are not set yet.
1. Select **Sprint 1** and click the **ellipsis (...)**. From the context menu, select **Edit**.

     ![Screenshot of the iterations tab edit settings.](images/m1/EShop-WEB-edit_iteration_v1.png)

    > **Note**: Specify the Start Date as the first work day of last week, and count 3 full work weeks for each sprint. For example, if March 6 is the first work day of the sprint, it goes until March 24th. Sprint 2 starts on March 27, which is 3 weeks out from March 6.

1. Repeat the previous step to add **Sprint 2** and **Sprint 3**. You could say that we are currently in the 2nd week of the first sprint.

    ![Screenshot of the sprints configuration under the iterations tab.](images/m1/EShop-WEB-3sprints_v1.png)

1. Still in the **Project Settings / Boards / Team Configuration** pane, at the top of the pane, select the **Areas** tab. You will find there an automatically generated area with the name matching the name of the team.

    ![Screenshot of the Areas settings page.](images/m1/EShop-WEB-areas_v1.png)

1. Click the ellipsis symbol (...) next to the **default area** entry and, in the dropdown list, select **Include sub areas**.

    ![Screenshot of the include sub areas option under the areas tab."](images/m1/EShop-WEB-sub_areas_v1.png)

    > **Note**: The default setting for all teams is to exclude sub-area paths. We will change it to include sub-areas so that the team gets visibility into all of the work items from all teams. Optionally, the management team could also choose to not include sub-areas, which automatically removes work items from their view as soon as they are assigned to one of the teams.

#### Task 2: Manage work items

In this task, you will step through common work item management tasks.

Work items play a prominent role in Azure DevOps. Whether describing work to be done, impediments to release, test definitions, or other key items, work items are the workhorse of modern projects. In this task you'll focus on using various work items to set up the plan to extend the eShopOnWeb site with a product training section. While it can be daunting to build out such a substantial part of a company's offering, Azure DevOps and the Scrum process make it very manageable.

> **Note**: This task is designed to illustrate a variety of ways you can create different kinds of work items, as well as to demonstrate the breadth of features available on the platform. As a result, these steps should not be viewed as prescriptive guidance for project management. The features are intended to be flexible enough to fit your process needs, so explore and experiment as you go.

1. Click on the project name in the upper left corner of the Azure DevOps portal to return to the project home page.

1. In the vertical navigational pane of the Azure DevOps portal, select the **Boards** icon and, select **Work Items**.

    > **Note**: There are many ways to create work items in Azure DevOps, and we'll explore a few of them. Sometimes it's as simple as firing one off from a dashboard.

1. On the **Work Items** window, click on **+ New Work Item > Epic**.

    ![Screenshot of the work item creation.](images/m1/EShop-WEB-create_epic_v1.png)

1. In the **Enter title** textbox, type **`Product training`**.
1. In the upper left corner, select the **No one selected** entry and, in the dropdown list, select your user account in order to assign the new work item to yourself. If your name doesn't appear to start with, begin typing your name and click **Search**.
1. Next to the **Area** entry, select the **eShopOnWeb** entry and, in the dropdown list, select **EShop-WEB**. This will set the **Area** to **eShopOnWeb\EShop-WEB**.
1. Next to the **Iteration** entry, select the **eShopOnWeb** entry and, in the dropdown list, select **Sprint 2**. This will set the **Iteration** to **eShopOnWeb\Sprint 2**.
1. Click **Save** to finalize your changes. **Do not close it**.

    ![Screenshot of the work item details, including title, area and iteration options.](images/m1/EShop-WEB-epic_details_v1.png)

    > **Note**: Ordinarily you would want to fill out as much information as possible, but this is sufficient for the purposes of this lab.

    > **Note**: The work item form includes all of the relevant work item settings. This includes details about who it's assigned to, its status across many parameters, and all the associated information and history for how it has been handled since creation. One of the key areas to focus on is the **Related Work**. We will explore one of the ways to add a feature to this epic.

1. In the **Related work** section on the lower right-side, select the **Add link** entry and, in the dropdown list, select **New item**.
1. On the **Add link** panel, in the **Link Type** dropdown list, select **Child**. Next, in the **Work item type** dropdown list, select **Feature**, in the **Title** textbox, type **`Training dashboard`**.

    ![Screenshot of the work item link creation.](images/m1/EShop-WEB-create_child_feature.png)

1. Click **Add link** to save the Child item.

    ![Screenshot of the work item related work area.](images/m1/EShop-WEB-epic_with_linked_item_v1.png)

    > **Note**: On the **Training dashboard** panel, note that the assignment, **Area**, and **Iteration** are already set to the same values as the epic that the feature is based on. In addition, the feature is automatically linked to the parent item it was created from.

1. On the (New Feature) **Training dashboard** panel, click **Save & Close**.

1. In the vertical navigation pane of the Azure DevOps portal, in the list of the **Boards** items, select **Boards**.
1. On the **Boards** panel, select the **EShop-WEB boards** entry. This will open the board for that particular team.

    ![Screenshot of the EShop-WEB board selection.](images/m1/EShop-WEB-_boards_v1.png)

1. On the **Boards** panel, in the upper right corner, select the **Backlog items** entry and, in the dropdown list, select **Features**.

    > **Note**: This will make it easy to add tasks and other work items to the features.

1. Hover with the mouse pointer over the rectangle representing the **Training dashboard** feature. This will reveal the ellipsis symbol (...) in its upper right corner.
1. Click the ellipsis (...) icon and, in the dropdown list, select **Add Product Backlog Item**.

    ![Screenshot of the Add Product Backlog Item option in the Training dashboard feature.](images/m1/EShop-WEB-add_pb_v1.png)

1. In the textbox of the new product backlog item, type **`As a customer, I want to view new tutorials`** and press the **Enter** key to save the entry.

    > **Note**: This creates a new product backlog item (PBI) work item that is a child of the feature and shares its area and iteration.

1. Repeat the previous step to add two more PBIs designed to enable the customer to see their recently viewed tutorials and to request new tutorials named, respectively, **`As a customer, I want to see tutorials I recently viewed`** and **`As a customer, I want to request new tutorials`**.

    ![Screenshot of the add Product Backlog.](images/m1/EShop-WEB-pbis_v1.png)

1. On the **Boards** panel, in the upper right corner, select the **Features** entry and, in the dropdown list, select **Backlog items**.

     ![Screenshot of the view backlog items.](images/m1/EShop-WEB-backlog_v1.png)

    > **Note**: Backlog items have a state that defines where they are relative to being completed. While you could open and edit the work item using the form, it's easier to just drag cards on the board.

1. On the **Board** tab of the **EShop-WEB** panel, drag the first work item named **As a customer, I want to view new tutorials** from the **New** to **Approved** stage.

    ![Screenshot of the board with work items.](images/m1/EShop-WEB-new2ap_v1.png)

    > **Note**: You can also expand work item cards to get to conveniently editable details.

1. Hover with the mouse pointer over the rectangle representing the work item you moved to the **Approved** stage. This will reveal the down facing caret symbol.
1. Click the down facing caret symbol to expand the work item card, replace the **Unassigned** entry with your name, then select your account to assign the moved PBI to yourself.
1. On the **Board** tab of the **EShop-WEB** panel, drag the second work item named **As a customer, I want to see tutorials I recently viewed** from the **New** to the **Committed** stage.
1. On the **Board** tab of the **EShop-WEB** panel, drag the third work item named **As a customer,  I want to request new tutorials** from the **New** to the **Done** stage.

    ![Screenshot of the board with work items moved to the specified columns from previous steps.](images/m1/EShop-WEB-board_pbis_v1.png)

    > **Note**: The task board is one view into the backlog. You can also use the tabular view.

1. On the **Board** tab of the **EShop-WEB** pane, at the top of the pane, click **View as Backlog** to display the tabular form.

    ![Screenshot of the EShop-WEB backlog.](images/m1/EShop-WEB-view_backlog_v1.png)

    > **Note**: You can use the plus sign directly under the **Backlog** tab label of the **EShop-WEB** panel to view nested tasks under these work items.

    > **Note**: You can use the second plus sign directly left to the first backlog item to add a new task to it.

1. On the **Backlog** tab of the **EShop-WEB** pane, in the upper left corner of the pane, click the plus sign next to the first work item. This will display the **NEW TASK** panel.

    ![Screenshot of the create task option in the backlog list.](images/m1/new_task_v1.png)

1. At the top of the **NEW TASK** panel, in the **Enter title** textbox, type **`Add page for most recent tutorials`**.
1. On the **NEW TASK** panel, in the **Remaining Work** textbox, type **5**.
1. On the **NEW TASK** panel, in the **Activity** dropdown list, select **Development**.
1. On the **NEW TASK** panel, click **Save & Close**.

    ![Screenshot of the new work item creation.](images/m1/EShop-WEB-save_task_v1.png)

1. Repeat the last five steps to add another task named **`Optimize data query for most recent tutorials`**. Set its **Remaining Work** to **3** and its **Activity** to **Design**. Click **Save & Close** once completed.

#### Task 3: Manage sprints and capacity

In this task, you will step through common sprint and capacity management tasks.

Teams build the sprint backlog during the sprint planning meeting, typically held on the first day of the sprint. Each sprint corresponds to a time-boxed interval which supports the team's ability to work using Agile processes and tools. During the planning meeting, the product owner works with the team to identify those stories or backlog items to complete in the sprint.

Planning meetings typically consist of two parts. In the first part, the team and product owner identify the backlog items that the team feels it can commit to completing in the sprint, based on experience with previous sprints. These items get added to the sprint backlog. In the second part, the team determines how it will develop and test each item. They then define and estimate the tasks required to complete each item. Finally, the team commits to implementing some or all the items based on these estimates.

The sprint backlog should contain all the information the team needs to successfully plan and complete work within the time allotted without having to rush at the end. Before planning the sprint, you'd want to have created, prioritized, and estimated the backlog and defined the sprints.

1. In the vertical navigational pane of the Azure DevOps portal, select the **Boards** icon and, in the list of the **Boards** items, select **Sprints**.
1. On the **Taskboard** tab of the **Sprints** view, in the toolbar, on the right hand side, select the **View options** symbol (directly to the left of the funnel icon) and, in the **View options** dropdown list, select the **Work details** entry. Select **Sprint 2** as filter.

    ![Screenshot of the EShop-WEB board with the work details.](images/m1/EShop-WEB-work_details_v1.png)

    > **Note**: The current sprint has a pretty limited scope. There are two tasks in the **To do** stage. At this point, neither task has been assigned. Both show a numeric value to the right of **Unassigned** entry representing the remaining work estimate.

1. Within the **ToDo** Column, notice the Task Item **Add page for most recent tutorials**, click the **Unassigned** entry and, in the list of user accounts, select your account to assign the task to yourself.

1. Select the **Capacity** tab of the **Sprints** view.

    ![Screenshot of the board with work items with capacity hours.](images/m1/EShop-WEB-capacity_v1.png)

    > **Note**: This view enables you to define what activities a user can take on and at what level of capacity.

1. On the **Capacity** tab of the **Sprints** view, for your user account, set the **Activity** field to **Development** and, in the **Capacity per day** textbox, type **1**. Then click **Save**.

    > **Note**: This represents 1 hour of development work per day. Note that you can add additional activities per user in the case they do more than just development.

    ![Screenshot of the capacity configuration.](images/m1/EShop-WEB-capacity-setdevelopment_v1.png)

    > **Note**: Let's assume you're also going to take some vacation. Which should be added to the capacity view too.

1. On the **Capacity** tab of the **Sprints** view, directly next to the entry representing your user account, in the **Days off** column, click the **0 days** entry. This will display a panel where you can set your days off.
1. In the displayed panel, use the calendar view to set your vacation to span five work days during the current sprint (within the next three weeks) and, once completed, click **OK**.

    ![Screenshot of the day off configuration.](images/m1/EShop-WEB-days_off_v1.png)

1. Back on the **Capacity** tab of the **Sprints** view, click **Save**.
1. Select the **Taskboard** tab of the **Sprints** view.

    ![Screenshot of of the work details section.](images/m1/EShop-WEB-work_details_window_v1.png)

    > **Note**: Note that the **Work details** panel has been updated to reflect your available bandwidth. The actual number displayed in the **Work details** panel might vary, but your total sprint capacity will be equal to the number of working days remaining till the end of the sprint, since you allocated 1 hour per day. Take a note of this value since you will use it in the upcoming steps.

    > **Note**: One convenient feature of the boards is that you can easily update key data in-line. It's a good practice to regularly update the **Remaining Work** estimate to reflect the amount of time expected for each task. Let's say you've reviewed the work for the **Add page for most recent tutorials** task and found that it will actually take longer than originally expected.

1. On the **Taskboard** tab of the **Sprints** view, in the square box representing the **Add page for most recent tutorials**, set the estimated number of hours to **14**, to match your total capacity for this sprint, which you identified in the previous step.

    ![Screenshot of the board and team assigned time capacity.](images/m1/EShop-WEB-over_capacity_v1.png)

    > **Note**: This automatically expands the **Development** and your personal capacities to their maximum. Since they're large enough to cover the assigned tasks, they stay green. However, the overall **Team** capacity is exceeded due to the additional 3 hours required by the **Optimize data query for most recent tutorials** task.

    > **Note**: One way to resolve this capacity issue would be to move the task to a future iteration. There are a few ways this could be done. You could, for example, open the task here and edit it within the panel providing access to the task details. Another approach would be to use the **Backlog** view, which provides an in-line menu option to move it. At this point though, don't move the task yet.

1. On the **Taskboard** tab of the **Sprints** view, in the toolbar, on the right hand side, select the **View options** symbol (directly to the left of the funnel icon) and, in the **View options** dropdown list, select the **Assigned To** entry.

    > **Note**: This adjusts your view such that you can review the progress of tasks by person instead of by backlog item.

    > **Note**: There is also a lot of customization available.

1. Click the **Configure team settings** cogwheel icon (directly to the right of the funnel icon).
1. On the **Settings** panel, select the **Styles** tab, click **+ Add styling rule**, under the **Rule name** label, in the **Name** textbox, type **`Development`**, and, in the **Color** dropdown list, select the green rectangle.

    > **Note**: This will color all cards green if they meet the rule criteria set directly below, in the **Rule criteria** section.

1. In the **Rule criteria** section, in the **Field** dropdown list, select **Activity**, in the **Operator** dropdown list, select **=**, and, in the **Value** dropdown list, select **Development**.

    ![Screenshot of the board style settings.](images/m1/EShop-WEB-styles_v2.JPG)

1. Click **Save** to save and close the settings.

    ![Screenshot of of the tasks styles in the teams board.](images/m1/EShop-WEB-sprint-green_v1.png)

    > **Note**: This will set all cards assigned to **Development** activities green.

1. Go back to the **Settings** panel, select the **General** tab, and under the **Backlogs** section, view and configure the navigation levels.

    > **Note**: Epics are not included by default, but you could change that.

1. In the **Settings** panel, select the **General** tab, and under the **Working days** section specify the working days the team follows.

    > **Note**: This applies to capacity and burndown calculations.

1. In the **Settings** panel, select the **General** tab, and under the **Working with bugs** section you can specify how you manage bugs in backlogs and boards.

    > **Note**: Entries on this tab allow you to specify how bugs are presented on the board.

1. On the **Settings** panel, click **Save** to save and close the styling rule.

    > **Note**: The task associated with **Development** is now green and very easy to identify.

#### Task 4: Customize Kanban boards

In this task, you will step through the process of customizing Kanban boards.

To maximize a team's ability to consistently deliver high quality software, Kanban emphasizes two main practices. The first, visualizing the flow of work, requires that you map your team's workflow stages and configure a Kanban board to match. The second, constraining the amount of work in progress, requires that you set work-in-progress (WIP) limits. You're then ready to track progress on your Kanban board and monitor key metrics to reduce lead or cycle time. Your Kanban board turns your backlog into an interactive signboard, providing a visual flow of work. As work progresses from idea to completion, you update the items on the board. Each column represents a work stage, and each card represents a user story (blue cards) or a bug (red cards) at that stage of work. However, every team develops its own process over time, so the ability to customize the Kanban board to match the way your team works is critical for the successful delivery.

1. In the vertical navigational pane of the Azure DevOps portal, in the list of the **Boards** items, select **Boards**.
1. On the **Boards** panel, click the **Configure board settings** cogwheel icon (directly to the right of the funnel icon).

    > **Note**: The team is emphasizing work done with data, so there is special attention paid to any task associated with accessing or storing data.

1. On the **Settings** panel, select the **Tag colors** tab, click **+ Add tag color**, in the **Tag** textbox, type **`data`** and select the yellow rectangle.

    ![Screenshot of the tag settings.](images/m1/EShop-WEB-tag_color_v1.png)

    > **Note**: Whenever a backlog item or bug is tagged with **data**, that tag will be highlighted.

1. Select the **Annotations** tab.

    > **Note**: You can specify which **Annotations** you would like included on cards to make them easier to read and navigate. When an annotation is enabled, the child work items of that type are easily accessible by clicking the visualization on each card.

1. Select the **Tests** tab.

    > **Note**: The **Tests** tab enables you to configure how tests appear and behave on the cards.

1. Click **Save** to save and close the settings.
1. From the **Board** tab of the **EShop-WEB** panel, open the Work Item representing the **As a customer, I want to view new tutorials** backlog item.
1. From the detailed item view, at the top of the panel, to the right of the **0 comments** entry, click **Add tag**.
1. In the resulting textbox, type **`data`** and press the **Enter** key.
1. Repeat the previous step to add the **`ux`** tag.
1. Save these edits by clicking **Save & Close**.

    ![Screenshot of the two new tags visible on the card.](images/m1/EShop-WEB-tags_v1.png)

    > **Note**: The two tags are now visible on the card, with the **data** tag highlighted in yellow as configured.

1. On the **Boards** panel, click the **Configure board settings** cogwheel icon (directly to the right of the funnel icon).
1. On the **Settings** panel, select the **Columns** tab.

    > **Note**: This section allows you to add new stages to the workflow.

1. Click **+ Add column**, under the **Column name** label, in the **Name** textbox, type **`QA Approved`** and, in the **WIP limit** textbox, type **1**

    > **Note**: The Work in progress limit of 1 indicates that only one work item should be in this stage at a time. You would ordinarily set this higher, but there are only two work items to demonstrate the feature.

    ![Screenshot of of the WIP settings.](images/m1/EShop-WEB-qa_column_v1.png)

1. Notice the ellipsis next to the **QA Approved** column you created. Select **Move right** twice, so that the QA Approved column gets positioned in-between **Committed** and **Done**.
1. Click **Save** to save and close the settings.

1. On the **Boards portal**, the **QA Approved** column is now visible in the Kanban board view.
1. Drag the **As a customer, I want to see tutorials I recently viewed** work item from the **Committed** stage into the **QA Approved** stage.
1. Drag the **As a customer, I want to view new tutorials** work item from the **Approved** stage into the **QA Approved** stage.

    ![Screenshot of the board that exceeds its WIP limit.](images/m1/EShop-WEB-wip_limit_v1.png)

    > **Note**: The stage now exceeds its **WIP** limit and is colored red as a warning.

1. Move the **As a customer, I want to see tutorials I recently viewed** backlog item back to **Committed**.
1. On the **Boards** panel, click the **Configure board settings** cogwheel icon (directly to the right of the funnel icon).
1. On the **Settings** panel, return to the **Columns** tab and select the **QA Approved** tab.

    > **Note**: A lag often exists between when work gets moved into a column and when work starts. To counter that lag and reveal the actual state of work in progress, you can turn on split columns. When split, each column contains two sub-columns: **Doing** and **Done**. Split columns let your team implement a pull model. Without split columns, teams push work forward, to signal that they've completed their stage of work. However, pushing it to the next stage doesn't necessarily mean that a team member immediately starts work on that item.

1. On the **QA Approved** tab, enable the **Split column into doing and done** checkbox to create two separate columns.

    > **Note**: As your team updates the status of work as it progresses from one stage to the next, it helps that they agree on what **done** means. By specifying the **Definition of done** criteria for each Kanban column, you help share the essential tasks to complete before moving an item into a downstream stage.

1. On the **QA Approved** tab, at the bottom of the panel, in the **Definition of done** textbox, type **`Passes **all** tests`**.
1. Click **Save** to save and close the settings.

    ![Screenshot of the settings split column and definition of done configuration.](images/m1/dd_v1.png)

    > **Note**: The **QA Approved** stage now has **Doing** and **Done** columns. You can also click the informational symbol (with letter **i** in a circle) next to the column header to read the **Definition of done**. You may need to refresh the browser to see changes.

    ![Screenshot of the split columns for QA Approved column.](images/m1/EShop-WEB-qa_2columns_v1.png)

1. On the **Boards** panel, click the **Configure boards settings** cogwheel icon (directly to the right of the funnel icon).

    > **Note**: Your Kanban board supports your ability to visualize the flow of work as it moves from new to done. When you add **swimlanes**, you can also visualize the status of work that supports different service-level classes. You can create a swimlane to represent any other dimension that supports your tracking needs.

1. On the **Settings** panel, select the **Swimlanes** tab.
1. On the **Swimlanes** tab, click **+ Add swimlane**, directly under the **Swimlane name** label, in the **Name** textbox, type **`Expedite`**.
1. In the **Color** dropdown list, select the **Green** rectangle.
1. Click **Save** to save and close the settings.

    ![Screenshot of Swimlane expedite creation.](images/m1/EShop-WEB-swimlane_v1.png)

1. Back on the **Board** tab of the **Boards** panel, drag and drop the **Committed** work item onto the **QA Approved \| Doing** stage of the **Expedite** swimlane so that it gets recognized as having priority when QA bandwidth becomes available.

    > **Note**: You may need to refresh the browser to make the swimlane visible.

In this exercise you learned how to use the Kanban board to visualize the flow of work in a way that is easy to understand and to track progress. You also learned how to configure the board to support your team's process and how to set the work in progress (WIP) limit to ensure that the team doesn't get overwhelmed with work.

### Exercise 2: Define dashboards

#### Task 1: Create and customize dashboards

In this task, you will step through the process of creating dashboards and their core components.

Dashboards allow teams to visualize status and monitor progress across the project. At a glance, you can make informed decisions without having to drill down into other parts of your team project site. The Overview page provides access to a default team dashboard which you can customize by adding, removing, or rearranging the tiles. Each tile corresponds to a widget that provides access to one or more features or functions.

1. In the vertical navigational pane of the Azure DevOps portal, select the **Overview** icon and, in the list of the **Overview** items, select **Dashboards**.
1. Select the **Overview** for the **eShopOnWeb Team** and review the existing dashboard.

    ![Screenshot of the dashboards overview page.](images/m1/EShop-WEB-dashboard_v1.png)

1. On the **Dashboards** pane, in the upper-right corner, select **+ New Dashboard**.

1. On the **Create a dashboard** pane, in the **Name** textbox, type **`Product training`**, in the **Team** dropdown list, select the **EShop-WEB** team, and click **Create**.

    ![Screenshot of the create dashboard panel.](images/m1/EShop-WEB-create_dash_v1.png)

1. On the new dashboard pane, click **Add a widget**.
1. On the **Add Widget** panel, in the **Search widgets** textbox, type **`sprint`** to find existing widgets that focus on sprints. In the list of results, select **Sprint Overview** and click **Add**.
1. In the rectangle representing the newly added widget, click the **Settings** cogwheel icon and review the **Configuration** pane.

    > **Note**: The customization level will vary by widget.

1. On the **Configuration** pane, click **Close** without making any changes.
1. Back on the **Add Widget** pane, in the **Search** textbox, type **`sprint`** again to find existing widgets that focus on sprints. In the list of results, select **Sprint Capacity** and click **Add**.
    > **Note**: If the widget shows "Set capacity to use the sprint capacity widget", you can select the **Set capacity** link to set the capacity. Set the Activity to Development and the Capacity to 1. Click **Save** and back to the dashboard.
1. In the **Dashboard** view, at the top of the pane, click **Done Editing**.

    ![Screenshot of the dashboard with two new widgets.](images/m1/EShop-WEB-finished_dashboard_v1.png)

    > **Note**: You can now review two important aspects of your current sprint on your custom dashboard.

    > **Note**: Another way of customizing dashboards is to generate charts based on work item queries, which you can share to a dashboard.

1. In the vertical navigational pane of the Azure DevOps portal, select the **Boards** icon and, in the list of the **Boards** items, select **Queries**.
1. On the **Queries** pane, click **+ New query**.
1. On the **Editor** tab of **Queries > My Queries** pane, in the **Value** dropdown list of the **Work Item Type** row, select **Task**.
1. Click on **Add new clause**, and in the **Field** column, select **Area Path** and, in the corresponding **Value** dropdown list, select **eShopOnWeb\\EShop-WEB**.
1. Click **Save**.

    ![Screenshot of the new query in the query editor."](images/m1/EShop-WEB-query_v1.png)

1. In the **Enter name** textbox, type **`Web tasks`**, in the **Folder** dropdown list, select **Shared Queries**, and click **OK**.
1. On the **Queries > Shared Queries** pane, click **Web tasks** to open the query.
1. Select the **Charts** tab and click **New chart**.
1. On the **Configure Chart** panel, in the **Name** textbox, type **`Web tasks - By assignment`**, in the **Group by** dropdown list, select **Assigned To**, and click **Save chart** to save the changes.

    ![Screenshot of the new web tasks pie chart.](images/m1/EShop-WEB-chart_v1.png)

    > **Note**: You can now add this chart to a dashboard.

1. Return to the **Dashboards** section in the **Overview** Menu. From the **EShop-Web** section, select the **Product Training** dashboard you used earlier, to open it.

1. Click **Edit** from the top menu. From the **Add Widget** list, search for **`Chart`** , and select **Chart for Work Items**. Click **Add** to add this widget to the EShop-Web dashboard.

1. Click the **configure** (cogwheel) within the **Chart for Work Items** to open the widget settings.

1. Accept the title as is. Under **Query**, select **Shared Queries / Web Tasks**. Keep **Pie** for Chart Type. Under **Group By**, select **Assigned To**. Keep the Aggregation (Count) and Sort (Value / Ascending) defaults.

1. Confirm the configuration by clicking **Save**.

1. Notice the query results pie chart is shown on the dashboard. **Save** the changes by pressing the **Done Editing** button on top.

## Review

In this lab you used Azure Boards to perform a number of common agile planning and portfolio management tasks, including management of teams, areas, iterations, work items, sprints and capacity, customizing Kanban boards, and defining dashboards.

---
lab:
    title: 'Lab: GitHub Actions Continuous Integration'
    module: 'Module 8: Implementing Continuous Integration with GitHub Actions'
---

# Lab: GitHub Actions Continuous Integration
# Student lab manual

## Lab overview

GitHub Actions simplify incorporating continuous integration (CI) into GitHub repositories.​ In this lab, you will set up two GitHub workflows that automate the build process.

> **Note**: In this lab, you will interact with the **github-learning-lab** bot, which will guide you through individual lab tasks.

## Objectives

After you complete this lab, you will be able to:

- Describe the importance of GitHub Actions in Continuous Integration
- Use and customize a templated workflow
- Create CI workflows that match the needs and behaviors of your team
- Use a repository's source code and build artifacts across jobs in a workflow
- Implement a unit testing framework using GitHub Actions
- Create a workflow that runs tests and produces test reports
- Set up a matrix build to create build artifacts for multiple target platforms
- Save and access a repository's build artifacts
- Choose virtual environments for an application's CI needs

> **Note**: In this lab, many of the tasks will be related to builds. Those builds sometimes take longer to build, up to several minutes. Make sure to wait for the builds to finish before moving on to your next step.

## Lab duration

-   Estimated time: **150 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 computer by using the following credentials:
    
-   Username: **Student**
-   Password: **Pa55w.rd**

#### Review applications required for this lab

Identify the applications that you'll use in this lab:
    
-   Microsoft Edge

#### Set up a GitHub account

If you don't already have a GitHub account that you can use for this lab, follow instructions available at [Signing up for a new GitHub account](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account).

### Exercise 1: Implement Continuous Integration with GitHub Actions

In this exercise, you will implement Continuous Integration with GitHub Actions

#### Task 1: Add a templated workflow to a GitHub repository

In this task, you will create a pull request with a templated workflow by using the following sequence of steps:

- Navigate to the Actions tab.
- Choose the template Node.js workflow.
- Commit the workflow to a new branch.
- Create a pull request titled CI for Node.

1.  On your lab computer, start a web browser, navigate to the launch page of the [GitHub Actions: Continuous Integration lab](https://lab.github.com/githubtraining/github-actions:-continuous-integration) and, if you're not already signed in to GitHub, click **Sign in** in the upper right corner.
1.  On the **GitHub Learning Lab** page, click **Start learning with GitHub learning lab**. 
1.  Back on the launch page of the [GitHub Actions: Continuous Integration lab](https://lab.github.com/githubtraining/github-actions:-continuous-integration), click **Start free course**.
1.  In the pop-up window, note that the **GitHub Learning Lab** will create a public repository named **github-actions-for-ci** in your GitHub account and click **Begin GitHub Actions: Continuous Integration**.
1.  Back on the launch page of the **GitHub Actions: Continuous Integration** page, in the list of **Course steps**, next to the first step **Use a templated workflow** click **Start**. You will be automatically redirected to the **Issues** tab of the **github-actions-for-ci** repository.
1.  On the **Issues** tab of the **github-actions-for-ci** repository, on the **There's a bug** issue page, review the **Welcome** section. 

    > **Note**: As per the **Welcome** section, there's a bug somewhere in your repository. You'll use the practice of Continuous Integration (CI) to set up automated testing to make it easier to discover, diagnose, and minimize scenarios like this. The codebase is written with Node.js. GitHub Actions allows you to use some templated workflows for common languages and frameworks, like Node.js. You will create a pull request with a templated workflow.

1.  On the **There's a bug** page, click the **Actions** top menu tab header. 
1.  On the **Actions** tab of the **github-actions-for-ci** repository, on the **Get started with GitHub Actions** page, in the **Node.js** pane, click **Set up this workflow**.
1.  On the **github-actions-for-ci/.github/workflows/node.js.yml** page, click **Start commit**.
1.  On the **Commit new file** pane, accept the default settings which will create a new branch for this commit and start a pull request, and click **Commit new file**.
1.  On the **Open a pull request** **Create node.js.yml** page, in the name of the pull request, rename **Create node.js.yml** to **CI for Node** and click **Create pull request**.

> **Note**: Adding templated workflow to the branch is sufficient for GitHub Actions to initiate CI for the repository. 

#### Task 2: Run a templated workflow

In this task, you will track execution of the templated workflow and review the results

1.  On the **Pull requests** tab of the **CI for Node #2** page of the **github-actions-for-ci** GitHub repo, on the **Conversation** tab of the pull request and review its intended changes, representing the content of the **.github/workflows/node.js.yml** file.
1.  As you review individual changes, click the corresponding **Resolve conversation** buttons.

    > **Note**: The **.github/workflows/node.js.yml** file includes the following components:

    - Workflow: A workflow is a unit of automation, which includes the definition of what triggers the automation, what environment or other aspects should be taken account during the automation, and what should happen as a result of the trigger.
    - Job: A job is a section of the workflow consisting of one or more steps. In our sample workflow, the template defines the steps that form a build job.
    - Step: A step represents one part of the automation. A step could be defined as a GitHub Action.
    - Action: A GitHub Action is a piece of automation written in a way that is compatible with workflows. You can use built-in actions available directly from GitHub, available from the open source community, or create a custom one. For example, **actions/checkout@v2** is used to ensure that virtual machine running the build has a copy of our codebase. The checked out code is used to run tests. The action **actions/setup-node@v1** is used to set up proper versions of Node.js since we'll be performing testing against multiple versions.

    > **Note**: The **on:** field in the **.github/workflows/node.js.yml** file tells GitHub Actions when to run. In this case, we're running the workflow anytime there's a push.

    > **Note**: The **jobs:** block in the **.github/workflows/node.js.yml** file defines the core component of an Actions workflow. Workflows are made of jobs, and our template workflow defines a single job with the identifier build. Every job also needs a specific host machine on which to run, which is designated by the value of the **runs-on:** field. This template workflow is running the build job by using the latest version of Ubuntu.

    > **Note**: In addition to running pre-built actions, the workflow can also execute commands, just as you would if you had direct access to the virtual machine running the build. By using the **run:** field of the template workflow, you can run arbitrary commands relevant to the project, such as **npm install** to install dependencies or **npm test** to run the chosen testing framework.

    ```yaml
    # This workflow will do a clean install of node dependencies, build the source code and run tests across different versions of node
    # For more information see: https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

    name: Node.js CI

    on:
      push:
        branches: [ master ]
      pull_request:
        branches: [ master ]

    jobs:
      build:

        runs-on: ubuntu-latest

        strategy:
          matrix:
            node-version: [10.x, 12.x, 14.x]

        steps:
        - uses: actions/checkout@v2
        - name: Use Node.js ${{ matrix.node-version }}
          uses: actions/setup-node@v1
          with:
            node-version: ${{ matrix.node-version }}
        - run: npm ci
        - run: npm run build --if-present
        - run: npm test
    ```

> **Note**: Ignore the fact that the workflow will fail. This is expected. To identify the reason for the failure, you can review the content of the **Actions** tab of the repository or clicking the **Details** links corresponding to the failures listed on the **Conversation** tab of the pull request.

> **Note**: In our case, the source of the error is the **npm test** command. The **npm test** command looks for a testing framework. In this lab, you will use Jest. Jest requires unit tests in a directory named **\_\_test\_\_**. The problem is that **\_\_test\_\_** directory doesn't exist on this branch.

#### Task 3: Add a jest test to the GitHub workflow

In this task, you will add a Jest-based test to the GitHub workflow to address the failure you identified in the previous task by using the following steps:

- Navigate to the open pull request named **Add Jest tests**.
- Merge the pull request

1.  On the **CI for Node #2** page of the **github-actions-for-ci** GitHub repo, click the **Pull requests** tab.
1.  In the list of pull requests, click **Add jest test**.
1.  On the **Add jest test** page, review the content of the **Conversation** tab of the pull request.

    > **Note**: This pull request introduces Jest, a popular JavaScript testing framework. We'll use it to learn how to use it for continuous integration.

1.  On the **Add jest test** page, on the **Conversation** tab of the pull request, click **Merge pull request** and then click **Confirm merge**.
1.  On the **Add jest test** page, on the **Conversation** tab of the pull request, in the comment from the **github-learning-lab** bot, click the **next step** link. This will redirect you back to the **Conversation** tab of the **CI for Node #2** pull request of the **github-actions-for-ci** GitHub repo.

#### Task 4: Review the Actions log and identify failed tests

In this task, you will review the Actions log and identify the failing test. 

> **Note**: Now that the testing framework is properly configured, the build process should automatically invoke it. You'll review the corresponding logs to determine the outcome and recommend the next step. As before, you can access logs via the **Actions** tab of the repository or the **Details** links corresponding to the failures listed on the **Conversation** tab of the pull request.

To complete this task, you will use the following sequence of steps:

- Navigate to the workflow logs
- Identify the name of the failing test
- Add a comment with the name of the failing test to the conversation

1.  Back on the **Conversation** tab of the **CI for Node #2** pull request of the **github-actions-for-ci** GitHub repo, scroll down to the **Waiting on tests** comment from the **github-learning-lab** bot and review its content.
1.  Scroll down to the list of comments on the **Conversation** tab and click each of the **Details** links next to individual failed checks. This will automatically redirect you to the **Checks** tab of the **CI for Node #2**.
1.  On the **Checks** tab of the **CI for Node #2**, review each of the build logs and verify that they all fail during **Run npm test**.
1.  Explore the **Run npm test** stage and, in the **Game** section, identify the names of the individual tests that have the **X** mark, indicating a failure.

    ```
    FAIL __test__/game.test.js
      App
        ✕ Contains the compiled JavaScript (8 ms)
      Game
        Game
          ✕ Initializes with two players (3 ms)
          ✓ Initializes with an empty board (1 ms)
          ✕ Starts the game with a random player
        turn
          ✓ Inserts an 'X' into the top center
          ✓ Inserts an 'X' into the top left
        nextPlayer
          ✕ Sets the current player to be whoever it is not (1 ms)
        hasWinner
          ✓ Wins if any row is filled (1 ms)
          ✓ Wins if any column is filled
          ✓ Wins if down-left diagonal is filled (1 ms)
          ✓ Wins if up-right diagonal is filled
    ```

1.  Switch back to the **Conversation** tab, scroll down to the very bottom in the list of comments and, on the **Write** tab of the last comment, replace **Leave a comment** with the following names you identified in the previous step, and click **Comment**.

    ```
    Initializes with two players
    Starts the game with a random player
    Sets the current player to be whoever it is not
    ```

> **Note**: This will automatically display another set of comments from the bot on the **Conversation** tab, starting with the **Reading failed logs** comment.

#### Task 5: Fix the failing test

In this task, you will modify the file that's causing the test to fail in order to correct the issue.

> **Note**: One of the failing tests is: Initializes with two players. If you dig explore the logs in more details, you may notice that a unit test expects the names the two players to be Salem and Nate, but somehow Bananas was used instead of Nate:

    ```
      ● Game › Game › Starts the game with a random player

        expect(received).toBe(expected) // Object.is equality

        Expected: "Nate"
        Received: "Bananas"

          39 | 
          40 |       Math.random = () => 0.6
        > 41 |       expect(new Game(p1, p2).player).toBe('Nate')
             |                                       ^
          42 |     })
          43 |   })
          44 | 
    ```

> **Note**: It's a common practice to name test files the same as the code file they are testing, and simply add the .test.js extension. We can assume that the test result from game.test.js is caused by a problem in game.js.

1.  On the **Conversation** tab of the **CI for Node #2** pull request of the **github-actions-for-ci** GitHub repo, locate the most recent comment from the **github-learning-lab** bot and review its content.
1.  To identify code changes referenced in the comment, click the **View changes** button, scroll down to the section representing the **\_\_test\_\_/game.test.js** file, and review the proposed changes.

    ```yaml
    Suggested change 
        this.p2 = 'Bananas'
        this.p2 = p2
    ```

1.  Navigate back to the **Conversation** tab of the **CI for Node #2** pull request of the **github-actions-for-ci** GitHub repo, scroll down to the most recent comment from the **github-learning-lab** bot, click **Commit suggestion** and, in the popup pane, click **Commit changes**.

    > **Note**: Once you commit the changes, the tests will run again and this time complete successfully. 

1.  On the **Conversation** tab of the **CI for Node #2** pull request of the **github-actions-for-ci** GitHub repo, scroll down to the **Changes approved** response, click **Merge pull request** and then click **Confirm merge**. 
1.  On the **Conversation** tab of the **CI for Node #2** pull request of the **github-actions-for-ci** GitHub repo, in the most recent comment from the **github-learning-lab** bot, click the **next step** link. This will redirect you to the **Issues** tab displaying the **A workflow for the entire team #4** page.

#### Task 6: Review the next steps

In this task, you will review the next steps that will allow you to share your first workflow so your entire team can use it.

> **Note**: Now that we've learned how to set up CI, let's try a more realistic use case. Your team has a custom workflow that goes beyond the template we've used so far. We would like the following features:

- The ability to test against multiple targets so that we can validate different combinations of operating systems and Node.js versions
- Dedicated test job so that we can separate out build from test details
- Access to build artifacts so that we can deploy them to a target environment
- Branch protections so that the master branch can't be deleted or inadvertently broken
- Required reviews so that any pull requests can be double-checked by teammates
- Obvious approvals so we can merge quickly and potentially automate merges and deployments

#### Task 7: Create a custom GitHub Actions workflow

In this task, you will create a custom GitHub Actions workflow.

To implement this functionality, you will perform the following sequence of steps:

- Edit your existing workflow file in a new branch
- In the workflow file, target versions 12.x and 14.x of Node
- Open a new pull request named **Improve CI** including the changes you made

1.  On the **Issues** tab of the repository **github-actions-for-ci** displaying the **A workflow for the entire team #4** page, scroll down to the section labeled **Step 7: Create a custom GitHub Actions workflow** and, in the **Activity: Edit the existing workflow with new build targets** listing, click the **existing workflow** link. This will redirect you to the **Edit file** view of the **github-actions-for-ci/.github/workflows/node.js.yml** file on the **Code** tab of the repository **github-actions-for-ci**.
1.  In the **github-actions-for-ci/.github/workflows/node.js.yml** file, replace `node-version: [10.x, 12.x, 14.x]` with `node-version: [12.x, 14.x]`
1.  On the **Code** tab of the **github-actions-for-ci** repository, in the upper right corner, click **Start commit**.
1.  On the **Commit changes** pane, accept the default settings which will create a new branch for this commit and start a pull request, and click **Commit changes**.
1.  On the **Open a pull request** page, in the name of the pull request, rename **Update node.js.yml** to **Improve CI** and click **Create pull request**. This will automatically redirect you to the **Create CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository.

> **Note**: By targeting specific versions of Node, you've configured a build matrix which allow for testing across multiple operating systems, platforms, and language versions. For more information regarding this topic, you can refer to [GitHub Docs](https://help.github.com/en/articles/configuring-a-workflow#configuring-a-build-matrix).

#### Task 8: Target a Windows environment

In this task, you will edit your workflow file to build for Windows environments

> **Note**: Since we'd like to support deploying our app to Windows environments, let's add Windows to the matrix build configuration.

To implement this functionality, you will perform the following sequence of steps:

- Add the **os** field to the **strategy.matrix** section of your workflow
- Add **ubuntu-latest** and **windows-2016** to the list of target operating systems
- Commit your workflow changes to the existing branch

    ```yaml
    node-version: [10.x, 14.x]
    os: [ubuntu-latest, windows-2016]
    node-version: [12.x, 14.x]
    ```

1.  On the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository, scroll down to the latest comment from the **github-learning-lab** bot and review its content.
1.  In the **Activity: Edit your workflow file to build for Windows environments** section, click the **.github/workflows/nodejs.yml** link in step 1.  This will redirect you to the **Edit file** view of the **github-actions-for-ci/.github/workflows/node.js.yml** file on the **Code** tab of the repository **github-actions-for-ci**.
1.  In the **github-actions-for-ci/.github/workflows/node.js.yml** file, review the matrix section and identify the changes that are supposed to be made. 
1.  Navigate back to the **Activity: Edit your workflow file to build for Windows environments** section of the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository, click **Commit suggestion** and, in the popup pane, click **Commit changes**.
1.  This will update the content of the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository with the latest comment from the **github-learning-lab** bot, labeled **New Job**.

> **Note**: If you review the logs, you'll notice that there are, at this point, 4 builds. That's because for each of the 2 operating systems we're running tests against 2 versions.

#### Task 9: Configure multiple jobs

In this task, you will edit your workflow file to separate build and test jobs

> **Note**: Let's now create a dedicated test job. This will allow us to separate the build and test functions of our workflow into more than one job that will run when our workflow is triggered.

1.  On the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository, in the latest comment from the **github-learning-lab** bot, labeled **New Job**, in the **Step 9: Use multiple jobs** section, click the **your workflow file** link in step 1.  This will redirect your browser to the **Edit file** view of the **github-actions-for-ci/.github/workflows/node.js.yml** file on the **Code** tab of the repository **github-actions-for-ci**.
1.  In the build job section, add the following content representing your existing workflow:

    ```yaml
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
        - name: npm install and build webpack
          run: |
            npm install
            npm run build
    ```

1.  At the very bottom of the **github-actions-for-ci/.github/workflows/node.js.yml** file, add a new entry representing the job named **test** (ensure that its indentation matches the **build** job entry) with the following content:

    ```yaml
    test:
      runs-on: ubuntu-latest
      strategy:
        matrix:
          os: [ubuntu-latest, windows-2016]
          node-version: [12.x, 14.x]
      steps:
      - uses: actions/checkout@v2
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v1
        with:
          node-version: ${{ matrix.node-version }}
      - name: npm install, and test
        run: |
          npm install
          npm test
        env:
          CI: true
    ```

1.  This should yield the following content:

    ```yaml
    name: Node CI

    on: [push]

    jobs:
      build:

        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v2
          - name: npm install and build webpack
            run: |
              npm install
              npm run build

      test:

        runs-on: ubuntu-latest

        strategy:
          matrix:
            os: [ubuntu-latest, windows-2016]
            node-version: [12.x, 14.x]

        steps:
        - uses: actions/checkout@v2
        - name: Use Node.js ${{ matrix.node-version }}
          uses: actions/setup-node@v1
          with:
            node-version: ${{ matrix.node-version }}
        - name: npm install, and test
          run: |
            npm install
            npm test
          env:
            CI: true
    ```

1.  On the **Code** tab of the repository **github-actions-for-ci** displaying the content of the **github-actions-for-ci/.github/workflows/node.js.yml** file you are editing, in the upper right corner, click **Start commit**.
1.  On the **Commit changes** pane, accept the default settings, which will commit the change directly to the current branch, and click **Commit changes**.

> **Note**: Following this commit, the workflow will run again. 

#### Task 10: Run multiple jobs
 
In this task, you will simply wait for the result of multiple jobs in your workflow.

> **Note**: There are no action required in this task. 

#### Task 11: Upload a job's build artifacts

In this task, you will use the upload action in your workflow file to save a job's build artifacts

> **Note**: You may noticed that the build succeeded, but each of the test jobs failed. This is because the build artifacts created in build aren't available to the test job, since each job executes in a new instance of the virtual environment. This is inherent part of design of the virtual environment. To address this, we can use the built-in artifact storage to save artifacts created from one job to be used in another job within the same workflow. Artifacts allow you to persist data after a job has completed and share that data with another job in the same workflow. An artifact is a file or collection of files produced during a workflow run.

> **Note**: To upload artifacts to the artifact storage, we can use an action built by GitHub: actions/upload-artifacts.

1.  Navigate back to the **Pull requests** tab of the **Improve CI #5** page of the **github-actions-for-ci** GitHub repo, on the **Conversation** tab, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository with the latest comment from the **github-learning-lab** bot, labeled **New Job**, in the **Step 11: Upload a job's build artifacts** section, click the **your workflow file** link in step 1. This will redirect your browser to the **Edit file** view of the **github-actions-for-ci/.github/workflows/node.js.yml** file on the **Code** tab of the repository **github-actions-for-ci**.
1.  In the build job section, update the build job to include the **upload-artifacts** action, such that it matches the following content:

    ```yaml
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v2
          - name: npm install and build webpack
            run: |
              npm install
              npm run build
          - uses: actions/upload-artifact@master
            with:
              name: webpack artifacts
              path: public/
    ```

1.  On the **Code** tab of the repository **github-actions-for-ci** displaying the content of the **github-actions-for-ci/.github/workflows/node.js.yml** file you are editing, in the upper right corner, click **Start commit**.
1.  On the **Commit changes** pane, accept the default settings, which will commit the change directly to the current branch, and click **Commit changes**.

> **Note**: Following this commit, the workflow will run again. 

#### Task 12: Download a job's build artifacts

In this task, you will use the download action in your workflow file to access a prior job's build artifacts

> **Note**: The build artifacts will now be uploaded to the artifact storage, but if track the workflow, you'll notice the test job still fails. There are two primary reasons for it:

- jobs run in parallel, unless explicitly configured to run sequentially.
- each job runs in its own virtual environment, so although we've pushed our artifacts to storage, the test job needs to retrieve them.

> **Note**: To remedy this, you'll run test only after build is finished so the artifacts are available. In addition, to leverage the upload action that copies artifacts to the designated artifact store, we'll use another GitHub built-in action **actions/download-artifact** to download these artifacts.

1.  Navigate back to the **Pull requests** tab of the **Improve CI #5** page of the **github-actions-for-ci** GitHub repo, on the **Conversation** tab, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository with the latest comment from the **github-learning-lab** bot, labeled **New Job**, in the **Step 12: Download a job's build artifacts** section, click the **your workflow file** link in step 1. This will redirect your browser to the **Edit file** view of the **github-actions-for-ci/.github/workflows/node.js.yml** file on the **Code** tab of the repository **github-actions-for-ci**.
1. In the test job section, update the test job to run only after the build job is completed by adding the **needs: build** component, such that it matches the following content:

    ```yaml
    test:
      needs: build
      runs-on: ubuntu-latest
    ```

1.  In the build job section, update the build job to include the **download-artifacts** action, such that it matches the following content:

    ```yaml
    steps:
    - uses: actions/checkout@v2
    - uses: actions/download-artifact@master
      with: 
        name: webpack artifacts
        path: public
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}
    - name: npm install, and test
      run: |
        npm install
        npm test
      env:
        CI: true
    ```

1.  On the **Code** tab of the repository **github-actions-for-ci** displaying the content of the **github-actions-for-ci/.github/workflows/node.js.yml** file you are editing, in the upper right corner, click **Start commit**.
1.  On the **Commit changes** pane, accept the default settings, which will commit the change directly to the current branch, and click **Commit changes**.

> **Note**: Following this commit, the workflow will run again. 

> **Note**: Our custom workflow now provides the following functionality:

- testing against multiple targets so that we know if our supported operating systems and Node.js versions are working
- dedicated testing job so that we can separate out build from test details
- access to build artifacts so that we can deploy them to a target environment

#### Task 13: Share the improved CI workflow with the team

In this task, you will merge the pull request with the improved workflow into the master branch.

1.  Navigate back to the **Pull requests** tab of the **Improve CI #5** page of the **github-actions-for-ci** GitHub repo, on the **Conversation** tab, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository, on the latest comment from the **github-learning-lab** bot, labeled **Merge the CI**, review the **Step 13: Share the improved CI workflow with the team** section.

    > **Note**: Here's our status on the list of requirements we implemented so far:

    - test against multiple targets so that we know if our supported operating systems and Node.js versions are working
    - dedicated test job so that we can separate out build from test details
    - access to build artifacts so that we can deploy them to a target environment

    > **Note**: In the next few steps, you'll make changes that will provide the following functionality to the team's workflow:

    - branch protections so that the master branch can't be deleted or inadvertently broken
    - required reviews so that any pull requests are double checked by teammates
    - obvious approvals so we can merge quickly and potentially automate merges and deployments

1.  Scroll down to the **Changes approved** section, click **Merge pull request** and then click **Confirm merge**.

#### Task 14: Automate the review process

In this task, you will add a new workflow file to automate the team's review process

1.  On the **Improve CI #5** page of the **Pull requests** tab of the **github-actions-for-ci** repository, on the latest comment from the **github-learning-lab** bot, click the **next step** link. This will redirect you to the **Conversation** tab of the pull request named **A custom workflow #6**.
1.  On the **Conversation** tab of the **A custom workflow #6** pull request, scroll down and review section labeled **Step 14: Automate the review process**.

    > **Note**: GitHub Actions can run multiple workflows for different event triggers. Let's create a new approval workflow that'll work together with our Node.js workflow.

1.  In the section labeled **Activity: Add a new workflow file to automate the team's review process**, in the list of actions, click the **new file** link in step 1. This will redirect you to the **Code** tab of the **github-actions-for-ci** repository, displaying the editor page with the **github-actions-for-ci/.github/workflows/approval-workflow.yml** file open.
1.  Within the editor pane, enter `name: AZ-400 Team approval workflow`, scroll down to the bottom of the page and click **Commit new file**

#### Task 15: Use an action to automate pull request reviews

In this task, you will use the community action in your new workflow

> **Note**: Workflows can be configured to run:

- Using events from GitHub
- At a scheduled time
- When an event outside of GitHub occurs

> **Note**: So far, we've used the push event for our Node.js workflow. That makes sense when we want to take action on code changes to our repository. For a review workflow, we want to engage with human reviews. For example, we'll use the Label approved pull requests action so that we can easily see when we've gotten enough reviews to merge a pull request. Let's prepare our review workflow by triggering it with the **pull_request_review** event.

1.  Navigate back to the **Conversation** tab of the pull request **A custom workflow #6**, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the page labeled **A custom workflow #6** of the **Pull requests** tab of the **github-actions-for-ci** repository, on the latest comment from the **github-learning-lab** bot, review the **Step 15: Use an action to automate pull request reviews** section.
1.  Note that this step involves simply adding **on: pull_request_review** to the workflow file you created in the previous step.
1.  Within the **Step 15: Use an action to automate pull request reviews** section, click **Commit suggestion** and, in the popup pane, click **Commit changes**.

#### Task 16: Create an approval job in your new workflow

In this task, you will in your new workflow file, create a new job that'll use the community action

1.  On the **Conversation** tab of the pull request **A custom workflow #6**, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the **A custom workflow #6** page of the **Pull requests** tab of the **github-actions-for-ci** repository, on the latest comment from the **github-learning-lab** bot, review the **Step 16: Create an approval job in your new workflow** section.
1.  Note that this step involves adding to the same workflow file you modified in the previous task a new job named **labelWhenApproved** in the format:

    ```yaml
    jobs:
      labelWhenApproved:
        runs-on: ubuntu-latest
    ```

1.  Within the **Step 16: Create an approval job in your new workflow** section, click **Commit suggestion** and, in the popup pane, click **Commit changes**.

#### Task 17: Automate approvals

In this task, you will use the community action to automate part of the review approval process

> **Note**: In the previous task, you created a job with the **labelWhenApproved** identifier. Now, you will use a community-created action named **pullreminders/label-when-approved-action**. The action adds a label to any pull requests after a preset number of approvals. These labels could be used as a visual indicator to your team that something is ready to merge, or even as a way to use other actions or tools to automatically merge pull requests when they receive the required number of approvals.

1.  On the **Conversation** tab of the pull request **A custom workflow #6**, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the **A custom workflow #6** page of the **Pull requests** tab of the **github-actions-for-ci** repository, on the latest comment from the **github-learning-lab** bot, review the **Step 17: Automate approvals** section.

    > **Note**: To implement it, use the following guidelines:

    - the workflow file needs a **steps:** block
    - step names are arbitrary
    - to use a community action, include the **uses:** keyword
    - the **label-when-approved-action** requires a block named **env:** with the following environment variables:
       - **APPROVALS** is the number of required approvals that are required for a label to be applied. For the purpose of this lab, set it to **1**.
       - **GITHUB_TOKEN** is necessary so the action can create and apply labels to this repository. 
       - **ADD_LABEL** is the name of the label which should be added when the number of approvals have been met. Its name is arbitrary.

1.  Switch to the **Code** tab of the **github-actions-for-ci** repository, in the list of branches, select the **team-workflow** entry, in the list of folders, click **.github/workflows**, in the list of files, click **approval-workflow.yml**, and, while viewing the content of **github-actions-for-ci/.github/workflows/approval-workflow.yml**, click the pencil icon on the right side of the code pane to enter the edit mode.
1.  Add the following content to the **approval-workflow.yml** file.

    ```yaml
    steps:
    - name: Label when approved
      uses: pullreminders/label-when-approved-action@master
      env:
        APPROVALS: "1"
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        ADD_LABEL: "approved"
    ```

1.  Click **Start commit**.
1.  On the **Commit changes** pane, accept the default settings, which will commit the change directly to the current branch, and click **Commit changes**.

#### Task 18: Use branch protections

In this task, you will complete the automated review process by protecting the master branch.

> **Note**: Protected branches ensure that collaborators on your repository cannot make irrevocable changes to branches. Enabling protected branches also allows you to enable other optional checks and requirements, like required status checks and required reviews.

1.  Navigate back to the **Conversation** tab of the pull request **A custom workflow #6**, scroll down to the most recent comment from the **github-learning-lab** bot and review its content.
1.  On the page labeled **A custom workflow #6** of the **Pull requests** tab of the **github-actions-for-ci** repository, on the latest comment from the **github-learning-lab** bot, review the **Step 18: Use branch protections** section.
1.  Scroll back up to the top of the page, in the top menu, click the **Settings** header and, in the vertical menu on the left side, click **Branches**.
1.  In the **Branch protection rules** section, click **Add rule**.
1.  On the **Branch protection rule**, in the **Branch name pattern** type **master**, in the list of **Protect matching branches** settings, enable **Require pull request reviews before merging**, **Require status checks to pass before merging**, select the checkboxes next to each of the build and test jobs, scroll to the bottom of the page, and click **Create**.
1.  Navigate back to the **Conversation** tab of the pull request **A custom workflow #6**, scroll down to the most recent comment from the **github-learning-lab** bot, review the **Step 18: Use branch protections** section, and, in step 8 of the **Activity: Complete the automated review process by protecting the master branch** subsection, click the **approve the requested review** link. This will redirect your browser session to the **Files changed** tab of **A custom workflow** pull requests.
1.  On the **Files changed** tab of **A custom workflow** pull requests, click **Review changes**, select the **Approve** option, and click **Submit review**. This will redirect your browser session back to the page labeled **A custom workflow #6** of the **Pull requests** tab of the **github-actions-for-ci** repository, with the most recent comment from the **github-learning-lab** bot, confirming that the workflow was successfully completed.

## Review

In this lab, you learned how to set up GitHub workflows that automate the build process.
# Hands-on Modernization Lab

Modernization is essential to preserve the security and stability of your critical enterprise applications, and to be able to increase the delivery of new innovations. Modernization can take many forms, including adopting a more modern runtime, adopting Kubernetes/OpenShift, and refactoring to Microservices.  Many modernization projects struggle to deliver because of the complexity inherent in uprooting an application and its dependencies, understanding code written decades ago, and acquiring new skill for the destination environment. In this lab you'll use two new preview technologies that address these challenges:
1. Enterprise Application Service for Java. This is a new managed service which provides an end-to-end application delivery experience, including DevOps, GitOps, build & delivery pipelines, running applications, and observability integration. Being a fully managed service, it insulates you from the infrastructure complexities so you can focus on delivering the code.
2. Application Modernization Accelerator. This tool enables you to discover existing enterprise applications and their database and messaging dependencies.  Assess the complexity of migration, plan a migration and then assist the migration (including database and messaging) through curated instructions, artefact generation and code rewrite rules.

In this lab you will use Application Modernization Accelerator to create a migration plan that includes automatically generated configuration files and scripts, guidance and pointers to tools that are needed to accelerate and complete the migration. 
 
This migration plan will include: 
- Details for the MQ administrator to configure the on-premises MQ network
- Details for the Db2 administrator to transfer the data from the on-premises database to the new Db2 SaaS
- Generated Liberty configuration files required to migrate the application

Application Modernization Accelerator can then automatically provision, configure and wire together the Enterprise Application Service, MQ and Db2 SaaS services to support the application running in the cloud.
 
During the code remediation process, you will have the opportunity to utilize the services of **watsonx Code Assistant for Enterprise Java Applications** (WCA4EJA) to assist with code explanation, code completion, test generation, and also consume the output of Application Modernization Accelerator to convert the application from WebSphere to Enterprise Application Service.
 
The generated files are then brought into GitHub where Enterprise Application Service will build the application. Deterministic changes to the application source code can be fixed automatically using pre-defined rules. Other fixes are facilitated through Generative AI where you can interact in a guided manner with a Java-trained LLM and generated code can be added to the application source.
 
Enterprise Application Service has a built-in CI/CD workflow for delivering applications, as follows:
1. Every time you create a Pull Request in GitHub, Enterprise Application Service will run a Pull Request build. You can then review the build, the test results.  
2. Once happy with the updates, you can merge the Pull Request to trigger a Release Build.  Again, you can review the build output, test results, and generated artefacts.
3. Releases are deployed through updates to a GitHub configuration repository.  Updating a staging environment yaml to reference the release will cause it to be deployed into a predefined Staging environment.  As with the code updates, PRs and PR merging can be used to achieve this.  Creating a PR will cause a Config validation build to run to check the configuration conforms to the deployment schema.
4. Once happy with the application in Staging, the release can be promoted to a pre-defined Production environment by making equivalent updates to the production environment yaml.

# Steps:

## 1. Discovering applications and planning your migration

Launch Application Modernization Accelerator (AMA) using the url **http://10.100.1.10:3000**:

![ama-home-page](images/ama-home-page.png)

### Discovery tool

A major capability of Application Modernization Accelerator is discovery. The discovery tool is downloadable from AMA and can be used on all major platforms:

![ama-discovery-tool](images/ama-discovery-tool.png)

The discovery tool can be executed on any system that you want analyzed. It will discover all of your relevant Java application runtimes (such as IBM WebSphere Application Server), and their database and messaging dependencies. The tools will also discover details about any discovered Db2 database instances.

The result of the discovery is then automatically uploaded into Application Modernization Accelerator as a **Workspace**.

For the convenience of this lab, we have already run the discovery tool, and the results are stored in the **SampleData** workspace.

### Explore the Discovery tool results

From the main AMA panel, click on the workspace labelled **SampleData**. This will display a topology map from the **Visualization** tab.

![ama-discovered-estate](images/ama-discovered-estate.png)

The map includes all the applications, databases and message queues, and their relationships, which were discovered from the scanned systems.

![ama-node-types](images/ama-node-types.png)

You also see the total number and type of each node:

![ama-asset-count](images/ama-asset-count.png)

You will also see several features to help you pan and zoom on the map.

Clicking on a specific node will bring up more details.

![ama-node-details](images/ama-node-details.png)

### Identify potential capabilities to modernize

Each set of connected nodes can be thought of as a **workload** that can be modernized. Let's filter our map to show the `Acme` workload.

Type **acme** in the field:

![ama-acme-capability](images/ama-acme-capability.png)

Here you see all the nodes associated with the **Acme** workload. As you can see, this is a very complex system, with multiple applications using multiple message queues and databases. You can use the dependency information to help you to decide where to start with modernization. Further details to help with this decision are also available and will be seen later. 

For the purpose of this lab, we will work on a simpler workload. This is how you would typically start a modernization project.  Starting simple helps you learn the tools and technologies.  Replace the field field with **mod**.

![ama-mod-capability](images/ama-mod-capability.png)

Now you see only three nodes highlighted, which looks like a much better option to start with.

Click on the application node (the blue hexagon) to see more details about ModResorts:

![ama-modresorts-details](images/ama-modresorts-details.png)

ModResorts appears to be a good candidate for modernization. Now let's explore this application and its dependencies in more detail.

### Modernize the ModResorts application

Click on the **Assessment** tab:

![ama-assessment-panel](images/ama-assessment-panel.png)

Note that the **Enterprise Application Service** is automatically selected as the **Target destination**. This is the IBM Enterprise Application Service for Java managed service where we wish to build and deploy the application.

As you can see, the overall total development cost for the whole workload is 21 days.

Now let's focus on the ModResorts application. Scroll down and enter **modresorts** in the search bar to show the **ModResorts.ear** application:

![ama-modresorts-assess](images/ama-modresorts-assess.png)

Additionally, you see:
- this application has **Moderate** complexity, which means that code changes are required for ModResorts to work on Enterprise Application Service.
- there are 2 **Critical** issues and 5 **Information** issues. If you mouse over the issues you will see that Critical issues must be addressed, and Information issues are relevant in testing.
- the **Required code changes** column shows that we have an **Automated** way to resolve all the issues in the application.

Now click on the **ModResorts.ear** link to see the applications details page:

![ama-modresorts-ear-details](images/ama-modresorts-ear-details.png)

This panel contains all the detailed information about this application. 

Scroll down to the **Required code changes** section. Here you will see the build configuration you can use in a maven or gradle build to execute the rewrite rules to automatically resolve the issues in the application.  These same rules can also be executed within watsonx Code Assistant (see `Run auto-fixes` later in the lab).

![ama-req-code-changes](images/ama-req-code-changes.png)

Scroll down to the **Issues** section to see what the automated code changes will resolve.

Open the technology issues accordion to see the two Critical issues. Expand them to review the help – but remember these can be resolved automatically by executing the configuration changes provided. [**NOTE** You will do this later in the lab using IBM watsonx code assistant].

![ama-issues-expanded](images/ama-issues-expanded.png)

### Generating the migration plan

Now that we have reviewed all the details of the ModResorts application, it still looks like a good candidate for modernizing to the Cloud. Reasons include:
- all code changes can be automated.
- all dependencies (database and message queue) are not shared with 
other applications.

To start the migration, click the **View migration plan** button at the top of the panel.

The Migration plan page contains all of the details about what will be included in your migration bundle:

![ama-migration-panel](images/ama-migration-panel.png)

Click through and read the contents of each section (Note, some are placeholder instructions which are under development). 

At the right panel of the **IBM Enterprise Application Service** section you can see a preview of the files that are provided. Click on the `server.xml` file to get a preview:

![ama-server-xml](images/ama-server-xml.png)

This is the starting configuration used to run the server and application locally during development and local testing.

At the top left side of the panel, click the **Dependency endpoints** button:

![ama-dependency-endpoints](images/ama-dependency-endpoints.png)

This panel shows all the dependencies that are part of this migration plan. If you do not want to modernize any of the dependencies, or if they are no longer relevant, you can remove them from the plan by deselecting them.

For this lab we will be modernizing all of the dependencies.

[Click on **Start in IBM Cloud**. Upon completion, it will create a Cloud project and stand up and configure service instances for the application, databases and message queues defined in the migration plan. From there you can customize the instances to suit your needs.]

At the top right side of the panel, click the **Download plan** button. This will result in a zip file being downloaded to your local system.

![ama-download-plan](images/ama-download-plan.png)

You can explode this zip file bundle to see all the artifacts that AMA provides, including a **README** file that gives you step by step instructions on how to use the bundle. You will use this bundle in the next section of the lab.

## 2. Updating the application to run on Enterprise Application Service for Java

For this step, we will start with the source for the applications which you discovered, assessed and planned in step 1.

### Clone the ModResorts GitHub repo

1. In a terminal, clone the ModResorts application.
   
    ```bash
    cd /home/admin
    git clone https://github.com/techxchange2024/mod-resorts.git
    ```

## Experiencing watsonx Code Assistant for Enterprise Java Applications

This section will provide guidance on the usage of the features of watsonx Code Assistant for Enterprise Java Applications.

### WebSphere Application Server modernization to Liberty with watsonx Code Assistant for Enterprise Java Applications Eclipse IDE plugin

First, let’s start with **Eclipse IDE**. Locate and launch the Eclipse IDE from the Activities task bar.

1. After launching Eclipse, import the ModResorts app as a Maven project by going to the
    **File** drop-down menu in the top left corner, then selecting
    **Import -\> Maven -\> Existing Maven Projects**.

2. Please note that to access watsonx Code Assistant for Enterprise Java Applications, you will need to log in using the API key from IBM Cloud. Your instructor will provide you with the necessary API key. In Eclipse IDE, you can find the watsonx Code Assistant for Enterprise Java Applications tab in the top panel of the interface.

    <div align="center">
        <img src="./images/wca-icon.png">
    </div>

    After successfully logging into watsonx Code Assistant for Enterprise Java Applications, you should see a greeting message from watsonx Code Assistant for Enterprise Java Applications in the chat panel.

    <div align="center">
        <img src="./images/wca-greeting.png">
    </div>

3.  To ensure that the project is properly configured with the dependencies for the WebSphere Application Server APIs, please run the following commands in a terminal window to install the required dependencies into your local Maven project.

    ```bash
    cd mod-resorts
    mvn install:install-file -Dfile="was-dependency/was_public.jar" -DpomFile="was-dependency/was_public-9.0.0.pom"
    ```

4.  The application was configured and written using **traditional WebSphere Application Server** code, watsonx Code Assistant for Enterprise Java Applications supports an automated AI powered feature to modernize this application to use the latest WebSphere Liberty Server. To access this feature, right-click on the project and select **watsonx Code Assistant for Enterprise Java Applications → Modernize to Liberty**. You should see the watsonx Code Assistant for Enterprise Java Applications extension interface.

    <div align="center">
        <img src="./images/wca-modernize.png">
    </div>

    > **NOTE**: Please wait for the console message `Retrieving the analysis report completed` before proceeding to the next step.
        <div align="center">
            <img src="./images/wca-report-complete.png">
        </div>

    Click on the **Modernize project: modresorts** tab.

    <div align="center">
        <img src="./images/wca-modernize-tab.png">
    </div>

5.  WCA has two options to migrate the project to Liberty:
    1.  **Upload the migration bundle** exported for Application Modernization Accelerator, which will provide watsonx Code Assistant with the artefacts required to migrate the application.
    2. Use the **binary scanner** if a migration bundle is not available. 
 
    Note that using the migration bundle is recommended because it has been generated with the configuration information from where the Java application is running, while the binary scanner will not be able to obtain this information. The migration bundle option will be used in this lab. Please locate and open the bundle zip file you built and downloaded using `Application Modernization Accelerator`.

    <div align="center">
        <img src="./images/wca-start-modernize-msg.png">
    </div>

    <div align="center">
        <img src="./images/wca-start-scan.png">
    </div>

6.  watsonx Code Assistant for Enterprise Java Applications will extract configuration files, including `server.xml` and `Containerfile`, from the migration bundle. The `server.xml` file contains the application's runtime configuration. The `Containerfile` can be used to build a container image for your applications. Enterprise Application Service only needs your application source and runtime configuration, so the `Containerfile` is not used. After reviewing these files, add them to the project folder by clicking on **Add Files to Project** to apply the changes.

    <div align="center">
        <img src="./images/wca-add-files.png">
    </div>

7.  Next, a modernization issues list will be presented with the must fix issue detected by watsonx Code Assistant for Enterprise Java Applications. You can review each issue in detail by clicking on the expand button on the right-hand side of each item. The **Additional information** tab highlights other issues you should be aware of when migrating your application, though these do not require code changes. After the review is complete, click the **Run auto-fixes** button to apply the automated changes (Note, these are the same `rewrite` changes you could have applied using maven and gradle with the configuration from Application Modernization Accelerator. Applying them in WCA is easier as it doesn't require you to customize and run a build).

    <div align="center">
        <img src="./images/wca-issues-list-sm.png">
    </div>

8.  After the auto-fix is completed, click on **Build and Refresh** to
    refresh the list. All the listed issue should be resolved.

    <div align="center">
        <img src="./images/wca-resolved-list.png">
    </div>

9.  Once migration is completed, add the Liberty maven plugin configuration in the **pom.xml** file.  This plugin makes it easy to build and test the application locally using the Liberty runtime.  This is the same runtime used in Enterprise Application Service for Java:

    ```bash
    <plugin>
        <groupId>io.openliberty.tools</groupId>
        <artifactId>liberty-maven-plugin</artifactId>
        <version>3.10.3</version>
    </plugin>
    ```

10. Additonally, add links to the Db2 and MQ drivers. First we copy the drivers by adding the following `resources` to the **pom.xml** file:

    ```bash
    <resources>
        <resource>
            <directory>${project.basedir}/src/main/resources</directory>
        </resource>
        <resource>
            <directory>${project.basedir}/db-drivers</directory>
            <targetPath>${project.build.directory}/liberty/wlp/usr/shared/config/lib/global</targetPath>
            <filtering>false</filtering>
        </resource>
        <resource>
            <directory>${project.basedir}/mq-drivers</directory>
            <targetPath>${project.build.directory}/liberty/wlp/usr/shared/config/lib/global</targetPath>
            <filtering>false</filtering>
        </resource>
    </resources>
    ```

    And then we reference the drivers by updating their location in the **server.xml** file:

    ```bash
    <library>
        <file name="{shared.config.dir}/lib/global/db2jcc.jar"/>
        <file name="{shared.config.dir}/lib/global/db2jcc_license_cu.jar"/>
        <file name="{shared.config.dir}/lib/global/db2jcc_license_cisuz.jar"/>
    </library>

    <resourceAdapter id="mqJmsRa" location="${shared.config.dir}/lib/global/wmq.jmsra.rar">
    ```

11. Run the application in Liberty Dev Mode in the same terminal used in step 3. Dev Mode is an interactive developer experience where you can make changes to your application or configuration and Dev Mode automatically updates the application to give you immediate feedback. You can also press **Enter** to run application test cases. To view the running application, visit <http://localhost:9080/resorts>.

    ```bash
    mvn liberty:dev
    ```

12. After you are finished checking out the application, stop the Liberty instance by pressing **CTRL+C** in the command-line session where you ran Liberty. 

## 3. Deploy application to cloud using IBM Enterprise Application Service for Java

Now that we have modernized our ModResorts application to use the Java **Liberty Runtime**, we can build and deploy it in Enterprise Application Service for Java.

Provide the instructor with your email address associated with your GitHub account. You will then be assigned a student number that you can use throughout this section of the lab.

For convenience, each student will be given authorization to access the following:

- GitHub Organization - **techxchange2024**
- GitHub **source** repo - **student-source-{number}**
- GitHub **config** repo - **student-config-{number}**
- IBM SaaS account - **techxchange2024-saas**
- IBM Enterprise Application Service subscription - **Application Flow**
- IBM Enterprise Application Service service instance - **student-{number}**

>**NOTE**: To save time and eliminate any mistakes made in the previous step, the **source** repository already contains the ModResorts app with the changes applied to run on Liberty.

>**IMPORTANT**: You will receive an email regarding access to the **techxchange2024** GitHub Organization. You must accept the invitation in order to complete this section of the lab.

### Access your Enterprise Application Service service instance

1. Use this link to start:

    https://ibm-cloud.console.saas.ibm.com

    <div align="center">
        <img src="./images/saas-accounts.png">
    </div>

    Click on the **techxchange2024-saas** tile to view the account details.

1. Click the **View instances** link to bring up the **Instance** list.

    <div align="center">
        <img src="./images/saas-subscription-details.png">
    </div>

1. Select your instance (it should be the only one with a clickable link).

    <div align="center">
        <img src="./images/saas-instance-list.png">
    </div>

    From the list of instances, **Open** the instance associated with your student number.

### Configure GitHub repos for your Enterprise Application Service service instance

1. Since no GitHub repos have been configured to your service instance, you will need to do this now. This step enables updates to the source repo to trigger builds, and updates to the config repo to trigger deployments in Enterprise Application Service for Java. You will see the following panel:

    <div align="center">
        <img src="./images/saas-instance-intro.png">
    </div>

    Click **Install and configure** to continue.

    <div align="center">
        <img src="./images/saas-build-deploy-run.png">
    </div>

    Click the **Build, Deploy & Run** option. (Note, the Deploy & Run option allows you to build your application in an existing CI pipeline and deploy it with Enterprise Application Service for Java).

    This will start a series of steps for you to setup and authorize access to your GitHub account and your application repos (**source** and **config**).

    Click `Next`.

1. Authorize access to GitHub.
   
    <div align="center">
        <img src="./images/saas-authorize-github.png">
    </div>

    Click **Authorize access to GitHub** and enter your GitHub account information. 
    
    <div align="center">
        <img src="./images/saas-github-authorize.png">
    </div>
       
    Once authorized, click **Next**.

    <div align="center">
        <img src="./images/saas-github-authorized.png">
    </div>

1. Select an organization

    <div align="center">
        <img src="./images/saas-select-org.png">
    </div>

    Select the **techxchange2024** organization and click **Next**.

    >**NOTE**: To use Enterprise Application Service, all GitHub repos must be assigned to a GitHub Organization.

1. Assign GitHub source and config repos

    Select your student assigned source GitHub repository and click **Next**.

    <div align="center">
        <img src="./images/saas-select-source-repo.png">
    </div>

    Select your student assigned configuration GitHub repository and click **Next**.

    <div align="center">
        <img src="./images/saas-select-config-repo.png">
    </div>

1. Confirm GitHub settings

    <div align="center">
        <img src="./images/saas-config-summary.png">
    </div>

    Confirm your settings and click **Finish**.

1. Navigate the Enterprise Application Service console

    Once confirmed, you will see the Enterprise Application Service console.

    <div align="center">
        <img src="./images/saas-home-screen.png">
    </div>

    Use the icon in the upper left to show the menu options.

    <div align="center">
        <img src="./images/saas-home-screen-with-menu.png">
    </div>

    Note that the **Builds** menu options are related to your **source** code repo, and the **Configuration jobs** menu options are related to your **config** repo.

### Enterprise Application Service GitHub flow

Once you configured your source repo, a **Release build** was automatically started.

1. From the menu, select **Builds** -> **Release builds**.

    <div align="center">
        <img src="./images/saas-release-build-list.png">
    </div>

    Click the **Build ID** to bring up the **Build details**.

    <div align="center">
        <img src="./images/saas-release-build-log.png">
    </div>

    If successful, you will see a **Deploy to staging** button. Click the button to get instructions on how to deploy your application.

    <div align="center">
        <img src="./images/saas-deploy-instructs.png">
    </div>

    In order to deploy, you will need to update the associated **config** repo, using the **Version ID** value displayed on the instructions page.

1. Update staging config file with Release build version ID
   
    Using the editor directly in your GitHub config repo, edit the environments/staging/environment.yaml file.

    Paste the ID as the version key value. [Note: keep the double quotes]

    <div align="center">
        <img src="./images/github-config-change.png">
    </div>

    After editing the file, click the **Commit changes...** button.

    In the **Propose changes** panel, make sure you select to **Create a new branch** option.

    <div align="center">
        <img src="./images/github-propose-changes.png">
    </div>

    Click **Propose changes** then click the button to create a Pull Request.

    Creating the Pull Request will trigger Enterprise Application Service to run **Configuration validation** job. This can be viewed by selecting the **Configuration jobs** -> **Configuration validations** menu option.

    <div align="center">
        <img src="./images/saas-config-validation.png">
    </div>

    Your GitHub PR panel will show that it is in a **Checks** phase until the validation check is complete. 

### Promote to staging environment and run application

Once the validation has completed successfully, merge the Pull Request into the **Main** branch of your repo.
    
Note: using Pull Requests is the recommended workflow.  This means you can have a peer review the changes and make the decision on whether they are correct and ready to merge.  Enterprise Application Service also has the same option for code changes.  Creating a Pull Request for code changes will trigger a Pull Request build where you can validate that the application builds and passes the tests (not just locally!). Merging the Pull Request triggers Enterprise Application Service to run a Release build, which we saw happen when the source repository was first configured. 

The PR merge will trigger Enterprise Application Service to run a **Deployment job** to deploy the Release into a Staging environment. The progress can be viewed by selecting the **Configuration jobs** -> **Deployments** menu option. [Note that this step may take a few minutes].

<div align="center">
        <img src="./images/saas-config-deploy.png">
    </div>

Upon succesful completion of the **Deployment**, Enterprise Application Service will automatically start the application in the **Staging** environment.

1. Click **Environments** -> **Staging** to view the Staging environment.

    <div align="center">
        <img src="./images/saas-staging.png">
    </div>

    Note that the **Release build** link should take you to the release build that was deployed to Staging, and the **Deployment job** link will take you to the deployment job that deployed the release to Staging.

    At the bottom of the panel is a build log. For our ModResorts app, you can see here that it was successfully started:

    <div align="center">
        <img src="./images/saas-staging-log.png">
    </div>

1. To view the running application, click on **Actions** -> **Open application**.

    Initially, you will get a `404` error because a context path must be provided.

    Add **/resorts** to the end of the URL to provide the proper path.

    <div align="center">
        <img src="./images/saas-modresorts.png">
    </div>

### Modify application source code

Now that you know how the process works, let's make a change to the source repo to see that part of the workflow.

1. Using the editor directly in your GitHub **source** repo, make a simple change to the **README.md** file. 

    After editing the file, click the **Commit changes...** button.

    In the **Propose changes** panel, make sure you select to **Create a new branch** option.

    <div align="center">
        <img src="./images/github-propose-changes.png">
    </div>

    Then click the button to create a **Pull Request**.

2. This will trigger a **PR build** in Enterprise Application Service. This can be viewed by selecting the **Builds** -> **PR builds** menu option.  

    <div align="center">
        <img src="./images/saas-pr-build.png">
    </div>

    Your GitHub PR panel will show that it is in a **Checks** phase until the PR build is complete. 

    <div align="center">
        <img src="./images/github-pull-request.png">
    </div>

    Once the PR build is complete, merge and confirm your pull request into the Main branch of your repo.

3. The PR merge will generate a **Release build** event in the EASeJ console. This can be viewed by selecting the **Builds** -> **Release builds** menu option.

    <div align="center">
        <img src="./images/saas-release-build.png">
    </div>

    Click on the **Build ID** number to bring up details on the build.

    <div align="center">
        <img src="./images/saas-release-build-log.png">
    </div>

    Once completed, it will display a status label to indicate if it **Succeeded** or **Failed**.

    If errors occurred, the **Build log** can help you track the issue.

    Click on the **Downloads** tab to see the assets generated by the build. You will see that one of the files is a SLSA provenance json.  This provides information about the artefacts and build system used for the application and is useful to help protect against supply chain security exploitation attacks.

    <div align="center">
        <img src="./images/saas-release-build-downloads.png">
    </div>

### Deploy the new version of the app

Once the application changes have been merged and built, we need to update the Staging **config** to deploy the new release.

The steps required will follow the same flow as described above.

>**NOTE**: **[Service]** and **[GitHub]** indicates which UI you will be using to perform the step, where **[Service]** is the Enterprise Application Service console and **[GitHub]** is the GitHub UI.

1. **[Service]** Click the **Deploy to staging** button.
   
2. **[Service]** Copy the **Version ID** value.
   
3. **[GitHub]** Edit the **environments/staging/environment.yaml** file in your GitHub **config** repo.
   
4. **[GitHub]** Paste the ID as the **version** key value. [Note: keep the double quotes]
   
5. **[GitHub]** Commit the change using the **Create a new branch** option.
   
6. **[GitHub]** Click the button to create a **Pull Request**.
   
7. **[Service]** The pull request will trigger a **Config validation** job. View the validation by selecting the **Configuration jobs** -> **Configuration validations** menu option.
   
8. **[GitHub]** Once complete, merge and confirm your Pull Request into the Main branch of your GitHub **config** repo.
   
9. **[Service]** Merging the Pull Request will trigger a **Deployment**. View the deployment by selecting the **Configuration jobs** -> **Deployments** menu option.
    
10. **[Service]** After successful completion of the **Deployment**,  Enterprise Application Service will automatically start the application in the **Staging** environment.
    
11. **[Service]** Click **Environments** -> **Staging** to view the staging environment.
    
12. **[Service]** Click on **Actions** -> **Open application** to view the application. [Don't forget to add the **/resorts** context to the URL]

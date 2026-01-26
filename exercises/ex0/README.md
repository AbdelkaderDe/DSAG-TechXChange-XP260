# Environment Setup and Initial Deployment

## 🚀 Quick Start

* **Duration:** 2 hours hands-on workshop
* **What you'll build:** A simple SAP BTP application with security features
* **Prerequisites:** Web browser, SAP BTP Trial Account
* **Setup time:** ~30 minutes
* **Start here:** [Step 1: Set Up Your Trial Account](#step-1-set-up-your-trial-account)

## Set up Process flow Diagram
  <p align="center">
    <img src="images/setup-process-flow-diagram.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>
    
## Table of Contents

- [Overview](#overview)
  - [Business Scenario](#business-scenario)
  - [Solution Diagram](#solution-diagram)
- [Step 1: Set Up Your BTP Trial Account](#step-1-set-up-your-trial-account)
- [Step 2: Set Up Subscriptions](#step-2-set-up-subscriptions)
   - [2.1. Subscribe to SAP HANA Cloud (Trial)](#21-subscribe-to-sap-hana-cloud-trial)
   - [2.2. Subscribe to Cloud Identity Services (Trial)](#22-subscribe-to-cloud-identity-services-trial)
   - [2.3. Establish Trust with SAP Cloud Identity (Trial)](#23-establish-trust-with-sap-cloud-identity-trial)
   - [2.4. Subscribe to SAP Build Work Zone, Standard Edition (Trial)](#24-subscribe-to-sap-build-work-zone-standard-edition-trial)
- [Step 3: Provision SAP HANA Cloud Service](#step-3-provision-sap-hana-cloud-service)
- [Step 4: Configure Custom SAP Cloud Identity](#step-4-configure-custom-sap-cloud-identity)
- [Step 5: Configure User Access (Role Collections & Platform Roles)](#step-5-configure-user-access-role-collections--platform-roles)
- [Step 6: Launch BAS, Import Project, and Deploy to Cloud Foundry](#step-6-launch-bas-import-project-and-deploy-to-cloud-foundry)
- [Step 7: Set Up SAP Build Work Zone](#step-7-set-up-sap-build-work-zone)

## Set up Process flow Diagram
  <p align="center">
    <img src="images/solution-diagram-incident-management.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

## Overview 

In these hands-on exercises, we will be using the Incident Management Application, which is designed as a reference application for the [SAP BTP Developer's Guide](https://help.sap.com/docs/btp/btp-developers-guide/btp-developers-guide). It showcases best practices for developing applications on SAP Business Technology Platform (SAP BTP).

- ⚠️ **Note:** To keep this page open, right-click each link and choose **"Open link in new tab"**.

### Business Scenario
ACME, a leading electronics company, uses this application to manage customer service incidents. The application supports the following business process:
  1. A customer contacts ACME's call center with an issue.
  2. A call center representative (processor) receives the call.
  3. The representative creates a new incident in the system based on the customer's complaint.
  4. The conversation details are recorded as part of the incident.

### Solution Diagram
The solution diagram illustrates the key components and their interactions within the Incident Management Application deployed on SAP Business Technology Platform (SAP BTP).

  <p align="center">
    <img src="images/solution-diagram-incident-management.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

## Step 1: Set Up Your Trial Account

1. Navigate to the [SAP BTP Trial Sign-Up/Login Page](https://account.hanatrial.ondemand.com/).
2. If you don't have a trial account:
    - Click Sign Up and follow the prompts to create your account using your email.
    - Verify your email address to complete registration.
3. Select your preferred region (e.g., US East, Singapore) when prompted.
  <p align="center">
    <img src="images/btp-trial-regions.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>
4.  Click on the subaccount tile (typically labeled trial) to open the SAP BTP Cockpit for your trial subaccount.
  <p align="center">
    <img src="images/trial-global-account.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

## Step 2: Set Up Subscriptions

In this step, you will ensure the necessary applications are subscribed to.

| Application                           | Subscription Plan             | Purpose             |
| :------------------------------       | :--------------- | :---------------    |
| [SAP HANA Cloud](https://discovery-center.cloud.sap/protected/index.html#/serviceCatalog/sap-hana-cloud/?region=all)              | tools             |Provides the database administration tools.|
| [Cloud Identity Services](https://discovery-center.cloud.sap/serviceCatalog/business-application-studio/?region=all)       | default |Manages user authentication|
| [SAP Build Work Zone, standard edition](https://discovery-center.cloud.sap/serviceCatalog/sap-build-work-zone-standard-edition/?region=all) | standard         |The Launchpad where you will access your deployed app.|


  ⚠️ **Note:** 
  - Your Trial account comes pre-configured with [SAP Business Application Studio](https://discovery-center.cloud.sap/serviceCatalog/business-application-studio/?region=all) (subscribed) and the
    [Cloud Foundry Environment](https://discovery-center.cloud.sap/protected/index.html#/serviceCatalog/cloud-foundry-runtime?region=all) (enabled). You do not need to add these manually.
  - Before subscribing to SAP Build Work Zone, standard edition, you must establish trust between your BTP subaccount and an SAP Cloud Identity Services – Identity Authentication (IAS) tenant using OpenID Connect (OIDC).

### 2.1. Subscribe to SAP HANA Cloud (Trial)
  1. From your Trial Subaccount (Cockpit), look at the navigation menu on the left.
  2. Click on **Service Marketplace**.
  3. Search for **"SAP HANA Cloud"** and select it from the results.
  4. Click the three-dot menu **(...)** next to the service name, then choose Create.
  5. In the Create Subscription wizard:
      * Confirm Service is set to "SAP HANA Cloud"
      * Select **Subscription Plan: tools** (free Trial plan)
      * Click **Create**.
  6. After subscription complete, Confirm **Status = "Subscribed"**

### 2.2. Subscribe to Cloud Identity Services (Trial)
Trial accounts have a pre-linked Identity Authentication (IAS) tenant, so subscription and instance setup are simplified:
  1. Return to the **Service Marketplace** in your subaccount.
  2. Search for **"Cloud Identity Services"** and select it from the results.
  3. Click the three-dot menu **(...)** next to the service name, then choose Create.
  4. In the Create Subscription wizard:
      * Confirm Service is set to **"Cloud Identity Services"**
      * Select **Subscription Plan: default**.
      * Click **Create** to initiate the subscription.
  5. After subscription complete, Confirm **Status = "Subscribed"**
  6. Activate your IAS Administration Console access via email:
      * Check your registered email inbox (including spam/junk folders) for an activation message from SAP Cloud Identity Services.
      * Click the activation link in the email, follow the prompts to set a secure password, and log into the Identity Authentication Administration Console to confirm access. This step is optional for Trial trust setup but required for advanced user management later, including adding business users to access your SAP Build Work Zone and deployed applications.

  <p align="center">
    <img src="images/cloud-identity_service_dashboard.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

### 2.3. Establish Trust with SAP Cloud Identity (Trial)
Establishing trust allows SAP Cloud Identity Services to act as your central identity provider, enabling secure Single Sign-On (SSO) and centralized management of business users. This connection is a technical requirement for services like SAP Build Work Zone to authenticate users and correctly assign the role collections needed to access applications.

  1. **Navigate to Trust Configuration:** In your BTP subaccount, go to **Security > Trust Configuration**.
  2. **Initiate Trust Setup:** Click the **Establish Trust** button.
  3. **Choose Tenant:** In the wizard, select your pre-linked SAP Cloud Identity tenant and click **Next**.
  4. **Complete and Review:** Follow the remaining steps for **"Configure Main Information"** and **"Configure Identiy provider and Parameters"**, keep the default values in all steps and click on next, then click **Finish** to activate the trust.

  <p align="center">
    <img src="images/IAS-trust configuration.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

### 2.4. Subscribe to SAP Build Work Zone, Standard Edition (Trial)
  1. Return to the **Service Marketplace** in your subaccount.
  2. Search for **"SAP Build Work Zone, standard edition"** and select it.
  3. Click the three-dot menu **(...)** next to the service name, then choose **Create**.
  4. In the Create Subscription wizard:
      * Confirm Service is set to "SAP Build Work Zone, standard edition"
      * Select **Subscription Plan: standard**
  5. After subscription complete, Confirm **Status = "Subscribed"**

At the conclusion of Step 2: Set Up Subscriptions, your Subscriptions tab will display the complete list of successfully subscribed required services, exactly as illustrated in the following 
image:

  <p align="center">
    <img src="images/btp-trial-subscriptions.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

- Access your SAP BTP account for the session XP260 using this link: [Global Account: SAP-TechEd-2025 – Account Explorer](https://emea.cockpit.btp.cloud.sap/cockpit?idp=akihlqzx8.accounts.ondemand.com#/globalaccount/4c772782-0751-42ee-93c3-897452fdcb63/accountModel&//?section=HierarchySection&view=TreeTableView).

- ⚠️ **Note:** You will use an SAP BTP subaccount with subaccount admin privileges. We use the Identity Authentication service tenant **akihlqzx8.accounts.ondemand.com** as custom identity provider, both for platform and application users.

- Login to open your subaccount **XP260_0XX**, where **XX** is your seat number: 
  - Username: **xp260-0XX@education.cloud.sap** ( with XX depending on your seat from 01 - 40 )
  - Password: Will be given to you as part of the session.
  - In the list of directories and subaccounts, click on the entry for your subaccount.

### Review the Subscribed Services and Instances 

#### Subscriptions

| Application                           | Plan             |
| :------------------------------       | :--------------- |
| [Audit Log Viewer Service](https://discovery-center.cloud.sap/serviceCatalog/audit-log-service/?region=all)              | free             |
| [SAP Business Application Studio](https://discovery-center.cloud.sap/serviceCatalog/business-application-studio/?region=all)       | standard-edition |
| [SAP Build Work Zone, standard edition](https://discovery-center.cloud.sap/serviceCatalog/sap-build-work-zone-standard-edition/?region=all) | standard         |

#### Instances

| Instance Name                       | Service                                    | Plan        |
| :------------------------------     | :-------------------------------------     | :---------- |
| incident-management-auth            | [Authorization and Trust Management Service](https://discovery-center.cloud.sap/serviceCatalog/authorization-and-trust-management-service/?region=all) | application |
| incident-management-db              | [SAP HANA Schemas & HDI Containers](https://help.sap.com/docs/hana-cloud-database/sap-hana-cloud-sap-hana-database-deployment-infrastructure-hdi-reference/sap-hdi-containers)          | hdi-shared  |
| incident-management-destination     | [Destination Service](https://discovery-center.cloud.sap/serviceCatalog/destination-service/?service_plan=lite&region=all&commercialModel=btpea)                        | lite        |
| incident-management-html5-repo-host | [HTML5 Application Repository Service](https://help.sap.com/docs/btp/sap-business-technology-platform/html5-application-repository)       | app-host    |
| incident-management-html5-runtime   | [HTML5 Application Repository Service](https://help.sap.com/docs/btp/sap-business-technology-platform/html5-application-repository)       | app-runtime |

#### Environments

| Environment Name    | Plan       |
| :------------------ | :-------   |
| [Cloud Foundry Runtime](https://discovery-center.cloud.sap/protected/index.html#/serviceCatalog/cloud-foundry-runtime?region=all) | standard |

- Check the subscriptions, instances, and the environment under subaccount **XP260_0xx > Instances and Subscriptions** in the **SAP BTP cockpit**.
 
  <p align="center">
    <img src="images/btp-subaccount-instances-subscriptions.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

### Review the Configured User Access:

- ⚠️ **Note:** In this workshop, your account **xp260-0xx@education.cloud.sap** fulfills two key roles: as a **platform administrator** for configuring role collections and infrastructure, and as a **business application user** for developing and testing secure Incident Management features. This duality simplifies the training environment by eliminating constant re-logins during exercises, but in real-world systems, strict role separation ensuring users hold only necessary permissions—is essential.

1- Check the users under **Security > Users**.

2- Check your user **xp260-0XX@education.cloud.sap**. There will be two representations in the cockpit, one as **platform user** and one as **business user**. 

3- Check **Role Collections** for the Business User
  - Locate and click on the business user representation in the user management interface.
  - Open the Role Collection Details in the right frame.
  - Confirm the Following Role Collections Are Assigned :
      - **Business_Application_Studio_Administrator**
      - **Business_Application_Studio_Developer**
      - **Business_Application_Studio_Extension_Deployer**
      - **Launchpad_Admin**
      - **Subaccount Viewer**

  <p align="center">
    <img src="images/btp-configured_user_platform.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

4- To demonstrate real-world access control, you'll test the application using dedicated accounts with **precisely scoped** role collections. Unlike your **xp260-0xx@education.cloud.sap** training account's broad privileges, these users showcase how proper role assignments enforce **least privilege** in production:
  - **bob.support@company.com** (Support user)
  - **alice.support@company.com** (Support user)
  - **david.admin@company.com** (Admin user)
  - Check the user role collections in the SAP BTP cockpit for **Bob, Alice, and David**
  - Select a user. In the right frame, check the role collections assigned:
      - Check if **bob.support@company.com** and **alice.support@company.com** are assigned to the role collection **Incident Management Support**. 
      - Check if **david.admin@company.com** is assigned to the role collection **Incident Management Admin**.

 <p align="center">
    <img src="images/btp-configured_user_application.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

## Review the Development Environment
 
1- As we are using Cloud Foundry, check under **Cloud Foundry > Org Members**, if your platform user **xp260-0XX@education.cloud.sap** has **Org Manager, Org User** under **Org Roles**. 

 <p align="center">
    <img src="images/btp-subaccount-org-members.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

2- Under **Cloud Foundry > Spaces**, verify the existence of your space called **'XP260-0XX'**.
  
 <p align="center">
    <img src="images/btp-subaccount-space.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

3- Under Space **XP260-0XX > Space Members**, verify if your platform user **xp260-0XX@education.cloud.sap** has **Space Developer, Space Manager** under **Space Roles**.

 <p align="center">
    <img src="images/btp-subaccount-space-members.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

## Launch SAP Business Application Studio

Now after these checks, you can open the SAP Business Application Studio. 

1- Navigate to **Services > Instances and Subscriptions** in the **SAP BTP cockpit**. Then click on the small **Go to Application** icon next to the name SAP Business Application Studio.

 <p align="center">
    <img src="images/btp-subaccount-open-BAS-application.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

2- On the logon screen, click on the custom identity provider (IDP) **akihlqzx8.accounts.ondemand.com** to login with single sign-on (SSO). 
- ⚠️ **Note:**
    - If prompted for an **Origin Key**, enter: **akihlqzx8-platform**.
    - Always select the Custom Identity Provider **akihlqzx8.accounts.ondemand.com** when authenticating during exercises.
 <p align="center">
    <img src="images/btp-subaccount-open-BAS-SSO.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>

3- Enter email and password in the login window.
- ⚠️ **Note:** You may not see this step if you're already authenticated through SSO.

 <p align="center">
    <img src="images/btp-subaccount-open-BAS-login-windows.png" alt="" width="450"/>
    <br>
    <b></b>
  </p>

4- You will see your Dev Space called 'secure_incident_management'. Make sure it is in a **RUNNING** state, if not start it.

 <p align="center">
    <img src="images/btp-subaccount-open-BAS-dev-space-stopped.png" alt="" width="900"/>
    <br>
    <b></b>
  </p>
  
5- When it is running, click on **secure_incident_management** to open the SAP Business Application Studio with your incident management application.

 <p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-space-running.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

6- Click on **incident-management** to open the application in your workspace.

 <p align="center">
  <img src="images/btp-subaccount-open-BAS-open-secure-incident-management.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

- In the workspace on the right side, you will find your incident management application in the list of projects.
 <p align="center">
  <img src="images/btp-subaccount-open-BAS-folder-explorer.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

7- **Bookmark your SAP Business Application Studio link**.

## Login to Your Cloud Foundry Environment from SAP Business Application Studio
Once you have SAP Business Application Studio open with your secure incident management project, you need to authenticate with your Cloud Foundry environment to deploy and manage applications. Choose one of the following methods to log in:

### 1. Login Using the User Interface (UI)

1. In SAP Business Application Studio, open the **Command Palette** (press **Ctrl+Shift+P** or select **View > Command Palette**) from the top menu.

<p align="center">
  <img src="images/btp-subaccount-open-BAS-command-palette.png" alt="" width="900"/>
  <br>
  <b></b>
</p>
  
2. Search for and select **CF: Login to Cloud Foundry**.

<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-command-palette-cf-login.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

3. You’ll see **Cloud Foundry Sign In and Targets**. Select **SSO Passcode** and click on the link **Open New Browser** to generate your SSO Passcode.
<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-command-cf-signIn-target.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

4. Select **Sign in** with your user **xp260-0XX@education.cloud.sap (origin: akihlqzx8-platform)**.
- ⚠️ **Note:** If your account is not displayed, click **Sign in to another account**.

<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-command-cf-signIn-idp.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

5. You’ll see a passcode page, copy the temporary authentication code generated in the **Passcode** field.
<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-command-cf-temp-code.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

6. Paste the **SSO Passcode** back into the SAP Business Application Studio and click on the **Sign In** button.
<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-command-cf-paste-code.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

7. In section **Cloud Foundry Target**, select **Organization** and **Space** (for example, `XP260-0XX`), then click on the **Apply** button.
<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-select-cf-target.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

8. Once connected, a notification message pops up in the status bar in the SAP Business Application Studio confirming that your Cloud Foundry organization and space have been set and are ready for use.

<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-UI-login-message.png" alt="" width="900"/>
  <br>
  <b></b>
</p>
  
### 2. Login Using the Command Line (Terminal)

1. **Open Terminal**
   - In the SAP Business Application Studio, go to **Terminal > New Terminal** from the top menu.
   - A terminal window will open at the bottom of your workspace in your project directory **secure-incident-management**.

<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-open-terminal.png" alt="" width="900"/>
  <br>
  <b></b>
</p>
   
2. **Run the following command to log in:**
  ```
  cf login -a https://api.cf.eu10-004.hana.ondemand.com  --origin akihlqzx8-platform
  ```
3. When prompted enter:
    * Email: xp260-0XX@education.cloud.sap
    * Password: Use the password provided during the session.
    
4.  To verify the login, run
  ```
    cf target
  ```
5- You should see the current organization and space listed.
  
<p align="center">
  <img src="images/btp-subaccount-open-BAS-dev-cf-target-message.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

## Launch SAP Build Work Zone

1- Go back to **Services > Instances and Subscriptions** in the **SAP BTP cockpit**. Click on the **Go to Application** icon next to the **SAP Build Work Zone, standard edition** subscription to open the SAP Build Work Zone Site. 

<p align="center">
  <img src="images/btp-subaccount-open-SAP-Build-Work-Zone.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

2- Check if the **Incident Management** Site is present. Click on the **Go to site** icon. 

<p align="center">
  <img src="images/btp-subaccount-open-SAP-Build-Work-Zone-Site.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

3- When the **Incident Management tile** is displayed, **Sign Out** from your current user **XP260-0xx@education.cloud.sap** and login to the application with the **alice.support@company.com** user. 

<p align="center">
  <img src="images/btp-subaccount-open-SAP-Build-Work-Zone-sign-out.png" alt="" width="900"/>
  <br>
  <b></b>
</p>

4- Click on the tile to open the incident management application and **bookmark the URL**. 

<p align="center">
  <img src="images/btp-subaccount-open-SAP-Build-Work-Zone-open-incident-management.png" alt="" width="900"/>
  <br>
  <b></b>
</p>


Now you are ready to start the exercises. 


## Summary

Now that you have made yourself familiar with the setup,
continue to - [Exercise 1 - Broken Access Control](../ex1/README.md)

@lab.Title

**Please enter your own Microsoft alias (without the @microsoft.com) in the field below before starting the lab**. This information is required to correctly associate your activity with your learner record and ensure your completion is accurately captured and reported. For data integrity and compliance reasons, learners may only enter their own alias.   After completing the lab, please allow up to 5 business days for all reporting systems to fully reflect your completion.

@lab.ActivityGroup(aliascapture)

===


## Survey

@lab.ActivityGroup(initialsurvey)

===

## Introduction
Declarative agents are a type of agent for Microsoft 365. You can build one by extending Microsoft 365 Copilot. You define custom knowledge and custom actions to create agents tailored to a specific scenario.

Declarative agents use the same infrastructure, orchestrator, foundation model, and security controls as Microsoft 365 Copilot, which ensures a consistent and familiar user experience.

## Objectives

After completing this lab, you'll be able to:

- Run and test an Azure Functions-based REST API locally using the Azurite storage emulator.
- Verify API behaviour using HTTP test requests in VS Code.
- Create a declarative agent backed by an API plugin and SharePoint knowledge.
- Configure a declarative agent manifest with instructions, conversation starters, and capabilities.
- Extend an existing API with new resource paths and register them in the application package.
- Update the OpenAPI Specification and plugin definition files to expose new functions to Copilot.
- Integrate a Microsoft 365 Copilot Connector as a knowledge capability in a declarative agent.

## Duration

**Estimated time**: 120 minutes

### Declarative agent architecture diagram 

!IMAGE[c5bvc5a2.jpg](instructions337966/c5bvc5a2.jpg)

At the very basis is the foundational model of Microsoft 365 Copilot, as well as the same orchestrator. The agent provides custom knowledge, grounding data, and custom skills as actions, triggers, and workflows. 

===

# Exercise 01: Build a backend API

In this exercise, you'll set up an API based on Azure Functions and install it as an API plugin for Microsoft 365 Copilot.

<!-- <div class="lab-intro-video">
    <div style="flex: 1; min-width: 0;">
        <iframe  src="//www.youtube.com/embed/XO2aG3YPbPc" frameborder="0" allowfullscreen style="width: 100%; aspect-ratio: 16/9;">          
        </iframe>
    </div>
</div> -->

## Introduction

In this exercise, you'll set up a REST API for Trey Research, a hypothetical consulting company. It provides API's for accessing information about consultants (using the **/api/consultants** path) and about the current user (using the **/api/me** path). For now, the API doesn't support authentication, so the current user will always be a dummy user, Avery Howard.

The code consists of Azure Functions written in TypeScript, backed by a database in Azure Table storage. When you run the app locally, table storage will be provided by the Azurite storage emulator.

>[!help] **How was this API created?**
>
>The project was created using the Microsoft 365 Agents Toolkit. You can create the same scaffolding for your own project by opening an empty folder in VS Code and going to Agents Toolkit. [Read more](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/build-declarative-agents).

===

## Task 01: Run the application

### Description

You'll open the Trey Research project in Visual Studio Code, sign in to the Microsoft 365 Agents Toolkit, and start the debugger to run the Azure Functions API locally.

### Success criteria

- You signed in to your Microsoft 365 account through the Agents Toolkit.
- The debugger started successfully and Microsoft Edge opened a sign-in window.
- You minimized Edge without signing in, leaving the debugger running.

### Key steps

---
1. [] Sign in to the lab VM with the following credentials:

	| Item     | Value                                                |
	|:---------|:---------|
	| Username | **@lab.VirtualMachine(Windows11(25H2)).Username**       |
	| Password | **+++@lab.VirtualMachine(Windows11(25H2)).Password+++** |

1. [] Open **Visual Studio Code**.

    >[!note] This will open a project folder with a declarative agent located in `C:\Lab Files\lab-build-api`.

1. [] In the leftmost pane, select the **Microsoft 365 Agents Toolkit** icon.

    !IMAGE[w889l2kw.jpg](instructions337966/w889l2kw.jpg)

1. [] In the **MICROSOFT 365 AGENTS TOOLKIT** pane, under the **ACCOUNTS** section, select **Sign in to Microsoft 365**.

1. [] In the dialog, select **Sign in**.

1. [] In the **Windows Security** dialog, select **Allow**.

1. [] Sign in with your lab credentials:

    | Item | Value |
    |:---------|:---------|
    | Username | `@lab.CloudPortalCredential(User1).Username` |
    | Password | `@lab.CloudPortalCredential(User1).AccessToken` |

1. [] Close the browser to return to Visual Studio Code.

1. [] In the top menu bar, select **Run**, then **Start Debugging**.

    !IMAGE[i5y9cfzw.jpg](instructions337966/i5y9cfzw.jpg)

    >[!alert] This may take a couple minutes to start.

1. [] In the **Windows Security** dialog, select **Allow**.

    >[!note] Once started, Microsoft Edge will open a sign in window.

1. [] Minimize Microsoft Edge without signing in.
    
    >[!alert] Do not close the window as that will stop the debugger.

===

## Task 02: Test the app's web services

### Description

You'll use the `.http` test file included in the project to send requests directly to the running API, exploring the available endpoints and verifying their responses.

### Success criteria

- You sent a GET request to `/api/me` and received a JSON response for the fictitious user Avery Howard.
- You charged 3 hours to the Woodgrove Bank project and confirmed the updated `deliveredThisMonth` value in a follow-up `/api/me` request.
- You sent additional GET requests to explore consultant data by skill, role, and availability.
- You stopped the debugger and closed the open files.

> [!NOTE]
> The Trey Research project is an API plugin, so it includes an API. In this task, you'll test the API manually to understand what it does before connecting it to an agent.

### Key steps

---


#### 01: Get the **/me** resource

1. [] In VS Code's leftmost pane, select the **Explorer** icon.

    !IMAGE[lca3jonk.jpg](instructions337966/lca3jonk.jpg)

1. [] In the **EXPLORER** pane, expand the **http** folder, then select **treyResearchAPI.http**.

    !IMAGE[g1w9sn8u.jpg](instructions337966/g1w9sn8u.jpg)

1. [] In the bottom pane, select the **DEBUG CONSOLE** tab.

    !IMAGE[q34m3eph.jpg](instructions337966/q34m3eph.jpg)

    >[!note] **Attach to Backend** is selected by default.

1. [] In the **treyResearchAPI.http** file, select **Send Request** below line 5, for **{{base_url}}/me**.

	!IMAGE[3xvlcsuf.jpg](instructions337966/3xvlcsuf.jpg)

    >[!note] You'll see a response in the right pane, with a log of the request in the bottom pane. The response shows the information about the logged-in user, but since we haven't implemented authentication yet, the app will return information on the fictitious consultant Avery Howard. 
    
1. [] Observe the response to see details about Avery, including a list of project assignments.

    !IMAGE[rhd05xo6.jpg](instructions337966/rhd05xo6.jpg)

===

#### 02: Try the other methods and resources

1. [] In **treyResearchAPI.http**, select **Send Request** below line 8.

    >[!note] This will charge 3 hours of Avery's time to the Woodgrove Bank project. This is stored in the project database, which is a locally hosted emulation of Azure Table Storage, so the system will remember that Avery has delivered these hours.

1. [] Verify the charge. Below line 5, select **Send Request** again for **{{base_url}}/me**. 

1. [] In the results, observe the **"deliveredThisMonth"** property under the Woodgrove Bank project.

    !IMAGE[1c47x5pw.jpg](instructions337966/1c47x5pw.jpg)

1. [] Try selecting **Send Request** for the various GET requests in the .http file to find consultants with various skills, certifications, roles, and availability. All the information will be available to your agent to answer user prompts.

1. [] In the top menu bar, select **Run**, thne **Stop Debugging**.

	!IMAGE[9qa4jsqp.jpg](instructions337966/9qa4jsqp.jpg)

1. [] Close the tabs for **treyResearchAPI.http** and the **Response** pane.

===

#### 03: Examine the database (optional)

1. [] On the VM's taskbar, open Microsoft Azure Storage Explorer.

    !IMAGE[eq6mzmiy.jpg](instructions337966/eq6mzmiy.jpg)

1. [] In the **EXPLORER** pane, expand **Storage Accounts**, select **(Emulator - Default Ports) (Key)**, then **Tables**.

    !IMAGE[klo0ti8e.jpg](instructions337966/klo0ti8e.jpg)

    >[!note] You can examine and modify the application's data. The data is stored in Azure Table Storage, which in this case is running locally using the Azurite emulator.
    >
    >Your lab came with the Azurite storage emulator pre-installed with the project folder. For more information check the [Azurite documention here](https://learn.microsoft.com/azure/storage/common/storage-use-azurite).
    >
    >When you start the project, Azurite is automatically started up. As long as your project is started successfully, you can view the storage.

1. [] Observe the three tables.

    >[!note]    
    >   - **Assignment:** This table stores consultant assignments to projects, such as Avery Howard's assignment to the Woodgrove Bank project. This table includes a **delivered** field that contains a JSON representation of the hours delivered by that consultant on the project over time.
    >   - **Consultant:** This table stores details about Trey Research consultants.
    >   - **Project:** This table stores details about Trey Research projects.

1. [] Close **Storage Explorer**.

---

**Congratulations!**

You've successfully built the lab sample API! You can now proceed to make it into a plugin and expose it via a declarative agent.

===

# Exercise 02: Add a declarative agent and API plugin


## Introduction

In this exercise, you'll create a declarative agent grounded in the Trey Research API plugin and a SharePoint document library. You'll upload sample consulting documents to a new SharePoint site, write the declarative agent JSON, configure environment variables, and register the agent in the Teams app manifest.


<!-- <div class="lab-intro-video">
    <div style="flex: 1; min-width: 0;">
        <iframe  src="//www.youtube.com/embed/XO2aG3YPbPc" frameborder="0" allowfullscreen style="width: 100%; aspect-ratio: 16/9;">          
        </iframe>
    </div>
</div> -->

You'll continue working from the same project folder. 

Reference: 
- [Exercise 02 Solution](https://github.com/microsoft/copilot-camp/tree/main/src/extend-m365-copilot/path-e-lab03-build-declarative-agent/trey-research-lab03-END)

===

## Task 01: Upload sample documents

### Description

You'll create a new SharePoint Team site and upload the sample consulting documents included in the project. The declarative agent will use these documents as grounding knowledge when answering prompts.

### Success criteria

- You created a SharePoint Team site named `Trey Research Legal Documents` with the correct group email address and site address suffix.
- You uploaded the `sampleDocs` folder from `C:\Lab Files\lab-build-api` to the site's **Documents** library.

### Key steps

---

#### 01: Create a SharePoint site

1. [] Open Microsoft Edge, then go to `lodsprodmca.sharepoint.com/_layouts/15/sharepoint.aspx`.

1. [] Sign in with your lab credentials:

    | Item | Value |
    | ---- | ----- |
    | Username | `@lab.CloudPortalCredential(User1).Username` |
    | Temporary Access Pass (TAP) | `@lab.CloudPortalCredential(User1).AccessToken` |

1. [] On the top bar, select **Create site**.

    !IMAGE[lnc5y50t.jpg](instructions337966/lnc5y50t.jpg)

1. [] In the dialog, select **Team site**.

    !IMAGE[x85n19v0.jpg](instructions337966/x85n19v0.jpg)

1. [] Select **Standard team**.

    !IMAGE[cuhbe1n1.jpg](instructions337966/cuhbe1n1.jpg)

1. [] In the lower-right corner of the dialog, select **Use template**.

1. [] In the **Give your site a name** dialog, enter the following:

    | Item | Value |
    | ---- | ----- |
    | Site name | `Trey Research Legal Documents` |
    | Group email address | `TreyResearchLegalDocuments@lab.LabInstance.Id` |
    | Site address suffix | **TreyResearchLegalDocuments@lab.LabInstance.Id** |

    !IMAGE[jrtj8new.jpg](instructions337966/jrtj8new.jpg)

	>[!alert] Use the provided values, as these will be referenced again in future steps. The numbers shown in the screenshot will not match your instance. 

1. [] In the lower-right corner of the dialog, select **Next**.

1. [] Select **Create site**.

1. [] Once the site is created, select **Finish**.

===

#### 02: Upload the sample documents

1. [] In the site menu, select **Documents**.

    !IMAGE[1pkgb3a5.jpg](instructions337966/1pkgb3a5.jpg)

1. [] Near the upper-right corner of the page, select **Create or upload**, then select **Folder upload**.

    !IMAGE[kce2mckf.jpg](instructions337966/kce2mckf.jpg)

1. [] Go to `C:\Lab Files\lab-build-api`, select the **sampleDocs** folder, then select **Upload**.

    !IMAGE[imp9pox7.jpg](instructions337966/imp9pox7.jpg)

1. [] In the dialog, select **Upload**.

    !IMAGE[mc3y5irl.jpg](instructions337966/mc3y5irl.jpg)

===

## Task 02: Create the declarative agent

### Description

You'll add the declarative agent JSON file to the project, configure it to reference your SharePoint site and the Trey Research API plugin, and register it in the Teams app manifest. You'll also review the existing API plugin files to understand how they describe the API to Copilot.

### Success criteria

- You created `appPackage/trey-declarative-agent.json` with the correct schema, instructions, conversation starters, SharePoint capability, and action reference.
- You added `SHAREPOINT_DOCS_URL` to `env/.env.local` pointing to your SharePoint site.
- You reviewed `trey-definition.json` and `trey-plugin.json` and can describe their roles.
- You added the `copilotAgents` block to `manifest.json` and removed the `staticTabs` and `validDomains` sections.

### Key steps

---
#### 01: Add the declarative agent JSON to your project

1. [] Go back to **Visual Studio Code**.

1. [] Right-click the **appPackage** folder, then select **New File**.

    !IMAGE[e1brs6xz.jpg](instructions337966/e1brs6xz.jpg)

1. [] Name the file `trey-declarative-agent.json`.

1. [] Enter the following JSON into the file:

    >[!hint] Select **Copy** in the following block, then paste with **Ctrl+V**.

    ```json
    {
        "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.6/schema.json",
        "version": "v1.6",
        "name": "Trey Genie Local",
        "description": "You are a handy assistant for consultants at Trey Research, a boutique consultancy specializing in software development and clinical trials. ",
        "instructions": "You are consulting agent. Greet users professionally and introduce yourself as the Trey Genie. Offer assistance with their consulting projects and hours. Remind users of the Trey motto, 'Always be Billing!'. Your primary role is to support consultants by helping them manage their projects and hours. Using the TreyResearch action, you can: You can assist users in retrieving consultant profiles or project details for administrative purposes but do not participate in decisions related to hiring, performance evaluation, or assignments. You can assist users to find consultants data based on their names, project assignments, skills, roles, and certifications. You can assist users to retrieve project details based on the project or client name. You can assist users to charge hours to a project. You can assist users to add a consultant to a project. If a user inquires about the hours they have billed, charged, or worked on a project, rephrase the request to ask about the hours they have delivered. Additionally, you may provide general consulting advice. If there is any confusion, encourage users to consult their Managing Consultant. Avoid providing legal advice.",
        "conversation_starters": [
            {
                "title": "Find consultants",
                "text": "Find consultants with TypeScript skills"
            },
            {
                "title": "My Projects",
                "text": "What projects am I assigned to?"
            },
            {
                "title": "My Hours",
                "text": "How many hours have I delivered on projects this month?"
            }
        ],
        "capabilities": [
            {
                "name": "OneDriveAndSharePoint",
                "items_by_url": [
                    {
                        "url": "${{SHAREPOINT_DOCS_URL}}"
                    }
                ]
            }
        ],
        "actions": [
            {
                "id": "treyresearch",
                "file": "trey-plugin.json"
            }
        ]
    }
    ```

    >[!note] The file includes a name, description, and instructions for the declarative agent. Notice that as part of the instructions, Copilot is instructed to "Always remind users of the Trey motto, 'Always be Billing!'." You'll see this when you prompt Copilot in the next task.

    !IMAGE[as0btl5b.jpg](instructions337966/as0btl5b.jpg)

===

#### 02: Add the URL of your SharePoint site to the declarative agent

1. [] In the same file, under the **"capabilities"** section, you'll find a SharePoint file container using the **SHAREPOINT_DOCS_URL** environment variable.

	!IMAGE[g59c3jcj.jpg](instructions337966/g59c3jcj.jpg)

1. [] Create the environment variable. In the **EXPLORER** pane, expand the **env** folder, then select **.env.local**.

    !IMAGE[qthc7fpf.jpg](instructions337966/qthc7fpf.jpg)

1. [] On the lower part of the file, create a new line, then enter the following:

    `SHAREPOINT_DOCS_URL=https://lodsprodmca.sharepoint.com/sites/TreyResearchLegalDocuments@lab.LabInstance.Id`

    !IMAGE[r42pa7sw.jpg](instructions337966/r42pa7sw.jpg)

===

#### 03: Examine the API Plugin files

In this task, you'll look at the **trey-plugin.json** and **trey-definition.json** files to see how they describe the API to Copilot for it to make the REST calls.

1. [] Go back to **appPackage**, then select **trey-declarative-agent.json**.

1. [] Observe the **"actions"** section, which tells the declarative agent to access the Trey Research API.

1. [] Open **appPackage**, select **apiSpecificationFile**, then **trey-definition.json** and observe the file.

    !IMAGE[uze7rnfs.jpg](instructions337966/uze7rnfs.jpg)

    >[!note] This is the [OpenAPI Specification (OAS)](https://swagger.io/specification/) or "Swagger" file, which is an industry standard format for describing a REST API.
    >
    >You'll find the general description of the application. This includes the server URL; Agents Toolkit will create a [developer tunnel](https://learn.microsoft.com/azure/developer/dev-tunnels/) to expose your local API on the internet and replace the token **${{OPENAPI_SERVER_URL}}** with the public URL. It then goes on to describe every resource path, verb, and parameter in the API. Notice the detailed descriptions; these are important to help Copilot understand how the API is to be used.

1. [] Open **appPackage**, then **trey-plugin.json**.

	!IMAGE[2dzteege.jpg](instructions337966/2dzteege.jpg)

    >[!note] This file contains all the Copilot-specific details that aren't described in the OAS file.

1. [] In the **functions** section, observe the first entry for **getConsultants**.

    !IMAGE[p91dkwq6.jpg](instructions337966/p91dkwq6.jpg)

    >[!note] The file breaks the API calls down into _functions_ which can be called when Copilot has a particular use case. For example, all GET requests for **/consultant** look up one or more consultants with various parameter options, and they are grouped into a function **getConsultants**.

1. [] Move through the file to the **runtimes** section.

    !IMAGE[6wv0wg8j.jpg](instructions337966/6wv0wg8j.jpg)

    >[!note] This includes a pointer to the **trey-definition.json** file, and an enumeration of the available functions.

===

#### 04: Add the declarative agent to your app manifest

1. [] Open **appPackage**, then **manifest.json**.

    !IMAGE[etgq8vsm.jpg](instructions337966/etgq8vsm.jpg)

1. [] Create a new line after **"accentColor": "#FFFFFF",** on line 24.

1. [] Create a new **copilotAgents** object by entering the following:

    ```
    "copilotAgents": {
      "declarativeAgents": [
        {
          "id": "treygenie",
          "file": "trey-declarative-agent.json"
        }
      ]
    }, 
    ```

    >[!note] This has a **declarativeAgents** object to reference the declarative agent JSON file you created in the previous step.

1. [] Right-click anywhere in the file, then select **Format Document** to fix the spacing.

    !IMAGE[zyyrrgz7.jpg](instructions337966/zyyrrgz7.jpg)
    !IMAGE[d63kx7ua.jpg](instructions337966/d63kx7ua.jpg)

===

#### 05: Remove the dummy feature from the app manifest

The solution from Exercise 01 didn't have a declarative agent yet, so the manifest couldn't be installed because it had no features. We added a "dummy" feature: a static tab pointing to the Copilot Developer Camp home page. This allows users to view the Copilot Developer Camp site in a tab within Teams, Outlook, and the [Microsoft 365 app](https://office.com).

If you've ever tried [Teams App Camp](https://aka.ms/app-camp), this will be familiar. 

---

1. [] In **manifest.json**, delete the following **staticTabs** and **validDomains** sections as they're no longer needed:

    ```json
    "staticTabs": [
      {
        "entityId": "index",
        "name": "Copilot Camp",
        "contentUrl": "https://microsoft.github.io/copilot-camp/",
        "websiteUrl": "https://microsoft.github.io/copilot-camp/",
        "scopes": [
          "personal"
        ]
      }
    ],
    "validDomains": [
      "microsoft.github.io"
    ],
    ```

    !IMAGE[ymrk2xj6.jpg](instructions337966/ymrk2xj6.jpg)

===

## Task 03: Run and test the declarative agent

### Description

You'll start the debugger to provision and launch the **Trey Genie Local** agent in Microsoft 365 Copilot, then verify it can answer prompts by combining data from the API plugin and documents from SharePoint.

### Success criteria

- You started the debugger and the **Trey Genie Local** agent opened in Microsoft 365 Copilot.
- You sent a prompt referencing both project data from the API and a Statement of Work from SharePoint, and received a combined response with citations.
- You sent a `/api/me` prompt and confirmed the agent returned Avery Howard's profile.
- You observed the API call logged in the VS Code terminal.

### Key steps

---
1. [] Save all your changes by selecting **File**, then **Save All**.

1. [] In the top menu bar, select **Run**, then **Start Debugging**.

    >[!note] Once started, Edge will automatically open a sign in window.

1. [] Sign in with your lab credentials:

    | Item | Value |
    |:---------|:---------|
    | Username | `@lab.CloudPortalCredential(User1).Username` |
    | Password | `@lab.CloudPortalCredential(User1).AccessToken` |

    >[!note] This should launch your **Trey Genie Local** agent in M365 Copilot.

    !IMAGE[zx8ti0he.jpg](instructions337966/zx8ti0he.jpg)

    >[!alert] If you receive an error in M365 Copilot, select **Try again** or refresh the page.
    >
    >You can also find the agent in the leftmost pane. Under the **Agents** section, select **Trey Genie Local**.

1. [] Close any dialogs.

1. [] Test a prompt like the following:

    `Please list my projects along with details from the Statement of Work doc.`

    !IMAGE[j0a4mpif.jpg](instructions337966/j0a4mpif.jpg)

    >[!note] You should see a list of your projects from the API plugin, enhanced with details from each project's Statement of Work. 

1. [] If prompted, select **Allow**.

3. [] Select any of the citations in its response to check out a document.

    >[!note] Notice these are the files you uploaded to SharePoint earlier. 

    >[!alert] If the SharePoint documents aren't referenced, perhaps there is an issue accessing the files. Has there been time for Search to index the site? Does the end user have permission to the site?

1. [] Send a new prompt to instruct the agent to retrieve details from the **api/me** endpoint:

    `List my information.`

1. [] If prompted, select **Allow**.
    
    !IMAGE[w60xt6j6.jpg](instructions337966/w60xt6j6.jpg)

1. [] Observe the response. 

    !IMAGE[uls6x3b6.jpg](instructions337966/uls6x3b6.jpg)

    >[!note] As seen earlier, this currently pulls in Avery Howard as the signed in user as you haven't implemented auth yet.

1. [] Go back to VS Code and observe the terminal to see how the agent called the API.

    !IMAGE[nlofjdul.jpg](instructions337966/nlofjdul.jpg)

1. [] In the top menu bar, select **Run**, then **Stop Debugging**.

1. [] Close any open tabs in VS Code.

---

**Congratulations!**

You've completed adding a declarative agent to your API plugin. You're now ready to enhance your API and the plugin for your agent. 

===

# Exercise 03: Enhance the API and Plugin

## Introduction
In this exercise, you'll add additional REST calls to the API and add them to the API plugin packaging so Copilot can call them. In the process you'll learn all the places where an API needs to be defined.


<!-- <div class="lab-intro-video">
    <div style="flex: 1; min-width: 0;">
        <iframe  src="//www.youtube.com/embed/9kb9whCKey4" frameborder="0" allowfullscreen style="width: 100%; aspect-ratio: 16/9;">    
        </iframe>
    </div>
</div> -->

Reference: 
- [Exercise 03 Solution](https://github.com/microsoft/copilot-camp/tree/main/src/extend-m365-copilot/path-e-lab04-enhance-api-plugin/trey-research-lab04-END) 

===

## Task 01: Add /projects resource

### Description

You'll create a new Azure Function file that handles GET and POST requests for the `/projects` path, then add HTTP test requests to the `.http` file so you can verify the new endpoints manually before wiring them into the plugin.

### Success criteria

- You created `src/functions/projects.ts` with the provided code and saved the file without errors.
- You added the HTTP test block for `/api/projects` to `treyResearchAPI.http`.
- You started the debugger, sent the POST assign-consultant request, and received a successful response.


### Key steps

---

#### 01: Add Azure function code

1. [] In the **EXPLORER** pane, expand **src**, then select **functions**.

1. [] Right-click the **functions** folder, then select **New File**.

1. [] Name the file `projects.ts`.

    !IMAGE[iaqyw7jj.jpg](instructions337966/iaqyw7jj.jpg)

1. [] Enter the following code block in **projects.ts**:

    >[!hint] Select **Copy** in the following block, then paste with **Ctrl+V**.

    ```
    /* This code sample provides a starter kit to implement server side logic for your Teams App in TypeScript,
    * refer to https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference for complete Azure Functions
    * developer guide.
    */

    import { app, HttpRequest, HttpResponseInit, InvocationContext } from "@azure/functions";
    import ProjectApiService from "../services/ProjectApiService";
    import { ApiProject, ApiAddConsultantToProjectResponse, ErrorResult } from "../model/apiModel";
    import { HttpError, cleanUpParameter } from "../services/Utilities";
    import IdentityService from "../services/IdentityService";

    /**
    * This function handles the HTTP request and returns the project information.
    *
    * @param {HttpRequest} req - The HTTP request.
    * @param {InvocationContext} context - The Azure Functions context object.
    * @returns {Promise<Response>} - A promise that resolves with the HTTP response containing the project information.
    */

    // Define a Response interface.
    interface Response extends HttpResponseInit {
        status: number;
        jsonBody: {
            results: ApiProject[] | ApiAddConsultantToProjectResponse | ErrorResult;
        };
    }
    export async function projects(
        req: HttpRequest,
        context: InvocationContext
    ): Promise<Response> {
        context.log("HTTP trigger function projects processed a request.");
        // Initialize response.
        const res: Response = {
            status: 200,
            jsonBody: {
                results: [],
            },
        };

        try {

            // Will throw an exception if the request is not valid
            const userInfo = await IdentityService.validateRequest(req);

            const id = req.params?.id?.toLowerCase();
            let body = null;
            switch (req.method) {
                case "GET": {

                    let projectName = req.query.get("projectName")?.toString().toLowerCase() || "";
                    let consultantName = req.query.get("consultantName")?.toString().toLowerCase() || "";

                    console.log(`➡️ GET /api/projects: request for projectName=${projectName}, consultantName=${consultantName}, id=${id}`);

                    projectName = cleanUpParameter("projectName", projectName);
                    consultantName = cleanUpParameter("consultantName", consultantName);

                    if (id) {
                        const result = await ProjectApiService.getApiProjectById(id);
                        res.jsonBody.results = [result];
                        console.log(`   ✅ GET /api/projects: response status ${res.status}; 1 projects returned`);
                        return res;
                    }

                    // Use current user if the project name is user_profile
                    if (projectName.includes('user_profile')) {
                        const result = await ProjectApiService.getApiProjects("", userInfo.name);
                        res.jsonBody.results = result;
                        console.log(`   ✅ GET /api/projects for current user response status ${res.status}; ${result.length} projects returned`);
                        return res;
                    }

                    const result = await ProjectApiService.getApiProjects(projectName, consultantName);
                    res.jsonBody.results = result;
                    console.log(`   ✅ GET /api/projects: response status ${res.status}; ${result.length} projects returned`);
                    return res;
                }
                case "POST": {
                    switch (id.toLocaleLowerCase()) {
                        case "assignconsultant": {
                            try {
                                const bd = await req.text();
                                body = JSON.parse(bd);
                            } catch (error) {
                                throw new HttpError(400, `No body to process this request.`);
                            }
                            if (body) {
                                const projectName = cleanUpParameter("projectName", body["projectName"]);
                                if (!projectName) {
                                    throw new HttpError(400, `Missing project name`);
                                }
                                const consultantName = cleanUpParameter("consultantName", body["consultantName"]?.toString() || "");
                                if (!consultantName) {
                                    throw new HttpError(400, `Missing consultant name`);
                                }
                                const role = cleanUpParameter("Role", body["role"]);
                                if (!role) {
                                    throw new HttpError(400, `Missing role`);
                                }
                                let forecast = body["forecast"];
                                if (!forecast) {
                                    forecast = 0;
                                    //throw new HttpError(400, `Missing forecast this month`);
                                }
                                console.log(`➡️ POST /api/projects: assignconsultant request, projectName=${projectName}, consultantName=${consultantName}, role=${role}, forecast=${forecast}`);
                                const result = await ProjectApiService.addConsultantToProject
                                    (projectName, consultantName, role, forecast);

                                res.jsonBody.results = {
                                    status: 200,
                                    clientName: result.clientName,
                                    projectName: result.projectName,
                                    consultantName: result.consultantName,
                                    remainingForecast: result.remainingForecast,
                                    message: result.message
                                };

                                console.log(`   ✅ POST /api/projects: response status ${res.status} - ${result.message}`);
                            } else {
                                throw new HttpError(400, `Missing request body`);
                            }
                            return res;
                        }
                        default: {
                            throw new HttpError(400, `Invalid command: ${id}`);
                        }
                    }

                }
                default: {
                    throw new Error(`Method not allowed: ${req.method}`);
                }
            }

        } catch (error) {

            const status = <number>error.status || <number>error.response?.status || 500;
            console.log(`   ⛔ Returning error status code ${status}: ${error.message}`);

            res.status = status;
            res.jsonBody.results = {
                status: status,
                message: error.message
            };
            return res;
        }
    }

    app.http("projects", {
        methods: ["GET", "POST"],
        authLevel: "anonymous",
        route: "projects/{*id}",
        handler: projects,
    });
    ```

    >[!note] This implements a new Azure function to provide access to Trey Research projects.

===

#### 02: Review the Azure function code (optional)

Let's take a moment to review the code you just added.

This is a version 4 Azure function, so the code looks a lot like traditional Express code for NodeJS. The **projects** class implements an HTTP request trigger, which is called when the **"/projects"** path is accessed. This is followed by some inline code that defines the methods and route. For now, access is anonymous.

- [] Observe the following section:

    ```typescript
    export async function projects(
        req: HttpRequest,
        context: InvocationContext
    ): Promise<Response> {
        // ...
    }
    app.http("projects", {
        methods: ["GET", "POST"],
        authLevel: "anonymous",
        route: "projects/{*id}",
        handler: projects,
    });
    ```

    >[!knowledge] The class includes a switch statement for handling GET vs. POST requests and obtains the parameters from the URL path (in the case of a project ID), query strings (such as **?projectName=foo**, in the case of a GET), and the request body (in the case of a POST). 
    >
    >It then accesses the project data using the [ProjectApiService](https://github.com/microsoft/copilot-camp/blob/main/src/extend-m365-copilot/path-e-lab04-enhance-api-plugin/trey-research-lab04-END/src/services/ProjectApiService.ts), which was part of the starting solution. It also sends responses for each request and the logging of requests to the debug console.

===

#### 03: Add HTTP test requests

You'll need to add the new requests to the **.http** file you configured in an earlier task.

1. [] Open **http**, then select **treyResearchAPI.http**.

1. [] At the bottom of the file, enter the following block of requests for the API:

    ```
    ########## /api/projects - working with projects ##########

    ### Get all projects
    {{base_url}}/projects

    ### Get project by id
    {{base_url}}/projects/1

    ### Get project by project or client name
    {{base_url}}/projects/?projectName=supply

    ### Get project by consultant name
    {{base_url}}/projects/?consultantName=dominique

    ### Add consultant to project
    POST {{base_url}}/projects/assignConsultant
    Content-Type: application/json

    {
        "projectName": "contoso",
        "consultantName": "sanjay",
        "role": "architect",
        "forecast": 30
    }
    ```

    !IMAGE[jdv25li8.jpg](instructions337966/jdv25li8.jpg)


===

#### 04: Test the new resource

1. [] Save all your changes by selecting **File**, then **Save All**.

3. [] In the top menu bar, select **Run**, then **Start Debugging**.

    >[!note] Once started, Edge will take you back to the **Trey Genie Local** agent.

1. [] Minimize Microsoft Edge.
    
    >[!alert] Do not close the window as that will stop the debugger.

1. [] In **treyResearchAPI.http**, try selecting **Send Request** to assign a new consultant to a project using the POST request under line 58.

    !IMAGE[da1fjpdk.jpg](instructions337966/da1fjpdk.jpg)

1. [] Observe the response.

    !IMAGE[k40zk3pt.jpg](instructions337966/k40zk3pt.jpg)

1. [] Feel free to try sending other requests for project details.

1. [] Once finished, in the top menu bar, select **Run**, then **Stop Debugging**.

1. [] Close any VS Code tabs and the **Response** pane.

===

## Task 02: Add projects to the application package

### Description

You'll update the OpenAPI Specification file and the plugin definition file to include the new `/projects` endpoints, so Copilot knows how to discover and call them at runtime.

### Success criteria

- You replaced `trey-definition.json` with the updated version including the `/projects/` GET path and the `/projects/assignConsultant` POST path.
- You replaced `trey-plugin.json` with the updated version including `getProjects` and `postAssignConsultant` in both the `functions` array and `run_for_functions` list.

### Key steps

---

#### 01: Update the Open API Specification file

An important part of the application package is the [Open API Specification (OAS)](https://swagger.io/specification/) definition file. OAS defines a standard format for describing a REST API and is based on the popular "Swagger" definition.

1. [] Open **appPackage**, select **apiSpecificationFile**, then **trey-definition.json**

    !IMAGE[yic7igs2.jpg](instructions337966/yic7igs2.jpg)

1. [] Delete everything contained in the file. 

    >[!hint] Select **Ctrl+A** to highlight the whole file.

1. [] Replace the file content with the following:

    >[!hint] Select **Copy** in the following block, then paste with **Ctrl+V**.

    ```json
    {
        "openapi": "3.0.1",
        "info": {
            "version": "1.0.0",
            "title": "Trey Research API",
            "description": "API to streamline consultant assignment and project management."
        },
        "servers": [
            {
                "url": "${{OPENAPI_SERVER_URL}}/api/",
                "description": "Production server"
            }
        ],
        "paths": {
            "/consultants/": {
                "get": {
                    "operationId": "getConsultants",
                    "summary": "Get consultants working at Trey Research based on consultant name, project name, certifications, skills, roles and hours available",
                    "description": "Returns detailed information about consultants identified from filters like name of the consultant, name of project, certifications, skills, roles and hours available. Multiple filters can be used in combination to refine the list of consultants returned",
                    "parameters": [
                        {
                            "name": "consultantName",
                            "in": "query",
                            "description": "Name of the consultant to retrieve",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        },
                        {
                            "name": "projectName",
                            "in": "query",
                            "description": "The name of the project",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        },
                        {
                            "name": "skill",
                            "in": "query",
                            "description": "Skills for a consultant. Retrieve consultants with this skill",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        },
                        {
                            "name": "certification",
                            "in": "query",
                            "description": "Certification for a consultant. Retrieve consultants with this certification",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        },
                        {
                            "name": "role",
                            "in": "query",
                            "description": "Role of a consultant. Retrieve consultants with this role",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        },
                        {
                            "name": "hoursAvailable",
                            "in": "query",
                            "description": "Hours a consultant is available for new work over this and next month. Please provide an integer value; for example 0 if no hours are available, 1 if any hours are available, or 20 if 20 or more hours are available.",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        }
                    ],
                    "responses": {
                        "200": {
                            "description": "Successful response",
                            "content": {
                                "application/json": {
                                    "schema": {
                                        "type": "object",
                                        "properties": {
                                            "results": {
                                                "type": "array",
                                                "items": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "string"
                                                        },
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "email": {
                                                            "type": "string",
                                                            "format": "email"
                                                        },
                                                        "phone": {
                                                            "type": "string"
                                                        },
                                                        "location": {
                                                            "type": "object",
                                                            "properties": {
                                                                "street": {
                                                                    "type": "string"
                                                                },
                                                                "city": {
                                                                    "type": "string"
                                                                },
                                                                "state": {
                                                                    "type": "string"
                                                                },
                                                                "country": {
                                                                    "type": "string"
                                                                },
                                                                "postalCode": {
                                                                    "type": "string"
                                                                },
                                                                "latitude": {
                                                                    "type": "number"
                                                                },
                                                                "longitude": {
                                                                    "type": "number"
                                                                }
                                                            }
                                                        },
                                                        "skills": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "string"
                                                            }
                                                        },
                                                        "certifications": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "string"
                                                            }
                                                        },
                                                        "roles": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "string"
                                                            }
                                                        },
                                                        "consultantPhotoUrl": {
                                                            "type": "string",
                                                            "format": "uri"
                                                        },
                                                        "projects": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "object",
                                                                "properties": {
                                                                    "projectName": {
                                                                        "type": "string"
                                                                    },
                                                                    "projectDescription": {
                                                                        "type": "string"
                                                                    },
                                                                    "projectLocation": {
                                                                        "type": "object",
                                                                        "properties": {
                                                                            "street": {
                                                                                "type": "string"
                                                                            },
                                                                            "city": {
                                                                                "type": "string"
                                                                            },
                                                                            "state": {
                                                                                "type": "string"
                                                                            },
                                                                            "country": {
                                                                                "type": "string"
                                                                            },
                                                                            "postalCode": {
                                                                                "type": "string"
                                                                            },
                                                                            "latitude": {
                                                                                "type": "number"
                                                                            },
                                                                            "longitude": {
                                                                                "type": "number"
                                                                            }
                                                                        }
                                                                    },
                                                                    "mapUrl": {
                                                                        "type": "string",
                                                                        "format": "uri"
                                                                    },
                                                                    "role": {
                                                                        "type": "string"
                                                                    },
                                                                    "forecastThisMonth": {
                                                                        "type": "integer"
                                                                    },
                                                                    "forecastNextMonth": {
                                                                        "type": "integer"
                                                                    },
                                                                    "deliveredLastMonth": {
                                                                        "type": "integer"
                                                                    },
                                                                    "deliveredThisMonth": {
                                                                        "type": "integer"
                                                                    }
                                                                }
                                                            }
                                                        },
                                                        "forecastThisMonth": {
                                                            "type": "integer"
                                                        },
                                                        "forecastNextMonth": {
                                                            "type": "integer"
                                                        },
                                                        "deliveredLastMonth": {
                                                            "type": "integer"
                                                        },
                                                        "deliveredThisMonth": {
                                                            "type": "integer"
                                                        }
                                                    }
                                                }
                                            },
                                            "status": {
                                                "type": "integer"
                                            }
                                        }
                                    }
                                }
                            }
                        },
                        "404": {
                            "description": "Consultant not found"
                        }
                    }
                }
            },
            "/me": {
                "get": {
                    "operationId": "getUserInformation",
                    "summary": "Get consultant profile of the logged in user.",
                    "description": "Retrieve the consultant profile for the logged-in user including skills, roles, certifications, location, availability, and project assignments.",
                    "responses": {
                        "200": {
                            "description": "Successful response",
                            "content": {
                                "application/json": {
                                    "schema": {
                                        "type": "object",
                                        "properties": {
                                            "results": {
                                                "type": "array",
                                                "items": {
                                                    "type": "object",
                                                    "properties": {
                                                        "id": {
                                                            "type": "string"
                                                        },
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "email": {
                                                            "type": "string",
                                                            "format": "email"
                                                        },
                                                        "phone": {
                                                            "type": "string"
                                                        },
                                                        "location": {
                                                            "type": "object",
                                                            "properties": {
                                                                "street": {
                                                                    "type": "string"
                                                                },
                                                                "city": {
                                                                    "type": "string"
                                                                },
                                                                "state": {
                                                                    "type": "string"
                                                                },
                                                                "country": {
                                                                    "type": "string"
                                                                },
                                                                "postalCode": {
                                                                    "type": "string"
                                                                },
                                                                "latitude": {
                                                                    "type": "number"
                                                                },
                                                                "longitude": {
                                                                    "type": "number"
                                                                }
                                                            }
                                                        },
                                                        "skills": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "string"
                                                            }
                                                        },
                                                        "certifications": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "string"
                                                            }
                                                        },
                                                        "roles": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "string"
                                                            }
                                                        },
                                                        "consultantPhotoUrl": {
                                                            "type": "string",
                                                            "format": "uri"
                                                        },
                                                        "projects": {
                                                            "type": "array",
                                                            "items": {
                                                                "type": "object",
                                                                "properties": {
                                                                    "projectName": {
                                                                        "type": "string"
                                                                    },
                                                                    "projectDescription": {
                                                                        "type": "string"
                                                                    },
                                                                    "projectLocation": {
                                                                        "type": "object",
                                                                        "properties": {
                                                                            "street": {
                                                                                "type": "string"
                                                                            },
                                                                            "city": {
                                                                                "type": "string"
                                                                            },
                                                                            "state": {
                                                                                "type": "string"
                                                                            },
                                                                            "country": {
                                                                                "type": "string"
                                                                            },
                                                                            "postalCode": {
                                                                                "type": "string"
                                                                            },
                                                                            "latitude": {
                                                                                "type": "number"
                                                                            },
                                                                            "longitude": {
                                                                                "type": "number"
                                                                            }
                                                                        }
                                                                    },
                                                                    "mapUrl": {
                                                                        "type": "string",
                                                                        "format": "uri"
                                                                    },
                                                                    "role": {
                                                                        "type": "string"
                                                                    },
                                                                    "forecastThisMonth": {
                                                                        "type": "integer"
                                                                    },
                                                                    "forecastNextMonth": {
                                                                        "type": "integer"
                                                                    },
                                                                    "deliveredLastMonth": {
                                                                        "type": "integer"
                                                                    },
                                                                    "deliveredThisMonth": {
                                                                        "type": "integer"
                                                                    }
                                                                }
                                                            }
                                                        },
                                                        "forecastThisMonth": {
                                                            "type": "integer"
                                                        },
                                                        "forecastNextMonth": {
                                                            "type": "integer"
                                                        },
                                                        "deliveredLastMonth": {
                                                            "type": "integer"
                                                        },
                                                        "deliveredThisMonth": {
                                                            "type": "integer"
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            },
            "/projects/": {
                "get": {
                    "operationId": "getProjects",
                    "summary": "Get projects matching a specified project name and/or consultant name",
                    "description": "Returns detailed information about projects matching the specified project name and/or consultant name",
                    "parameters": [
                        {
                            "name": "consultantName",
                            "in": "query",
                            "description": "The name of the consultant assigned to the project",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        },
                        {
                            "name": "projectName",
                            "in": "query",
                            "description": "The name of the project or name of the client",
                            "required": false,
                            "schema": {
                                "type": "string"
                            }
                        }
                    ],
                    "responses": {
                        "200": {
                            "description": "Successful response",
                            "content": {
                                "application/json": {
                                    "schema": {
                                        "type": "object",
                                        "properties": {
                                            "results": {
                                                "type": "array",
                                                "items": {
                                                    "type": "object",
                                                    "properties": {
                                                        "name": {
                                                            "type": "string"
                                                        },
                                                        "description": {
                                                            "type": "string"
                                                        },
                                                        "location": {
                                                            "type": "object",
                                                            "properties": {
                                                                "street": {
                                                                    "type": "string"
                                                                },
                                                                "city": {
                                                                    "type": "string"
                                                                },
                                                                "state": {
                                                                    "type": "string"
                                                                },
                                                                "country": {
                                                                    "type": "string"
                                                                },
                                                                "postalCode": {
                                                                    "type": "string"
                                                                },
                                                                "latitude": {
                                                                    "type": "number"
                                                                },
                                                                "longitude": {
                                                                    "type": "number"
                                                                }
                                                            }
                                                        },
                                                        "mapUrl": {
                                                            "type": "string",
                                                            "format": "uri"
                                                        },
                                                        "role": {
                                                            "type": "string"
                                                        },
                                                        "forecastThisMonth": {
                                                            "type": "integer"
                                                        },
                                                        "forecastNextMonth": {
                                                            "type": "integer"
                                                        },
                                                        "deliveredLastMonth": {
                                                            "type": "integer"
                                                        },
                                                        "deliveredThisMonth": {
                                                            "type": "integer"
                                                        }
                                                    }
                                                }
                                            },
                                            "status": {
                                                "type": "integer"
                                            }
                                        }
                                    }
                                }
                            }
                        },
                        "404": {
                            "description": "Project not found"
                        }
                    }
                }
            },
            "/me/chargeTime": {
                "post": {
                    "operationId": "postBillhours",
                    "summary": "Charge time to a project on behalf of the logged in user.",
                    "description": "Charge time to a specific project on behalf of the logged in user, and return the number of hours remaining in their forecast.",
                    "requestBody": {
                        "required": true,
                        "content": {
                            "application/json": {
                                "schema": {
                                    "type": "object",
                                    "properties": {
                                        "projectName": {
                                            "type": "string"
                                        },
                                        "hours": {
                                            "type": "integer"
                                        }
                                    },
                                    "required": [
                                        "projectName",
                                        "hours"
                                    ]
                                }
                            }
                        }
                    },
                    "responses": {
                        "200": {
                            "description": "Successful charge",
                            "content": {
                                "application/json": {
                                    "schema": {
                                        "type": "object",
                                        "properties": {
                                            "results": {
                                                "type": "object",
                                                "properties": {
                                                    "status": {
                                                        "type": "integer"
                                                    },
                                                    "clientName": {
                                                        "type": "string"
                                                    },
                                                    "projectName": {
                                                        "type": "string"
                                                    },
                                                    "remainingForecast": {
                                                        "type": "integer"
                                                    },
                                                    "message": {
                                                        "type": "string"
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            },
            "/projects/assignConsultant": {
                "post": {
                    "operationId": "postAssignConsultant",
                    "summary": "Assign consultant to a project when name, role and project name is specified.",
                    "description": "Assign (add) consultant to a project when name, role and project name is specified.",
                    "requestBody": {
                        "required": true,
                        "content": {
                            "application/json": {
                                "schema": {
                                    "type": "object",
                                    "properties": {
                                        "projectName": {
                                            "type": "string"
                                        },
                                        "consultantName": {
                                            "type": "string"
                                        },
                                        "role": {
                                            "type": "string"
                                        },
                                        "forecast": {
                                            "type": "integer"
                                        }
                                    },
                                    "required": [
                                        "projectName",
                                        "consultantName",
                                        "role",
                                        "forecast"
                                    ]
                                }
                            }
                        }
                    },
                    "responses": {
                        "200": {
                            "description": "Successful assignment",
                            "content": {
                                "application/json": {
                                    "schema": {
                                        "type": "object",
                                        "properties": {
                                            "results": {
                                                "type": "object",
                                                "properties": {
                                                    "status": {
                                                        "type": "integer"
                                                    },
                                                    "clientName": {
                                                        "type": "string"
                                                    },
                                                    "projectName": {
                                                        "type": "string"
                                                    },
                                                    "consultantName": {
                                                        "type": "string"
                                                    },
                                                    "remainingForecast": {
                                                        "type": "integer"
                                                    },
                                                    "message": {
                                                        "type": "string"
                                                    }
                                                }
                                            },
                                            "status": {
                                                "type": "integer"
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
    ```

    >[!note] You can review the file in the next task to understand the changes.

    !IMAGE[pgsezxrn.jpg](instructions337966/pgsezxrn.jpg)

===

#### 02: Review the updates (optional)

The first update was to add the **/projects/** path to the **paths** collection in the JSON. 

- [] Observe the **"/projects/"** section:

    ```json
    "/projects/": {
        "get": {
            "operationId": "getProjects",
            "summary": "Get projects matching a specified project name and/or consultant name",
            "description": "Returns detailed information about projects matching the specified project name and/or consultant name",
            "parameters": [
                {
                    "name": "consultantName",
                    "in": "query",
                    "description": "The name of the consultant assigned to the project",
                    "required": false,
                    "schema": {
                        "type": "string"
                    }
                },
    ...
    ```

    >[!note] You can see this includes all the available query strings when retrieving the **/projects/** resource, along with data types and required fields. It also includes the data that will be returned in API responses, including different payloads for status **200 (successful)** and **400 (failed)** responses.
    >
    >You'll also find that a path has been added at **/projects/assignConsultant** to handle the POST requests.

---

>[!knowledge] **Descriptions are important**
>
>All files in the app package will be read by AI. You can help Copilot properly use your API by using descriptive names and descriptions in this and all other application package files.

===

#### 03: Add projects to the plugin definition file

1. [] Open **appPackage**, then select **trey-plugin.json**.
    
    !IMAGE[o681vvye.jpg](instructions337966/o681vvye.jpg)

    >[!note] This file contains extra information not included in the OAS definition file.

1. [] Delete everything contained in the file. 

1. [] Replace the file content with the following:

    >[!hint] Select **Copy** in the following block, then paste with **Ctrl+V**.

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.2/schema.json",
      "schema_version": "v2.2",
      "namespace": "treyresearch",
      "name_for_human": "Trey-Research${{APP_NAME_SUFFIX}}",
      "description_for_human": "Plugin to streamline consultant assignment and project management.",
      "description_for_model": "Plugin for managing repairs, consultant assignments, and project tracking within Trey Research. Provides functionalities to list repairs with details and images.",
      "functions": [
        {
          "name": "getConsultants",
          "description": "Returns detailed information about consultants identified from filters like name of the consultant, name of project, certifications, skills, roles and hours available. Multiple filters can be used in combination to refine the list of consultants returned",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.results",
              "properties": {
                "title": "$.name",
                "subtitle": "$.id",
                "url": "$.consultantPhotoUrl"
              }
            }
          }
        },
        {
          "name": "getUserInformation",
          "description": "Retrieve the consultant profile for the logged-in user including skills, roles, certifications, location, availability, and project assignments.",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.results",
              "properties": {
                "title": "$.name",
                "subtitle": "$.id",
                "url": "$.consultantPhotoUrl"
              }
            }
          }
        },
        {
          "name": "getProjects",
          "description": "Returns detailed information about projects matching the specified project name and/or consultant name by searching scripts under ./scripts/db/",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.results",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        },
        {
          "name": "postBillhours",
          "description": "Charge time to a specific project on behalf of the logged in user, and return the number of hours remaining in their forecast.",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.results.clientName",
                "subtitle": "$.results.status"
              }
            },
            "confirmation": {
              "type": "AdaptiveCard",
              "title": "Charge time to a project on behalf of the logged in user.",
              "body": "**ProjectName**: {{function.parameters.projectName}}\n* **Hours**: {{function.parameters.hours}}"
            }
          }
        },
        {
          "name": "postAssignConsultant",
          "description": "Assign (add) consultant to a project when name, role and project name is specified.",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.results.clientName",
                "subtitle": "$.results.status"
              }
            },
            "confirmation": {
              "type": "AdaptiveCard",
              "title": "Assign consultant to a project when name, role and project name is specified.",
              "body": "**ProjectName**: {{function.parameters.projectName}}\n* **ConsultantName**: {{function.parameters.consultantName}}\n* **Role**: {{function.parameters.role}}\n* **Forecast**: {{function.parameters.forecast}}"
            }
          }
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/trey-definition.json"
          },
          "run_for_functions": [
            "getConsultants",
            "getUserInformation",
            "getProjects",
            "postBillhours",
            "postAssignConsultant"
          ]
        }
      ],
      "capabilities": {
        "conversation_starters": [
          {
            "text": "What Trey projects am i assigned to?"
          },
          {
            "text": "Charge 5 hours to the Contoso project for Trey Research"
          },
          {
            "text": "Which Trey consultants are Azure certified?"
          },
          {
            "text": "Find a Trey consultant who is available now and has Python skills"
          },
          {
            "text": "Add Avery as a developer on the Contoso project for Trey"
          }
        ]
      }
    }
    ```

    !IMAGE[bwsvhedk.jpg](instructions337966/bwsvhedk.jpg)

===

#### 04: Review the changes to the plugin definition file (optional)

The plugin JSON file contains a collection of _functions_, each of which corresponds to a type of API call. Copilot will choose among these functions when utilizing your plugin at runtime.

The new **trey-plugin.json** file includes new functions **getProjects** and **postAssignConsultant**. 

Here's **getProjects**:

```json
{
    "name": "getProjects",
    "description": "Returns detailed information about projects matching the specified project name and/or consultant name",
    "capabilities": {
        "response_semantics": {
            "data_path": "$.results",
            "properties": {
            "title": "$.name",
            "subtitle": "$.description"
            }
        }
    }
},
```

This includes **response_semantics** which instructs Copilot's orchestrator on how to interpret the response payload. It defines the mapping of structured data in a response payload to specific properties required by the function. It enables the orchestrator to interpret and transform raw response data into meaningful content for rendering or further processing.

For example, look at the response semantics below:

```json
"functions": [
    {
      "name": "getConsultants",
      "description": "Returns detailed information about consultants identified from filters like name of the consultant, name of project, certifications, skills, roles and hours available. Multiple filters can be used in combination to refine the list of consultants returned",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.results",
          "properties": {
            "title": "$.name",
            "subtitle": "$.id",
            "url": "$.consultantPhotoUrl"
          }
        }
      }
    },
    ...
]
```

Here **data_path** is **$.results** in the **response_semantics** of function **getConsultants**. This means the main data resides under the **results** key in the JSON, which ensures the system extracts data starting at that path. It also defines mappings of specific fields from the raw data to corresponding semantic properties under **properties** field to readily use it for rendering.

```json
"title": "$.name",
"subtitle": "$.id",
"url": "$.consultantPhotoUrl"
```

The POST request has a similar function:

```json
{
    "name": "postAssignConsultant",
    "description": "Assign (add) consultant to a project when name, role and project name is specified.",
    "capabilities": {
        "response_semantics": {
            "data_path": "$",
            "properties": {
            "title": "$.results.clientName",
            "subtitle": "$.results.status"
            }
        },
        "confirmation": {
            "type": "AdaptiveCard",
            "title": "Assign consultant to a project when name, role and project name is specified.",
            "body": "**ProjectName**: {{function.parameters.projectName}}\n* **ConsultantName**: {{function.parameters.consultantName}}\n* **Role**: {{function.parameters.role}}\n* **Forecast**: {{function.parameters.forecast}}"
        }
    }
}
```

It includes an [adaptive card](https://adaptivecards.io) to be used in the confirmation card, which is shown to users to confirm an action prior to issuing a POST request.

Moving down, you can see the **runtimes** object which defines the type of plugin, the OAS definition file location, and a list of functions. The new functions have been added to the list.

```json
"runtimes": [
    {
        "type": "OpenApi",
        "auth": {
        "type": "None"
        },
            "spec": {
            "url": "trey-definition.json"
        },
        "run_for_functions": [
            "getConsultants",
            "getUserInformation",
            "getProjects",
            "postBillhours",
            "postAssignConsultant"
        ]
    }
],
```

Finally, it includes some conversation starters which are prompt suggestions shown to users. The new file has a conversation starter relating to projects.

```json
"capabilities": {
    "localization": {},
    "conversation_starters": [
        {
            "text": "What Trey projects am i assigned to?"
        },
        {
            "text": "Charge 5 hours to the Contoso project for Trey Research"
        },
        {
            "text": "Which Trey consultants are Azure certified?"
        },
        {
            "text": "Find a Trey consultant who is available now and has Python skills"
        },
        {
            "text": "Add Avery as a developer on the Contoso project for Trey"
        }
    ]
}
```

===

## Task 03: Test the plugin in Copilot

### Description

You'll increment the app manifest version to ensure the updated package is installed, start the debugger, and verify the agent can answer project-related queries using the new `/projects` endpoint.

### Success criteria

- You incremented the manifest version number.
- You started the debugger and the **Trey Genie Local** agent opened without errors.
- You sent a project query and received a valid response referencing Adatum project data.

### Key steps

---

#### 01: Update manifest version

Before you test the application, update the manifest version of your app package in the **manifest.json** file.

1. [] Open **appPackage**, then select **manifest.json**.

1. [] Locate the **version** property on line 5.

1. [] Increment the version number. If it's at **1.0.2**, update to `"1.0.3"`.

    !IMAGE[fuvd7ln3.jpg](instructions337966/fuvd7ln3.jpg)

===

#### 02: Test the agent

1. [] Save all your changes by selecting **File**, then **Save All**.

1. [] In the top menu bar, select **Run**, then **Start Debugging**.

    >[!note] Once started, Edge will take you back to the **Trey Genie Local** agent.

1. [] Prompt the Trey Genie Local agent:

    `What projects do we have for Adatum?`

    !IMAGE[j0v99gkw.jpg](instructions337966/j0v99gkw.jpg)

    >[!alert] If it's unable to find any projects: check the leftmost pane to see if you have another **Trey Genie Local** agent listed. Try from there.
    >
    >!IMAGE[9kgw4z9b.jpg](instructions337966/9kgw4z9b.jpg)

1. [] In VS Code's top menu bar, select **Run**, then **Stop Debugging**.

1. [] Close any open tabs in VS Code.

---

**Congratulations!**

You've now completed enhancing your API plugin.

===

# Exercise 04: Integration - Add knowledge capability to Trey Genie using a Microsoft Copilot connector

## Introduction
In this exercise, you'll learn how to add your own data into Microsoft Graph to then be organically used by the declarative agent as its own knowledge. In the process, you'll learn all how to deploy a Microsoft Copilot Connector and use the connector in Trey Genie declarative agent. 

You'll learn to:

- Deploy a Microsoft Copilot Connector of your own data into Microsoft Graph and have it power various Microsoft 365 experiences.
- Customize the Trey Genie declarative agent to use the Copilot Connector as a capability to extend its knowledge.

Knowledge:
- [Microsoft Copilot Connector source code](https://github.com/microsoft/copilot-camp/tree/main/src/extend-m365-copilot/path-e-bonus-gc-lab/trey-feedback-connector)



<!-- !!! note "Prerequisites: Tenant Admin Access"
    Additonal prerequisites are needed to run this lab. You will need <mark>tenant administrator privileges</mark> as Microsoft Copilot Connectors use app-only authentication to access the connector APIs.

!!! note "Prerequisites: Azure Functions Visual Studio Code extension"
    - [Azure Functions Visual Studio Code extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions){target=_blank} -->

===

## Task 01: Deploy Copilot Connector

### Description
Microsoft 365 Copilot connectors allow you to bring external, line-of-business data into Microsoft 365 Copilot so your users can search, reason over, and act on more of your enterprise content. The platform supports two connector models:

- **Synced connectors** ingest and index external content into Microsoft Graph.
- **Federated connectors (early access preview)** retrieve content in real time using Model Context Protocol (MCP) without indexing data into Microsoft Graph.

Both connector types power Microsoft 365 Copilot and other Microsoft 365 intelligent experiences, such as Microsoft Search, Context IQ, and Microsoft 365 Copilot.

### Success criteria

- You navigated to the **Connectors** page in the Microsoft 365 admin center and identified the connector setup steps.
- You confirmed the pre-configured GitHub Cloud Issues connector returns results for "Project Alpha" in Microsoft 365 Search.
- You added the `GraphConnectors` capability block to `trey-declarative-agent.json` and updated the agent instructions.

### Key steps

---

#### 01: Setting up a Microsoft 365 Copilot connector

>[!alert] **NOTE:** Observe the following steps on this page, as your lab account does not have tenant level permissions. 
>
>If you have your own test tenant, you can follow along from there.

1. [] Open Microsoft Edge, then go to ++admin.cloud.microsoft++.

1. [] In the leftmost pane, go to **Copilot** > **Connectors**.

    !IMAGE[fi1a9160.jpg](instructions337966/fi1a9160.jpg)

1. [] Above the table, select **Add Connection**.

	!IMAGE[w7f9hyka.jpg](instructions337966/w7f9hyka.jpg)

    >[!note] You'll see many prebuilt third-party services you can create connections to.

1. [] In the search box, enter ++GitHub++.

1. [] In the tile for **GitHub Cloud Issues**, select **Add**.

	!IMAGE[x655gfnj.jpg](instructions337966/x655gfnj.jpg)

1. [] In the flyout pane, observe the following:

	1. [] You'll see various options for **Authentication type**.
    1. [] Entry for your GitHub Enterprise **Organization name**.
    1. [] Under **Authenticate your Github instance**, there's a link to install the **Microsoft 365 Copilot** app within your GitHub organization. 
    
    	>[!knowledge] The GitHub connector is designed for **GitHub Enterprise**. There may be limitations with **Free** or **Team** plans.

    1. [] You can then **Authorize** your GitHub account to set up the connector.
    1. [] Near the top of the pane, select **Custom setup**.
    
    	!IMAGE[lg20mspx.jpg](instructions337966/lg20mspx.jpg)

    1. [] Select the **User** tab.
    
    	!IMAGE[4cx6dmzg.jpg](instructions337966/4cx6dmzg.jpg)

    	>[!note] Once you've authorized your account, you can restrict which users in your tenant can access the data source.

===

#### 02: Test the GitHub Cloud Issues connector

>[!alert] You can perform tasks in the lab environment again from this point forward. 
>
>We've preconfigured a connector using a GitHub Enterprise organization with sample data.

1. [] In a new tab, go to `m365.cloud.microsoft/chat`.

1. [] Close any dialogs.

1. [] In the leftmost pane, select **Search**.
    
    !IMAGE[xba2xe15.jpg](instructions337966/xba2xe15.jpg)

1. [] In the search box, enter `Project Alpha`, then select **Enter**.

    !IMAGE[zpj9hesv.jpg](instructions337966/zpj9hesv.jpg)

    >[!note] In the results, you'll see it pulls an existing issue from the GitHub Enterprise organization through the Copilot connector we've preconfigured. 

===

#### 03: Add the connector to your agent

With your GitHub Enterprise organization now part of your Microsoft 365 data, you can add the connector data as focused knowledge for our declarative agent.

1. [] In VS Code, open **appPackage**, then select **trey-declarative-agent.json**.
    
    !IMAGE[ssy3om6p.jpg](instructions337966/ssy3om6p.jpg)

1. [] In the **capabilities** array, on line 29, enter a comma `,` after the curly brace, then create a new line.

    !IMAGE[f36eji7z.jpg](instructions337966/f36eji7z.jpg)

1. [] Add **GraphConnectors** to the array:

    ```json
    {
        "name": "GraphConnectors",
        "connections": [
            {
                "connection_id": "GitHubCloudIssues2603062230"
            }
        ]
    }
    ```

    >[!knowledge] This uses the **connection_id** of the preconfigured M365 Copilot connector. After creating a connector in the M365 admin center, open it to find the ID near the top of the pane.
    >
    >!IMAGE[rbf8134y.jpg](instructions337966/rbf8134y.jpg)

1. [] Right-click anywhere in the file, then select **Format Document** to fix the spacing.

    !IMAGE[7i3it0ko.jpg](instructions337966/7i3it0ko.jpg)
    !IMAGE[1iaujkff.jpg](instructions337966/1iaujkff.jpg)

1. [] Near the top of the file, replace the entire line for **"instructions"** with the following:

	`"instructions": "You are a consulting agent. Greet users professionally and introduce yourself as the Trey Genie. Offer assistance with their consulting projects and hours. Remind users of the Trey motto, 'Always be Billing!'. Your primary role is to support consultants by helping them manage their projects and hours, as well as tracking project issues. Using the TreyResearch action, you can assist users in retrieving consultant profiles or project details for administrative purposes, charge hours, and assign consultants. Do not participate in decisions related to hiring or performance evaluation. If a user inquires about hours billed or charged, rephrase the request to ask about hours 'delivered'. Additionally, you have access to GitHub Cloud Issues through Microsoft 365 Copilot connectors. You can assist users in searching, summarizing, and retrieving data regarding technical issues and bugs associated with their projects. If there is any confusion, encourage users to consult their Managing Consultant. Avoid providing legal advice.",`

    >[!note] This includes new instructions to use the GitHub Cloud Issues connector.

1. [] Right-click anywhere in the file, then select **Format Document** to fix the spacing.

	!IMAGE[kn5axzvd.jpg](instructions337966/kn5axzvd.jpg)

===

#### 04: Update manifest version

1. [] Open **.\appPackage\manifest.json**.

1. [] Increment the **version** number on line 5. If it's at **1.0.3**, update to `"1.0.4"`.

    !IMAGE[fuvd7ln3.jpg](instructions337966/fuvd7ln3.jpg)


===

#### 05: Test the agent

1. [] Ensure you've saved all your changes.

1. [] In the top menu bar, select **Run**, ,then **Start Debugging**.

    >[!note] Once started, Edge will take you back to the **Trey Genie Local** agent.

1. [] Ask about an issue from GitHub:

    `Has the Bellows College network vulnerability assessment issue been closed yet in GitHub?`

    !IMAGE[4niie7ra.jpg](instructions337966/4niie7ra.jpg)

    >[!alert] If it's unable to find anything, check the leftmost pane to see if you have another **Trey Genie Local** agent listed and try from there.

1. [] Test a prompt that has the agent check both GitHub and SharePoint:
    
    `Show me the open GitHub issue related to Woodgrove Bank and tell me how many hours have been logged on the plugin.`

    !IMAGE[u1uo21ub.jpg](instructions337966/u1uo21ub.jpg)

    >[!note] The 2nd prompt should pull issue data from GitHub and hours from SharePoint, based on the file name referenced within the issue.

===

>[!Alert] **IMPORTANT:** These labs are hosted on the Skillable platform. Completion data is collected and then exported to Success Factors every Monday. SF require another 1-3 days to process that data. The status for this lab will be visible in Viva and Learning Path next week. 
>
Be sure to select "**Submit**" in the bottom right corner to get credit for completing this lab. 

@lab.ActivityGroup(completionsurvey)

>[!Alert] After answering the survey questions, select **submit** to complete and end the lab. **This is required in order to receive credit for lab completion**.



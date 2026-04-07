## Usecase 03- Build Fabric Data Agent using Mirrored Azure SQL Database in Microsoft Fabric

**Introduction**

Modern organizations require intelligent systems that can quickly
analyze operational data and provide meaningful insights without complex
data movement. In this use case, Microsoft Fabric is used to mirror data
from an Azure SQL Database into a Fabric environment and create a Fabric
Data Agent that can query and analyze the mirrored data.

The process begins with creating an Azure SQL Database containing sample
business data. This database is then mirrored into Microsoft Fabric
using Azure SQL Mirroring, enabling near real-time access to operational
data within the Fabric workspace. Once the mirrored database is
available, a Fabric Data Agent is configured to connect to the data
source and answer natural language queries.

This approach enables users to interact with enterprise data through
intelligent agents, allowing faster insights into product performance,
customer distribution, and sales trends without writing complex SQL
queries.

**Objective**

The objective of this lab is to demonstrate how to build and configure a
Fabric Data Agent that can analyze mirrored operational data from an
Azure SQL Database.

By completing this lab, you will learn how to:

- Create an **Azure SQL Database** with sample data.

- Create a **Microsoft Fabric workspace** to host data and analytics
  resources.

- Mirror an **Azure SQL Database into Microsoft Fabric** using Azure SQL
  Mirroring.

- Configure a **Fabric Data Agent** and connect it to the mirrored
  database.

- Query the data using **natural language prompts** to generate
  insights.

- Validate the agent responses using sample analytical questions.

## **Task 0: Sync Host environment time**

1.  In your VM, navigate and click in the **Search bar**, type
    **Settings** and then click on **Settings** under **Best match**.

> ![A screenshot of a computer Description automatically
> generated](./media/image1.png)

2.  On Settings window, navigate and click on **Time & language**.

![A screenshot of a computer Description automatically
generated](./media/image2.png)

3.  On **Time & language** page, navigate and click on **Date & time**.

![A screenshot of a computer Description automatically
generated](./media/image3.png)

4.  Scroll down and navigate to **Additional settings** section, then
    click on **Syn now** button. It will take 3-5 minutes to syn.

![A screenshot of a computer Description automatically
generated](./media/image4.png)

5.  Close the **Settings** window.

![A screenshot of a computer Description automatically
generated](./media/image5.png)

## Task 1: Create a single database - Azure SQL Database

In this task, you will create a fully configured Azure SQL Database with
sample data. You will deploy the AdventureWorksLT sample schema, verify
tables, and prepare your server connection details for later mirroring
in Fabric.

1.  Open a browser go to +++https://portal.azure.com+++ and sign in with
    your cloud slice account below.
    |   |   |
    |---|---|
    | Username | +++@lab.CloudPortalCredential(User1).Username+++ |
    | Password | +++@lab.CloudPortalCredential(User1).Password+++ |

3.  From the Azure portal home page, click on **Azure portal
    menu** represented by three horizontal bars on the left side of the
    Microsoft Azure command bar. Select SQL database

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image6.png)

3.  Click on **+ Create**

> ![](./media/image7.png)

4.  On **Create a storage account** window, under the **Basics** tab,
    enter the below details to create a storage account and then click
    on **Next:Networking**

| Setting | Value  |
|--------|----------------|
| Subscription | Select your subscription |
| Resource group | Select your Resource group |
| Database name | +++sqldatabaseXXXX+++ *(XXXX = last 4 digits of Lab Instance ID)* |
| Server | Select **Create new** |
| Server name | +++sqlserverXXXX+++ |
| Location | Southeast Asia |
| Server admin login | +++sqladmin+++ |
| Password | `+++password321!+++` |
| Confirm password | `+++password321!+++` |
| Action | Click **OK** |

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image8.png)

![A screenshot of a computer AI-generated content may be
incorrect.](./media/image9.png)

5.  In the Compute + Storage section, click on **Configure database**.

![](./media/image10.png)

6.  For Service tier from the dropdown select **Standard(Budget
    Friendly) and for DTU enter 100 **and click** Apply**

![](./media/image11.png)

![](./media/image12.png)

7.  On the **Networking** tab, select **Public endpoint**, set **Allow
    Azure services and resources** to **Yes**, enable **Add current
    client IP address**, and then click **Next: Security\>**

![](./media/image13.png)

8.  On the **Security** page, after reviewing, select **Next :
    Additional settings**

> ![](./media/image14.png)

9.  On the *Additional settings* tab, select **Sample** under *Use
    existing data*, choose **AdventureWorksLT** when prompted, click
    **OK**, and then select **Review + create** to proceed.

> ![](./media/image15.png)

10. On the **Review + create** page, after reviewing, select **Create**

> ![](./media/image16.png)
>
> ![](./media/image17.png)

11. On **Microsoft.SQLDatabase** window, after the deployment is
    completed, click on the **Go to resource** button.

> ![](./media/image18.png)

12. In SQL database page select **Query editor**.

> ![](./media/image19.png)

13. In the **Query editor (preview)**, enter the SQL
    server **login** as **sqladmin** and **password** as +++**password321!+++**,
    then click **OK** to connect to the database.

> ![](./media/image20.png)

14. Make sure all the sample tables have been successfully
    deployed.![](./media/image21.png)

15. Go back to your SQL Database. Copy **Server name** (1) and **SQL
    Database name** (2), paste them in a notepad, and then **Save** the
    notepad to use the information in the upcoming task.

> ![](./media/image22.png)

1.  Click **Home** to return to the main page

> ![](./media/image23.png)

2.  Click on **Resource groups**.

> ![](./media/image24.png)

3.  Click on the **ResourceGroup1** resource group.

> ![](./media/image25.png)

4.  Select **SQL server**

> ![](./media/image26.png)

5.  Navigate to Identity, switch the System assigned managed identity
    status to **On**, and then click **Save** to apply the change.

> ![](./media/image27.png)
>
> ![](./media/image28.png)

## Task 2: Create a Fabric workspace

In this task, you create a Fabric workspace. The workspace contains all
the items needed for this lakehouse tutorial, which includes lakehouse,
dataflows, Data Factory pipelines, the notebooks, Power BI datasets, and
reports.

1.  Open your browser, navigate to the address bar, and type or paste
    the following
    URL:+++https://app.fabric.microsoft.com/+++ press the **Enter** button
    and sign in with your credentials

    |  |   |
    |---|----|
    |Username	|+++@lab.CloudPortalCredential(User1).Username+++|
    |TAP	|+++@lab.CloudPortalCredential(User1).AccessToken+++|


2.  Fabric home page, select **+New workspace** tile.

> ![A screenshot of a computer Description automatically
> generated](./media/image29.png)

3.  In the **Create a workspace** pane that appears on the right side,
    enter the following details, and click on the **Apply** button.

| Property | Value |
|---------|-------|
| Name | +++FabricAgent-mirroringdatabase@lab.LabInstance.Id+++ |
| Advanced | Under **License mode**, select **Fabric** |
| Default storage format | Small dataset storage format |
| Template apps | Check **Develop template apps** |

> ![](./media/image30.png)

Note: To find your lab instant ID, select 'Help' and copy the instant
ID.

> ![A screenshot of a computer Description automatically
> generated](./media/image31.png)
>
> ![](./media/image32.png)

4.  Wait for the deployment to complete. It takes 2-3 minutes to
    complete.

> ![](./media/image33.png)

## Task 3: Create a Solution to Mirror Data using Azure SQL Mirroring

In this task, you will connect the Azure SQL Database to Microsoft
Fabric using Azure SQL Mirroring. You will select tables, create the
mirrored database, and validate that the data has synced successfully.

1.  Create a new lakehouse by clicking on the **+New item** button in
    the navigation bar

![](./media/image34.png)

1.  In the **Filter by keyword** search box, enter **+++Mirroed Azure
    SQL Database+++** and select the **Mirroed Azure SQL Database**
    item.

![](./media/image35.png)

2.  In the **Choose a database connection to get started** window,
    select **Azure SQL database**

![](./media/image36.png)

3.  In Connection settings tab enter the below detail and click on
    Connect button

| Field | Value |
|------|-------|
| Server | SQL server URL saved in **Task 2 → Step 15** |
| Database | Enter your SQL database |
| Username | +++sqladmin+++ |
| Password | +++password321!+++ |

![](./media/image37.png)

7.  In the **Choose data** window, select **Select all** and click on
    **Connect** button

> ![](./media/image38.png)

8.  In the Destination tab, click on **Create mirrored database**

> ![](./media/image39.png)

9.  Click **Refresh** to update and view the latest changes.

> ![](./media/image40.png)
>
> ![](./media/image41.png)

1.  In the left-sided navigation menu, navigate and click on
    ***FabricAgent-mirroringdatabaseXXXX***, as shown in the below
    image.

> ![](./media/image42.png)

## Task 4: Create a Data Agent and Connect the Mirrored Database

Here, you will create a new Fabric Data Agent and configure it to use
the mirrored Azure SQL Database as its data source. This agent will
respond to natural language prompts using the mirrored data.

1.  In the **Fabric** home page, select **+New item.**

![](./media/image43.png)

3.  In the **Filter by item type** search box, enter **+++data agent+++** and select the **Data agent.**

> ![](./media/image44.png)

4.  Enter **+++FabricDataAgent@lab.LabInstance.Id+++** as the Data
    agent name and select **Create**.

> ![](./media/image45.png)

5.  Select **Add data source** to configure a new data source.

> ![](./media/image46.png)

6.  Select your Mirrored database resource for this workshop

> ![](./media/image47.png)
>
> ![](./media/image48.png)

## Task 5: Test the agent

You will test the Data Agent by asking analytical questions like:

- Which product categories generate the highest sales?

- List products with high list price but low sales volume.

- Which cities have the highest number of customers?

This validates the agent’s ability to understand and respond to business
queries.

1.  Select the **SalesLT** schema for all tables.

2.  In the query panel of your Fabric data agent, type the question
    **+++Which product categories generate the highest sales?+++** and
    click the Send icon to view the agent’s response

![](./media/image49.png)

![](./media/image50.png)

3.  To test the agent, run the application and enter the sample
    questions to verify the responses.

++++List products with high list price but low sales volume.+++

![](./media/image51.png)

![](./media/image52.png)

+++**List the cities with the highest number of customers**+++

![](./media/image53.png)

> ![](./media/image54.png)

4.  Click **Agent instructions** from top menu.

![](./media/image55.png)

5.  Click Publish from the top menu and select **Publish**.

![](./media/image56.png)

![](./media/image57.png)

![](./media/image58.png)

6.  Now, click on **FabricAgent-mirroringdatabaseXXXXXX** on the
    left-sided navigation pane.

![](./media/image59.png)

## Task 6: Delete the Resources

1.  Select the **...** option under the workspace name and
    select **Workspace settings**.

> ![](./media/image60.png)

2.  Select **General** and **Remove this workspace.**

> ![A screenshot of a computer AI-generated content may be
> incorrect.](./media/image61.png)

3.  Click on **Delete** in the warning that pops up.

> ![](./media/image62.png)

4.  Wait for a notification that the Workspace has been deleted, before
    proceeding to the next lab.

> ![](./media/image63.png)

7.  Open a browser go to +++https://portal.azure.com+++ and sign in with
    your cloud slice account below.

8.  To delete resources , type **Resource groups** in the Azure portal
    search bar, navigate and click on **Resource
    groups** under **Services**.

![A screenshot of a computer Description automatically
generated](./media/image64.png)

9.  In the Resource groups page, select your resource group.

10. On the **Resource Group** home page, select all resources except
    **Fabric Capacity**, and then click **Delete**.

![](./media/image65.png)

11. In the **Delete Resources** pane that appears on the right side,
    navigate to **Enter “delete” to confirm deletion** field, then click
    on the **Delete** button

![](./media/image66.png)

![](./media/image67.png)

**Summary**

In this lab, you successfully created an Azure SQL Database and mirrored
its data into Microsoft Fabric using Azure SQL Mirroring. You then
configured a Fabric Data Agent to connect to the mirrored database and
analyze the data through natural language queries.

The agent was able to answer analytical questions such as identifying
high-selling product categories, products with high prices but low
sales, and cities with the largest number of customers. This
demonstrates how Microsoft Fabric can integrate operational data sources
with intelligent agents to simplify data exploration and enable faster
business insights.

This use case highlights the power of combining **data mirroring and
AI-powered data agents** to create interactive and intelligent data
experiences within the Microsoft Fabric ecosystem.


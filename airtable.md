# Interact with Airtable

Retool can connect to Airtable using either SQL via [Sequin](https://www.sequin.io/) or GraphQL via [BaseQL](https://www.baseql.com/).

## Connect via Sequin

Let's build a customer support dashboard from scratch using Airtable and Retool. The app will allow you to search through all your customers and then see all the contacts, tasks, meetings and invoices related to the customer in one, clean view.

![Alt text](https://docs.sequin.io/assets/retool-svoc/001_single_view_of_customer.png)

Retool doesn't come with a native Airtable integration. And while you can use Retool's REST API connector with Airtable, because Airtable's API comes with rate limits and pagination, it's a bit clumsy to work with.

Luckily, Retool comes out of the box with first-class support for SQL. So in this tutorial we'll use [Sequin](https://sequin.io/) to turn Airtable into a Postgres database. This will allow you to connect Airtable to Retool as a database so you can build the apps you need in minutes.

### Airtable Setup

For this tutorial we'll use Airtable's [customer success management template](https://airtable.com/templates/sales-and-customers/expWibrfsinGaxuX5/customer-success-management). This base is a great example of how Airtable can manage all your customer data and relationships. But it also showcases the limits of using Airtable when you need finer workflows or permissions. This is where Retool comes in.

![Alt text](https://docs.sequin.io/assets/retool-svoc/002_airtable_base.png)

First, add the Airtable customer success management template to your Airtable workspace:

1. Log into your [Airtable workspace](https://airtable.com/) and then open the [customer success management template](https://airtable.com/templates/sales-and-customers/expWibrfsinGaxuX5/customer-success-management) in a new tab.

2. Click the **Use Template** button to add the customer success management template to your workspace.

![Alt text](https://docs.sequin.io/assets/retool-svoc/003_copy_template.png)

### Sequin Setup

Now, let's turn the customer success management base in Airtable into a Postgres database that works with Retool using [Sequin](https://sequin.io/).

1. Go to [https://app.sequin.io/signup](https://app.sequin.io/signup) and create an account.

2. Connect your Airtable base by going through the tutorial or clicking the **Add Base** button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/004_add_base.png)

3. You'll be prompted to enter your Airtable API key. After that, select the **Customer Success Management** base and all its tables. Then click **Create**.

![Alt text](https://docs.sequin.io/assets/retool-svoc/005_create.png)

5. Sequin will immediately provision you a Postgres database and begin syncing all the data in your Airtable base to it. You'll be provided with credentials for you new database. Keep these handy as you'll use them to connect your Sequin database to Retool.

![Alt text](https://docs.sequin.io/assets/retool-svoc/006_connection.png)

### Retool Resource Setup

Now, just add your Sequin database to Retool like any other Postgres database:

1. In a new tab, log into your [Retool dashboard](https://retool.com/). In the top menu bar click **Resources** and then the blue **Create New **button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/007_create_resource.png)

2. Select **Postgres** from the list of resource types.

![Alt text](https://docs.sequin.io/assets/retool-svoc/008_select_postgres.png)

3. Enter the name for your resource (i.e. "Airtable - Customer Success") and then enter the **Host**, **Port**, **Database name**, **Database username**, and **Password** for your Sequin database. You can copy and paste these from Sequin (make sure there aren't any leading spaces). Then click the blue **Create resource** button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/009_enter_db_cred.png)

4. Retool will confirm that your resource was created. Click **Create an app** to return to the app page.

![Alt text](https://docs.sequin.io/assets/retool-svoc/010_confirm_resource.png)

### Retool App Setup

With Airtable successfully connected to Retool using Sequin, we are ready to build an app that shows you all your customer data in one view. Or, as some call it, a _single view of customer_.

1. On the Retool app page, click the blue **Create new** button and select the **Generate app from data** option.

2. Because your Airtable data is now in a Postgres database, Retool can jump-start your application with a table and search bar. This is a good foundation for your single view of customer application. So in the modal that appears, select the Sequin Postgres database you created (i.e. **Airtable - Customer Success**), the **accounts** table, and the **account** column as the initial data for the app. Then click the blue **Next** button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/012_select_initial_data.png)

3. Now, give your application a name. You can go with something like "Customer Central" and click the blue **Create app** button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/013_name_app.png)

4. Retool will now generate an app with a table and a search bar. Because we selected the **accounts** table and the **account** column, the app will initially show the entire **accounts** table and allow you to search by **account**. Give it a try. If you enter "long" in the search bar and hit enter, the table automatically shows "Long Beach College." Isn't that nifty!

![Alt text](https://docs.sequin.io/assets/retool-svoc/014_starting_app.png)

5. Now, let's begin to edit this app. Click the **Edit** button in the top right corner.

![Alt text](https://docs.sequin.io/assets/retool-svoc/015_start_edit.png)

6. First, we can remove the helpful hints that Retool loads into the template app. We don't need them. Select and delete the two text boxes at the top of the page as well as the extra button at the bottom. This will leave you with a clean table and search bar:

![Alt text](https://docs.sequin.io/assets/retool-svoc/016_clean_app.png)

### Create the customer accounts table

With the basic scaffolding of your Retool application in place, you can begin to tailor the app to our needs using SQL.

First, look at the query section in bottom panel. You can see that because Sequin has transformed your Airtable base into a proper Postgres database, you have access to all of Retool's SQL capabilities. For instance, you can see the entire schema of your base as well as the column types. And as you write SQL queries, you get the benefits of autocomplete:

![Alt text](https://docs.sequin.io/assets/retool-svoc/017_sql_editor.png)

Now, go ahead and edit the query that populates the account table so that you only pull in the data you need for each customer account:

```sql
select accounts.id, accounts.account, accounts.primary_csm ->> 'name' AS "CSM", accounts.partnership_growth, accounts.customer_stage, count(task_manager.id) as "tasks", accounts.next_meeting
from accounts
left join task_manager on task_manager.account[1] = accounts.id
where ({{ !searchBar.value }} or accounts.account ilike {{ '%' + searchBar.value + '%' }})
group by accounts.id;
```

This query is selecting just the columns you need from the `accounts` table. You'll note that the syntax for pulling in the `primary_csm` is a little different. The `primary_csm` column is a collaborator field in Airtable - so it's stored as JSON in your Sequin database. The `->> 'name'` syntax pulls out just the value of the `name` key in the JSON.

You'll also note that to bring in the number of tasks for each account you `JOIN` the `accounts` table with the `task_manager` table. Linked records appear in Airtable as arrays because it is possible for one records to be linked to many other records. In this case, you know that each task is linked to one account, so the `[1]` syntax extracts the one and only account from the array to complete the join.

The query then filters the results at the `WHERE` clause based on the value in the search bar. If nothing is entered in the search bar, the entire list of accounts is returned.

When you click to **Save and Run** button, the results of the query are shown as a preview. Everything looks good. So as a last step, rename this query to `list_accounts` by clicking and then editing the name of the query in the left bar.

![Alt text](https://docs.sequin.io/assets/retool-svoc/018_list_accounts.png)

Finally, let's make this table look good. With accounts table selected in the canvas, go into the inspector in the right panel to re-organize the columns and give them clear titles. Hide the `id` column by clicking the little "eye" icon since its more of a reference field. Lastly, adjust the table sort and pagination to your liking.

![Alt text](https://docs.sequin.io/assets/retool-svoc/019_clean_accounts_table.png)

With the table looking good, give it a title. Drag and drop a `text` component above the table.

![Alt text](https://docs.sequin.io/assets/retool-svoc/020_add_title.png)

Then, in the inspector, use markdown to give the text component a value of `# 📂 Accounts`. This will make the text an `H1` header.

![Alt text](https://docs.sequin.io/assets/retool-svoc/021_final_accounts_table.png)

### Add the remaining customer data

Voila, your first table is complete. The accounts table is now pulling data live from Airtable and is easily searchable. With SQL and Retool at your fingertips, that was pretty quick. Now, lets add the remaining customer data.

We're going to repeat the same steps we used to create the account table, but in a slightly more efficient flow:

1. Layout the rest of the application by dragging and dropping components.
2. Add data to each of the components.
3. Make the components look good.

#### 1. Layout the remaining tables and titles

In order to show all the relevant information for any given account, your app should show the **Contacts**, **Tasks**, **Meetings**, and **Invoices** for the account selected. Each of these will be their own table. So we can layout the entire application by dragging and dropping tables and text components into the canvas. In a couple clicks you'll end up with an app that looks something like this:

![Alt text](https://docs.sequin.io/assets/retool-svoc/022_layout.png)

> Note: as you drag tables onto the canvas, Retool will auto populate them with the `list_accounts` query. Don't worry about it.

#### 2. Add data to the tables

Filling each table up with data is a two step process:

1. Write a query to retrieve the data for the selected account.
2. Configure the table to pull in the data from the query.

Let's start with the contacts table.

First, open the bottom panel and click **New** to add a new query:

![Alt text](https://docs.sequin.io/assets/retool-svoc/023_create_query.png)

To retrieve the contacts, we can write a simple SQL query:

```sql
SELECT contacts.name, contacts.role, contacts.email, contacts.linkedin
FROM contacts
WHERE contacts.company[1] = {{table.selectedRow.data.id}};
```

Here, the `SELECT` statement is pulling in the specific columns for the table. While the `WHERE` clause is filtering the returned results based on the account selected in the accounts table (i.e. `table`).

> You might be wondering why you are using `contacts.company[1]` in the `WHERE` clause. In short, the `company` column is originally a linked record field in Airtable. Linked records can potentially hold more than one value. So in your Sequin Postgres database, `company` is stored as a Postgres array in case there are multiple values. The `[1]` is how you select the first value in the Postgres Array. For a complete explanation check out the [Sequin cheat sheet](https://docs.sequin.io/cheat-sheet) or this [dev.to tutorial](https://dev.to/thisisgoldman/go-further-with-airtable-using-postgres-arrays-4hia).

When you select an account in the accounts table (I recommend selecting `Soho Real Estate` account for testing) and then click the **Save and Run** button, you'll see this query returns just the contacts for the selected account. Exactly what you are looking for.

Name the query `list_contacts` so it's easy to identify. Now, click on the empty table under the Contacts heading in the canvas. The inspector on the right will now show the settings for the table. Enter `{{ list_contacts.data }}` into the data field. This will fill the table with the data returned from the `list_contacts` query.

![Alt text](https://docs.sequin.io/assets/retool-svoc/024_contacts.png)

Just repeat this pattern to populate the remaining tables. Here are the PostgreSQL queries you can use for each table. They all follow the same pattern:

**Tasks** (`list_tasks`)

```sql
SELECT task_manager.task_name, task_manager.status, task_manager.assignee, task_manager.due_date
FROM task_manager
WHERE task_manager.account[1] = {{table.selectedRow.data.id}};
```

**Meetings** ('list_meetings`)

```sql
SELECT meetings.date, meetings.name, meetings.meeting_complete
FROM meetings
WHERE meetings.account[1] = {{table.selectedRow.data.id}};
```

Invoices ('list_invoices`)

```sql
SELECT invoices.invoice_number, invoices.invoice_frequency, invoices.invoicing_status, invoices.invoice_amount, invoices.invoice_period_beginning_date, invoices.invoice_period_end_date, invoices.invoice_type
FROM invoices
WHERE invoices.account[1] = {{table.selectedRow.data.id}};
```

After creating and connecting all your queries, your app should now feel almost complete. You can search for an account like "Soho Real Estate", select it, and all the tables will show the details for the account.

![Alt text](https://docs.sequin.io/assets/retool-svoc/025_working_app.png)

#### 3. Make the tables look pretty

To make the app easy to use, let's make each table a little more functional.

As a first step, to make these additional tables easy to read, turn on **Compact mode** in the inspector. This will allow each table to show more information.

![Alt text](https://docs.sequin.io/assets/retool-svoc/026_compact_mode.png)

Next, add readable column names and column types to each table. Simply select the table in the canvas, then select the column in the inspector and give it a name and type. For example, change the column title for 'invoice_amount' to 'Amount' and make it's column type 'USD (dollars)'.

![Alt text](https://docs.sequin.io/assets/retool-svoc/027_name_and_type.png)

Retool has many options to make these tables sparkle, but with just these minor changes your app should be easy to use.

You now have a single view of customer app with all your key customer data in one, clean window.

![Alt text](https://docs.sequin.io/assets/retool-svoc/028_final_read_app.png)

### Writing to Airtable

Let's say you want to allow your customer success agents to create new tasks for accounts.

To add this functionality, you'll first set up the components and then connect the data.

In this case, you'll use a simple modal to collect the details of the task.

#### The Sequin Proxy

Sequin promotes a one-way data-flow. You read data from Airtable using the Sequin Postgres database, and then you write to Airtable through the [Sequin Proxy](https://docs.retool.com/reference#writes).

The proxy sits between your code and Airtable's API so that any CREATES, UPDATES, or DELETES are written to both Airtable and your Sequin database simultaneously. This ensures your Retool app is responsive and fast.

![Alt text](https://docs.sequin.io/assets/retool-svoc/039_proxy_flow.png)

To use the Sequin Proxy, you just need to change the hostname for all your Airtable API calls from `https://api.airtable.com` to `https://proxy.sequin.io/api.airtable.com`. After that change, you use the Airtable API like you normally would.

So in this case, to create a new task on Airtable, you'll perform a `POST` to Airtable via the Sequin Proxy. This update will be applied to Airtable and your Sequin Postgres database simultaneously, which means Retool will always display the latest data.

#### Add a modal

From the components list, find the modal component, and then drag and drop its button onto the canvas just above the Task table.

You can make the button green with the text "New Task".

![Alt text](https://docs.sequin.io/assets/retool-svoc/029_add_modal.png)

Now, let's build a form in the modal to collect the details of a new task.

When you click on the "New Task" button the modal will appear. The modal doesn't need to be huge, so you can adjust the height of the modal to `350px` in the inspector.

Now, drag and drop several components into the modal:

- A text component that says `Add a new task`
- A text input component with the label `Task`
- A date and time selector component with the label `Due Date`
- A drop down component with the label Status. Set the values and display values to be the following array: `["Not Started", "In Progress", "Complete", "On Hold"]`.
- Two buttons at the bottom. One that reads Save and the other Cancel.
- By the end, your modal should look something like this:

![Alt text](https://docs.sequin.io/assets/retool-svoc/030_build_modal.png)

#### Connect the modal

First, make sure the user can close the modal:

Open the bottom panel and click to create a new query
For the resource, select `Run JS code (JavaScript)`
Enter `modal1.close()` for the query and save the query as `close_modal`.

![Alt text](https://docs.sequin.io/assets/retool-svoc/031_close_modal.png)

Trigger the `close_modal` query when the user closes the modal or clicks the cancel button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/032_connect_close.png)

Next, let's trigger the creation of a new task when the user clicks save.

To do so, we first need to add the Sequin Proxy as a resource.

Click the Retool icon in the upper left corner, and select Resources.

![Alt text](https://docs.sequin.io/assets/retool-svoc/033_click_resource.png)

On the Resource page, click the blue **Create New** button and then select **REST API** from the options.

![Alt text](https://docs.sequin.io/assets/retool-svoc/034_select_api.png)

Configure your the Airtable API:

- Name the API `Airtable API - Customer Success`
- To get the Base URL, go to [https://airtable.com/api](https://airtable.com/api) and select the Customer Success Management base. In the middle of the page you'll see your base ID (the same one you used for Sequin) in green. Just append that base ID to this URL: `https://proxy.sequin.io/api.airtable.com/v0/{YOUR_BASE_ID}`
- Then, retrieve your API key from the your [Airtable account page](https://airtable.com/account) and add these two headers:
  - Authorization: Bearer {YOUR_API_KEY}
  - Content-Type: application/json

Then click the blue **Create resource** button.

![Alt text](https://docs.sequin.io/assets/retool-svoc/035_configure_API.png)

Navigate back to your app, open the bottom panel and click to create a new query as follows:

- For the resource, select the Airtable API resource that you just created.
- For action type, select `POST` and enter `Task%20Manager` (the `%20` is the URL-friendly version of a space)
- For body, select `Raw` and enter the following JSON:

```json
'{
  "records": [
    {
      "fields": {
        "Status": "{{select1.value}}",
        "Due Date": "{{datetimepicker1.formattedString}}",
        "Account": [
          "{{table.selectedRow.data.id}}"
        ],
        "Task name": "{{textinput2.value}}"
      }
    }
  ]
}'
```

The body of this API query is creating a new record in the `Task Manager` table. The details of the task are being pulled from the values entered in the modal.

As a last step, after a new task is created, we want to refresh the data in the app. To do so, in the **On success trigger** field at the bottom of the page, select the `list_accounts` query. This will cause all the tables on the page to refresh.

Click **Save** and name the query `ceate_new_task`.

![Alt text](https://docs.sequin.io/assets/retool-svoc/036_create_task_query.png)

When the user clicks save in the modal, we want to create a new task, close the modal, and then refresh the data on the page.

So as a last step, let's create one more query to string these functions together.

Click to create a new query and select `Run JS Code (JavaScript)` as the resource. Then enter this code:

```js
create_new_task.trigger();
modal1.close();
```

1. The first line triggers a new task.
2. The second line closes the modal.
3. Click the save button and name this query `run_modal`:

![Alt text](https://docs.sequin.io/assets/retool-svoc/037_run_modal.png)

Now, let's connect this new query to the modal. We want to trigger the run_modal query when the user clicks the save button. So open the modal, and select the Save button. In the inspector, select to `Run a query` on click. Select `run_modal` from the drop down.

![Alt text](https://docs.sequin.io/assets/retool-svoc/038_final_connect.png)

Last step! Test it out. Open the modal and create a task. Click the save button. The modal will close and your new task appears instantly ✨

Now, with the power of SQL and the Sequin Proxy, you can build Retool apps on your Airtable data in no time.

## Connecting via BaseQL

_Note: BaseQL is only available on the Airtable Pro Plan._

To get started, we'll need to install BaseQL from the Airtable Marketplace. Head over to your marketplace tab and search for BaseQL:

![Alt text](https://d3399nw8s4ngfo.cloudfront.net/docs/rd/345cf32-Screen_Shot_2020-11-20_at_2.55.44_PM.png)

Once you've got BaseQL installed, head over to whichever Airtable base you want to query and use in Retool. We've got a base of Sharks (from Shark Tank, of course) queued up here that we want to use in Retool.

Click "Apps" on the right side, and select BaseQL.

![Alt text](https://d3399nw8s4ngfo.cloudfront.net/docs/rd/3423896-Screen_Shot_2020-11-20_at_2.35.43_PM.png)

First, you'll need to get your Airtable API Key. Click on your account icon on the top right of your screen.

![Alt text](https://d3399nw8s4ngfo.cloudfront.net/docs/rd/83bdaa4-Screen_Shot_2020-11-20_at_2.36.10_PM.png)

This account doesn't have an API Key yet, so we'll need to generate one with the "Generate API Key" button.

![Alt text](https://d3399nw8s4ngfo.cloudfront.net/docs/rd/914c7ce-Screen_Shot_2020-11-20_at_2.36.31_PM.png)

Once you've got this API Key copied to your clipboard, head back to your base and paste it into BaseQL's API Key field. BaseQL gives you two options - you can use an inline IDE for writing queries ("GraphQL Explorer"), or open the BaseQL app directly and write queries there. Either way, you should see an endpoint that BaseQL generates on their servers. It should look something like this:

`https://api.baseql.com/airtable/graphql/appb5EQO5dqP1YdprX`

This is the endpoint we'll query in Retool.

Pop open a Retool app (or [create an account](https://login.retool.com/auth/signup) if you don’t have one), head down to your query editor on the bottom of your screen, and create a new query with the "+ new" button. Select "GraphQL" from the dropdown, and paste your endpoint from above into the "URL" field. Then you should be good to go!

### A basic BaseQL Airtable query

Going back to our Sharks example, here's a simple GraphQL query that pulls in some of the data from our base:

```javascript
query {
 sharks {
  sharkName
  sharkNumber
  sharkImage
  biography
  }
}

Click preview, and you should see your data returned in a nice JSON format (thank you GraphQL).

![](https://d3399nw8s4ngfo.cloudfront.net/docs/rd/fcaccda-Screen_Shot_2020-11-20_at_2.44.19_PM.png)

To get our data into a table, we can reference `{{ AirtableQuery.data.sharks }}`:

![](https://d3399nw8s4ngfo.cloudfront.net/docs/rd/5e1d051-Screen_Shot_2020-11-20_at_2.44.40_PM.png)
```

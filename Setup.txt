
# Setup Instructions:



# SharePoint


## Ticket List Creation
If you have the ability to use the Graph API, that is the fastest option.  You don't need an azure app or anything setup, Graph Explorer lets you interact with your data without one. You can create the list manually, just reference the JSON file for the columns and their settings.  Ignore this section if you will be creating the list manually.  

1.	Go here and sign-in on the left side: https://developer.microsoft.com/en-us/graph/graph-explorer
1.	Click on the ellipses next to your name and click Select Permissions.  Grant yourself:
    1.	Sites.Manage.All
1.	Will this list live on your default SharePoint site?  If yes, move to step 4.  If not, we need to find the site id.
    1. In Graph Explorer, we can run a query to find that.
        1. GET https://graph.microsoft.com/v1.0/sites/HOSTNAME:/sites/SITENAME
	        1. _NOTE THE COLON AFTER HOSTNAME_
            1. Where HOSTNAME = yourtenant.sharepoint.com
            1. Where SITENAME = name of the site as shown in the URL
            1. The ID field returned will look like _skycamplabs.sharepoint.com, *c9622d08-a30b-4acd-a363-bb7a664a5d18* ,8207f6c1-e48e-4c36-9688-c5b69e93706f_, 3 values separated by commas.  You want to copy the middle value.
    1. Alternatively, append _\_api/site/id_ to the end of the site url and copy the Edm.Guid field 
1.	Copy the the SyncroSync-createList.json script and modify the JSON (optional)
    1.	Line 2: This is the name of the SharePoint list.  You can change that if you’d like.
    1.	For any of the columns, you can change the Display Name value (NOT the name value).
    1.	Do not change anything else
1.	Create the list
    1.	In Graph Explorer, start a new query
        1. POST
        1. If using root site:   https://graph.microsoft.com/v1.0/sites/root/lists
        1. If using different site:  https://graph.microsoft.com/v1.0/sites/{site-id}/lists 
            1. Where site-id is the value you copied above.  You do not need the { }
        1. Under the Request Header tab, add a key/value pair
            1. Key:  content-type
            1. Value:  application/json
            1. Click Add
        1. Under the Request Body tab, paste in the JSON data from above
        1. Click Run Query
        1. If successful, you should get a ‘Created – 201’ response with the new list properties
            1.	If you get a permission error, give yourself the following permissions and try again
                1. Sites.Read.All
                1. Sites.ReadWrite.All


### Ticket List Formatting

1. There is a Link column that will open the ticket in Syncro, but if you’d also like to be able to click on the ticket number to open it in Syncro, do the following (this does not apply when viewing the item in the form pane, so that’s what the Link column is for)
  1.	Right-Click on the Ticket Number column header and go to Column Settings -> Format this column.  Click on Advanced Mode at the bottom of the window that pops up.
  2.	Replace YOUR-SYNCRO-DOMAIN with your actual Syncro subdomain, then paste this in:

    {
      "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
      "elmType": "a",
      "style": {
        "color": "white"
      },
      "txtContent": "@currentField",
      "attributes": {
        "target": "_blank",
        "href": "='https://YOUR-SYNCRO-DOMAIN.syncromsp.com/tickets/' + [$TicketID]"
      }
    }

_If you use a light theme, change the “color” value from “white” to “black” or whatever fits your theme.  You can use hex color codes, just wrap it in double quotes._


### Create better default view
1. In your ticket list, click the gear icon up top and go to list settings
2. Click Create View at the very bottom
3. Choose Standard View
4. Name: Whatever you want
	1. Check 'Make this the default view'
	1. All the default columns should already be selected, but you want to uncheck Comments.  
	1. Sort:
		1. First sort by Resolved At in descending order
	1.  Item Limit:
		1. 50 - Display items in batches
	1. Click OK



## ReOpen List Creation
Following the same steps as the ticket list creation, use the SyncroSync-ReOpenList.json file to create that list
Post-creation- Rename the Title field to Subject if you'd prefer

## KB ToDo List
If you want to utilize the KB functionality, create a shared ToDo list.



# Import Solution into Power Automate
1. Go to the Power Automate site https://us.flow.microsoft.com/
    1. Click on Solutions
    1. Click Import 
    1. Select the SyncroSync_vx.zip file and click Next
    1. Assuming it doesn't give any errors, click Import
    
1. You'll be prompted to select or create some Connections.  This is how Flow authenticates to various services.
	1. For each Connection, Select 'New Connection' and select the account to use.


3. You'll be prompted to set some environment variables now.  You must put a value in for each.
    1. Notification List- List of email addresses to notifiy if the flow fails, separated by semi-colon
    1. SP Site Address- Full URL or ID of the site where your list will live.  If root site, it's just https://yourdomain.sharepoint.com/
    1. Syncro Subdomain - The X in https://X.syncromsp.com
    1. SP Ticket List Name - The exact name or ID of the ticket list
    1. Ticket Type Ignore - If you want to prevent certain Problem Types from syncing, you can add those here.  Must be formatted like so:   ["Alerts","whatever"]   If none, just put in [""]
    1. SP ReOpen List Name - The exact name or ID of the reopen list
    1. ToDo List Name -- Exact name of the list in ToDo.  If you don't want to utilize, put N/A
    1. Enable KB - Set to No if you don't want to utilize the KB ToDo functionality
    1. Enable ReOpen - Set to No if you don't want to utilize the ticket ReOpen functionality
    1. Timezone - Enter the name of your timezone.  Standard US ones are-
    	1. Eastern Standard Time
    	1. Central Standard Time
    	1. Mountain Standard Time
    	1. Pacific Standard Time
    	1. See here for an entiere list- https://ss64.com/nt/timezones.html
    2. Syncro API Key - Create an API Token in Syncro (Admin -> API Tokens)
    	3. Requires 
    		4. Ticket - Edit
    		5. Ticket - View Details
    		6. Ticket - List/Search



1. Click Import
1. Once finished, click on Publish All Customizations in the menu.
2. Then click on the SyncroSync flow to open, then click Edit
3. Save the flow to generate webhook URL
4. Click on the 'When a HTTP request is received' action to expand it
5. Copy the HTTP POST URL provided there.  



# Syncro

## Notification Set
1. Go to Admin -> Notification Center
1. Create a New Notification Set
    1. Name: SyncroSync (or whatever you want, has no impact elsewhere)
    1. Paste the Flow webhook into the Webhook URL box
    1. Find the 'Ticket - Was Resolved' event and check the Webhook box
    1. Click Create Notification Set

### To utilize the ReOpen and KB functionality, configure a Custom Ticket Field Type.  
If you don't want to utilize either, just don't create the fields

1. Go to Admin -> Tickets -> Ticket Custom Fields
1. Create New Custom Field Type.  Name it whatever you'd like, we used 'Post-Ticket Actions'
1. Create New Field (Names must be exactly what it defined below)
	1. Name: Reopen On - Date Field - Hidden=true
	1. Name: KB Article - Check box - Hidden=true
	1. Name: KB Title - Text Field - Hidden=true


Special Note:  Reopen on daily check
Currently, you can't use environment variables in certain flow triggers, including the Recurrence trigger used here.  This means you can't change the timezone or time of day that the flow runs.  It's set to run at 6am EST.
If that doesn't work for you, you can disable the Reopen Daily Check flow in the solution and just import the flow itself.  That requires some extra customization.  Please see the ReopenDailyCheck document for further instructions.

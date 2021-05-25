# SyncroSync
Syncro Sync is a simple utility that will automatically sync your Syncro or RepairShopr tickets to SharePoint.  This provides a couple great benefits:

-	Searching!  You can search, sort, and filter based on any ticket criteria.  Yes, you can search ticket comments as well.
-	If you happen to leave Syncro, you have all your ticket history to reference.

Our production SharePoint list contains nearly 75,000 tickets, search is still fast and accurate.

This will also work for RepairShopr, just substitute repairshopr for syncro wherever you see it.

### Additional (optional) features:

- Ticket Reopen –  Set a date, close the ticket, it’ll automatically reopen on the date specified

- KB Add – If working a ticket that needs further documentation, you can mark it in the ticket and it’ll automatically get added to a shared To-Do list



## Requirements

- SharePoint Online
- Power Automate / Flow with premium connectors (Requires Power Automate Per User license, $15/mo)

## How it works:

When a ticket is marked as Resolved, a webhook fires and initiates the Flow.  The ticket data then gets processed and added to SharePoint.  In the event a ticket was resolved, re-opened, and resolved again, it will simply overwrite the previous entry.



# Setup Instructions:

## SharePoint List Creation
If you have the ability to use the Graph API, that is the fastest option.  You don't need an azure app or anything setup, Graph Explorer lets you interact with your data without one. You can create the list manually, just reference the JSON file for the columns and their settings.  Ignore this section if you will be creating the list manually.  

1.	Go here and sign-in on the left side: https://developer.microsoft.com/en-us/graph/graph-explorer
1.	Click on the gear icon next to your name and click Select Permissions.  Grant yourself:
    1.	Sites.Manage.All
1.	Will this list live on your default SharePoint site?  If yes, move to step 4.  If not, we need to find the site id.
    1. In Graph Explorer, we can run a query to find that.
        1. GET https://graph.microsoft.com/v1.0/sites/HOSTNAME:/sites/SITENAME
            1. Where HOSTNAME = yourtenant.sharepoint.com
            1. Where SITENAME = name of the site as shown in the URL
            1. Copy the id property that is returned
    1. Alternatively, append _\_api/site/id_ to the end of the site url and copy the Edm.Guid field 
1.	Modify the JSON code (optional)
    1.	Line 2: This is the name of the SharePoint list.  You can change that if you’d like.
    1.	For any of the columns, you can change the Display Name value (NOT the name value).
    1.	Do not change anything else
1.	Create the list
    1.	In Graph Explorer, start a new query
        1.	POST
        1.	If using root site:   https://graph.microsoft.com/v1.0/sites/root/lists
        1.	If using different site:  https://graph.microsoft.com/v1.0/sites/{site-id}/lists 
            1.	Where site-id is the value you copied above.  You do not need the { }
        1.	Under the Request Header tab, add a key/value pair
            1.	Key:  content-type
            1.	Value:  application/json
            1.	Click Add
        1.	Under the Request Body tab, paste in the JSON data
        1.	Click Run Query
        1.	If successful, you should get a ‘Created – 201’ response with the new list properties
            1.	If you get a permission error, give yourself the following permissions and try again
                1.	Sites.Read.All
                1.	Sites.ReadWrite.All

## SharePoint List Formatting

1. There is a Link column that will open the ticket in Syncro, but if you’d also like to be able to click on the ticket number to open it in RS, do the following (this does not apply when viewing the item in the form pane, so that’s what the Link column is for)
  1.	Right-Click on the Ticket Number column header and go to Column Settings -> Format this column.  Click on Advanced Mode at the bottom of the window that pops up.
  2.	Replace YOUR-RS-DOMAIN with your actual Syncro subdomain, then paste this in:

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


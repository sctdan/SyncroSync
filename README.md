# SyncroSync
Syncro Sync is a simple utility that will automatically sync your Syncro or RepairShopr tickets to SharePoint.  This provides a couple great benefits:

-	Searching!  You can search, sort, and filter based on any ticket criteria.  Yes, you can search ticket comments as well.
-	If you happen to leave Syncro, you have all your ticket history to reference.
-	
Our production SharePoint list contains nearly 75,000 tickets, search is still fast and accurate.

This will also work for RepairShopr, just substitute repairshopr for syncro wherever you see it.

Additional (optional) features:

-Ticket Reopen –  Set a date, close the ticket, it’ll automatically reopen on the date specified

-KB Add – If working a ticket that needs further documentation, you can mark it in the ticket and it’ll automatically get added to a shared To-Do list



Requirements

SharePoint Online
Power Automate / Flow with premium connectors (Requires Power Automate Per User license, $15/mo)

How it works:

When a ticket is marked as Resolved, a webhook fires and initiates the Flow.  The ticket data then gets processed and added to SharePoint.  In the event a ticket was resolved, re-opened, and resolved again, it will simply overwrite the previous entry.

---
name: check-orders
description: Check Gmail for new TCGPlayer orders and listing notifications, then process them into the vault database.
disable-model-invocation: true
context: fork
agent: tcgplayer-order-processor
argument-hint: "[optional: number of days to override search window]"
---

Check my Gmail inbox for TCGPlayer notification emails since the last check date stored in `Database/.last-check`. If a number of days is provided as an argument, override the search window to the last $ARGUMENTS days instead.

Search for ALL matching emails (read and unread) and deduplicate against existing database records:
1. Listing notifications — emails about newly listed cards on TCGPlayer
2. Sale notifications — emails about new sales/orders on TCGPlayer

Process each NEW (non-duplicate) email:
- Parse the email content to extract card and order details
- Create or update the appropriate markdown files in the Database/ directory
- Move cards through their lifecycle (Unlisted > Listed > Sold) as needed
- Create sale records for new orders

After processing, provide:
1. A full summary of all changes made (files created, moved, updated)
2. A "Sold Pending Shipping" report — all orders across the database that need to be shipped
3. A "Payments Received" report — all completed transactions across the database
4. Any issues that need my manual attention

Update `Database/.last-check` with the current datetime when finished.

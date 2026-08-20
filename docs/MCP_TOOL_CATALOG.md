# MCP Tool Catalog

This document lists every MCP tool exposed by the server and classifies each as **Read-only**, **Write/Update**, or **Mixed (read/write)**.

- Source of truth: `/src/handlers/tool.definitions.ts`
- Total tools: **101**

## Access type definitions

- **Read-only**: fetch/search/list/get operations that do not mutate Autotask data.
- **Write/Update**: create/update/delete operations that mutate Autotask data.
- **Mixed (read/write)**: routing or raw execution tools that can call either read or write operations depending on arguments.

## Full tool list

| Tool | Access type | Category | Description |
| --- | --- | --- | --- |
| `autotask_test_connection` | Read-only | utility | Test Autotask API connection |
| `autotask_search_companies` | Read-only | companies | Search companies by name or status. Max 200/page. |
| `autotask_create_company` | Write/Update | companies | Create new company record |
| `autotask_update_company` | Write/Update | companies | Update company record. invoiceTemplateID sets payment terms (103=Due on Receipt, 104=NET 30). Billing address fields separate from regular address. |
| `autotask_get_company_site_configuration` | Read-only | companies | Get company site configuration records. Call first to discover available fields. |
| `autotask_update_company_site_configuration` | Write/Update | companies | Update company site configuration. Fields are tenant-defined; call get first. |
| `autotask_search_contacts` | Read-only | contacts | Search contacts by name, email, or company. Max 200/page. |
| `autotask_create_contact` | Write/Update | contacts | Create new contact record |
| `autotask_update_contact` | Write/Update | uncategorized | Update contact record. Only provided fields are changed. |
| `autotask_search_tickets` | Read-only | tickets | Search tickets by company, queue, status, priority. Use autotask_get_ticket_details for full data. Max 500/page. |
| `autotask_get_ticket_details` | Read-only | tickets | Get full ticket details including notes, time entries, and custom fields. |
| `autotask_create_ticket` | Write/Update | tickets | Create new ticket record |
| `autotask_update_ticket` | Write/Update | tickets | Update ticket record. Only provided fields are changed. |
| `autotask_get_ticket_charge` | Read-only | tickets | Get a specific ticket charge by ID |
| `autotask_search_ticket_charges` | Read-only | tickets | Search ticket charges (materials, costs, expenses). Provide ticketId for best performance. Max 10 if unfiltered. |
| `autotask_create_ticket_charge` | Write/Update | tickets | Create charge on ticket for materials, costs, or expenses. |
| `autotask_update_ticket_charge` | Write/Update | tickets | Update an existing ticket charge. Only fields provided will be changed. |
| `autotask_delete_ticket_charge` | Write/Update | tickets | ⚠ DESTRUCTIVE — IRREVERSIBLE. Permanently deletes a ticket charge |
| `autotask_get_ticket_history` | Read-only | tickets | Get a single ticket history entry by ID. Each entry records one audited change to a ticket field (who, when, before/after). |
| `autotask_search_ticket_history` | Read-only | tickets | Get the audit trail of field changes for a ticket (status transitions, assignment changes, priority edits, etc.). Use this to answer questions like "when did this ticket move from In Progress to Waiting Customer" or "who changed the priority". Returns entries ordered by Autotask; sort/filter client-side if needed. |
| `autotask_create_time_entry` | Write/Update | time and billing | Create a time entry in Autotask. Can be tied to a ticket, task, or project, OR created as "Regular Time" (no parent) for meetings, admin work, etc. For Regular Time, specify a category like "Internal Meeting", "Office Management", "Training", etc. |
| `autotask_search_projects` | Read-only | projects | Search for projects in Autotask. Returns 25 results per page by default. Use page parameter for more results. |
| `autotask_create_project` | Write/Update | projects | Create a new project in Autotask |
| `autotask_update_project` | Write/Update | uncategorized | Update an existing project in Autotask. Only the fields you provide will be updated. Common use case: set status=5 to mark a project Complete. |
| `autotask_search_resources` | Read-only | resources | Search for resources (users) in Autotask. Returns 25 results per page by default. Use page parameter for more results. |
| `autotask_get_ticket_note` | Read-only | tickets | Get a specific ticket note by ticket ID and note ID |
| `autotask_search_ticket_notes` | Read-only | tickets | Search for notes on a specific ticket. Iterating across many tickets trips Autotask's per-integration API threshold — scope the parent list first. |
| `autotask_create_ticket_note` | Write/Update | tickets | Create a new note for a ticket |
| `autotask_search_ticket_checklist_items` | Read-only | uncategorized | List all checklist items on a ticket, including their completion status. Checklist items are a sub-resource of a ticket and cannot be queried without a ticket ID. |
| `autotask_create_ticket_checklist_item` | Write/Update | uncategorized | Add a new checklist item to a ticket. |
| `autotask_update_ticket_checklist_item` | Write/Update | uncategorized | Update a checklist item on a ticket — edit text, mark complete/incomplete, or change position. |
| `autotask_delete_ticket_checklist_item` | Write/Update | uncategorized | ⚠ DESTRUCTIVE — IRREVERSIBLE. Permanently deletes a checklist item |
| `autotask_get_project_note` | Read-only | projects | Get a specific project note by project ID and note ID |
| `autotask_search_project_notes` | Read-only | projects | Search for notes on a specific project. Fan-out across many projects trips Autotask's API threshold (see issue #69) — scope the parent list (status, company, date range) first. |
| `autotask_create_project_note` | Write/Update | projects | Create a new note for a project |
| `autotask_get_company_note` | Read-only | company notes | Get a specific company note by company ID and note ID |
| `autotask_search_company_notes` | Read-only | company notes | Search for notes on a specific company. Iterating across many companies trips Autotask's API threshold — scope the parent list first. |
| `autotask_create_company_note` | Write/Update | company notes | Create a new note for a company |
| `autotask_get_ticket_attachment` | Read-only | tickets | Get a ticket attachment. With includeData=false (default) returns metadata only — fast, suitable for browsing. With includeData=true returns the base64 binary content via the top-level /TicketAttachments/{id} endpoint (the child endpoint never populates data). The attachment is verified to belong to the given ticketId. Oversized binaries are stripped from the response with a dataOmittedReason field — Autotask attachments can be up to 3 MB, which is ~4 MB as base64 and may exceed the MCP client tool-result limit (~1 MB). |
| `autotask_search_ticket_attachments` | Read-only | tickets | Search for attachments on a specific ticket. Each parent triggers a separate query — scope the parent ticket list before iterating. |
| `autotask_create_ticket_attachment` | Write/Update | tickets | Upload a file attachment to an existing ticket. The file content must be passed as a base64-encoded string in the `data` field (MCP is JSON-RPC, so binary bytes must be base64-encoded). Autotask enforces a 3 MB hard limit on ticket attachments; this tool validates the decoded size before calling the API and returns a clear error if the limit is exceeded. Example: { ticketId: 12345, title: "screenshot.png", data: "iVBORw0KGgoAAAANSUhEUgAA..." } |
| `autotask_get_expense_report` | Read-only | time and billing | Get a specific expense report by ID |
| `autotask_search_expense_reports` | Read-only | time and billing | Search for expense reports with optional filters |
| `autotask_create_expense_report` | Write/Update | time and billing | Create a new expense report |
| `autotask_create_expense_item` | Write/Update | time and billing | Create an expense item on an existing expense report |
| `autotask_get_quote` | Read-only | financial | Get a specific quote by ID |
| `autotask_search_quotes` | Read-only | financial | Search for quotes with optional filters |
| `autotask_create_quote` | Write/Update | financial | Create a new quote |
| `autotask_get_opportunity` | Read-only | financial | Get a specific opportunity by ID |
| `autotask_search_opportunities` | Read-only | financial | Search for opportunities with optional filters |
| `autotask_create_opportunity` | Write/Update | financial | Create a new sales opportunity in Autotask |
| `autotask_get_product` | Read-only | products and services | Get a specific product by ID |
| `autotask_search_products` | Read-only | products and services | Search for products with optional filters |
| `autotask_get_service` | Read-only | products and services | Get a specific service by ID |
| `autotask_search_services` | Read-only | products and services | Search for services with optional filters |
| `autotask_get_service_bundle` | Read-only | products and services | Get a specific service bundle by ID |
| `autotask_search_service_bundles` | Read-only | products and services | Search for service bundles with optional filters |
| `autotask_get_quote_item` | Read-only | financial | Get a specific quote item by ID |
| `autotask_search_quote_items` | Read-only | financial | Search for quote items, typically filtered by quote ID |
| `autotask_create_quote_item` | Write/Update | financial | Create a line item on a quote. Set exactly ONE item reference (serviceID, productID, or serviceBundleID). Required: quoteId, quantity. Defaults: unitDiscount=0, lineDiscount=0, percentageDiscount=0, isOptional=false. |
| `autotask_update_quote_item` | Write/Update | financial | Update an existing quote item (quantity, price, etc.) |
| `autotask_delete_quote_item` | Write/Update | financial | ⚠ DESTRUCTIVE — IRREVERSIBLE. Permanently deletes a quote item |
| `autotask_search_configuration_items` | Read-only | configuration items | Search for configuration items in Autotask with optional filters |
| `autotask_search_contracts` | Read-only | financial | Search for contracts in Autotask with optional filters |
| `autotask_get_contract` | Read-only | financial | Get a single contract by ID (header fields only, no service lines) |
| `autotask_list_expiring_contracts` | Read-only | financial | List contracts whose end date falls within the next N days (expiring-contracts report). Optionally include already-expired contracts, and scope to one company or the whole org. |
| `autotask_search_invoices` | Read-only | financial | Search for invoices in Autotask with optional filters |
| `autotask_get_invoice_details` | Read-only | uncategorized | Get a single Autotask invoice with its nested line items (billing items posted to the invoice). Use for finance workflows that need to see exactly what an invoice contains. |
| `autotask_search_tasks` | Read-only | projects | Search for tasks in Autotask. Returns 25 results per page by default. Use page parameter for more results. |
| `autotask_create_task` | Write/Update | projects | Create a new task in Autotask |
| `autotask_list_phases` | Read-only | projects | List phases for a project in Autotask |
| `autotask_create_phase` | Write/Update | projects | Create a new phase in an Autotask project |
| `autotask_list_queues` | Read-only | utility | List all available ticket queues in Autotask. Use this to find queue IDs for filtering tickets by queue. |
| `autotask_list_ticket_statuses` | Read-only | utility | List all available ticket statuses in Autotask. Use this to find status values for filtering or creating tickets. |
| `autotask_list_ticket_priorities` | Read-only | utility | List all available ticket priorities in Autotask. Use this to find priority values for filtering or creating tickets. |
| `autotask_get_field_info` | Read-only | utility | Get field definitions for an Autotask entity type, including picklist values. Useful for discovering valid values for any picklist field. |
| `autotask_search_billing_items` | Read-only | time and billing | Search for billing items in Autotask. Billing items represent approved and posted billable items from the "Approve and Post" workflow. Returns 25 results per page by default. |
| `autotask_get_billing_item` | Read-only | time and billing | Get detailed information for a specific billing item by ID |
| `autotask_search_billing_item_approval_levels` | Read-only | time and billing | Search for billing item approval levels. These describe multi-level approval records for Autotask time entries, enabling visibility into tiered approval workflows. |
| `autotask_search_time_entries` | Read-only | time and billing | Search for time entries in Autotask. Returns 25 results per page by default. Time entries can be filtered by resource, ticket, project, task, date range, or approval status. Use approvalStatus="unapproved" to find entries not yet posted. Common fan-out target — scope by date range first to avoid Autotask's API threshold. |
| `autotask_list_categories` | Read-only | uncategorized | List available tool categories. Use this to discover what types of Autotask operations are available before loading specific tools. |
| `autotask_list_category_tools` | Read-only | uncategorized | List tools in a specific category with full schemas. Use after autotask_list_categories to see available tools and their parameters. |
| `autotask_execute_tool` | Mixed (read/write) | uncategorized | Execute any Autotask tool by name. Use after discovering tools via autotask_list_category_tools. |
| `autotask_router` | Mixed (read/write) | uncategorized | Intelligent tool router - describe what you want to do and get the right tool suggestion with pre-filled parameters. Use this when unsure which tool to call. |
| `autotask_get_service_call` | Read-only | service calls | Get a specific service call by ID |
| `autotask_search_service_calls` | Read-only | service calls | Search for service calls in Autotask. Filter by company, status, or date range. |
| `autotask_create_service_call` | Write/Update | service calls | Create a new service call in Autotask. Service calls are used to schedule and plan work on tickets. |
| `autotask_update_service_call` | Write/Update | service calls | Update an existing service call. Use this to change status, times, or description. To complete/close a service call, set complete: true or update the status. |
| `autotask_delete_service_call` | Write/Update | service calls | ⚠ DESTRUCTIVE — IRREVERSIBLE. Permanently deletes a service call |
| `autotask_search_service_call_tickets` | Read-only | service calls | Search for ticket associations on service calls. Use this to find which tickets are linked to a service call, or which service calls contain a specific ticket. |
| `autotask_create_service_call_ticket` | Write/Update | service calls | Link a ticket to a service call. This associates the ticket with the service call for scheduling purposes. |
| `autotask_delete_service_call_ticket` | Write/Update | service calls | ⚠ DESTRUCTIVE — IRREVERSIBLE. Permanently removes a ticket |
| `autotask_search_service_call_ticket_resources` | Read-only | service calls | Search for resource (technician) assignments on service call tickets. |
| `autotask_create_service_call_ticket_resource` | Write/Update | service calls | Assign a resource (technician) to a service call ticket. |
| `autotask_delete_service_call_ticket_resource` | Write/Update | service calls | ⚠ DESTRUCTIVE — IRREVERSIBLE. Permanently removes a resource |
| `autotask_create_contract` | Write/Update | financial | Create a new Contract in Autotask. Field names match the Autotask REST API exactly. status: 1=In Effect, 0=Inactive. Dates are ISO format (YYYY-MM-DD). |
| `autotask_create_contracts_bulk` | Write/Update | financial | Create multiple contract shells (header records, no service lines) in one call — e.g. onboarding a customer with several location-based contracts. Shells are created one at a time; a failure on one shell does not stop the rest, and each item reports its own success or error. |
| `autotask_update_contract` | Write/Update | financial | Update an existing Contract in Autotask (PATCH). Pass only fields you want to change; everything except id is optional. status: 1=In Effect, 0=Inactive. |
| `autotask_create_contract_service` | Write/Update | financial | Add a ContractService (service line item) to an existing Contract. |
| `autotask_update_contract_service` | Write/Update | financial | Update an existing ContractService line on a Contract. Pass only fields you want to change. |
| `autotask_raw_request` | Mixed (read/write) | uncategorized | Escape hatch for Autotask REST endpoints not yet wrapped by a typed tool. Use sparingly — typed tools are preferred for safety. The existing Content-Type, Accept, ApiIntegrationcode, UserName, Secret headers are added automatically. The path is resolved against the zone-resolved base URL (https://webservices<N>.autotask.net/ATServicesRest/v1.0). Pass queryParams as a flat object of string/number/boolean values; they will be URL-encoded and appended to the path. |

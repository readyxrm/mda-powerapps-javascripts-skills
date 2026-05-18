---
name: js-webresource
user-invocable: true
# Use when the user wants to create, edit, upload, download, or publish a JavaScript file as a Dataverse web resource for Model-Driven Power Apps.
description: >
  Manage local JavaScript files as Dataverse web resources for Model-Driven Power Apps. Use this skill to:
  - Create or edit a local JS file for client-side logic
  - Upload the file as a web resource to Dataverse
  - Download the latest version from Dataverse
  - Publish the web resource so changes go live
  - Automatically read schema names from the Dataverse MCP server for correct field references
---

# JavaScript Web Resource Workflow for Model-Driven Power Apps

## Prerequisite (Before Starting)
- Ensure MCP server connectivity before any other step. If the MCP server is unavailable, halt and notify the user. This is the highest-priority gate.
- Do not draft, scaffold, or modify any JavaScript content until MCP connectivity is confirmed.

## Schema Validation (During Schema Validation Step, Upload, and Publish)
- Validate all table and column names against the MCP server during the Validate Schema step, and re-validate before any Upload or Publish action. Do not guess or assume field or table names. If schema validation fails, return a clear error that includes the unresolved table/column name and stop the workflow before any create/edit/upload/publish action.
- If the MCP server becomes unavailable during schema validation, retry the connection up to three times before halting and notifying the user.
- MCP validation belongs to the skill workflow (pre-authoring), not to generated JavaScript runtime code, unless the user explicitly asks for runtime MCP calls.

## Publishing (Before Publish Step)
- Ensure PAC CLI is installed and available in PATH before any publish step. If PAC CLI is unavailable, stop and provide an actionable error.

## General
- If MCP connectivity drops during the workflow, retry MCP connection checks up to three times before halting and notifying the user.

## Steps
1. **Check MCP Server**: Verify that the MCP server is running and accessible by sending a test request. Halt execution if the server is unavailable and notify the user.
  - If the check fails due to intermittent connectivity, retry up to three times.
  - If any retry succeeds, continue the workflow.
  - If all retries fail, halt and notify the user.
2. **Validate Schema**: Use MCP to validate table and column names before creating or editing any JavaScript file. Halt execution if schema names cannot be confirmed.
  - If the MCP server becomes unavailable during schema validation, retry the connection up to three times before halting and notifying the user.
  - On failure, emit an explicit validation error with: failed schema name, operation attempted, and next action (for example, ask user for correct schema name).
  - Do not continue to any write operation after a schema validation error.
  - Treat this as a recurring requirement: re-run schema validation immediately before Upload and again immediately before Publish.
3. **Create/Edit**: Author or update a local JavaScript file for client-side form logic only after MCP server connectivity and schema validation are confirmed. Do not begin any code-writing step before Steps 1 and 2 pass.
4. **Upload**: Re-validate all referenced table and column names with MCP, then encode and upload the file as a web resource to Dataverse. If re-validation fails, halt and notify the user.
5. **Publish**: Before publishing, run a PAC CLI availability and auth check (for example, `Get-Command pac` and `pac auth who`).
   - Re-validate all referenced table and column names with MCP before publishing. If re-validation fails, halt and notify the user.
   - If PAC CLI is missing, halt with an error that instructs the user to install PAC CLI.
   - If PAC CLI is present but not authenticated, halt with an error that instructs the user to run PAC authentication.
   - Only publish when both checks succeed.
6. **Download**: Retrieve the latest version from Dataverse to sync local and server copies.
7. **Schema Awareness**: Use MCP to look up table and column schema names for robust, error-free scripting.

## Decision Points
- If the MCP server is not running, stop the process and notify the user.
- If the MCP server becomes unavailable mid-workflow, retry the connection up to three times before terminating.
- If schema names cannot be identified, halt execution and notify the user.
- If schema validation throws an error or returns no match, report the error details to the user and terminate the workflow.
- If schema re-validation fails before Upload or Publish, do not proceed with that operation.
- If MCP connectivity is unconfirmed, do not generate JavaScript code snippets, files, or edits.
- If PAC CLI is not installed or cannot be resolved from PATH, do not attempt publish.
- If PAC CLI authentication fails or no active environment is available, do not attempt publish.
- If the file does not exist locally, create it.
- If the web resource does not exist in Dataverse, create it; otherwise, update it.
- Always publish after upload to activate changes.
- Use MCP schema lookup before scripting to ensure correct field names.

## Completion Criteria
- MCP server connectivity is verified before any operation.
- Intermittent MCP connectivity failures are handled with up to three retries, with a clear halt-and-notify outcome when retries fail.
- Schema names for tables and columns are validated and identified before creating or editing JavaScript files.
- Schema names are re-validated before Upload and before Publish; no upload/publish occurs on failed re-validation.
- No JavaScript authoring occurred before MCP connectivity and schema validation succeeded.
- Schema validation failures are surfaced with actionable error details, and no write actions occur after a failed validation.
- PAC CLI availability and authentication are verified before publishing.
- Local JS file matches the intended logic.
- Web resource in Dataverse is updated and published.
- Field and table schema names are correct (verified via MCP).
- Downloaded file matches Dataverse version.

## Example Prompts
- "Create a new JS web resource for the outdoor activity form and upload it to Dataverse."
- "Download the latest liability tab script from Dataverse and update my local file."
- "Publish the instructor change handler script for Model-Driven Power Apps."
- "Check the schema name for the instructor field before scripting."

## Related Customizations
- Add a prompt for quick schema lookup
- Create a checklist for web resource deployment
- Add a hook to auto-publish after upload

---
# End of SKILL.md

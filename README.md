# Apache Camel CRM MCP demo

This example turns a tiny, local SQLite CRM into three MCP tools using Apache Camel YAML routes. Connect an MCP-capable local model in LM Studio and ask it to find customers, retrieve their notes, or save new notes.

## Requirements

- Apache Camel CLI 4.22 or newer, available as `camel` on your PATH
- Java appropriate for your Camel CLI release
- LM Studio 0.4.25 or newer
- A locally loaded LLM that supports tool/function calling

## Start the example

From this directory, run:

```shell
camel run init.camel.yaml crm-mcp.camel.yaml
```

Camel CLI automatically discovers the adjacent `application.properties`. On the first run, `init.camel.yaml` creates `crm.db`, initializes two tables, and inserts sample customers and notes. On later runs, the `INSERT OR IGNORE` statements keep those seed records from being duplicated.

The MCP endpoint is available at:

```text
http://localhost:12380/mcp
```

The server uses MCP **streamable HTTP**. The `crm` tag makes only the three intended tools available:

- `find_customers(query)` — find a company by name or partial name;
- `get_crm_notes(customer_id)` — retrieve the latest CRM notes;
- `create_crm_note(customer_id, note)` — add a note.

**Sample customers:** `Acme Industries`, `Acme Manufacturing`, and `BluePeak Industries`. Asking about just `Acme` is deliberately ambiguous, so the assistant should ask which company you mean.

## Connect LM Studio

1. Open LM Studio and load a model with tool-calling support.
2. Open a new chat
3. In the **right sidebar**, open **Integrations (hammer symbol) > Install > Edit mcp.json**.
4. Add the following entry to your MCP configuration, preserving any other servers already present:

```json
{
  "mcpServers": {
    "crm-demo": {
      "url": "http://localhost:12380/mcp"
    }
  }
}
```

5. Save the configuration, enable the CRM server for your chat, and check that the three CRM tools are available.

These instructions assume Camel and LM Studio run on the same computer. If they do not, change `localhost` to a reachable, appropriately protected address.

## Suggested system prompt

The following prompt helps the LLM choose tools, resolve ambiguous company names, and avoid making unrequested changes. It improves behavior but is **not** a security or authorization boundary.

```text
You are a CRM assistant. Use the available CRM tools to retrieve information or save a user-requested CRM note.

Always identify the relevant company before reading or writing CRM data.

Rules:
1. Search for a company using the available CRM tools when its identity is not already unambiguous from the conversation.
2. If exactly one company matches, use it.
3. If no company matches, say so and ask for clarification. Never invent a customer ID or a company.
4. If more than one company matches, show their names and ask which one the user means.
5. Never store a CRM note against a company unless its identity is unambiguous.
6. Only store a note when the user explicitly asks you to save, add, record, or create it. Otherwise, offer to save it.
7. When asked about prior interactions, notes, status, or history, use the tools to retrieve records rather than guessing.
8. Do not expose internal database IDs unless necessary for clarification.
9. Summarize retrieved information accurately and concisely. Do not invent missing details.
10. If a tool fails, acknowledge the failure rather than claiming the operation succeeded.
```

## Try these conversations

**Resolve an ambiguous company name:**

```text
What do we know about Acme?
```

Since the demo contains both `Acme Industries` and `Acme Manufacturing`, the assistant should ask which company you mean. Reply with `Acme Industries` to see its earlier interactions.

**Write a new CRM note:**

```text
I met with Acme Industries today. Please add a CRM note:
The customer would like a proof of concept to connect ERP and production data.
```

**Verify the new note:**

```text
Show me the latest CRM notes for Acme Industries.
```

LM Studio's exact wording and tool choices vary by model; the actual source of truth is the data returned by the Camel tools.

## Files

```text
application.properties    Camel CLI dependency and MCP server configuration
init.camel.yaml           SQLite DataSource, schema and sample data
crm-mcp.camel.yaml        Three MCP tools implemented as Camel routes
README.md                 Setup and suggested prompts
```

## Notes on safety and scope

This is a **local demo**, not a production-ready CRM authorization service. The model can request tools, but tool endpoints are real operations. Protect the MCP endpoint, enforce user- or service-level authorization, validate inputs and limit who can write to the CRM before using real company data. MCP tool annotations such as `readOnlyHint` are advisory metadata, not access controls.

For repeatable screenshots, you can stop the Camel process, delete the local `crm.db` demo file, and restart the example to restore its initial sample data. This deletes any notes you added during the demo.

## References

- [Apache Camel: AI Tool component](https://camel.apache.org/components/latest/ai-tool-component.html)
- [Apache Camel: MCP Server](https://camel.apache.org/components/latest/others/mcp-server.html)
- [Apache Camel CLI: Running Camel](https://camel.apache.org/manual/camel-jbang-running.html)
- [LM Studio: Use MCP Servers](https://lmstudio.ai/docs/app/mcp)

## How to enable KG

``` js
fetch("/api/admin/kg/config", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    vendor: "Ditec",
    vendor_domains: ["Ditec.sk"],
    ignore_domains: [],
    coverage_start: "2025-01-01T00:00:00Z"
  })
}).then(r => r.ok ? "KG enabled!" : r.text()).then(console.log)
```

### enable KG for a persona

#### First, check the KG tool id
``` js
fetch("/api/tool").then(r => r.json()).then(tools => 
  console.log(tools.filter(t => t.in_code_tool_id === "KnowledgeGraphTool"))
)
```

#### Then update your target persona (replace PERSONA_ID and KG_TOOL_ID):
``` js
fetch("/api/admin/persona/PERSONA_ID")
  .then(r => r.json())
  .then(p => {
    const toolIds = p.tools.map(t => t.id);
    if (!toolIds.includes(KG_TOOL_ID)) toolIds.push(KG_TOOL_ID);
    return fetch("/api/admin/persona/PERSONA_ID", {
      method: "PATCH",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({tool_ids: toolIds})
    });
  })
  .then(r => r.ok ? "Done!" : r.text())
  .then(console.log)
```

### disable image processin (can be done in the UI)
``` js
fetch("/api/admin/settings", {
  method: "PUT",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ image_extraction_and_analysis_enabled: false })
}).then(r => r.ok ? "Disabled!" : r.text()).then(console.log)
 ```

 ### Some questions:

The KG-specific LLM timeouts are in kg_configs.py. The relevant ones for entity/relationship extraction (which is what's running when your SKILL_OF merge happens):

Env var	Default	Purpose
KG_ENTITY_EXTRACTION_TIMEOUT	15s	Entity extraction LLM call
KG_RELATIONSHIP_EXTRACTION_TIMEOUT	15s	Relationship extraction LLM call
Set them via environment variable in your docker-compose or .env:


KG_ENTITY_EXTRACTION_TIMEOUT=60
KG_RELATIONSHIP_EXTRACTION_TIMEOUT=60
If the timeout is happening during query (not indexing), the relevant ones are KG_STRATEGY_GENERATION_TIMEOUT (20s) and KG_SQL_GENERATION_TIMEOUT (40s).

There's also the global LLM_SOCKET_READ_TIMEOUT in chat_configs.py:30 set to 300s — that's the raw socket timeout for all LLM calls.

Which operation is timing out — indexing (extraction) or query time?
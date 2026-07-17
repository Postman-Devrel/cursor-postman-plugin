---
description: Ask questions about Postman and how to accomplish workflows. Searches the official Postman documentation and learning resources.
allowed-tools: Read, Glob, Grep, mcp__postman__searchLearningCenter
---

# /postman:learn -- Learn Postman

Answer "how do I..." questions about Postman itself and how to accomplish different workflows. Search the official Postman documentation at https://learning.postman.com and return authoritative, cited guidance.

Use this for questions about the Postman **product** ("how do I create a mock server?", "how do variables work?", "how do I write a test script?"). For questions about the user's own APIs and resources, use `/postman:search` instead.

## Prerequisites

Postman MCP Server must be configured. If MCP tools fail, tell the user to run `/postman:setup`.

## Workflow

### Step 1: Understand the Question

Take the user's question as-is. If they didn't include one with the command, ask what they'd like to learn about Postman.

### Step 2: Search the Learning Center

Use the `searchLearningCenter` tool with a focused query derived from the user's intent.

1. Call `searchLearningCenter` with a concise, feature-oriented query (e.g. `"how to create a mock server"`, `"write a test script"`, `"set a collection variable"`, `"schedule a monitor"`).
2. If results are sparse or off-target, refine the query — rephrase around the specific Postman feature or concept, or break a broad "how do I do X end-to-end" question into multiple targeted searches (one per step).
3. For multi-step workflows, run several searches to cover each stage, then stitch the guidance together into one coherent walkthrough.

### Step 3: Answer

Synthesize the documentation passages into a clear, actionable answer. Answer first, then cite.

- Lead with a direct answer or the concrete steps.
- Use numbered steps for procedures.
- Include the relevant source URL(s) returned by the tool so the user can read more.
- Keep it grounded in what the docs actually say — don't invent UI details or menu paths that weren't in the results. If the docs don't cover it, say so.

**Example — a "how do I" question:**
```
To create a mock server from a collection:

  1. Select the collection in the sidebar.
  2. Open the "..." (More actions) menu and choose "Mock collection".
  3. Give the mock a name, optionally link an environment, then create it.
  4. Postman generates a mock URL you can send requests to.

Postman matches incoming requests to your saved example responses,
so add examples to each request for realistic mock data.

  Source: https://learning.postman.com/docs/design-apis/mock-apis/mock-with-collections/

Want me to create one for you now? (/postman:mock)
```

**Example — a broader workflow question:**
```
Setting up CI-style API testing in Postman has three parts:

  1. Write tests -- Add test scripts to your requests using pm.test()
     and pm.expect() assertions.
     https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts/

  2. Run the collection -- Use the Collection Runner, or run it headless
     with the Newman CLI or the postman-cli.
     https://learning.postman.com/docs/collections/running-collections/intro-to-collection-runs/

  3. Automate it -- Schedule a Monitor, or run Newman in your CI pipeline.
     https://learning.postman.com/docs/monitoring-your-api/intro-monitors/

Want to run your collection tests now? (/postman:test)
```

## Related Commands

Point the user to the command that acts on their intent when relevant:
- Learning how to search your own APIs -> `/postman:search`
- Learning how to test -> `/postman:test`
- Learning how to mock -> `/postman:mock`
- Learning how to document -> `/postman:docs`

## Error Handling

| Error | Response |
|-------|----------|
| No results found | "I couldn't find docs matching that. Try rephrasing around the specific Postman feature (e.g. 'mock server', 'environment variable', 'monitor'), or browse https://learning.postman.com directly." |
| Tool unavailable | "The learning center search isn't available. Make sure the Postman MCP Server is configured -- run /postman:setup." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |

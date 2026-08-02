---
description: Generate or update an OpenAPI 3.0 spec by analyzing the API routes in your codebase.
allowed-tools: Bash, Read, Write, Glob, Grep
---

# /postman:generate-spec -- Generate an OpenAPI Spec

Generate or update an OpenAPI 3.0 specification by analyzing the API routes in your codebase. Scan the project for route definitions, derive schemas from the code, write a valid spec, and validate it.

## Prerequisites

The Postman CLI is used to validate the generated spec (`postman spec lint`). If it isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`. Validation is optional; if the CLI is unavailable, still write the spec and tell the user to lint it later.

## Workflow

### Step 1: Check for an Existing Spec

```bash
ls postman/specs/**/*.yaml postman/specs/**/*.yml postman/specs/**/*.json 2>/dev/null
ls openapi.yaml openapi.yml swagger.yaml swagger.yml 2>/dev/null
```

If a spec exists, read it to understand the current state. You'll update it rather than replace it.

### Step 2: Discover API Endpoints

Scan the project for route definitions based on the framework:

- **Express/Node**: `app.get()`, `router.post()`, `@Get()` (NestJS)
- **Python**: `@app.route()`, `@router.get()` (FastAPI), `path()` (Django)
- **Go**: `http.HandleFunc()`, `r.GET()` (Gin/Echo)
- **Java**: `@GetMapping`, `@PostMapping`, `@RequestMapping`
- **Ruby**: `get`, `post`, `resources` in `routes.rb`

Read the route files and extract methods, paths, parameters, request bodies, response schemas, and auth requirements.

### Step 3: Generate or Update the Spec

Write a valid OpenAPI 3.0.3 YAML spec including:

- `info` with title, version, and description
- `servers` with a local dev URL
- `paths` with all discovered endpoints
- `components/schemas` with models derived from the code (types, models, structs)
- `components/securitySchemes` if auth is used

**When updating**: add new endpoints, update changed ones, and remove endpoints no longer in the code. Preserve existing descriptions and examples.

**When creating**: write to `postman/specs/openapi.yaml`.

### Step 4: Validate

```bash
postman spec lint ./postman/specs/openapi.yaml
```

Fix any validation errors and re-run until clean.

### Step 5: Report

Show what was created or changed:

```
Spec written: postman/specs/openapi.yaml

  Endpoints documented: 8
    GET    /users
    GET    /users/{id}
    POST   /users
    ... and 5 more

  Schemas defined: User, CreateUserRequest, UserListResponse, Error
  Security schemes: bearerAuth
  Validation: passed (0 errors, 0 warnings)
```

When updating, also summarize the changes from the previous spec (added / updated / removed endpoints).

## Error Handling

| Error | Response |
|-------|----------|
| No routes found | "I couldn't find API route definitions in this project. Tell me where your routes live, or which framework you're using." |
| Can't detect framework | "I can't tell which web framework this project uses. Point me at the routing file and I'll take it from there." |
| Validation errors | "The generated spec has validation issues — I'll fix them and re-lint until it's clean." |
| Postman CLI not installed | "I wrote the spec but couldn't validate it — the Postman CLI isn't installed. Install it, then run `postman spec lint ./postman/specs/openapi.yaml`." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |

## Related Commands

- Push the generated spec to Postman as a collection -> `/postman:sync`
- Run the collection's tests -> `/postman:test`

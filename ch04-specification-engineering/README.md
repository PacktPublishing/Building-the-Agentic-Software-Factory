# Chapter 4 - Specification Engineering

## Chapter Summary

Specification Engineering is the discipline of replacing vague, natural-language intent with machine-readable, precise contracts. This chapter introduces the **Algorithmic Specification** as the core communication artifact between the human engineer and the autonomous agent - a document precise enough that the agent can execute it without ambiguity, and verifiable enough that a CI pipeline can confirm success.

The walkthrough builds a `BlogPost` API endpoint driven entirely by an OpenAPI YAML contract, showing how a schema-first approach eliminates the "telephone game" of iterative prompt correction.

### Key Concepts

- **The Telephone Game Problem:** Vague prompts force the agent to make interpretive choices, which leads to iterative correction cycles. Each correction adds prompt ambiguity, causing the agent to regress. Algorithmic specifications cut this feedback loop short.
- **Algorithmic Specifications:** Structured documents that define what the system must do (behavior), how it must behave (constraints), and what success looks like (verification criteria). Written in formats the agent can parse deterministically - YAML, JSON Schema, or structured markdown.
- **Schema-First Development:** Define the data contract (OpenAPI spec, TypeScript interface, Pydantic model) before writing any implementation code. The schema is the specification. The agent implements to the schema.
- **The OpenAPI Specification:** The industry standard for describing REST API contracts. When injected into an agent's context window, an OpenAPI spec removes ambiguity about field names, types, required fields, and response codes.
- **Ambiguity Budget:** Every English word in a prompt has an "ambiguity budget" - a range of statistically valid interpretations the model might choose from. Specifications reduce that budget to near zero.

---

## Code Snippets

### The BlogPost OpenAPI Specification

This is the core specification artifact from the chapter walkthrough. Save this as `specs/blog-post-api.yaml`:

```yaml
openapi: 3.0.3
info:
  title: Blog Post API
  version: 1.0.0

paths:
  /posts:
    post:
      summary: Create a new blog post
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreatePostRequest'
      responses:
        '201':
          description: Post created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PostResponse'
        '400':
          description: Validation error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

  /posts/{id}:
    get:
      summary: Retrieve a single blog post by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Post found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PostResponse'
        '404':
          description: Post not found

components:
  schemas:
    CreatePostRequest:
      type: object
      required:
        - title
        - body
        - authorId
      properties:
        title:
          type: string
          minLength: 5
          maxLength: 120
        body:
          type: string
          minLength: 50
        authorId:
          type: string
          format: uuid
        tags:
          type: array
          items:
            type: string
          maxItems: 10

    PostResponse:
      type: object
      required:
        - id
        - title
        - body
        - authorId
        - slug
        - createdAt
      properties:
        id:
          type: string
          format: uuid
        title:
          type: string
        body:
          type: string
        authorId:
          type: string
          format: uuid
        slug:
          type: string
          description: URL-friendly version of the title, auto-generated from the title on creation.
        tags:
          type: array
          items:
            type: string
        createdAt:
          type: string
          format: date-time

    ErrorResponse:
      type: object
      required:
        - error
        - message
      properties:
        error:
          type: string
        message:
          type: string
```

### Prompt Template: Schema-First Specification Prompt

Use this prompt template when driving agent implementation from a spec file:

```
Read the OpenAPI specification in specs/blog-post-api.yaml completely before writing any code.

Your task is to implement the Express.js route handlers for the POST /posts and GET /posts/:id endpoints.

Constraints:
- Every field in PostResponse MUST be present in the response. Missing any required field is a test failure.
- The `slug` field MUST be auto-generated from the `title` using kebab-case (e.g., "My Blog Post" -> "my-blog-post").
- Return HTTP 400 for any missing required field with the ErrorResponse schema.
- Return HTTP 404 when a post ID does not exist.
- Use in-memory storage (a Map) for this exercise - no database required.
- Do NOT use any library not already in package.json.
```

### Slug Generation Utility

The agent should produce something equivalent to this for the `slug` field:

```javascript
function generateSlug(title) {
    return title
        .toLowerCase()
        .trim()
        .replace(/[^\w\s-]/g, '')   // Remove non-word characters
        .replace(/[\s_-]+/g, '-')   // Replace spaces and underscores with hyphens
        .replace(/^-+|-+$/g, '');   // Trim leading/trailing hyphens
}
```

### Algorithmic Specification Template (Markdown Format)

For specifications that go beyond OpenAPI (business logic, sequencing, constraints):

```markdown
# Specification: [Feature Name]

## Objective
[One sentence: what does this feature accomplish for the end user?]

## Inputs
| Field | Type | Required | Constraints |
|---|---|---|---|
| fieldName | string | Yes | minLength: 5, maxLength: 120 |

## Outputs
| Field | Type | Always Present | Description |
|---|---|---|---|
| id | uuid | Yes | Auto-generated on creation |

## Behavioral Constraints
- MUST [specific behavioral rule]
- MUST NOT [specific prohibition]
- IF [condition] THEN [required behavior]

## Error Conditions
| Condition | HTTP Status | Error Code |
|---|---|---|
| Missing required field | 400 | VALIDATION_ERROR |
| Resource not found | 404 | NOT_FOUND |

## Verification Criteria
- [ ] All required output fields are present in every response
- [ ] Invalid input returns 400 with correct error schema
- [ ] [Additional testable assertions]
```

---

## Supplemental Resources

### OpenAPI Specification
- **[OpenAPI Specification (Official)](https://spec.openapis.org/oas/latest.html)** - The authoritative specification for OpenAPI 3.x. Use this as the reference when writing your own YAML contracts.
- **[Swagger Editor](https://editor.swagger.io/)** - Browser-based OpenAPI editor with live validation and preview. The fastest way to author and debug YAML specs.
- **[OpenAPI Generator](https://openapi-generator.tech/)** - Automatically generates client SDKs, server stubs, and documentation from an OpenAPI spec. Extremely useful in the schema-first workflow.

### Schema Validation Tools
- **[Ajv - JSON Schema Validator (Node.js)](https://ajv.js.org/)** - The fastest JSON Schema validator for Node.js. Use it to add runtime request validation driven directly by your OpenAPI schemas.
- **[Pydantic (Python)](https://docs.pydantic.dev/)** - Python's data validation library. Pydantic models serve as algorithmic specifications for Python-based agents.
- **[Zod (TypeScript)](https://zod.dev/)** - TypeScript-first schema validation with static type inference. A solid specification tool for TypeScript projects.

### Specification-Driven Development
- **[Design-First API Development - Stoplight](https://stoplight.io/api-design-guide/basics)** - A practical guide to design-first API development workflows.
- **[API Design Patterns](https://www.manning.com/books/api-design-patterns)** by JJ Geewax - A useful reference for designing consistent API contracts, directly applicable to writing high-quality algorithmic specifications.

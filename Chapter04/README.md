# Chapter 4 - Shaping Intent

## Chapter Summary

When agents write the code, the human's primary artifact shifts from syntax to intent. This chapter introduces **shaping intent**, the practice of replacing ambiguous natural-language prompts with a precise specification the agent can build against. It covers why "just describing what you want" breaks down at scale, how **spec-driven development** uses schemas to prevent hallucination, and how to run the **Plan-Build-Verify cycle** so generation stays inside the lines you drew.

The walkthrough builds a **Task Management API** endpoint driven entirely by an OpenAPI YAML contract, demonstrating the full Plan-Build-Verify cycle.

### Key Concepts

- **Specifications as the New Source Code:** When agents handle the typing, the developer's primary deliverable shifts from implementation to intent. A specification captures the reasoning that used to live in the act of writing code.
- **Essential vs. Accidental Complexity:** Agents absorb accidental complexity (syntax recall, boilerplate) but cannot shoulder essential complexity (domain modeling, threat modeling, architectural trade-offs). Specifications are how essential complexity gets captured.
- **Spec-Driven Development (SDD):** Writing the specification before the code and treating it as the source of truth for both you and the agent. Schemas bound the *shape*; tests bound the *behavior*.
- **The Plan-Build-Verify Cycle:** A three-phase execution framework: (1) **Plan** - harden the intent into a spec with the agent in plan mode; (2) **Build** - authorize the agent to execute against the approved plan; (3) **Verify** - confirm the implementation matches the specification using automated contract tests.
- **Ambiguity Budget:** Every English word in a prompt has an "ambiguity budget," a range of statistically valid interpretations. A typed schema (string, integer, enum) reduces that budget to near zero.
- **From Player to Coach:** The developer's role shifts from typing every line of code to deciding what should be built, defining the constraints, and coordinating execution.

---

## Code Snippets

### Setup: Node.js Environment

```bash
mkdir specification-exercise && cd specification-exercise
npm init -y
npm install express
touch server.js
mkdir specs
```

### The Task Management API - OpenAPI Specification

This is the core specification artifact from the chapter walkthrough. Save this as `specs/openapi.yaml`:

```yaml
openapi: 3.0.0
info:
  title: Task Management API
  version: 1.0.0
paths:
  /tasks:
    post:
      summary: Create a new task
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - title
                - status
              properties:
                title:
                  type: string
                  description: The title of the task
                status:
                  type: string
                  enum: [pending, in-progress, completed]
      responses:
        '201':
          description: Task created successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  id:
                    type: string
                  title:
                    type: string
                  status:
                    type: string
        '400':
          description: Bad request (missing required fields or invalid status)
```

### Prompt Template: Schema-First Specification Prompt

Once the spec is written, inject it into the agent's context and issue this bounded prompt:

```
Read the provided openapi.yaml specification. Implement it exactly:
build an Express.js server inside server.js with the /tasks POST
endpoint defined in the schema. Include middleware to parse JSON.
Enforce all 'required' fields and 'enum' constraints from the
schema. Return the exact HTTP status codes defined in the responses.
Store tasks in a simple in-memory array for now.
```

### Verification: Testing the Implementation with curl

Start the server:

```bash
node server.js
```

**Test 1 - 400 Bad Request (missing required field):**

```bash
curl -X POST http://localhost:3000/tasks \
-H "Content-Type: application/json" \
-d '{"title": "Learn Specification Engineering"}'
```

Expected: `400` status code because the `status` field is missing.

**Test 2 - 400 Bad Request (invalid enum value):**

```bash
curl -X POST http://localhost:3000/tasks \
-H "Content-Type: application/json" \
-d '{"title": "Test Enum Validation", "status": "started"}'
```

Expected: `400` status code because `"started"` is not in the `[pending, in-progress, completed]` enum.

**Test 3 - 201 Created (valid payload):**

```bash
curl -X POST http://localhost:3000/tasks \
-H "Content-Type: application/json" \
-d '{"title": "Master the Plan-Build-Verify cycle", "status": "pending"}'
```

Expected: `201` status code with the completed JSON object including `id`, `title`, and `status`.

### From a Vague Wish to a Precise Specification

The chapter works through the transformation of a vague natural-language request into a precise OpenAPI contract. Here is the example:

**The vague wish:**

> "I need a new API endpoint that lets users update their profile. They should be able to change their email, their bio, and their profile picture URL. If they don't provide an email, throw an error. Make sure it returns the updated user object."

**The precise specification (worked out in plan mode with the agent):**

```yaml
openapi: 3.0.3
info:
  title: User Profile API
  version: 1.0.0
paths:
  /api/v1/users/profile:
    patch:
      summary: Update user profile
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - email
              properties:
                email:
                  type: string
                  format: email
                bio:
                  type: string
                  maxLength: 500
                avatarUrl:
                  type: string
                  format: uri
      responses:
        '200':
          description: Profile updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Bad Request - Missing required email
```

Every hole the prompt left open is now closed. The method is `PATCH`. The route is `/api/v1/users/profile`. The bio is bounded to 500 characters. The error code is `400`.

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
- **[Understanding Spec-Driven Development - Birgitta Boeckeler (martinfowler.com)](https://martinfowler.com/)** - Surveys the current tools and separates spec-first, spec-anchored, and spec-as-source as three distinct depths of the practice.
- **[Leadership is Language - L. David Marquet](https://www.amazon.com/Leadership-Language-Hidden-Power-What/dp/0735217548)** - The intent-based leadership model ("I intend to...") that maps directly to shaping intent in plan mode.
- **[Painless Functional Specifications - Joel Spolsky](https://www.joelonsoftware.com/2000/10/02/painless-functional-specifications-part-1-why-bother/)** - The clearest case for the functional spec, predating agents entirely, and every argument in it got stronger.
- **[No Silver Bullet - Fred Brooks (1986)](https://en.wikipedia.org/wiki/No_Silver_Bullet)** - The source of the essential vs. accidental complexity distinction this chapter leans on.
- **[One Developer, Two Dozen Agents, Zero Alignment - Maggie Appleton](https://maggieappleton.com/zero-alignment/)** - On why a team agreeing on what it is building becomes the real bottleneck once code generation is fast and cheap.

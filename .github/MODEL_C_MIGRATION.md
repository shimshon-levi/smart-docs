You are acting as a senior system architect.

This repository is a multi-service system with the following structure:

Root:

- auth-service
- client-service
- document-service
- frontend
- nginx
- docker-compose

The project is migrating to **Model C**.

## Model C – Canonical Rules (DO NOT VIOLATE)

- User = authentication & identity only
- Client = business entity, managed by an advisor/admin
- User is NOT a Client
- Client may exist without a User account
- Client.userId is optional and initially null
- auth-service must NEVER:
  - store business fields (phone, address, company, cases, documents)
  - create or assume Client entities
  - link User to Client automatically
- client-service owns:
  - Clients
  - advisor-client relationship
  - cases and templates
- document-service owns:
  - documents only
  - authorization based on case ownership
- frontend must:
  - never send advisorId or userId
  - rely on JWT context only
  - treat admin and client flows separately

## Task

1. Analyze the ENTIRE repository:
   - all services
   - all folders
   - frontend included
2. Identify all violations of Model C.
3. For EACH service, describe:
   - what must be rewritten
   - what must be removed
   - what must stay unchanged
4. Then propose a **service-by-service rewrite plan**:
   - auth-service
   - client-service
   - document-service
   - frontend (admin)
   - frontend (client)

Rules:

- Do NOT write code yet
- Do NOT refactor yet
- Only analysis and clear directives

Respond with a structured plan, not code.

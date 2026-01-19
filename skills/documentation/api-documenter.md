---
name: api-documenter
description: |
  Create comprehensive API documentation including endpoints, parameters,
  responses, and examples.
---

# API Documenter Skill

## Purpose
Document APIs clearly and comprehensively for developers consuming them.

## OpenAPI/Swagger Specification

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: MyApp API
  description: |
    REST API for MyApp.

    ## Authentication
    Most endpoints require a Bearer token in the Authorization header:
    ```
    Authorization: Bearer <access_token>
    ```

    ## Rate Limiting
    API requests are limited to 100 requests per 15-minute window.

  version: 1.0.0
  contact:
    email: api@myapp.com

servers:
  - url: https://api.myapp.com/v1
    description: Production
  - url: https://staging-api.myapp.com/v1
    description: Staging
  - url: http://localhost:3000/api
    description: Local Development

tags:
  - name: Authentication
    description: User authentication endpoints
  - name: Users
    description: User management
  - name: Posts
    description: Post CRUD operations

paths:
  /auth/login:
    post:
      tags: [Authentication]
      summary: User login
      description: Authenticate a user and return access tokens.
      operationId: login
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password]
              properties:
                email:
                  type: string
                  format: email
                  example: user@example.com
                password:
                  type: string
                  format: password
                  minLength: 8
                  example: securePassword123
      responses:
        '200':
          description: Login successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  accessToken:
                    type: string
                    description: JWT access token (expires in 15 minutes)
                  refreshToken:
                    type: string
                    description: Refresh token for obtaining new access tokens
                  user:
                    $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          description: Invalid credentials
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                error: Invalid email or password
                code: INVALID_CREDENTIALS

  /users:
    get:
      tags: [Users]
      summary: List users
      description: Retrieve a paginated list of users.
      operationId: listUsers
      security:
        - bearerAuth: []
      parameters:
        - name: page
          in: query
          description: Page number
          schema:
            type: integer
            default: 1
            minimum: 1
        - name: limit
          in: query
          description: Items per page
          schema:
            type: integer
            default: 10
            minimum: 1
            maximum: 100
        - name: search
          in: query
          description: Search by name or email
          schema:
            type: string
      responses:
        '200':
          description: List of users
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  meta:
                    $ref: '#/components/schemas/Pagination'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      tags: [Users]
      summary: Create user
      description: Create a new user account.
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          description: Email already registered
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /users/{id}:
    parameters:
      - name: id
        in: path
        required: true
        description: User ID
        schema:
          type: string

    get:
      tags: [Users]
      summary: Get user
      description: Retrieve a user by ID.
      operationId: getUser
      security:
        - bearerAuth: []
      responses:
        '200':
          description: User details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

    patch:
      tags: [Users]
      summary: Update user
      description: Update user profile. Users can only update their own profile.
      operationId: updateUser
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateUser'
      responses:
        '200':
          description: User updated
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '403':
          $ref: '#/components/responses/Forbidden'

    delete:
      tags: [Users]
      summary: Delete user
      description: Delete a user account. Requires admin role.
      operationId: deleteUser
      security:
        - bearerAuth: []
      responses:
        '204':
          description: User deleted
        '403':
          $ref: '#/components/responses/Forbidden'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT access token

  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          description: Unique identifier
          example: cuid_abc123
        email:
          type: string
          format: email
          example: user@example.com
        name:
          type: string
          nullable: true
          example: John Doe
        role:
          type: string
          enum: [USER, ADMIN]
          example: USER
        createdAt:
          type: string
          format: date-time
          example: '2024-01-01T00:00:00Z'

    CreateUser:
      type: object
      required: [email, password]
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
        name:
          type: string

    UpdateUser:
      type: object
      properties:
        name:
          type: string
        email:
          type: string
          format: email

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    Error:
      type: object
      properties:
        error:
          type: string
          description: Human-readable error message
        code:
          type: string
          description: Machine-readable error code
        details:
          type: object
          description: Additional error details

  responses:
    BadRequest:
      description: Bad request - validation failed
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            error: Validation failed
            code: VALIDATION_ERROR
            details:
              email: ['Invalid email format']

    Unauthorized:
      description: Unauthorized - missing or invalid token
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            error: Unauthorized
            code: UNAUTHORIZED

    Forbidden:
      description: Forbidden - insufficient permissions
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            error: Forbidden
            code: FORBIDDEN

    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            error: Resource not found
            code: NOT_FOUND
```

## Markdown API Docs

```markdown
# API Reference

Base URL: `https://api.myapp.com/v1`

## Authentication

Include the access token in the Authorization header:

```
Authorization: Bearer <access_token>
```

## Endpoints

### POST /auth/login

Authenticate a user.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "cuid_abc123",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

**Errors:**
| Status | Code | Description |
|--------|------|-------------|
| 400 | VALIDATION_ERROR | Invalid request body |
| 401 | INVALID_CREDENTIALS | Wrong email or password |

---

### GET /users

List all users (requires authentication).

**Query Parameters:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| page | integer | 1 | Page number |
| limit | integer | 10 | Items per page (max 100) |
| search | string | - | Search by name or email |

**Response:** `200 OK`
```json
{
  "data": [
    {
      "id": "cuid_abc123",
      "email": "user@example.com",
      "name": "John Doe",
      "role": "USER",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  }
}
```
```

## Quality Checklist

- [ ] All endpoints documented
- [ ] Request/response examples included
- [ ] Error codes documented
- [ ] Authentication explained
- [ ] Query parameters listed
- [ ] Status codes explained
- [ ] Schema definitions complete
- [ ] Examples are realistic

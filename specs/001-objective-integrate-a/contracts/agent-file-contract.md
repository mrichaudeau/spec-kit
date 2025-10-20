# Agent File Contract

**Location**: `.claude/agents/*.md`
**Purpose**: Define specialized Claude Code agent definitions with capabilities, behavioral traits, and response approaches
**Format**: Markdown with YAML frontmatter

## File Naming Convention

**Pattern**: `<specialty>.md`

**Rules**:
- Lowercase with underscores for multi-word specialties
- No feature-specific identifiers (enables cross-feature reuse per FR-008)
- Descriptive of agent's primary specialty domain

**Valid Examples**:
- `frontend_specialist.md`
- `api_developer.md`
- `database_architect.md`
- `test_specialist.md`
- `deployment_engineer.md`
- `general_purpose.md`

**Invalid Examples**:
- `001-api-developer.md` ❌ (contains feature ID)
- `ApiDeveloper.md` ❌ (uses PascalCase instead of snake_case)
- `agent.md` ❌ (non-descriptive)
- `my_custom_agent_v2.md` ❌ (version suffix inappropriate)

---

## File Structure

### Required Components

**1. YAML Frontmatter** (delimited by `---`)
**2. Markdown Body** with required sections

---

## YAML Frontmatter Schema

```yaml
---
name: string                    # Required: kebab-case or snake_case identifier
description: string             # Required: One-line purpose (50-150 chars)
model: string                   # Required: "sonnet" | "opus" | "claude-sonnet-4"
capabilities: List[string]      # Optional: Quick reference capability tags
template_version: string        # Optional: For future compatibility tracking
---
```

### Frontmatter Field Specifications

**`name`** (required)
- **Type**: String
- **Format**: kebab-case or snake_case
- **Example**: `"api-developer"` or `"api_developer"`
- **Purpose**: Machine-readable identifier for agent
- **Constraint**: Should match filename (e.g., `api_developer.md` → `name: api_developer`)

**`description`** (required)
- **Type**: String
- **Length**: 50-200 characters
- **Purpose**: One-line summary of agent's role and when to use it
- **Best Practice**: Include "Use PROACTIVELY when..." guidance
- **Example**: `"Build React components, implement responsive layouts, and handle client-side state management. Use PROACTIVELY when creating UI components."`

**`model`** (required)
- **Type**: String
- **Values**: `"sonnet"`, `"opus"`, `"claude-sonnet-4"`, or other Claude model identifiers
- **Purpose**: Specify which Claude model powers this agent
- **Default**: `"sonnet"` for most agents

**`capabilities`** (optional)
- **Type**: List of strings
- **Purpose**: Quick-reference tags for matching algorithm
- **Example**: `["rest_api", "fastapi", "authentication"]`
- **Note**: Detailed capabilities go in `## Capabilities` section

**`template_version`** (optional)
- **Type**: String (semver)
- **Example**: `"1.0.0"`
- **Purpose**: Track agent template evolution for future migrations

---

## Markdown Body - Required Sections

### Section 1: `## Purpose` (Required)

**Purpose**: High-level description of agent's role and expertise

**Content Guidelines**:
- 1-3 paragraphs
- Describe what the agent specializes in
- Explain the agent's value proposition
- Mention key technologies or domains

**Example**:
```markdown
## Purpose

Expert frontend developer specializing in React 19+, Next.js 15+, and modern web application development. Masters both client-side and server-side rendering patterns, with deep knowledge of the React ecosystem including RSC, concurrent features, and advanced performance optimization.
```

---

### Section 2: `## Capabilities` (Required)

**Purpose**: Detailed list of technical skills and expertise areas

**Content Guidelines**:
- Bulleted list (can use H3 subsections for organization)
- Specific, actionable capabilities
- Group related capabilities under H3 headings (optional but recommended)
- Include frameworks, patterns, tools, and techniques

**Format Options**:

**Option A: Flat List**
```markdown
## Capabilities

- REST API design and implementation
- FastAPI framework expertise
- Authentication and authorization (JWT, OAuth2)
- Database query optimization
- API documentation with OpenAPI
- Error handling and validation
- Rate limiting and throttling
```

**Option B: Organized with H3 Subsections** (Preferred for complex agents)
```markdown
## Capabilities

### API Development
- REST API design patterns
- FastAPI and Pydantic integration
- GraphQL schema design
- API versioning strategies

### Security
- Authentication (JWT, OAuth2, API keys)
- Authorization and RBAC
- Input validation and sanitization
- SQL injection prevention

### Performance
- Query optimization
- Caching strategies (Redis, in-memory)
- Rate limiting and throttling
- Async/await patterns
```

**Update Semantics** (FR-026):
- **Additive only**: New capabilities appended, existing never removed
- **Deduplication**: Check for semantic overlap before adding
- **Placement**: Append to appropriate H3 subsection or create new subsection

---

### Section 3: `## Behavioral Traits` (Required)

**Purpose**: Describe how the agent approaches tasks and makes decisions

**Content Guidelines**:
- Bulleted list
- Describes agent's working style, priorities, and principles
- Focus on *how* the agent works, not *what* it does
- Include quality standards, best practices adherence, and trade-off preferences

**Example**:
```markdown
## Behavioral Traits

- Prioritizes user experience and performance equally
- Writes maintainable, scalable component architectures
- Implements comprehensive error handling and loading states
- Uses TypeScript for type safety and better developer experience
- Follows React and Next.js best practices religiously
- Considers accessibility from the design phase
- Optimizes for Core Web Vitals and Lighthouse scores
- Documents components with clear props and usage examples
```

---

### Section 4: `## Response Approach` (Required)

**Purpose**: Step-by-step methodology for how agent tackles tasks

**Content Guidelines**:
- Numbered or bulleted list
- Sequential steps the agent follows
- Describes thought process and workflow
- Can include decision points and validation steps

**Example**:
```markdown
## Response Approach

1. **Analyze requirements** for modern React/Next.js patterns
2. **Suggest performance-optimized solutions** using React 19 features
3. **Provide production-ready code** with proper TypeScript types
4. **Include accessibility considerations** and ARIA patterns
5. **Consider SEO and meta tag implications** for SSR/SSG
6. **Implement proper error boundaries** and loading states
7. **Optimize for Core Web Vitals** and user experience
8. **Include Storybook stories** and component documentation
```

---

## Optional Sections

### `## Knowledge Base` (Optional)

**Purpose**: Document domain-specific knowledge areas and reference materials

**Example**:
```markdown
## Knowledge Base

- React 19+ documentation and experimental features
- Next.js 15+ App Router patterns and best practices
- TypeScript 5.x advanced features
- Modern CSS specifications and browser APIs
- Web Performance optimization techniques
- Accessibility standards (WCAG 2.1/2.2)
```

### `## Example Interactions` (Optional)

**Purpose**: Sample prompts/tasks the agent handles well

**Example**:
```markdown
## Example Interactions

- "Build a server component that streams data with Suspense boundaries"
- "Create a form with Server Actions and optimistic updates"
- "Implement a design system component with Tailwind and TypeScript"
- "Optimize this React component for better rendering performance"
```

---

## Complete Example

```markdown
---
name: api-developer
description: Design and implement RESTful APIs with authentication, error handling, and performance optimization. Use PROACTIVELY for backend API tasks.
model: sonnet
capabilities:
  - rest_api
  - fastapi
  - authentication
template_version: "1.0.0"
---

## Purpose

Expert backend developer specializing in RESTful API design and implementation using modern Python frameworks. Masters FastAPI, Pydantic validation, and async patterns with deep knowledge of authentication, authorization, and API security best practices.

## Capabilities

### API Development
- REST API design following OpenAPI 3.0 specifications
- FastAPI framework with Pydantic models
- GraphQL schema design and resolvers
- API versioning strategies (header, path, query param)
- Endpoint documentation with automatic OpenAPI generation

### Authentication & Security
- JWT token generation and validation
- OAuth2 flows (authorization code, client credentials)
- API key management and rotation
- Role-based access control (RBAC)
- Input validation and sanitization

### Performance & Scalability
- Async/await patterns with asyncio
- Database connection pooling
- Redis caching strategies
- Rate limiting and throttling (token bucket, sliding window)
- Query optimization and N+1 prevention

### Testing & Quality
- pytest fixtures and parametrized tests
- API contract testing with Pydantic
- Integration testing with test clients
- Load testing with locust
- API versioning and backward compatibility

## Behavioral Traits

- Prioritizes security and input validation
- Follows REST principles and HTTP semantics
- Implements comprehensive error handling with proper status codes
- Uses type hints and Pydantic for runtime validation
- Documents all endpoints with clear examples
- Considers API consumer experience (rate limits, pagination, filtering)
- Optimizes for performance without sacrificing code clarity
- Tests all error paths and edge cases

## Response Approach

1. **Analyze API requirements** and design resource models
2. **Define Pydantic schemas** for request/response validation
3. **Implement endpoints** with proper HTTP methods and status codes
4. **Add authentication and authorization** checks
5. **Implement comprehensive error handling** with custom exception handlers
6. **Add rate limiting** and request validation middleware
7. **Generate OpenAPI documentation** with examples
8. **Write tests** for happy paths, error cases, and edge cases
9. **Optimize queries** and add caching where appropriate

## Knowledge Base

- FastAPI documentation and advanced patterns
- Pydantic v2 features and performance optimizations
- OAuth2/OpenID Connect specifications
- HTTP/1.1 and HTTP/2 standards
- RESTful API design best practices
- Security headers and CORS configuration

## Example Interactions

- "Create a user registration endpoint with email validation"
- "Implement JWT authentication with refresh tokens"
- "Add rate limiting to prevent API abuse"
- "Design a paginated list endpoint with filtering and sorting"
- "Create comprehensive error handling for database errors"
```

---

## Validation Rules

### File-Level Validation
- File must be valid Markdown
- File must have YAML frontmatter delimited by `---`
- Frontmatter must parse as valid YAML
- All required frontmatter fields must be present

### Frontmatter Validation
- `name`: Non-empty string
- `description`: Non-empty string, 50-200 chars
- `model`: Non-empty string

### Section Validation
- Must contain `## Purpose` heading
- Must contain `## Capabilities` heading
- Must contain `## Behavioral Traits` heading
- Must contain `## Response Approach` heading
- Capabilities section must not be empty (at least 1 bullet point)

---

## Update Contract (FR-026)

When updating an existing agent file to add capabilities:

**Rules**:
1. **Preserve frontmatter**: Do not modify name, description, or model
2. **Preserve all sections**: Do not remove Purpose, Behavioral Traits, Response Approach
3. **Add capabilities only**: Append to `## Capabilities` section
4. **Check for duplicates**: Use semantic similarity to avoid redundant bullets
5. **Maintain structure**: If capabilities use H3 subsections, add to appropriate subsection
6. **Preserve formatting**: Maintain bullet style, indentation, line breaks

**Example Update**:
```markdown
## Capabilities (before)

### API Development
- REST API design
- FastAPI framework

## Capabilities (after - added GraphQL)

### API Development
- REST API design
- FastAPI framework
- GraphQL schema design and resolvers  # <-- ADDED
```

---

## Error Conditions

**Invalid Frontmatter**:
- Missing required field → Error: "Missing required frontmatter field: 'name'"
- Invalid YAML syntax → Error: "Frontmatter is not valid YAML"

**Missing Required Sections**:
- No `## Capabilities` → Error: "Missing required section: '## Capabilities'"

**Empty Capabilities**:
- Capabilities section has no bullets → Warning: "Capabilities section is empty"

---

This contract ensures all agent files follow a consistent structure, enabling reliable parsing, updating, and reuse across features.

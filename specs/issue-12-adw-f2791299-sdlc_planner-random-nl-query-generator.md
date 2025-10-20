# Feature: Random Natural Language Query Generator

## Feature Description
A new button will be added to the Natural Language SQL Interface that generates interesting natural language queries based on the existing database tables and their structure. When clicked, this button will use the LLM processor to analyze the available tables, columns, and data types to create contextually relevant, natural language questions that users can execute. The generated query will automatically populate the query input field, replacing any existing content, allowing users to immediately run the query with a single click.

## User Story
As a user of the Natural Language SQL Interface
I want to see example queries that are relevant to my current data
So that I can discover what kinds of questions I can ask and learn by example without having to think of queries myself

## Problem Statement
New users often face a "blank slate" problem when first using the Natural Language SQL Interface. They may not know what types of questions to ask or how to phrase their queries effectively. Additionally, users with newly uploaded data may want to quickly explore their data without having to manually compose queries. This creates friction in the user experience and slows down the data exploration process.

## Solution Statement
We will add a "Generate Random Query" button positioned next to the existing primary buttons (Query and Upload Data) that, when clicked, will:
1. Analyze the current database schema (tables, columns, data types, row counts)
2. Use the existing LLM processor to generate a contextually relevant natural language query
3. Automatically populate the query input field with the generated query (overwriting existing content)
4. Limit the generated query to two sentences maximum for brevity

This provides immediate value by giving users concrete examples they can execute, helping them understand the system's capabilities and inspiring their own queries.

## Relevant Files
Use these files to implement the feature:

- `app/server/core/llm_processor.py` - Contains the LLM generation logic that will be extended to create a new function for generating random queries based on schema
- `app/server/core/sql_processor.py` - Contains `get_database_schema()` function that retrieves current database structure needed for query generation
- `app/server/core/data_models.py` - Where new Pydantic models will be defined for the random query request/response
- `app/server/server.py` - FastAPI server where a new endpoint `/api/generate-random-query` will be added
- `app/client/src/api/client.ts` - API client where the new endpoint method will be added
- `app/client/src/main.ts` - Main TypeScript file where the new button's event handler will be implemented
- `app/client/index.html` - HTML structure where the new button will be added to the query controls section
- `app/client/src/style.css` - Existing styles will be reused for the new button (using the `secondary-button` class to match the Upload Data button style)
- `app/server/tests/core/test_llm_processor.py` - Where unit tests for the new query generation function will be added

### New Files
- `.claude/commands/e2e/test_random_query.md` - New E2E test file that validates the random query generation feature works end-to-end

## Implementation Plan
### Phase 1: Foundation
Create the backend infrastructure for generating random queries by extending the existing LLM processor module with a new function that takes database schema information and generates contextually relevant natural language queries. This includes creating appropriate data models and a new FastAPI endpoint.

### Phase 2: Core Implementation
Implement the frontend button and wire it to the backend API. The button will trigger an API call, receive a generated query, and populate the input field. The generated queries should be interesting, varied, and demonstrate the system's capabilities.

### Phase 3: Integration
Ensure the new feature integrates seamlessly with existing functionality, including proper error handling, loading states, and user feedback. Create comprehensive E2E tests to validate the feature works as expected.

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### 1. Create Backend Data Models
- Add new Pydantic models in `app/server/core/data_models.py`:
  - `RandomQueryRequest` - Empty model (no parameters needed)
  - `RandomQueryResponse` - Contains `query: str` field for the generated query and `error: Optional[str]`

### 2. Implement Random Query Generation Function
- Create a new function `generate_random_query()` in `app/server/core/llm_processor.py`
- Function signature: `generate_random_query(schema_info: Dict[str, Any]) -> str`
- Use the same routing logic as `generate_sql()` (check for OpenAI key first, then Anthropic)
- Create prompts for both OpenAI and Anthropic that instruct the LLM to:
  - Analyze the provided database schema (tables, columns, types, row counts)
  - Generate an interesting, natural language query that would yield useful results
  - Limit the query to two sentences maximum
  - Focus on demonstrating various query capabilities (filtering, aggregation, sorting, multi-table joins when applicable)
  - Return ONLY the natural language query text, no explanations or metadata
- Handle edge cases: no tables available (should return helpful message), single table vs multiple tables

### 3. Add Unit Tests for Query Generation
- Add tests to `app/server/tests/core/test_llm_processor.py`:
  - Test successful query generation with single table
  - Test successful query generation with multiple tables
  - Test handling of empty schema (no tables)
  - Test that generated queries are within two-sentence limit
  - Mock LLM API calls to avoid actual API usage in tests

### 4. Create Backend API Endpoint
- Add new endpoint in `app/server/server.py`:
  - Route: `POST /api/generate-random-query`
  - Response model: `RandomQueryResponse`
  - Endpoint should:
    1. Get current database schema using `get_database_schema()`
    2. Call `generate_random_query()` with the schema
    3. Return the generated query in the response
    4. Handle errors gracefully and return error messages in the response

### 5. Add Frontend API Client Method
- Add new method to `app/client/src/api/client.ts`:
  - Method name: `generateRandomQuery()`
  - Returns: `Promise<RandomQueryResponse>`
  - Makes POST request to `/api/generate-random-query`

### 6. Add Frontend Button to UI
- Modify `app/client/index.html`:
  - Add new button in the `.query-controls` div after the "Upload Data" button
  - Button ID: `random-query-button`
  - Button class: `secondary-button` (to match Upload Data button style)
  - Button text: "Generate Random Query"

### 7. Implement Frontend Button Functionality
- Modify `app/client/src/main.ts`:
  - Create new function `initializeRandomQuery()`
  - Add event listener for the random query button click
  - On click:
    1. Disable the button and show loading state (spinner)
    2. Call `api.generateRandomQuery()`
    3. If successful, populate the query input field with the generated query (overwrite existing content)
    4. If error, display error message using existing `displayError()` function
    5. Re-enable the button and remove loading state
  - Call `initializeRandomQuery()` in the `DOMContentLoaded` event handler

### 8. Create E2E Test File
- Create `.claude/commands/e2e/test_random_query.md` based on the format of `test_basic_query.md`
- Include test steps that:
  1. Navigate to the application
  2. Verify the "Generate Random Query" button is present
  3. Click the "Generate Random Query" button
  4. Verify the query input field is populated with generated text
  5. Verify the generated query is not empty and is reasonable (not too long)
  6. Click the "Query" button to execute the generated query
  7. Verify the query executes successfully and returns results
  8. Take screenshots at key points
- Define success criteria that validate the button works and queries are executable

### 9. Run Validation Commands
- Execute all validation commands to ensure zero regressions:
  - Read `.claude/commands/test_e2e.md` to understand E2E test execution
  - Execute the new E2E test file `.claude/commands/e2e/test_random_query.md`
  - Run `cd app/server && uv run pytest` to validate server tests pass
  - Run `cd app/client && bun tsc --noEmit` to validate TypeScript compilation
  - Run `cd app/client && bun run build` to validate frontend builds successfully

## Testing Strategy
### Unit Tests
- Test `generate_random_query()` function with various schema configurations
- Mock LLM API responses to ensure consistent test results
- Test edge cases: empty database, single table, multiple tables, complex schemas
- Verify generated queries are properly formatted and within length limits
- Test error handling when LLM API fails
- Test proper routing between OpenAI and Anthropic based on available API keys

### Edge Cases
- No tables in database (should return a helpful message like "Upload data to generate queries")
- Only one table with minimal columns (should generate simple query)
- Multiple tables with relationships (should potentially generate JOIN queries)
- LLM API timeout or failure (should return error gracefully)
- Very long table/column names (should not break query generation)
- Special characters in table/column names (should handle safely)
- Query generation during active query execution (button should be disabled during loading)

## Acceptance Criteria
- A new "Generate Random Query" button appears in the query controls section next to the Upload Data button
- The button has the same visual style as the Upload Data button (using `secondary-button` class)
- Clicking the button triggers an API call to generate a random query
- The button shows a loading state while the query is being generated
- The generated query automatically populates the query input field, replacing any existing content
- Generated queries are limited to two sentences maximum
- Generated queries are contextually relevant based on the current database schema
- If no tables exist, the feature provides helpful feedback to the user
- The generated queries are executable and return valid results when the Query button is clicked
- All existing functionality continues to work without regression
- Unit tests pass with >90% coverage for new code
- E2E test passes and validates the entire user workflow
- Error states are handled gracefully with user-friendly messages
- The feature works with both OpenAI and Anthropic LLM providers

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/server && uv run pytest tests/core/test_llm_processor.py -v` - Run specific LLM processor tests with verbose output
- `cd app/client && bun tsc --noEmit` - Run frontend TypeScript compilation to validate no type errors
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions
- Read `.claude/commands/test_e2e.md` to understand the E2E test execution process
- Read and execute the new `.claude/commands/e2e/test_random_query.md` E2E test file to validate the feature works end-to-end with real browser interactions

## Notes

### Design Decisions
- **Button Placement**: Positioned next to Upload Data button to group secondary actions together, keeping the primary Query button visually distinct
- **Button Style**: Uses `secondary-button` class to match Upload Data button, creating visual consistency for non-primary actions
- **Overwrite Behavior**: Always overwrites query input field content as specified in requirements, providing clean slate for each generation
- **Two-Sentence Limit**: Keeps queries concise and readable, preventing overwhelming text in the input field
- **LLM Provider Routing**: Reuses existing routing logic from `generate_sql()` to maintain consistency

### Future Enhancements
- Consider adding a "Copy Query" button to save interesting generated queries
- Could implement query history to avoid repeating recently generated queries
- Potential to add query complexity parameter (simple/medium/complex)
- Could analyze query execution results to improve future query generation

### Implementation Notes
- No new dependencies required - leverages existing LLM infrastructure
- Generated queries should showcase various SQL capabilities: filtering, aggregation, sorting, joins
- The feature gracefully degrades when no tables are available
- Error handling follows existing patterns in the codebase for consistency

# E2E Test: Random Query Generation

Test the random query generation feature in the Natural Language SQL Interface application.

## User Story

As a user
I want to generate random natural language queries based on my database schema
So that I can discover what kinds of questions I can ask and learn by example

## Test Steps

1. Navigate to the `Application URL`
2. Take a screenshot of the initial state
3. **Verify** the page title is "Natural Language SQL Interface"
4. **Verify** core UI elements are present:
   - Query input textbox
   - Query button
   - Upload Data button
   - Generate Random Query button
   - Available Tables section

5. **Verify** the "Generate Random Query" button is visible and has the `secondary-button` class
6. Take a screenshot of the button placement

7. Click the "Generate Random Query" button
8. **Verify** the button shows a loading state (disabled with spinner)
9. Wait for the query to be generated (should take 1-3 seconds)
10. **Verify** the query input field is populated with generated text
11. **Verify** the generated query is not empty
12. **Verify** the generated query is reasonable in length (not excessively long)
13. Take a screenshot of the generated query in the input field

14. Click the "Query" button to execute the generated query
15. **Verify** the query executes successfully without errors
16. **Verify** query results are displayed
17. **Verify** the SQL translation is shown
18. Take a screenshot of the query results

19. Click the "Generate Random Query" button again
20. **Verify** a new query is generated (may be different from the first)
21. **Verify** the previous query is overwritten in the input field
22. Take a screenshot of the second generated query

## Edge Cases to Test

1. **No Tables Available**:
   - Clear all tables from the database
   - Click "Generate Random Query" button
   - **Verify** a helpful message is shown (e.g., "Upload data to start exploring...")
   - Take a screenshot

2. **Single Table**:
   - Ensure only one table exists
   - Click "Generate Random Query" button
   - **Verify** query is generated based on that single table
   - Take a screenshot

3. **Multiple Tables**:
   - Ensure multiple tables exist (e.g., users and products)
   - Click "Generate Random Query" button multiple times
   - **Verify** queries are varied and contextually relevant
   - Take screenshots

## Success Criteria

- "Generate Random Query" button is visible in the query controls section
- Button has the same visual style as the "Upload Data" button (secondary-button class)
- Clicking the button shows a loading state while generating
- Generated query automatically populates the query input field
- Generated query overwrites any existing content in the input field
- Generated queries are limited to approximately two sentences
- Generated queries are contextually relevant to the current database schema
- Generated queries are executable and return valid results when run
- When no tables exist, a helpful message is displayed
- Button can be clicked multiple times to generate different queries
- All existing functionality continues to work (no regressions)
- At least 5 screenshots are taken showing key states

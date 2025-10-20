# E2E Test: Input Disable and Request Debouncing

Test that the query input area and buttons are properly disabled during query execution, and that debouncing prevents rapid successive query submissions.

## User Story

As a user
I want the query input and buttons to be disabled when a query is executing
So that I cannot trigger multiple simultaneous query requests or interfere with ongoing operations

## Test Steps

1. Navigate to the `Application URL`
2. Take a screenshot of the initial state
3. **Verify** the page title is "Natural Language SQL Interface"
4. **Verify** core UI elements are present and enabled:
   - Query input textbox (should be enabled)
   - Query button (should be enabled)
   - Generate Random Query button (should be enabled)

5. Enter the query: "Show me all users from the users table"
6. Take a screenshot showing the query input is enabled
7. **Verify** the query input textarea is enabled (check the `disabled` property is false)
8. **Verify** the Query button is enabled
9. **Verify** the Generate Random Query button is enabled

10. Click the Query button
11. **Immediately verify** while the loading spinner is visible:
    - The query input textarea is disabled (check the `disabled` property is true)
    - The Query button is disabled
    - The Generate Random Query button is disabled
12. Take a screenshot showing all elements disabled during query execution

13. **Verify** that attempting to type in the textarea during execution is blocked (textarea should remain disabled)
14. **Verify** that attempting to click the Query button again during execution has no effect
15. **Verify** that attempting to click the Generate Random Query button during execution has no effect

16. Wait for the query to complete (loading spinner disappears)
17. **Verify** after query completion:
    - The query input textarea is re-enabled (check the `disabled` property is false)
    - The Query button is re-enabled
    - The Generate Random Query button is re-enabled
18. Take a screenshot showing all elements re-enabled after query completion

19. Enter a new query: "Count all users"
20. **Test debouncing**: Rapidly click the Query button 5 times in quick succession (within 300ms)
21. **Verify** that only ONE query request is sent (check network requests or observe that only one loading state occurs)
22. Take a screenshot after the debounce test

23. Click the Generate Random Query button
24. **Verify** while the random query is being generated:
    - The query input textarea is disabled
    - The Query button is disabled
    - The Generate Random Query button is disabled
25. Take a screenshot showing disabled state during random query generation

26. Wait for the random query to complete
27. **Verify** all elements are re-enabled after random query generation completes
28. Take a screenshot showing all elements re-enabled

29. Type a query in the textarea: "Show me all products"
30. **Test keyboard shortcut blocking**: While a query is running, press Cmd+Enter (Mac) or Ctrl+Enter (Windows/Linux)
31. **Verify** the keyboard shortcut does not trigger a new query while one is already running
32. Take a screenshot showing final state

## Success Criteria
- Query input textarea is enabled initially
- Query input textarea is disabled when Query button is clicked
- Query input textarea is disabled when Generate Random Query button is clicked
- All query-related buttons are disabled during query execution
- Keyboard shortcuts (Cmd+Enter / Ctrl+Enter) are blocked during query execution
- Rapid successive clicks are debounced (only one query executes)
- All elements re-enable after query completes
- All elements re-enable after random query generation completes
- Visual feedback (opacity, cursor) indicates disabled state
- 7 screenshots are taken showing all state transitions

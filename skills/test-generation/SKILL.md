---
name: test-generation
description: Use to generate ViewModel unit tests following the project's convention
---

When generating tests for a ViewModel:
- Use a Mock of the corresponding repository/service
- Cover the happy path and at least 2 error scenarios
- Name tests in the format test_<condition>_<expectedResult>
- Don't test UI details, only the ViewModel's state and logic

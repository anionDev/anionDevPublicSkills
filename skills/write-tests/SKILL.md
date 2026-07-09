---
name: write-tests
description: Ensures that there are tests for the most important functions of the project.
---
Write unit-tests for the most important non-trivial functions which do not have unit-tests yet.
Things to consider:
- Try to write tests that verify that an attacker can not do anything harmful.
- Tests should be independent and atomic.
- If the code to test throws an exception anywhere then for this case tests must check if there is a defined crash-behavior without revealing any sensitive information.
- Write more tests if the user explicitly asks for it.
If the repository defines any rules/scripts for tests, etc. then follow them.

## Usage

`/unit_test <COMPONENT_OR_FEATURE>`

## Context

- Target component/feature: $ARGUMENTS
- Test location: `*_test.go` files alongside implementation
- Frameworks: Go standard testing, testify/assert, testify/mock
- Pattern: Table-driven tests with mocks

## Your Role

You are the Test Coordinator managing four testing specialists:

1. **Unit Test Expert** – creates comprehensive unit tests with mocks
2. **Integration Test Engineer** – designs handler and repository tests
3. **Mock Designer** – creates testify mocks for interfaces
4. **Coverage Analyst** – ensures high test coverage

## Process

1. **Test Analysis**: Identify test scenarios and edge cases
2. **Test Implementation**:
   - Unit Test Expert: Write table-driven tests for usecases
   - Integration Test Engineer: Test handlers with httptest
   - Mock Designer: Create mocks for repositories and external services
   - Coverage Analyst: Ensure >80% coverage with go test -cover
3. **Test Organization**: Structure tests with setup, execution, assertion
4. **CI Integration**: Configure tests for GitHub Actions

## Output Format

1. **Test Implementation** – complete test files with all scenarios
2. **Mock Definitions** – testify mocks for interfaces
3. **Test Helpers** – utility functions for test setup
4. **Coverage Report** – analysis of test coverage
5. **CI Configuration** – GitHub Actions workflow for tests

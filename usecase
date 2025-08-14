# Clean Architecture Usecase Development Guide

## Usage

`/usecase <BUSINESS_LOGIC_DESCRIPTION>`

## Context

- **Business logic to implement**: $ARGUMENTS
- **Location**: `src/domain/usecase/`
- **Pattern**: Clean architecture with dependency injection
- **Error handling**: usecase_error domain errors

## Your Role

You are the Usecase Coordinator orchestrating four domain experts:

1. **Domain Analyst** – models business rules and entities
2. **Usecase Designer** – structures business logic flow
3. **Repository Architect** – designs data access patterns
4. **Error Strategist** – implements proper error handling

## Process

1. **Business Logic Analysis**: Break down requirements into domain concepts
2. **Implementation Design**:
   - Domain Analyst: Define entities and value objects
   - Usecase Designer: Structure Execute() method with validation
   - Repository Architect: Design repository interfaces and methods
   - Error Strategist: Define domain errors and error flows
3. **Clean Architecture**: Ensure proper dependency direction
4. **Testing Strategy**: Design unit tests with mocked repositories

## Output Format

1. **Usecase Implementation** – complete usecase with Execute() and Validate()
2. **Entity Definitions** – domain entities if needed
3. **Repository Interface** – repository methods required
4. **Error Definitions** – custom domain errors
5. **Unit Tests** – comprehensive test coverage with mocks

## Conventions & Best Practices

### 1. Clean Architecture Requirements

**CRITICAL: All dependencies MUST use interfaces, not concrete implementations**

✅ **Correct Interface Usage:**

```go
type MyUseCase struct {
    userRepo       adapter_interface.UserRepository      // ✅ Interface
    productRepo    adapter_interface.ProductRepository   // ✅ Interface
    notifier       adapter_interface.Notifier            // ✅ Interface
}

func NewMyUseCase(
    userRepo    adapter_interface.UserRepository,      // ✅ Interface
    productRepo adapter_interface.ProductRepository,   // ✅ Interface
    notifier    adapter_interface.Notifier,            // ✅ Interface
) *MyUseCase {
    // ...
}
```

❌ **Incorrect Concrete Usage:**

```go
type MyUseCase struct {
    userRepo *repository.PostgresUserRepository  // ❌ Concrete
    notifier *service.EmailNotifier              // ❌ Concrete
}
```

### 2. Interface Creation

- **Check existing interfaces first**: Look in `adapter_interface/` before creating new ones
- **Create new interfaces if needed**: Follow existing patterns in `adapter_interface/`
- **Create mocks**: Add to `adapter_interface/mocks/` following existing patterns

### 3. Mock Usage

**ALWAYS use existing mocks from `adapter_interface/mocks/` to avoid duplication:**

✅ **Correct Mock Usage:**

```go
import (
    "github.com/yourorg/yourproject/src/domain/usecase/adapter_interface/mocks"
)

func TestMyUseCase(t *testing.T) {
    mockProductRepo := new(mocks.MockProductRepository)     // ✅ Existing mock
    mockUserRepo := new(mocks.MockUserRepository)           // ✅ Existing mock
    mockNotifier := new(mocks.MockNotifier)                 // ✅ New mock if needed

    uc := NewMyUseCase(mockUserRepo, mockProductRepo, mockNotifier)
    // ...
}
```

❌ **Incorrect Custom Mock:**

```go
// Don't create custom mocks if existing ones are available
type CustomMockProductRepository struct {
    mock.Mock
}
```

### 4. Error Handling

- **Check existing errors**: Look in `usecase_error/` before creating new ones
- **Use proper field names**: Check existing error struct fields
- **Follow domain error patterns**: Use appropriate error types

### 5. Import Management

- **Use only interfaces**: Remove concrete implementation imports from use cases
- **Clean unused imports**: Remove any unused imports
- **Proper organization**: Group imports logically

### 6. File Formatting

- Always have a newline at the end of every file
- Use gofmt/goimports for consistent formatting

### 7. Testing Strategy

- **Use existing mocks**: Never duplicate existing mock implementations
- **Focus on business logic**: Test validation, error handling, and success paths
- **Mock all dependencies**: Ensure complete isolation of use case logic

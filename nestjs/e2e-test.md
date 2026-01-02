## Description
Generates comprehensive E2E tests for NestJS APIs. Supports both consumer-facing APIs and service-to-service communication.

## Generated Files Structure

### For API
For `User` entity with list API:

```
test/modules/user/
├── get-user-list.spec.ts              
├── helper.ts                          
└── factory/
    └── user.factory.ts                
```

## Test File Structures

### Setup Test Application, create if not exists

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import {
  INestApplication,
  ExecutionContext,
  ValidationPipe,
} from '@nestjs/common';
import { EventBus } from '@nestjs/cqrs';
import { PrismaService } from '../src/modules/prisma/prisma.service';
import { AppModule } from '../src/modules/app/app.module';
import { JwtAuthGuard } from '../src/modules/auth/auth.jwt.guard';

export interface ITestContext {
  app: INestApplication;
  module: TestingModule;
  prisma: PrismaService;
  testUser?: any;
  mockAuthGuard?: any;
  generateJwtToken: (payload: {
    userId: number;
    businessId: number;
    email: string;
  }) => string;
}

declare global {
  // eslint-disable-next-line @typescript-eslint/no-namespace
  namespace NodeJS {
    interface Global {
      testContext: ITestContext;
    }
  }
}

beforeAll(async () => {
  // Create a mock auth guard instance with a canActivate method that can be spied on
  const mockAuthGuard = {
    canActivate: jest.fn().mockImplementation((context: ExecutionContext) => {
      const request = context.switchToHttp().getRequest();
      // Set a default test user that can be overridden by individual tests
      request.user = {
        userId: 1,
        email: 'test@example.com',
        businessId: 1,
        roleId: 1,
        isActive: true,
      };
      return true;
    }),
  };

  const moduleFixture: TestingModule = await Test.createTestingModule({
    imports: [AppModule],
  })
    .overrideGuard(JwtAuthGuard)
    .useValue(mockAuthGuard)
    .compile();

  const testApp: INestApplication = moduleFixture.createNestApplication();

  // Apply the same global pipes as in main.ts
  testApp.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
      transformOptions: {
        enableImplicitConversion: true,
      },
    }),
  );

  await testApp.init();

  const testPrisma: PrismaService = testApp.get(PrismaService);

  // Clear database to ensure clean state
  await testPrisma.userRole.deleteMany({});
  await testPrisma.user.deleteMany({});

  // Ensure base role exists
  await testPrisma.role.upsert({
    where: { roleId: 1 },
    update: {},
    create: {
      roleId: 1,
      roleName: 'test-user',
    },
  });

  // Create base test user for authentication
  const testUser = await testPrisma.user.create({
    data: {
      email: 'test@example.com',
      passwordHash: 'hashed_password',
      isActive: true,
      userRoles: {
        create: {
          roleId: 1, // Assume role 1 exists or will be created
        },
      },
    },
  });
  console.log('Created test user with ID:', testUser.userId);

  global.testContext = {
    app: testApp,
    module: moduleFixture,
    prisma: testPrisma,
    testUser, // Store the test user for reference
    mockAuthGuard, // Store the mock auth guard for individual test customization
    generateJwtToken: ({}) => {
      // For tests, we just return a mock token since the JWT strategy is mocked
      return 'mock-jwt-token';
    },
  };

  console.log('✅ Test setup completed');
}, 30000);

afterAll(async () => {
  if (global.testContext) {
    try {
      // Disconnect Prisma
      await global.testContext.prisma.$disconnect();

      // Close the NestJS application
      await global.testContext.app.close();

      console.log('✅ Test cleanup completed');
    } catch (error) {
      console.error('Error during cleanup:', error);
    }
  }
}, 10000);

beforeEach(async () => {
  if (global.testContext) {
    const eventBus = global.testContext.app.get(EventBus);
    jest.spyOn(eventBus, 'publish').mockImplementation(() => {});
  }
});

```

### API Test

```typescript
import request from 'supertest';
import { INestApplication, ExecutionContext } from '@nestjs/common';
import { User } from '@prisma/client';
import { JwtAuthGuard } from 'src/modules/auth/auth.jwt.guard';
import { ROLES_ID } from 'src/shared/constants/global.constants';
import { UserHelper } from './helper';

describe('Get list user (e2e)', () => {
  let app: INestApplication;
  let helper: UserHelper;

  beforeAll(async () => {
    app = global.testContext.app;
    helper = new UserHelper(global.testContext.module);
  });

  beforeEach(async () => {
  });

  afterEach(async () => {
    await helper.clear();
  });

  describe('GET /users - Get list user', () => {
    let authGuardMock: jest.SpyInstance;

    beforeEach(async () => {
      // Mock guards
      const authGuard = app.get(JwtAuthGuard);

      authGuardMock = jest
        .spyOn(authGuard, 'canActivate')
        .mockImplementation((context: ExecutionContext) => {
          const request = context.switchToHttp().getRequest();
          request.user = {
            userId: 1,
            email: 'test@example.com',
            businessId: 1,
            roleId: ROLES_ID.ADMIN,
          };
          return true;
        });
    });

    afterEach(() => {
      authGuardMock?.mockRestore();
      jest.clearAllMocks();
    });

    it('should successfully get list user', async () => {
      const users = await Promise.all([
        helper.createTestUser({
          email: 'test@example.com',
          isActive: true,
        }),
        helper.createTestUser({
          email: 'test2@example.com',
          isActive: true,
        }),
      ])

      const response = await request(app.getHttpServer())
        .get('/users')
        .set('Authorization', 'Bearer mock-token')
        .expect(200);

      expect(response.body.data.length).toBe(users.length);
    });
  });
});

```

## Test Scenarios

### Standard Test Scenarios (Auto-generated based on API type)

#### List API Scenarios
- **Basic Success**: Returns data successfully
- **Empty Results**: Handles empty result sets
- **Pagination**: Tests offset, limit, and total count
- **Sorting**: Tests all available sort options
- **Filtering**: Tests each filter parameter
- **Search**: Tests text search functionality (if applicable)
- **Validation**: Tests input validation for all parameters
- **Error Handling**: Tests database errors and edge cases

#### Detail API Scenarios
- **Basic Success**: Returns single entity successfully
- **Not Found**: Returns 404 for non-existent entities
- **Relationships**: Tests included related entities
- **Validation**: Tests UUID validation for ID parameter
- **Error Handling**: Tests various error conditions

#### Create API Scenarios
- **Basic Success**: Creates entity successfully
- **Validation**: Tests input validation for all parameters
- **Error Handling**: Tests database errors and edge cases

#### Update API Scenarios
- **Basic Success**: Updates entity successfully
- **Not Found**: Returns 404 for non-existent entities
- **Validation**: Tests input validation for all parameters
- **Error Handling**: Tests database errors and edge cases

#### Delete API Scenarios
- **Basic Success**: Deletes entity successfully
- **Not Found**: Returns 404 for non-existent entities
- **Error Handling**: Tests various error conditions

## Service Mocking Patterns


**Rule**: Mock all external services, keep only database operations real.

#### Service-Specific Mocking
```typescript
// Mock specific service methods
jest
  .spyOn(externalService, 'createExternalUser')
  .mockImplementationOnce((): Promise => {
    return Promise.resolve({
      data: {
        id: '1',
        email: 'test@example.com',
        name: 'test',
        role: 'test',
        businessId: 1,
      },
    });
  });
```

#### Mock Reset Pattern
```typescript
afterEach(async () => {
  jest.resetAllMocks(); // Reset all mocks after each test
  await entityHelper.clearData();
});
```

#### Error Simulation Mocking
```typescript
it('should handle service failures gracefully', async () => {
  jest.resetAllMocks();
  jest
    .spyOn(externalService, 'createExternalUser')
    .mockRejectedValueOnce(new Error('External service unavailable'));
});
```

#### Service Dependencies Testing
```typescript
describe('Service Integration', () => {
  it('should call external service with correct parameters', async () => {
    const externalSpy = jest.spyOn(externalService, 'createExternalUser')
      .mockImplementation(() => Promise.resolve({ data: fakeExternalUserData }));

    await request(server)
      .post('/users')
      .set('Authorization', 'Bearer mock-token')
      .send({
        email: 'test@example.com',
        name: 'test',
        role: 'test',
        businessId: 1,
      })

    expect(externalSpy).toHaveBeenCalledWith(
      expect.objectContaining({
        cmd: 'createExternalUser',
        data: expect.objectContaining({
          email: 'test@example.com',
          name: 'test',
          role: 'test',
          businessId: 1,
        })
      })
    );
  });
});
```

## Mock Data Factory (`{entity}.factory.ts`)

```typescript
import { User } from '@prisma/client';
import { v4 as uuidv4 } from 'uuid';

export const createUser = (overrides?: any) => {
  const userId = overrides?.userId || uuidv4();

  return {
    userId,
    email: overrides?.email || 'test@example.com',
    name: overrides?.name || 'Test User',
    role: overrides?.role || 'test',
    businessId: overrides?.businessId || 1,
    isActive: overrides?.isActive !== undefined ? overrides.isActive : true,
    ...overrides,
  };
};
```

## Helper Functions (`helper.ts`)

```typescript
import { TestingModule } from '@nestjs/testing';
import { PrismaService } from 'src/modules/prisma/prisma.service';
import {
  createTestUser,
} from '../../factory';

export class UserHelper {
  public readonly prisma: PrismaService;

  constructor(module: TestingModule) {
    this.prisma = module.get(PrismaService);
  }

  async createTestUser(overrides?: any) {
    const userData = createTestUser(overrides);
    return this.prisma.user.create({
      data: userData as any,
    });
  }

  async createTestUser(overrides?: any) {
    const userData = createTestUser(overrides);
    return this.prisma.user.create({
      data: userData as any,
    });
  }
  
  async clear() {
    // Clean up in correct order due to foreign key constraints
    await this.prisma.user.deleteMany({});
  }
}
```

## Test Data Patterns

### Realistic Test Data
- Use faker.js for generating realistic data
- Follow entity validation rules
- Include edge cases (empty strings, null values, etc.)
- Test with various data types and formats

### Relationship Testing
- Test APIs with and without related entities
- Verify correct relationship loading
- Test relationship filtering and sorting

### Permission Testing
- Test different user roles and permissions
- Verify organization-scoped access
- Test cross-tenant data isolation

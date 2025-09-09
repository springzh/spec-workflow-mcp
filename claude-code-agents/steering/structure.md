# Project Structure

## Directory Layout
```
project-root/
├── src/
│   ├── components/     # UI components and views
│   ├── services/       # Business logic and services
│   ├── repositories/   # Data access layer
│   ├── types/          # TypeScript type definitions
│   ├── utils/          # Utility functions and helpers
│   ├── hooks/          # Custom React hooks
│   ├── styles/         # CSS/styling files
│   └── tests/          # Test files
├── public/             # Static assets
├── docs/               # Documentation
├── scripts/            # Build and deployment scripts
├── config/             # Configuration files
└── .spec-workflow/      # Spec workflow documents
    ├── specs/          # Feature specifications
    └── steering/       # Project guidance documents
```

## Naming Conventions
- **Files**: kebab-case (e.g., user-profile.ts, button-component.tsx)
- **Components**: PascalCase (e.g., UserProfile, ButtonComponent)
- **Services**: camelCase with Service suffix (e.g., userService, authService)
- **Types**: PascalCase with Type/Interface suffix (e.g., UserType, UserProfile)
- **Constants**: UPPER_SNAKE_CASE (e.g., API_BASE_URL, MAX_RETRIES)
- **Functions**: camelCase (e.g., getUserData, validateInput)
- **Variables**: camelCase (e.g., userName, isLoading)
- **CSS Classes**: kebab-case (e.g., user-profile, primary-button)

## File Organization Principles
- **Single Responsibility**: Each file handles one specific concern
- **Related Files**: Group related functionality in the same directory
- **Index Files**: Use index.ts for exporting related modules
- **Barrel Exports**: Export frequently used items from directory index
- **Test Files**: Place test files alongside implementation files

## Component Structure
```
src/components/
├── common/            # Reusable UI components
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.styles.ts
│   │   ├── Button.test.tsx
│   │   └── index.ts
│   └── Input/
│       ├── Input.tsx
│       ├── Input.styles.ts
│       ├── Input.test.tsx
│       └── index.ts
├── features/          # Feature-specific components
│   ├── Auth/
│   │   ├── Login/
│   │   ├── Register/
│   │   └── PasswordReset/
│   └── Dashboard/
│       ├── Overview/
│       ├── Analytics/
│       └── Settings/
└── layout/            # Layout components
    ├── Header/
    ├── Sidebar/
    └── Footer/
```

## Service Layer Structure
```
src/services/
├── api/              # External API integrations
│   ├── userService.ts
│   ├── authService.ts
│   └── apiClient.ts
├── business/         # Business logic services
│   ├── validationService.ts
│   ├── calculationService.ts
│   └── workflowService.ts
├── data/            # Data access services
│   ├── userRepository.ts
│   ├── cacheService.ts
│   └── databaseService.ts
└── types/           # Service-related types
    ├── serviceTypes.ts
    └── apiTypes.ts
```

## Type Definitions Structure
```
src/types/
├── api/             # API-related types
│   ├── requestTypes.ts
│   ├── responseTypes.ts
│   └── errorTypes.ts
├── domain/          # Domain/business types
│   ├── userTypes.ts
│   ├── productTypes.ts
│   └── orderTypes.ts
├── ui/             # UI/component types
│   ├── componentProps.ts
│   ├── themeTypes.ts
│   └── formTypes.ts
└── common/         # Common/shared types
    ├── baseTypes.ts
    ├── utilityTypes.ts
    └── constants.ts
```

## Utility Functions Structure
```
src/utils/
├── validation/     # Validation utilities
│   ├── formValidation.ts
│   ├── inputValidation.ts
│   └── schemaValidation.ts
├── formatting/     # Formatting utilities
│   ├── dateUtils.ts
│   ├── numberUtils.ts
│   └── stringUtils.ts
├── helpers/        # General helper functions
│   ├── arrayUtils.ts
│   ├── objectUtils.ts
│   └── asyncUtils.ts
└── constants/      # Application constants
    ├── appConstants.ts
    ├── apiConstants.ts
    └── validationConstants.ts
```

## Test Organization
```
src/tests/
├── __mocks__/      # Mock files
├── fixtures/       # Test data fixtures
├── helpers/        # Test helper functions
├── unit/           # Unit tests
│   ├── components/
│   ├── services/
│   └── utils/
├── integration/    # Integration tests
│   ├── api/
│   ├── database/
│   └── workflows/
└── e2e/           # End-to-end tests
    ├── auth-flows/
    ├── user-flows/
    └── api-flows/
```

## Configuration Files
```
config/
├── app.config.ts       # Application configuration
├── database.config.ts  # Database configuration
├── api.config.ts       # API configuration
├── test.config.ts      # Test environment configuration
└── deployment.config.ts # Deployment configuration
```

## Documentation Structure
```
docs/
├── api/               # API documentation
├── guides/            # User guides and tutorials
├── architecture/      # Architecture documentation
├── deployment/        # Deployment guides
└── troubleshooting/   # Troubleshooting guides
```

## Build and Deployment Scripts
```
scripts/
├── build/            # Build scripts
├── deploy/           # Deployment scripts
├── migrate/          # Database migration scripts
├── test/             # Test execution scripts
└── utils/            # Utility scripts
```

## Environment Configuration
```
.env                 # Local development environment
.env.staging        # Staging environment
.env.production     # Production environment
.env.example        # Environment template
```

## Git Hooks
```
.husky/
├── pre-commit        # Pre-commit hooks
├── commit-msg        # Commit message validation
└── pre-push          # Pre-push hooks
```

## Example: E-commerce Application Structure

## Directory Layout
```
ecommerce-app/
├── src/
│   ├── components/
│   │   ├── product/
│   │   ├── cart/
│   │   ├── checkout/
│   │   └── user/
│   ├── services/
│   │   ├── productService.ts
│   │   ├── cartService.ts
│   │   ├── orderService.ts
│   │   └── userService.ts
│   ├── types/
│   │   ├── productTypes.ts
│   │   ├── orderTypes.ts
│   │   └── userTypes.ts
│   └── utils/
│       ├── currency.ts
│       ├── validation.ts
│       └── formatting.ts
├── public/
│   ├── images/
│   ├── fonts/
│   └── favicon.ico
└── .spec-workflow/
    ├── specs/
    └── steering/
```

## Key Principles
1. **Consistency**: Follow established patterns throughout the project
2. **Scalability**: Structure should support project growth
3. **Maintainability**: Easy to find and modify code
4. **Testability**: Structure should facilitate testing
5. **Documentation**: Clear organization makes the project self-documenting
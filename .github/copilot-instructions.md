# GitHub Copilot Instructions for OctoCAT Supply

## Project Overview

OctoCAT Supply is a supply chain management demo application built to showcase GitHub Copilot capabilities. This is a TypeScript monorepo featuring:

- **Backend API**: Express.js REST API with OpenAPI/Swagger documentation
- **Frontend**: React 18+ with TypeScript, Vite, and Tailwind CSS
- **Purpose**: Demonstrate Copilot features including Agent Mode, test generation, and AI-assisted development

## Architecture

### Monorepo Structure

```
├── api/                 # Backend Express.js API
│   ├── src/
│   │   ├── models/     # Entity interface definitions with Swagger schemas
│   │   ├── routes/     # Express route handlers with Swagger docs
│   │   ├── seedData.ts # In-memory data storage
│   │   └── index.ts    # Main server entry point
│   └── package.json
├── frontend/            # React frontend application
│   ├── src/
│   │   ├── components/ # React components (entity, admin, general)
│   │   ├── context/    # Context API providers (Auth, Theme)
│   │   ├── api/        # API configuration
│   │   └── main.tsx    # Application entry point
│   └── package.json
├── docs/               # Project documentation
├── infra/              # Deployment configuration
└── package.json        # Root workspace configuration
```

### Entity-Relationship Model

The application models a supply chain with these core entities:
- Headquarters → Branches → Orders → OrderDetails → Products
- Suppliers → Deliveries → OrderDetailDeliveries
- Each entity has corresponding model, route, and component files

## Backend API Conventions

### File Organization

- **Models** (`api/src/models/*.ts`): TypeScript interfaces with JSDoc Swagger schemas
- **Routes** (`api/src/routes/*.ts`): Express Router with Swagger documentation annotations
- **Tests** (`api/src/routes/*.test.ts`): Vitest tests using Supertest

### API Patterns

#### Model Definitions

Models are TypeScript interfaces with Swagger schema documentation:

```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     EntityName:
 *       type: object
 *       required:
 *         - requiredField
 *       properties:
 *         field:
 *           type: string
 *           description: Field description
 */
export interface EntityName {
    field: string;
}
```

#### Route Handlers

All routes follow this pattern:
- Use Express Router
- Include comprehensive Swagger documentation
- Use in-memory data from seedData.ts
- Export router as default
- Export data reset function for testing
- Support full CRUD operations: GET (all), GET (by ID), POST, PUT, DELETE

Example route structure:
```typescript
/**
 * @swagger
 * /api/entities:
 *   get:
 *     summary: Returns all entities
 *     tags: [EntityName]
 *     responses:
 *       200:
 *         description: List of all entities
 */
router.get('/', (req, res) => {
    res.json(entities);
});

export function resetEntities() {
    entities = [...seedEntities];
}

export default router;
```

#### API Configuration

- CORS is configured to allow localhost and GitHub Codespaces
- Environment variable `API_CORS_ORIGINS` can override CORS settings
- Default port is 3000 (configurable via `PORT` environment variable)
- Swagger UI available at `/api-docs`
- API endpoints use `/api/` prefix

### Testing Conventions

- Use **Vitest** for testing framework
- Use **Supertest** for API endpoint testing
- Test files named `*.test.ts`
- Always reset data with `reset*()` function in `beforeEach`
- Test all CRUD operations: create, read all, read by ID, update, delete
- Verify response status codes and body content

Test structure:
```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import request from 'supertest';
import express from 'express';

describe('Entity API', () => {
    beforeEach(() => {
        app = express();
        app.use(express.json());
        app.use('/entities', entityRouter);
        resetEntities();
    });

    it('should create a new entity', async () => {
        const response = await request(app).post('/entities').send(newEntity);
        expect(response.status).toBe(201);
        expect(response.body).toEqual(newEntity);
    });
});
```

## Frontend Conventions

### Component Organization

- **Entity components**: `src/components/entity/{entityName}/`
  - List views: `{EntityName}s.tsx` (e.g., `Products.tsx`)
  - Forms: `{EntityName}Form.tsx`
- **Admin components**: `src/components/admin/`
- **General components**: `src/components/` (Navigation, Footer, Login, etc.)
- **Context providers**: `src/context/` (AuthContext, ThemeContext)

### React Patterns

#### State Management

- Use **React Query** for server state (data fetching)
- Use **Context API** for global state (auth, theme)
- Use `useState` for local component state
- Use custom hooks in `src/context/` for context consumers

#### Component Structure

Components follow this pattern:
```typescript
import { useState } from 'react';
import axios from 'axios';
import { useQuery } from 'react-query';
import { api } from '../../../api/config';
import { useTheme } from '../../../context/ThemeContext';

interface EntityType {
    id: number;
    name: string;
}

const fetchEntities = async (): Promise<EntityType[]> => {
    const { data } = await axios.get(`${api.baseURL}${api.endpoints.entities}`);
    return data;
};

export default function Entities() {
    const { data: entities, isLoading, error } = useQuery('entities', fetchEntities);
    const { darkMode } = useTheme();
    
    // Component logic...
}
```

#### Styling

- Use **Tailwind CSS** utility classes exclusively
- Support dark mode via `darkMode` from ThemeContext
- Conditional classes: `{darkMode ? 'bg-gray-800' : 'bg-white'}`
- Responsive design with Tailwind breakpoints: `sm:`, `md:`, `lg:`

#### API Integration

- API configuration in `src/api/config.ts`
- Base URL determined by environment (localhost or Codespaces)
- All API calls use axios with React Query
- Query keys match entity names for cache management

### Frontend Testing

- Use **Vitest** with **jsdom** environment
- Use **Testing Library** (@testing-library/react)
- Test files colocated or in `__tests__` directories
- Test component rendering, user interactions, and API integration

## TypeScript Conventions

### General Rules

- Use TypeScript strict mode
- Define interfaces for all data models
- Avoid `any` type; use `unknown` or proper typing
- Export interfaces that are used across files
- Use optional chaining (`?.`) for potentially undefined values

### Naming Conventions

- Interfaces: PascalCase (e.g., `Product`, `OrderDetail`)
- Functions/variables: camelCase (e.g., `fetchProducts`, `handleSubmit`)
- Components: PascalCase (e.g., `ProductForm`, `Navigation`)
- Constants: camelCase or UPPER_SNAKE_CASE for true constants
- Files: Match primary export name (e.g., `product.ts` for Product interface)

### Type Safety

- Define return types for functions
- Use proper typing for async functions (e.g., `Promise<Type[]>`)
- Type event handlers explicitly: `React.FormEvent`, `React.ChangeEvent`
- Avoid type assertions; prefer type guards

## Development Workflow

### Commands

Root level (runs in all workspaces):
```bash
npm install              # Install all dependencies
npm run build           # Build API and frontend
npm run dev             # Run both API and frontend in dev mode
npm test                # Run all tests
npm run lint            # Lint frontend code
```

API workspace:
```bash
npm run dev --workspace=api      # Run API in dev mode (tsx)
npm run build --workspace=api    # Build API (TypeScript compilation)
npm run test --workspace=api     # Run API tests
npm run test:coverage --workspace=api  # Run tests with coverage
```

Frontend workspace:
```bash
npm run dev --workspace=frontend     # Run Vite dev server
npm run build --workspace=frontend   # Build for production
npm run lint --workspace=frontend    # Run ESLint
```

### VS Code Integration

- Debug configurations available in `.vscode/launch.json`
- Build tasks in `.vscode/tasks.json`
- Use "Start API & Frontend" debug configuration to run both services

### Development Servers

- **API**: http://localhost:3000
  - Swagger docs: http://localhost:3000/api-docs
- **Frontend**: http://localhost:5137 (or as configured by Vite)

## Coding Best Practices

### API Development

1. **Always add Swagger documentation** for new endpoints
2. **Export reset function** for testing: `export function reset{Entity}()`
3. **Use descriptive error messages** in responses
4. **Follow RESTful conventions**: GET (list/detail), POST (create), PUT (update), DELETE (remove)
5. **Validate input data** before processing
6. **Use appropriate HTTP status codes**: 200 (OK), 201 (Created), 404 (Not Found), etc.

### Frontend Development

1. **Use React Query** for all data fetching
2. **Implement loading and error states** for async operations
3. **Support dark mode** in all new components
4. **Use semantic HTML** elements
5. **Make components responsive** with Tailwind breakpoints
6. **Extract reusable logic** into custom hooks
7. **Keep components focused** - single responsibility

### Testing

1. **Write tests for all new routes/endpoints**
2. **Test both success and error cases**
3. **Reset data before each test** using reset functions
4. **Test user interactions** in frontend components
5. **Aim for meaningful coverage**, not just high percentages
6. **Use descriptive test names** that explain what is being tested

### Common Anti-Patterns to Avoid

❌ **Don't**:
- Mix inline styles with Tailwind classes
- Use `any` type without justification
- Forget to add Swagger documentation for API endpoints
- Skip error handling in API routes
- Create components without TypeScript interfaces
- Ignore loading/error states in data fetching
- Hardcode URLs - use API configuration
- Commit `node_modules`, `dist`, or `coverage` directories

✅ **Do**:
- Use Tailwind utilities for all styling
- Define proper TypeScript interfaces
- Document all API endpoints with Swagger
- Handle errors gracefully with try-catch
- Type all component props and state
- Show loading spinners and error messages
- Use centralized API configuration
- Follow the established patterns in existing code

## Project-Specific Context

### Data Storage

- **No database**: All data is in-memory via `seedData.ts`
- Data resets on server restart
- This is intentional for demo purposes
- Tests use reset functions to ensure clean state

### Demo Purpose

This application is designed to demonstrate:
- GitHub Copilot Agent Mode capabilities
- Test generation with Copilot
- Infrastructure as Code generation
- Custom instructions and prompt files
- MCP Server integration (Playwright)

When adding features, prioritize:
- Clear, demonstrable functionality
- Good documentation for learning
- Patterns that showcase Copilot capabilities

### Environment Variables

- `PORT`: API server port (default: 3000)
- `API_CORS_ORIGINS`: Comma-separated CORS origins
- `VITE_API_BASE_URL`: Frontend API base URL (auto-detected in Codespaces)

### Compatibility

- **Node.js**: >= 18 (specified in engines)
- **Package Manager**: npm (uses workspaces)
- **Browsers**: Modern browsers with ES6+ support
- **Development Environment**: Works in GitHub Codespaces and local development

## Documentation

Project documentation is in the `/docs` folder:
- `architecture.md`: System architecture and design
- `build.md`: Build and deployment instructions
- `tao.md`: Observability patterns
- `demo-script.md`: Demo presentation guide
- `model-comparison.md`: AI model comparisons

When making significant changes:
1. Update relevant documentation files
2. Keep README.md in sync with major features
3. Update API Swagger docs for endpoint changes
4. Document breaking changes clearly

## Common Tasks

### Adding a New Entity

1. Create interface in `api/src/models/{entity}.ts` with Swagger schema
2. Add seed data to `api/src/seedData.ts`
3. Create route handler in `api/src/routes/{entity}.ts` with Swagger docs
4. Register route in `api/src/index.ts`
5. Create React component in `frontend/src/components/entity/{entity}/`
6. Add API endpoint to `frontend/src/api/config.ts`
7. Write tests in `api/src/routes/{entity}.test.ts`

### Adding a New API Endpoint

1. Add Swagger documentation annotation
2. Implement route handler with proper error handling
3. Add to appropriate router
4. Write test cases
5. Update frontend to consume if needed

### Adding a New React Component

1. Create component file in appropriate directory
2. Define TypeScript interfaces for props
3. Implement with Tailwind styling
4. Support dark mode
5. Add loading/error states if fetching data
6. Export as default

## Getting Help

- Check existing code patterns before implementing new features
- Review Swagger docs at `/api-docs` for API reference
- Use GitHub Copilot to understand code context
- Refer to documentation in `/docs` folder
- The entire project was created with Copilot - ask Copilot to explain patterns!

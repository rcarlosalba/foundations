---
description: Ruleset for NestJS backend development. Enforces strict modular architecture, Dependency Injection, Data Transfer Objects (DTOs) with validation, and standardized error handling.
globs: "**/*.ts"
---

# NestJS Development Rules

**Core Goal:** Generate highly modular, testable, and strictly typed backend code leveraging NestJS's native decorators, Dependency Injection system, and ecosystem (Pipes, Guards, Interceptors).

## 1. Core Principles & Architecture
1. **The Nest Way:** Always use NestJS native building blocks. Do NOT bypass them to use underlying Express/Fastify request/response objects directly unless strictly necessary.
    - ✅ **CORRECT:** `@Body() dto: CreateUserDto`, `return service.create(dto)`
    - ❌ **WRONG:** `req.body`, `res.status(201).json(...)`
2. **Strict Modularity:** Every major feature MUST be encapsulated in its own Module (`FeatureModule`), which provides its own Services and Controllers. Avoid dumping everything into `AppModule`.
3. **TypeScript Strictness:** The use of `any` is strictly forbidden. Everything from DTOs to Service returns MUST be strongly typed using classes, interfaces, or types.

## 2. Controllers vs. Services (Separation of Concerns)
1. **Skinny Controllers:** Controllers are ONLY responsible for handling HTTP routing, extracting parameters/body using decorators, and calling a Service. They must contain ZERO business logic.
2. **Fat Services:** All business logic, database calls, and third-party API integrations belong in `@Injectable()` Services.
3. **Repository Pattern / Data Access:** Services should not write raw SQL. They must inject a Repository (TypeORM) or a Client (Prisma) to interact with the database.

## 3. Data Validation & DTOs
Incoming data MUST be validated at the application's edge.

1. **Class Validator:** ALWAYS use `class-validator` and `class-transformer` decorators inside DTO classes to validate incoming payloads.
2. **DTOs as Classes:** DTOs must be defined as `class` (not `interface` or `type`) so that NestJS Pipes can access their metadata at runtime.
    - ✅ **CORRECT:** ```typescript
      import { IsString, IsEmail, MinLength } from 'class-validator';
      export class CreateUserDto {
        @IsEmail() email: string;
        @IsString() @MinLength(8) password: string;
      }
      ```
3. **No Manual Validation:** Never write manual `if (!body.email) throw new Error(...)` inside Controllers or Services. Rely on the `ValidationPipe`.

## 4. Security & Configuration
1. **Environment Variables:** NEVER hardcode credentials. ALWAYS use `@nestjs/config` (`ConfigService`) to retrieve environment variables.
2. **Authentication/Authorization:** ALWAYS use NestJS `Guards` (e.g., `AuthGuard`) to protect endpoints. Do not check tokens manually inside controllers.
3. **User Extraction:** Use custom decorators (e.g., `@CurrentUser()`) to extract user information from the request object after it passes the Guard.



## 5. Error Handling & Standardization
1. **Standardized Exceptions:** Use built-in NestJS exceptions (`NotFoundException`, `BadRequestException`, `UnauthorizedException`) instead of throwing generic JavaScript `Error` objects.
2. **Global Response Format:** If the Spec requires it, use an Interceptor to format all successful responses into a standard JSON structure (e.g., `{ data: ..., meta: ... }`).

## 6. Directory Structure Reference
```text
src/
├── app.module.ts
├── common/                 # Global decorators, filters, guards, interceptors
│   ├── decorators/
│   ├── exceptions/         # Domain-specific exception classes
│   ├── filters/
│   └── guards/
└── modules/                # Feature modules
    └── users/
        ├── dto/            # Data Transfer Objects (Validation)
        ├── users.controller.ts
        ├── users.module.ts
        └── users.service.ts
```

---

### ✅ LLM Pre-Generation Checklist

Before finalizing any NestJS code, verify:

1. Is the Controller free of business logic? Only decorators + service method calls allowed.
2. Are all incoming DTOs validated with `class-validator` decorators (`@IsEmail()`, `@IsString()`, etc.)?
3. Are DTOs defined as `class`, not `interface` or `type`?
4. Is `any` used anywhere? Remove it — use proper types, interfaces, or generics.
5. Are environment variables read via `ConfigService`, not `process.env` directly?
6. Are endpoints that require authentication protected by a `Guard`?
7. Are NestJS built-in exceptions used (`NotFoundException`, `BadRequestException`) instead of generic `Error`?
8. Are custom domain exceptions defined in `common/exceptions/` and extending the appropriate NestJS base exception?
---
name: ali-java-code-style
description: Apply Alibaba-style Java coding conventions before generating or modifying Java code. Use for Java classes, methods, services, controllers, DTOs, DAOs, tests, Spring/SOF code, and any task that asks Codex to create, refactor, or review Java code; enforce standard class/method comments with class author fixed as Demon.
---

# Ali Java Code Style

## Pre-Generation Checklist

Before generating or modifying Java code, apply these rules unless the repository has a stricter local convention:

1. Match the existing package/module structure and dependency direction.
2. Keep APIs small, cohesive, and explicit; avoid speculative abstractions.
3. Prefer readable code over clever code; remove dead code and unused imports.
4. Validate inputs at boundaries and fail with clear exceptions or result objects.
5. Do not hard-code secrets, environment values, magic numbers, or locale/time assumptions.
6. Add focused tests for business logic, edge cases, and bug fixes.

## Naming & Layout

- Use UpperCamelCase for classes and enums; lowerCamelCase for methods, fields, and local variables; UPPER_SNAKE_CASE for constants.
- Use meaningful names; avoid `a`, `b`, `temp`, `data`, `obj`, or abbreviations that are not domain terms.
- Name interfaces and implementations according to local style. If no style exists, use business names for interfaces and `XxxImpl` for implementations.
- Name DTO/VO/BO/DO/DAO classes consistently with their role.
- Keep one top-level public class per file and keep methods short enough to read without scrolling heavily.

## Comments & Documentation

Generate standard comments for every new public or protected class and method. For private methods, add comments only when the logic is non-obvious.

Class comment template:

```java
/**
 * Briefly describe the class responsibility.
 *
 * @author Demon
 */
```

Method comment template:

```java
/**
 * Briefly describe what the method does.
 *
 * @param paramName describe the parameter
 * @return describe the return value
 * @throws ExceptionType describe when it is thrown
 */
```

Omit `@param`, `@return`, or `@throws` tags that do not apply. Comments must explain intent and contract, not restate code.

## Java Practices

- Compare strings with constants first when null safety helps: `"OK".equals(status)`.
- Use `BigDecimal` for money and precision-sensitive decimal calculations; avoid `double`/`float`.
- Avoid catching broad `Exception` unless rethrowing or translating with context.
- Do not swallow exceptions. Log useful context without leaking secrets.
- Use parameterized logging instead of string concatenation.
- Check collection emptiness with utility methods already used by the project, or `collection != null && !collection.isEmpty()`.
- Avoid modifying collections while iterating unless using safe iterator APIs.
- Be explicit about transaction boundaries, idempotency, and retry behavior in service/integration code.

## Spring/SOF & Persistence

- Keep controllers thin; place business rules in service/biz modules.
- Keep DAL objects focused on persistence and avoid business decisions there.
- Validate external request objects before invoking business logic.
- Keep Spring XML bean names clear and consistent with module naming.

## Testing

- Use JUnit 5 and Mockito unless the module already uses another test stack.
- Name tests `XxxTest` and test methods by behavior, such as `shouldRejectInvalidPassengerName`.
- Cover normal, boundary, and failure paths for changed logic.

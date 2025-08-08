---
description: "Playwright BDD test generation instructions"
applyTo: "**"
---

# BDD Test Writing Guidelines

## Code Quality Standards

- **Locators**: Prioritize user-facing, role-based locators (`getByRole`, `getByLabel`, `getByText`, etc.) for resilience and accessibility within step definitions.
- **Assertions**: Use auto-retrying web-first assertions with `await` keyword (e.g., `await expect(locator).toHaveText()`). Avoid `expect(locator).toBeVisible()` unless specifically testing visibility changes.
- **Timeouts**: Rely on Playwright's built-in auto-waiting mechanisms. Avoid hard-coded waits or increased default timeouts in step implementations.
- **Clarity**: Write step definitions with descriptive names that match Gherkin language. Use comments to explain complex logic or business rules.

## BDD Structure

### Feature Files (.feature)

- **Location**: Store all feature files in `e2e/features/` directory
- **Naming**: Use descriptive names like `user-login.feature`, `product-search.feature`
- **Language**: Write in natural language following Gherkin syntax
- **Organization**: One feature per business capability or user journey

### Step Definitions (.ts)

- **Imports**: Start with `import { expect } from '@playwright/test';` and `import { Given, When, Then } from './fixtures';`
- **Categories**: Organize steps by type:
  - **Given** (Navigation/Setup): `Given('estoy en la página principal')`
  - **When** (Actions): `When('busco el producto {string}')`
  - **Then** (Assertions): `Then('debería ver {int} resultados')`

## File Organization

### Feature Files Structure

```gherkin
Feature: [Business Capability]
  Como [tipo de usuario]
  Quiero [objetivo]
  Para [beneficio]

  Background:
    Given [contexto común]

  @smoke @regression
  Scenario: [Escenario específico]
    Given [precondición]
    When [acción]
    Then [resultado esperado]
```

### Step Definitions Structure

```typescript
import { expect } from "@playwright/test";
import { Given, When, Then } from "./fixtures";

// Navigation Steps (Given)
Given("estoy en la página {string}", async ({ page }, pageName) => {
  await page.goto(`/${pageName}`, { waitUntil: "domcontentloaded" });
});

// Action Steps (When)
When("hago clic en {string}", async ({ page }, buttonText) => {
  await page.getByRole("button", { name: buttonText }).click();
});

// Assertion Steps (Then)
Then("debería ver el texto {string}", async ({ page }, expectedText) => {
  await expect(page.getByText(expectedText)).toBeVisible();
});
```

## BDD Best Practices

### Gherkin Writing

- **User-Centric**: Write from user perspective using natural language
- **Declarative**: Focus on WHAT the user wants to achieve, not HOW
- **Reusable**: Create generic steps that can be reused across features
- **Readable**: Use clear, unambiguous language that business stakeholders understand

### Step Definition Guidelines

- **Atomic**: Each step should perform one clear action or verification
- **Parameterized**: Use parameters `{string}`, `{int}` for reusability
- **Maintainable**: Keep step implementations focused and avoid complex logic
- **Consistent**: Follow naming patterns across all step definitions

## Assertion Best Practices

### UI Structure Validation

```typescript
Then("la estructura de la página debería ser correcta", async ({ page }) => {
  await expect(page.getByRole("main")).toMatchAriaSnapshot(`
    - main:
      - heading "Título Principal" [level=1]
      - section:
        - list "navegación":
          - listitem:
            - link "Inicio"
  `);
});
```

### Common Assertion Patterns

- **Element Counts**: `await expect(page.getByRole('listitem')).toHaveCount(expectedCount);`
- **Text Content**: `await expect(page.getByRole('heading')).toHaveText(expectedText);`
- **Navigation**: `await expect(page).toHaveURL(/.*\/success/);`
- **Form States**: `await expect(page.getByRole('button')).toBeDisabled();`

## Example BDD Implementation

### Feature File (e2e/features/page-title.feature)

```gherkin
Feature: Verificar título de página
  Como usuario
  Quiero verificar que la página carga correctamente
  Para confirmar que el sitio funciona

  @smoke
  Scenario: Verificar título de la página principal
    Given estoy en la página principal
    When verifico el título de la página
    Then debería ver un título válido
```

### Step Definitions (e2e/steps/steps.ts)

```typescript
import { expect } from "@playwright/test";
import { Given, When, Then } from "./fixtures";

// Navigation Steps
Given("estoy en la página principal", async ({ page }) => {
  await page.goto("/", { waitUntil: "domcontentloaded" });
});

// Action Steps
When("verifico el título de la página", async ({ page }) => {
  await page.waitForFunction(() => document.title.length > 0, {
    timeout: 10000,
  });
});

// Assertion Steps
Then("debería ver un título válido", async ({ page }) => {
  const title = await page.title();
  expect(title).toBeTruthy();
  expect(title.length).toBeGreaterThan(0);
});
```

## Test Execution Strategy

1. **Feature Development**: Write features first in collaboration with stakeholders
2. **Step Implementation**: Implement step definitions following the patterns above
3. **Execution**: Run tests with `npm run test` or `npm run test:smoke`
4. **Debug**: Use `npm run test:headed` for visual debugging
5. **Reports**: Generate cucumber reports with `npm run test:report`

## Quality Checklist

Before finalizing BDD tests, ensure:

- [ ] Features are written in natural language and readable by business stakeholders
- [ ] Steps are atomic, reusable, and well-parameterized
- [ ] All locators use accessible, user-facing selectors
- [ ] Assertions are meaningful and reflect business expectations
- [ ] Tags (@smoke, @regression) are used appropriately for test categorization
- [ ] Background steps eliminate redundancy across scenarios
- [ ] Step definitions follow consistent naming patterns
- [ ] Complex business logic is abstracted into helper functions

## Common Patterns

### Data Tables

```gherkin
Scenario: Verificar múltiples usuarios
  Given los siguientes usuarios existen:
    | nombre | email           | rol     |
    | Juan   | juan@test.com   | admin   |
    | María  | maria@test.com  | usuario |
```

### Scenario Outlines

```gherkin
Scenario Outline: Login con diferentes credenciales
  When inicio sesión con "<usuario>" y "<password>"
  Then <resultado>

  Examples:
    | usuario | password | resultado                    |
    | admin   | admin123 | debería estar en el dashboard |
    | user    | wrong    | debería ver error de login    |
```

### Hooks and Setup

```typescript
// En fixtures.ts o hooks específicos
BeforeAll(async () => {
  // Setup global del entorno de pruebas
});

Before({ tags: "@database" }, async () => {
  // Setup específico para tests que requieren base de datos
});
```

<script src="https://gist.github.com/ShayneDATS/6d8cec51fa001309f0277bf728ad5b9d.js"></script>

### Estimation

## Estimated Effort: ~46–58 hours
- Model + Repository + Migration setup (Orders, Products, Templates, Workflows, Users) → 7–9h
- Service layer for order logic (status, workflows, transitions) → 7–9h
- Policies/Gates for permissions → 5–7h
- FormRequests for validations → 4–5h
- Blade templates + Components (UI helpers) → 7–9h
- Routes + Middleware setup → 3–4h
- Unit + Feature tests → 9–11h
- Buffer & docs → 3–4h
- Any surprises → 8–10h

## Analysis

**1. Separation of Concerns (Target Architecture)**

- Controller → Handles request/response, validations, delegates to Service, returns Blade view or JSON.
- Service Layer → Encapsulates business logic for orders (status transitions, workflow handling, actions: approve/cancel/ship).
- Repository Layer → Abstracts all queries using Eloquent models and relationships.

**2. Laravel Features to Apply**

- Enums → Replace CASE statements for order statuses (Pending, Late, Overdue, Approved, Cancelled, Shipped).
- FormRequests → Replace legacy check_incomplete.php with Laravel validation classes.
- Policies & Gates → Centralize user permissions (approve, ship, cancel, edit).
- Migrations → To ensure the database schema consistency on all environments.
- Eloquent Models → Represent Orders, Products, Templates, Workflows, and Users with relationships. Relationships will replace JOINS. Indexing for Performance optimization.
- Middleware & Routes → Restrict access to authenticated/authorized users and organize endpoints via resource routes.
- Blade Components → Replace UI helpers (create_button, order_status_badge, show_user_avatar).

**3. Dependencies to Investigate**

- $APP_CONFIG → Move to Laravel config files - config/app.php or module-specific config file.
- Utility/Helper functions (show_user_avatar, order_status_badge, create_button) → Determine which should be converted into Blade components versus <a href="https://github.com/nidhiG2610/Migration-Analysis/blob/master/create-laravel-helper.md"> helper class methods </a>.
- Validation include (check_incomplete.php) → Reimplement as FormRequest logic.  Can be imlementd on Controller method, hence no invalid or senetized data beyond controllers.
- Legacy SQL table structures → Ensure Eloquent models and relationships correctly replicate existing joins and constraints.
- Session variables ($_SESSION['user']) → Map to Laravel Auth user context.

**4. Testing Approach**

- Unit Tests → Validate transitions and computed statuses logic (pending → late, approved → shipped, etc.), Ensure template fields are correctly mapped and interpreted, Workflow Data Parsing.
- Policy Tests → Confirm permissions enforce correctly with Edge cases like
    - User with edit_overdue_orders permission → can act on overdue orders.
    - User assigned but not manager → cannot approve.
- Feature Tests → Cover approval, shipping, cancellation flows. Test real user flows and interaction with HTTP endpoints or Blade views.
- Edge Cases → Handle overdue deadlines, shipped + cancelled orders, missing required dates.
- Test Data Strategy → Laravel factories
- Continuous Testing & CI/CD  → Integrate tests in GitHub Actions / CI pipeline to ensure meet with coding standard and all implemented tests run correctly.

**5. Key Risk if Migrated Incorrectly**

The riskiest part of migrating this code lies in the status calculation and permission logic, which is currently scattered across SQL CASE statements, inline PHP, and session checks. Errors in these areas could have serious operational and business consequences:

- Incorrect Order Status → An order may be misclassified (e.g., shipped marked as cancelled), leading to wrong user actions.
- Broken Permissions → Users may approve, ship, or cancel orders without proper authorization.
- Workflow and Validation Failures → Orders may move forward without completing required steps, breaking audit trails.
- UI/UX Discrepancies → Buttons, badges, or action links may display incorrectly if helpers/components are not implemented consistently.

Mitigation Needs:
- Provide a truth table of valid order states and transitions.
- Supply sample edge-case orders for testing.
- Review legacy helper functions and validation logic.
- Implement rigorous unit, policy, and feature tests covering all possible state and permission scenarios.
- maintain feature gates to handle, revert rollout. Logging system to handle feature usage and user tracking. To maintain code quality and consistency use code reviews, Enums, constructor, helper, PHP-cs fixer with custom rules.

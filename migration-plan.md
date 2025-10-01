### Business Problem Solved
- This legacy file drives the order review / detail page: collects order + product + template + workflow data, computes the current status (Pending, Late, Overdue, Approved, Shipped, Cancelled), and renders action buttons (Approve, Ship, Cancel) depending on the user’s permissions.
- It ties UI presentation and business rules together.
- It ensures that only users with the right roles can perform transitions (approve, cancel, ship), directly in the view layer.

### Three Specific Issues
- Monolithic / tightly coupled code — SQL, business logic, permission checks, and HTML are all in one file
- Hard to test and fragile — status logic embedded in SQL CASEs and inline if blocks; changing business rules risks breaking display or actions
- Migration risk when doing big rewrite — If you try to rewrite this all at once, you risk regressions; the legacy app is doing real work in production, so you need a gradual, safe migration path.

### Migration Approach
I’ll follow a hybrid, incremental migration rather than a total rewrite. There are few ways to hanlde this like,
 - Apache Rewrite Rules : Use Apache to route requests to either the Laravel or legacy application based on a query parameter.
 - Laravel Middleware : Use Laravel middleware to toggle between the two applications.
 - Environment Variable : Use an environment variable to toggle between the applications.
 - Laravel Routes : Use Laravel routes to conditionally serve the legacy application.

# Step A: Bootstrapping Laravel alongside legacy
- Becasue this is already live, it should be served from the same virtual host. Install a fresh Laravel app within the existing system (or parallel) and use a Query Parameter in Laravel Routes. 
- For entire system, use a “legacy fallback route” or catch-all that passes unknown routes into the original system (so the legacy app continues functioning while we incrementally migrate) 
- Introduce helper/wrapper classes: these sit between the legacy code and new Laravel code — you migrate one endpoint at a time (rather than all at once) 

# Step B: Migrating this specific page incrementally
- Create a new Laravel app  under same directory. To allow existing page to work, define a route with query parameter, and return the legacy app file.
- Create a new Laravel route for /orders/{id} pointing to an OrderController@show.
- In that controller, call a Service layer (e.g. OrderReviewService) to fetch data, compute status, etc.
- Behind the scenes, start with a Repository or helper that wraps the legacy queries or logic (so you don’t have to rewrite all SQL at once).
- Gradually replace the legacy logic in the helper, refactor to Eloquent, then decommission the helper.
- Throw a custom exception or use a LegacyView pattern when transitioning from legacy to Blade views. 
- Continue migrating other actions (approve, ship, cancel) one by one, with fallbacks to legacy until fully replaced.

# Step C: Structure in Laravel (once app structer is ready) -  Repository Pattern
- Controller → handles request, calls Service
- Service → contains business logic (status, transitions, validations)
- Repository / Eloquent → handles data access
- Enums → represent statuses (Pending, Late, etc.) instead of SQL CASE
- FormRequests → for validate approve/ship/cancel requests
- Policies / Gates → centralize permission checks
- Blade components / helpers → for UI parts like status badge, buttons, avatars

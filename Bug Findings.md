

Reviewed revision: `d2338e516f608f5defcff1d8f45fdea514966661`.

## Summary

**44 findings: 16 high priority, 26 normal priority, 2 low priority.** This report covers correctness, data integrity, account workflows, configuration, and deployment. It does not claim that the project has no additional bugs.

- **P1:** Fix before the next release. These defects cause data loss, broken primary workflows, incorrect account state, or deployment failures.
- **P2:** Fix in the normal maintenance cycle. These defects affect specific operations or configurations.
- **P3:** Lower-impact development or interface defects.
- **Reproduced:** A local characterization check confirmed the current, incorrect behavior.
- **Source-confirmed:** The finding follows from the implementation and its callers. The report states any verification limit.

### Verification and scope

- Inventoried 706 tracked files under `app`, `config`, `database`, `routes`, `resources`, `tests`, `bootstrap`, and `docs`; also reviewed dependency manifests, build configuration, deployment configuration, and first-party JavaScript.
- Checked PHP syntax in 499 files. The sole failure was the intentional `tests/Unit/PHPStan/Rules/Data/late-declare.php` fixture.
- PHPStan completed with zero errors under the existing configuration. Its baseline suppresses existing errors; this result does not clear the findings below.
- Existing suite: **234 tests, 447 assertions, passed** with `APP_URL=http://localhost`. An initial run with a different host failed the fixture-dependent credit-page test; that environment mismatch is not a finding.
- Additional temporary characterization suite: **33 tests, 101 assertions, passed against the incorrect behavior**. These checks demonstrate bugs; they are not tests of fixes.
- Compiled Blade views and syntax-checked 145 generated PHP files. Checked syntax in the first-party JavaScript entry points and helpers.
- Ran fresh migrations, a populated-table upgrade probe, and rollback checks against disposable MySQL 8.4 databases. Existing application databases were not used.
- Tests used installed dependencies. Fortify and Livewire versions differed from the lockfile (`1.37.2` vs `1.37.3`, and `4.3.3` vs `4.3.5`). No dependencies were changed.
- No browser end-to-end run, external identity-provider login, or real S3 upload was performed. The asset build was not run: `node_modules` was absent and the `theme` submodule was not initialized. Third-party/generated bundles and concurrent, untracked `docs/` additions were outside the review scope.
- Application source was not changed. The disposable database container was removed after verification. Temporary checks and logs remain under `/tmp/core-bug-scan/`, including `ReviewCharacterizationTest.php` and `characterization-current.txt`.

## High priority

### B001 Â· P1 Â· Changing an established user's email makes cleanup delete the account

**Locations:** `app/Services/User/UserService.php:253-277`, `app/Repositories/User/UserRepository.php:37-43`, `app/Console/Kernel.php:29`.

An email change sets `is_active=false` without changing `created_at`. The nightly new-user cleanup selects inactive accounts by their original creation time. An account older than 72 hours becomes eligible for deletion as soon as an administrator changes its email, even though the user has just received a verification message. Cleanup can also delete its group and associated data.

**Evidence:** Reproduced with a year-old active account: change its email, run cleanup, and the account no longer exists.

**Fix:** Distinguish pending initial registration from email reverification. Base any expiry on a dedicated activation timestamp, and exclude established accounts from new-user cleanup.

### ==B002 Â· P1 Â· The registration form cannot pass its request validation==

==**Locations:** `app/Http/Requests/Auth/RegisterRequest.php:77-80`, `resources/views/auth/register.blade.php:100-133`.==

==The request requires a `require_activation` field. The form never submits it, and the middleware does not add it. The controller already obtains the setting from the white-label model.==

==**Evidence:** Reproduced a valid form submission. Validation reports `require_activation`, and no account is created.==

==**Fix:** Read this setting from the white-label context; remove the required client field.==

### ==B003 Â· P1 Â· Editing another user's profile disables their two-factor authentication==

==**Locations:** `app/Http/Controllers/UserController.php:216-250`, `resources/views/users/edit.blade.php:129-133`.==

==The form renders `2fa_enabled` only for the current user's own profile. The update controller interprets an absent field as a request to disable two-factor authentication. An administrator who changes another user's name clears their secret, recovery codes, and confirmation timestamp.==

==**Evidence:** Reproduced a name-only update to a user with confirmed two-factor authentication; all three settings were cleared.==

==**Fix:** Change two-factor state only through an explicit, authorized operation. An omitted control must leave the existing state unchanged.==

### B004 Â· P1 Â· An SSO user's normal profile save disconnects SSO

**Locations:** `app/Http/Controllers/UserController.php:179-211`, `resources/views/users/edit.blade.php:37-45`.

The form restricts the SSO control to users with `support users`. The controller treats the missing control as false. A normal SSO user saving their own profile triggers a password-reset notification and clears their SSO association.

**Evidence:** Reproduced a self-profile name change by a user with profile permissions but no support permission; `sso` changed from true to false.

**Fix:** Preserve SSO state unless an authorized user submits an explicit disconnect action.

### ==B005 Â· P1 Â· Logout does not clear impersonation state==

==**Locations:** `app/Http/Controllers/Auth/AuthenticationController.php:108-114`, `app/Http/Middleware/UserLoginAs.php:15-19`.==

==Logout clears the guard but retains `login_as`, its queue, and `original_user`. On the next request, the impersonation middleware calls `onceUsingId()` with the retained target and authenticates the user again.==

==**Evidence:** Reproduced logout during impersonation, followed by a middleware pass. The guard changed from logged out back to the impersonated account.==

==**Fix:** Clear impersonation state and invalidate the session during logout. Regenerate the CSRF token.==

### ==B006 Â· P1 Â· The active login implementation ignores the blocked-user setting==

==**Locations:** `app/Providers/FortifyServiceProvider.php:81-98`, `routes/core/auth.php:15-16`.==

==The active Fortify callback validates email, password, and white-label ID, then returns the user. It does not check `is_blocked`. The blocked-user check remains in `AuthenticationController::login()`, which the login route no longer calls.==

==**Evidence:** Reproduced a successful POST login for a user with `is_blocked=true`.==

==**Fix:** Apply account-state checks in the active authentication pipeline and test the routed login flow, not only the blocking services.==

### B007 Â· P1 Â· SSO resolves accounts across white-label boundaries

**Locations:** `app/Http/Controllers/SingleSignOnController.php:31-36,71`, `database/migrations/2022_03_16_130824_add_white_label_and_group_foreign_keys_into_users_table.php:20`.

The database allows the same email in different white-labels. The SSO callback looks up the first user by email alone. A callback for one white-label can therefore select the account belonging to another.

**Evidence:** Reproduced with a mocked identity provider and two white-label accounts sharing an email. The callback for the second white-label authenticated the first account.

**Fix:** Resolve the account with the current white-label ID and the expected provider identity, consistent with the database uniqueness contract.

### B008 Â· P1 Â· Table deletion omits the scope used to list records

**Locations:** `app/Livewire/Tables/BaseTable.php:486-528`, `app/Livewire/Tables/UserTable.php:35-47`, `app/Livewire/Tables/GroupTable.php:35-47`, `app/Livewire/Tables/MenuTable.php:37-45`.

These table subclasses add group or white-label restrictions in `hookQuery()`. Both delete methods use `query()` instead of the scoped builder, so they omit those restrictions. The delete permission and self-delete checks do not restore the missing record scope.

**Evidence:** Reproduced with a user-table record outside the actor's group hierarchy. The listing excluded it, but `deleteRecord()` deleted it.

**Fix:** Define the mandatory record scope in `query()` or a shared scoped query that both listing and mutation use. Apply record-level checks to single and bulk deletion.

### B009 Â· P1 Â· Production credit pages depend on a development-only helper

**Locations:** `app/Services/Credits/CreditService.php:159,167,175`, `composer.json:39`, `.ebextensions/02-deploy.config:8-9`.

The chart code calls the global function `array_merge_recursive_distinct()`. Its sole definition is in `spatie/flare-client-php`, which the lockfile lists under `packages-dev`. Deployment installs with `--no-dev`. The default credit-page request calls this function and will fail without the development package.

**Evidence:** Runtime reflection located the function in `vendor/spatie/flare-client-php/src/helpers.php`; the dependency classification and production install command confirm the mismatch. A separate no-dev installation was not performed.

**Fix:** Implement the required chart aggregation in application code. Do not add a debug package to production to supply this helper.

### B010 Â· P1 Â· Group editing permits cycles in the group hierarchy

**Locations:** `app/Services/Group/GroupService.php:110-126`, `app/Http/Requests/Group/UpdateGroupRequest.php:33-39`, `resources/views/groups/edit.blade.php:17-30`.

The form excludes the current group but still offers its descendants as parents. The service checks that the selected parent exists, but does not reject descendants. A resulting cycle breaks recursive hierarchy queries and tree traversal.

**Evidence:** Reproduced a two-group parent/child cycle through the update service.

**Fix:** Reject the group itself and its descendants as parent candidates in both validation and persistence logic.

### B011 Â· P1 Â· A dismissed notification breaks notification-center rendering

**Locations:** `app/Repositories/NotificationDismissals/NotificationDismissalRepository.php:100-108`, `app/Livewire/Notifications/NotificationCenterComponent.php:206-218`, `resources/views/livewire/notifications/notification-center-component.blade.php:198`.

The joined query returns `Notification` models with an extra `dismissed_at` attribute. That attribute has no date cast on `Notification`, so it remains a string. The component passes it to `@formattedDate`, which calls a Carbon method on it.

**Evidence:** Reproduced the center-rendering failure after one dismissal: `Call to a member function toFormattedDate() on string`.

**Fix:** Cast or normalize the joined dismissal date before rendering, or return a typed notification/dismissal structure.

### B012 Â· P1 Â· Role-set dry runs modify and delete permissions

**Location:** `app/Console/Commands/SyncRoleSets.php:32-38,85-94`.

The command runs `permissions:sync --force` before checking `--dry-run` or asking for confirmation. A preview or a later declined confirmation can therefore alter production permissions and roles.

**Evidence:** Reproduced `role-sets:sync --dry-run` deleting a permission that was not in configuration.

**Fix:** Propagate dry-run behavior and obtain confirmation before either command writes to the database.

### B013 Â· P1 Â· The notification white-label migration fails on existing rows

**Location:** `database/migrations/2026_04_01_111952_add_whitelabel_id_to_notifications_table.php:14-18`.

The migration adds a required foreign key without assigning existing notifications to a white-label. MySQL gives existing rows a value of zero, then rejects the foreign key when no white-label with that ID exists. The successful column addition can remain after the failed constraint addition.

**Evidence:** Reproduced against a disposable database with one existing notification and white-label ID 1. Adding the constraint failed with SQLSTATE `23000`, error `1452`. Fresh-database migration tests do not cover this case.

**Fix:** Add a nullable column, backfill with an explicit ownership rule, then add the required constraint. Account for partially applied installations.

### B014 Â· P1 Â· Public S3 upload URLs retain a signature for the wrong host

**Location:** `app/Services/Aws/S3Service.php:123-138`.

The service signs a request for the internal endpoint and then replaces that endpoint with `AWS_URL`. AWS Signature V4 includes the host in the signed headers. A browser using the public hostname sends a request that does not match the original signature unless an intermediary restores the signed host.

**Evidence:** Reproduced with the real SDK and dummy credentials, without network access. Signing for the public host produced a different signature at the same signing time; the returned rewritten URL retained the internal signature.

**Fix:** Generate the browser URL with a client configured for the public endpoint. Verify uploads against the actual proxy/storage configuration.

### B015 Â· P1 Â· Inline translation editing saves data and then throws

**Locations:** `app/Livewire/Tables/TranslationEditTable.php:98-105`, `app/Console/Commands/UpdateLangFiles.php:21`.

The component calls `update:langfiles ` with a trailing space and a `domain` argument. The registered command is `update:langfiles` and accepts `whiteLabel`. The database update happens before this call, leaving the database changed but language files unchanged.

**Evidence:** Reproduced `CommandNotFoundException` after the translation text had saved. Removing only the trailing space would still leave the argument-name mismatch.

**Fix:** Use the registered command and argument names. Report regeneration failures and test the full save/regeneration sequence.

### B016 Â· P1 Â· Updating another user's password stores it without hashing

**Locations:** `app/Services/User/UserService.php:237-272`, `app/Http/Requests/User/UpdateUserRequest.php:72`, `app/Models/User.php:84-91`.

The service hashes a password only when the actor edits their own account. An accepted password field for another user reaches the repository unchanged. The model has no hashed password cast. The resulting value is incompatible with normal authentication.

**Evidence:** Reproduced through `UserService::update()` as an administrator; the database contained the supplied non-hashed value. The current other-user form does not display a password field, but the request/service accept one.

**Fix:** Hash at a shared persistence boundary, or reject other-user password updates and route them through a dedicated reset operation.

## Normal priority

### B017 Â· P2 Â· Configured login retry handling is disconnected

**Locations:** `app/Providers/FortifyServiceProvider.php:45-46,101-107`, `routes/core/auth.php:15-19`, `routes/testing.php:17-19`.

The replacement login and two-factor routes have only `web` middleware. The login limiter returns `Limit::none()`, and the custom pipeline omits Fortify's default limiter because a named limiter is configured. The application's blocked-login middleware appears on a test route, not the active login route.

**Evidence:** Source-confirmed and checked against the runtime route collection. The configured attempt thresholds do not govern ordinary password-login requests.

**Fix:** Attach the intended limiters and failure handling to the active routes/pipeline. Add routed retry tests.

### B018 Â· P2 Â· White-label settings require a nonexistent permission name

**Locations:** `routes/web.php:30`, `routes/core/white-label.php:6-9`, `config/permission-sync.php:42-45`.

The outer middleware requires `list white-labels`; the role configuration and inner middleware use `list whitelabels`. A user with `WhiteLabelsFullAccess` cannot open settings unless they qualify for the super-admin shortcut.

**Evidence:** Reproduced authorization failure for `WhiteLabelsFullAccess`.

**Fix:** Use one permission identifier throughout routes, menus, and role configuration.

### B019 Â· P2 Â· Notification readers cannot open their notification center

**Locations:** `routes/core/notification.php:8`, `config/roleset-sync.php:101-112`.

The center route requires `create notifications` in addition to list permission. The normal user role set grants `NotificationsListOnly`, so its users cannot open the center intended for reading and dismissing their messages.

**Evidence:** Reproduced authorization failure for `NotificationsListOnly`.

**Fix:** Require the reading permission for the center; keep creation permission on management operations.

### B020 Â· P2 Â· The global gate prevents group ownership policy checks

**Locations:** `app/Providers/AuthServiceProvider.php:34-75`, `app/Http/Controllers/GroupController.php:182`, `app/Policies/GroupOwnershipPolicy.php:29-61`.

The gate's `before` callback returns false for abilities that are not named role permissions. Group deletion authorizes the policy ability `delete`, so the gate rejects it before Laravel invokes the ownership policy. A group administrator with `destroy groups` cannot use this controller operation.

**Evidence:** Reproduced with `GroupsFullAccess` deleting a child group.

**Fix:** Return null when the global gate should defer to a policy. Keep permission and ownership checks as separate requirements.

### B021 Â· P2 Â· Administrators cannot move users between sibling groups

**Locations:** `app/Http/Requests/User/UpdateUserRequest.php:44-53,89`, `app/Http/Controllers/UserController.php:133`.

The edit form offers groups from the actor's hierarchy. Validation instead switches to the edited user's hierarchy. A sibling or parent group that the administrator can manage appears in the form but fails `group` validation.

**Evidence:** Reproduced an administrator moving a user between sibling groups; validation rejected the destination.

**Fix:** Validate destinations against the actor's permitted groups and validate the target user as a separate check.

### B022 Â· P2 Â· Profile password confirmation has no effect

**Locations:** `app/Http/Requests/User/UpdateUserRequest.php:71-72`, `resources/views/users/edit.blade.php:102-111`.

The form asks for password confirmation, but the update request validates the password only as nullable. It checks neither confirmation nor the application's password rules.

**Evidence:** Reproduced a successful self-profile save with different password and confirmation values. The first value became the new password.

**Fix:** Apply conditional string, length, confirmation, and password-rule validation when a new password is supplied.

### B023 Â· P2 Â· The credit table displays credits from other groups

**Locations:** `app/Livewire/Tables/CreditTable.php:10`, `app/Livewire/Tables/BaseTable.php:295-298`, `config/table-component.php:348-380`.

`CreditTable` inherits the unrestricted base query. The chart and balance use the current group, but the table does not, so users see an inconsistent ledger. Users with credit-delete permission can also act on records in that unscoped table.

**Evidence:** Reproduced a credit from another group appearing in the current group's table.

**Fix:** Restrict the credit query to the intended group and apply the same scope to mutations.

### B024 Â· P2 Â· Credit charts merge unrelated dates by array position

**Location:** `app/Services/Credits/CreditService.php:154-177`.

The credit and tick queries each return zero-indexed arrays. Recursive merging combines rows by index, not by day, month, or year. Different sets of transaction dates therefore move amounts between periods or discard them.

**Evidence:** Reproduced a 100-credit entry on June 14 and a 10-credit debit on June 15. The chart returned 90 then 0, instead of 100 then -10.

**Fix:** Key both aggregates by a complete period identifier, or calculate a signed transaction union in SQL.

### B025 Â· P2 Â· Monthly credit totals combine the same month from two years

**Locations:** `app/Repositories/Credits/CreditRepository.php:101-112`, `app/Repositories/Credits/TickRepository.php:80-91`, `app/Services/Credits/CreditService.php:201-215`.

The query includes part of the same month from the previous year and groups only by month number. The renderer shows 12 buckets, so last year's current-month transactions inflate this year's current-month total. The date-versus-timestamp lower bound also omits the first day of the boundary month.

**Evidence:** Reproduced June 2025 and June 2026 credits combined into one June row with the clock fixed to June 2026.

**Fix:** Define exact month boundaries and group by year and month together.

### B026 Â· P2 Â· The tick form and controller disagree on required fields

**Locations:** `resources/views/ticks/create.blade.php:4-10`, `app/Http/Requests/Credits/StoreTickRequest.php:24-29`, `app/Http/Controllers/TickController.php:34-37`.

The form submits `amount` and `model_description`. Validation requires `id`, `model_id`, `model_type`, and `price`; the controller then reads `model_price`, which validation does not require. The shipped form cannot create a tick, and the accepted field contract still does not supply the controller's price.

**Evidence:** Reproduced the form submission failing on the four absent required fields.

**Fix:** Define one purchase input contract and obtain the product and price from the server-side catalog.

### B027 Â· P2 Â· The dummy Buy button never purchases an item

**Locations:** `resources/views/components/table/buttons/dummy-buy.blade.php:1`, `routes/core/dummy.php:8`, `app/Http/Controllers/DummyController.php:16-18`.

The Buy link routes to the index method. That method ignores the selected item and renders the listing again, without creating a tick or changing the credit balance.

**Evidence:** Reproduced clicking the routed Buy operation with enough credits; no purchase was recorded.

**Fix:** Connect the button to a validated purchase action, or remove the nonfunctional control.

### B028 Â· P2 Â· Searching or sorting menus references a removed column

**Locations:** `config/table-component.php:258-262`, `database/migrations/2026_03_24_145611_add_parent_id_to_menus_table.php:24-25`.

The table still declares `group_name` as searchable and sortable after the hierarchy migration drops it. Any menu search includes the invalid column; selecting that sort also generates invalid SQL.

**Evidence:** Reproduced a menu search failing with an unknown `group_name` column.

**Fix:** Replace the old field with the parent relationship or remove it from the table configuration.

### B029 Â· P2 Â· Repeated column names produce duplicate Livewire keys

**Locations:** `resources/views/livewire/base-table.blade.php:136,173`, `config/table-component.php:9-42,190-205`.

Header and cell keys contain the column name but not its relationship. The user table has `name`, `group.name`, and `roleSet.name`; the group table has two `name` columns. Their sibling DOM elements receive identical keys, which breaks Livewire's element identity contract during updates.

**Evidence:** Source-confirmed key collisions. Browser behavior after sorting/filtering was not tested.

**Fix:** Include the relation-qualified column key, such as `getSortKey()`, in header, cell, and nested-component keys.

### B030 Â· P2 Â· Moving a menu subtree can exceed the supported depth

**Locations:** `app/Http/Controllers/MenuController.php:187-193`, `app/Services/Menu/MenuService.php:64-81`, `resources/views/components/metronic/menu.blade.php:45-65`.

The move check considers the destination parent's depth but not the height of the subtree being moved. Moving a three-level tree below another root creates four levels, while the menu template renders only three.

**Evidence:** Reproduced a successful move that created a four-level hierarchy.

**Fix:** Check destination depth plus subtree height before saving the move.

### B031 Â· P2 Â· Deleting a menu leaves position gaps that disable reordering

**Locations:** `app/Livewire/Tables/BaseTable.php:498,528`, `app/Livewire/MenuPositionManager.php:63-68,94-99`, `app/Repositories/Menu/MenuRepository.php:49-84,106-142`.

The table deletes menu records without renumbering remaining siblings. Reordering searches for the exact adjacent numeric position and stops if that number is missing.

**Evidence:** Reproduced deleting position 2 from positions 1, 2, 3. The remaining position-3 item could not move up.

**Fix:** Normalize sibling positions after deletion, or choose the adjacent record from the ordered collection instead of assuming consecutive numbers.

### B032 Â· P2 Â· Notification edit uses the wrong hidden checkbox field

**Location:** `resources/views/notifications/edit.blade.php:62-82`.

The hidden fallback beneath `show_on_login` is named `show_in_center_only`. It overrides a checked center-only value later in the form. An unchecked login checkbox submits no `show_on_login` field, so an existing true value remains true.

**Evidence:** Reproduced login visibility remaining enabled after an unchecked submission. Parsing the form's duplicate center-only fields produces a final value of zero.

**Fix:** Rename the second hidden field to `show_on_login`.

### B033 Â· P2 Â· Notification filtering resets a different paginator state

**Locations:** `app/Livewire/Notifications/NotificationCenterComponent.php:28-29,141-145,159-162,256-265`.

The component slices notifications with its custom `$page`. `resetPage()` resets Livewire's `$paginators['page']` instead. Filtering from a later page retains the old slice index and can show an empty page despite matching notifications.

**Evidence:** Reproduced filtering from page 3: the custom page stayed 3 while Livewire's page became 1.

**Fix:** Use one pagination state for page links, filtering, and collection slicing. Also adjust it after deletions shrink the result set.

### B034 Â· P2 Â· White-label updates leave request context stale

**Locations:** `app/Services/WhiteLabel/WhiteLabelContextInitializer.php:12-18`, `app/Services/WhiteLabel/WhiteLabelService.php:28-30`.

The initializer caches the entire white-label model by domain. Settings updates do not invalidate that cache. Requests continue using old registration settings, remember-me settings, colors, or domain metadata until expiration.

**Evidence:** Reproduced updating the label name while the initializer continued returning the previous name.

**Fix:** Invalidate both the old and new domain cache keys when a white-label changes.

### B035 Â· P2 Â· White-label language context does not update the translation loader

**Locations:** `app/Http/Kernel.php:38-39`, `app/Http/Middleware/LanguageSetter.php:18-32`, `app/Services/WhiteLabel/WhiteLabelContextInitializer.php:37-38`.

Language selection resolves Laravel's translator before the white-label middleware replaces `path.lang`. The singleton file loader retains its original paths; changing the container string does not update it. Generated white-label translations therefore do not replace the base translations for the request.

**Evidence:** Runtime inspection showed `path.lang` pointing at `lang/core.evalue8.local`, while the loader still used the framework language directory and `resources/lang`.

**Fix:** Initialize white-label context before translation resolution, or use a domain-aware loader with explicit per-request context handling.

### B036 Â· P2 Â· Carbon 3 makes automatic block durations grow instead of expire

**Locations:** `app/Services/User/BlockingService.php:58,87,131,166,218`.

Carbon 3 returns signed differences by default. `now()->diffInSeconds($past)` is negative, so subtracting it adds elapsed time to the configured duration. Persisted automatic blocks can remain effective after their intended expiry, including checks made while editing a user.

**Evidence:** Reproduced a 30-minute block reporting 9,000 seconds remaining two hours later.

**Fix:** Calculate elapsed time in the correct direction or request an absolute difference. Clamp the remaining duration at zero.

### B037 Â· P2 Â· IP intervals spanning subnets do not work

**Location:** `app/Services/Group/IpFilterService.php:23-34`.

The range implementation fixes the first three octets to the start address and loops only over the last octet. Validation accepts full start/end IPv4 ranges, but the service cannot evaluate intervals across octet boundaries.

**Evidence:** Reproduced rejecting `192.0.2.250` from `192.0.2.240-192.0.3.10`.

**Fix:** Compare validated numeric IPv4 addresses against numeric start/end bounds, and validate start <= end.

### B038 Â· P2 Â· Seven registered routes call nonexistent controller methods

**Locations:** `routes/core/translation.php:9-15`, `routes/core/menu.php:15`, `routes/core/credit.php:13-14`.

Runtime route/action inspection found these missing methods:

| Route | Missing method |
| --- | --- |
| GET `translations/create` | `TranslationController::create` |
| POST `translations` | `TranslationController::store` |
| PATCH `translations/{translation}` | `TranslationController::update` |
| POST `translations/bulk-update` | `TranslationController::bulkUpdate` |
| GET `menus/details` | `MenuController::getDetails` |
| GET `credits/{credit}/edit` | `CreditController::edit` |
| PATCH `credits/{credit}` | `CreditController::update` |

Requests that reach these actions fail instead of completing the operation. Some current tables hide the associated controls, but the routes remain registered.

**Fix:** Implement the supported actions or remove obsolete routes and their callers.

### B039 Â· P2 Â· Password-reset URLs corrupt plus-addressed email addresses

**Location:** `app/Services/User/Notifications/PasswordResetNotification.php:21-26`.

The notification inserts the email into the query string without URL encoding. Query parsing converts a plus sign into a space, so a reset link for a valid plus-addressed mailbox carries a different email and fails account lookup.

**Evidence:** Reproduced `review+tag@example.org` becoming `review tag@example.org` after parsing the generated reset URL.

**Fix:** Generate query parameters with a URL/query builder rather than string substitution.

### B040 Â· P2 Â· User forms offer only one page of groups

**Locations:** `app/Http/Controllers/UserController.php:70,133`, `app/Repositories/Group/GroupRepository.php:62`, `resources/views/users/edit.blade.php:79-87`.

The create/edit forms use the paginated group-list service to populate a plain selector. At the default page size, only 20 groups appear, with no selector pagination. A user's current group or a valid destination outside that page is unavailable.

**Evidence:** Source-confirmed caller/return-type mismatch. This differs from B021: even valid destinations under the chosen validation scope can be absent from the selector.

**Fix:** Use the existing group-collection service or a searchable, server-backed selector.

### B041 Â· P2 Â· Several migration rollbacks do not reverse their forward changes

**Locations and defects:**

- `database/migrations/2026_05_06_104554_change_white_label_id_to_notifications_table.php:29-33`: renames the column back without restoring the old foreign-key name. The April migration then tries to drop a nonexistent `notifications_whitelabel_id_foreign` constraint.
- `database/migrations/2026_04_01_111952_add_whitelabel_id_to_notifications_table.php:24-28`: drops the foreign key but leaves the added column, preventing a clean reapplication.
- `database/migrations/2025_01_14_000000_add_notifications_table.php:22-26`: adds `show_on_login` but tries to drop `show_in_footer`.
- `database/migrations/2024_11_27_120305_create_user_notification_deletions_table.php:28-30`: creates `deleted_notifications` but drops `user_notification_deletions`.
- `database/migrations/2022_09_09_130752_update_whitelabels_add_role_and_customer_group.php:30`: supplies two independently constrained columns to one `dropForeign()` call, which constructs a combined constraint name that was never created.

**Evidence:** A fresh migration followed by rollback stopped at the notification foreign-key name mismatch, with MySQL error `1091`. The remaining name/column mismatches are source-confirmed; the first failure prevented a complete rollback run.

**Fix:** Restore/drop the exact foreign keys, columns, and tables introduced by each migration. Test full rollback and reapplication on both empty and populated databases.

### B042 Â· P2 Â· Creating a new white-label through the command throws

**Location:** `app/Console/Commands/CreateWhiteLabelAdmin.php:56-64`.

The new-domain branch calls `$this->print()`, which neither the command nor Laravel's base command implements. The command stops before creating the new white-label.

**Evidence:** Source-confirmed; runtime reflection confirmed the missing method. PHPStan already suppresses this error in its baseline.

**Fix:** Use a supported output method such as `info()`, and test the new-domain branch with console input expectations.

## Low priority

### B043 Â· P3 Â· The Blade-formatting npm script invokes an uninstalled executable

**Location:** `package.json:12,14-23`.

`format-blade` invokes `blade-formatter`, but neither the manifest nor the lockfile installs a package providing that executable. A clean dependency installation cannot run the script without an unrelated global installation.

**Evidence:** Source-confirmed against both npm files. No npm installation was performed.

**Fix:** Use the configured Prettier Blade plugin or declare the formatter that the script requires.

### B044 Â· P3 Â· The reusable password score handler targets a nonexistent input

**Location:** `resources/views/components/core/input/password.blade.php:4-10,39-43`.

The rendered password input has a `name` but no `id="password"`. Its score handler binds to `#password`, so it attaches to no element on forms using this component. The hidden score stays at its initial value instead of reflecting the entered password.

**Evidence:** Source-confirmed selector/markup mismatch. The separate change-password template has its own matching input and handler; this finding concerns the reusable component.

**Fix:** Bind within the component's own wrapper to its actual password and score inputs. Ensure the scoring library is available where the component is used.

## Recommended follow-up

1. Add regression tests for B001-B016 before fixing them. Test expected behavior, rather than preserving the characterization assertions used during this scan.
2. Add routed tests for registration, profile updates, notification dismissal, and account-state handling. Add deployment tests without development dependencies and migration tests with existing rows.
3. Add browser tests for menu reordering and table updates. Recheck the source-confirmed UI findings in those tests.

Suggested report commit message: `docs: record project-wide bug scan findings`
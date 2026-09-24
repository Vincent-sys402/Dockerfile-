# Existing-demo personalization and approved setup

The Pages and Plugins screens expose **Make this site yours**. Chat requests classified as `content_personalization`, `demo_import`, `plugin_configuration`, or `woocommerce_setup` stop the replacement-site generation pipeline and open this workflow. Connecting or inspecting a site does not import anything.

## Client workflow

1. Inspect the connected site's documents, shared block templates/patterns, menus, images, theme and installed plugins.
2. Describe the client's business and request text suggestions, or edit individual fields and choose replacement images from the existing WordPress media library. Upload new client images through the existing asset upload. Existing/demo text is unconfirmed material; the suggestion prompt only treats explicitly confirmed client facts as evidence. Review suggestions before approval.
3. Alternatively choose a supported demo. Catalog adapters handle **OCDI (`ocdi/import_files`), legacy OCDI (`pt-ocdi/import_files`) and Merlin WP (`merlin_import_files`)**, using WordPress Importer (tested with 0.9.6). Both theme-local WXR and public HTTPS WXR exports are supported. Remote bytes are pinned in discovery and fetched again before execution; changed content requires another approval. Redirecting/private URLs, unsupported data files and unknown hooks are rejected. Review the named demo and importer dependency before approving. After import, rescan and create a separate personalization plan against the actual imported content.
4. Include reviewed WordPress.org plugin candidates and existing plugin configuration plans in the setup approval. Installed plugins can be researched without having been installed by this platform. Plugin settings can be suggested from requirements; secrets require existing secure references. Newly installed plugins are researched after installation, so newly discovered configuration requires another approval.
5. Review the exact plan and approve it. Applying is a separate action. Both the platform site and WordPress environment must identify a non-production target. API `staging`/`disposable` corresponds to WordPress `staging`/`development`/`local`.
6. Review the staging preview, desktop/mobile layout and affected plugin behavior. Setup approval **does not authorize production publication**. An account administrator can choose the matching production site and prepare a separate publication plan. Confirm the review, content/asset rights and licenses, then type the exact production address and select **Approve and publish to production**. That separate approval starts publication automatically, verifies resources and checks the public homepage. A failed homepage check triggers recovery.

## Interfaces and persistence

All new API routes require a real account session and site ownership. Reads allow `view_site`; actions require `edit_site`.

| Route relative to `/sites/:siteId/setup` | Method | Purpose |
| --- | --- | --- |
| `/` | GET | Saved inspection and plans |
| `/inspect` | POST | Read WordPress inventory and refresh platform records |
| `/suggest` | POST | Validated plain-text suggestions from `brief` |
| `/suggest-plugin-settings` | POST | Propose researched settings from `researchId` and `brief` |
| `/plans` | POST | Compile `baselineHash`, `intent`, `demoId`, `edits`, `pluginCandidateIds`, `pluginPlanIds`, and `brief` |
| `/plans/:id` | GET | Current plan and execution evidence |
| `/plans/:id/approve` | POST | Approve the exact `planHash`, target and revision |
| `/plans/:id/reject` | POST | Decline/cancel an unexecuted plan |
| `/plans/:id/execute` | POST | Apply approved operations on staging |
| `/plans/:id/reconcile` | POST | Read the durable WordPress execution record |
| `/plans/:id/rollback` | POST | Restore and verify the saved staging baseline |

`siteSetupDiscoveries` and `siteSetupPlans` use the existing file/PostgreSQL collection mechanism. Applied field edits also create a `pageEditChangesets` reference; its old Gutenberg apply/restore routes reject setup-managed records. Plugin configuration subplans retain their existing configuration service and snapshots. Every plan transition records an audit event.

The API signs `/setup/inspect`, `/setup/execute`, `/setup/status/:planId` and `/setup/rollback` control calls. WordPress requires `manage_options`. It saves a local execution journal and bounded database snapshot before mutation and holds a MySQL advisory lock. A repeated plan ID reads the recorded outcome; it never repeats an import. An unresolved running journal blocks different setup plans as well. Lost API responses can be reconciled without re-execution. Plan changes or external site/database changes require a new review.

Text edits patch original HTML offsets or known Elementor settings, retaining surrounding markup, layout and responsive controls. Images update attachment references and alt text, including Gutenberg image attributes. Shared theme patterns are expanded into a database template override only when that template is edited; the theme files remain intact. Unrecognized builders and controls are preserved.

## Current capability limits

- Customizer/widget/Redux/WPForms payloads, other providers and arbitrary theme hooks still need dedicated adapters. Merlin's two default pre/post-content methods are allowed only when their normalized source hashes match the reviewed upstream methods; their actual Home/Blog settings and sample-post effect appear in approval evidence. Registered required theme plugins are inspected; missing/inactive dependencies block the demo until separately installed/configured. There is no claim that every theme or wizard works.
- The snapshot adapter is limited to non-multisite databases, at most 20,000 rows per table and 30 MiB total. Larger installations need a hosting backup adapter. Plugin schema changes or other unhandled restoration effects can produce `rollback_failed`; only a verified restoration is reported as `rolled_back`.
- A PHP process lost during an importer can leave an uncertain journal. Automatic re-import is blocked; administrator reconciliation of that partial run is required. Failed runs with a known post-run baseline can use the rollback API.
- Images already in the media library require explicit permission confirmation, since demo images do not automatically become licensed client assets. Actual image creation or new asset upload uses existing platform flows.
- Live AI suggestion quality was not exercised. The API suggestion contract used a stubbed model. Real WordPress setup/publication tests use isolated databases, a real WordPress Importer and a fixture plugin; they do not constitute a customer publication or a signed plugin release.

## Verification

- `npm run test:site-setup`: intent routing, model proposal boundary, authenticated API, tenant/viewer isolation, exact approval, staging gates, stale baselines, lost-response reconciliation, no duplicate execution and verified rollback status.
- `npm run test:site-setup-ui`: actual setup component in Chromium, edit/review/approve/apply sequencing, runtime errors and mobile overflow. Set `CHROME_PATH` outside the default Windows Chrome installation.
- `scripts/site-setup-wordpress-smoke.php`, invoked with WP-CLI only against `DB_NAME=aiwp_setup_test`: real Gutenberg/Elementor field preservation, images and metadata, shared template overrides, fixture-plugin frontend configuration, external-edit rejection, WXR import, replay protection and verified rollback. It deliberately refuses other databases.
- Existing editing, intent, Elementor/cross-builder, plugin configuration and contract checks passed, as did dashboard compilation and JS lint. Desktop/mobile screenshots are retained under `.dev-logs/site-setup/`.
- Broader failures were not hidden: `dynamic-planning-regression.mjs` has the same unwanted booking-page failure in an extracted unchanged HEAD; the license scan blocks existing `autoskills@0.3.6` (`CC-BY-NC-4.0`); the secret scan flags three unchanged existing files. These are not new feature failures.

Importer interfaces were checked against the installed WordPress Importer 0.9.6 source and [WordPress Importer upstream](https://github.com/WordPress/wordpress-importer), with the catalog convention from [One Click Demo Import](https://github.com/awesomemotive/one-click-demo-import).

## Automatic publication after separate approval

The `wordpress-clone-presentation-v1` adapter transfers the reviewed result, rather than running the demo importer again on production. It accepts up to 30 contiguous applied setup journals, with a final unchanged staging baseline. Journals created before operation recording was added require a fresh staging preparation.

The bundle contains only supported presentation changes: page/post content, native builder metadata, shared templates, navigation, taxonomy relationships, branding, widgets already covered by setup, explicitly reviewed plugin option values and raster image files. It preserves GUIDs, rebases site URLs inside ordinary strings/JSON/serialized arrays, and rejects serialized objects and detected credentials. Secrets and licenses must be configured independently through secure handling.

Plugin options are exportable only when every stored field falls within the reviewed configuration paths. An option containing unreviewed sibling/nested fields is refused before its values leave WordPress. This conservative boundary prevents a selected setting from bringing along unrelated credentials; such plugins need independent secure production configuration or a dedicated field-transfer adapter.

Production must be a matching single-site clone with **the same theme, parent theme and active plugin versions and source-file hashes**. This adapter does not install production plugins or migrate plugin schemas. It reports missing dependencies before approval. Existing resource preimages and clone identities must match; occupied new IDs block promotion rather than overwriting an unrelated record. Client images uploaded after cloning can be copied when their IDs are free. Unknown changed options, tables, custom content and schema changes require another adapter.

The production journal stores the reviewed bundle and exact target preimages privately in WordPress. InnoDB transactions and resource locks protect content writes. Interrupted calls reconcile the durable result and actual pre/post-state; repeated IDs do not rerun imports or publication. Read-back includes changed content/settings, plugin/theme code and image hashes. The platform additionally checks the public homepage's HTTP status, HTML response type and final origin. This is an availability check, not a substitute for the explicitly required human desktop/mobile and plugin behavior review.

Recovery restores only the publication's touched resources, verifies them, and preserves unrelated live users, orders, posts and comments. Uploaded image files are retained because new live content may reference them. Conflicting subsequent edits block recovery. Failed HTTP health checks trigger this same recovery automatically; unverified recovery is never labeled successful.

Bounds: 2,000 resources, 20 MiB serialized bundle, raster images at most 5 MiB each, code manifests at most 10,000 files/100 MiB per theme/plugin directory, plus existing staging-snapshot limits. Larger sites, unrelated installations needing ID remapping, custom database data and production dependency installation require hosting/provider adapters.

### Publication interfaces

All routes are relative to `/sites/:stagingSiteId/setup`, authenticated and account-scoped. Reads require `view_site`; publication mutations require `manage_sites` (admin/owner).

| Route | Method | Purpose |
| --- | --- | --- |
| `/publications` | GET | Plans and same-account production targets; private bundles excluded |
| `/publications` | POST | Prepare from `setupPlanId` and `targetSiteId`, export staging and check production |
| `/publications/:id` | GET | Review/status evidence |
| `/publications/:id/approve-publish` | POST | Exact `planHash`, `confirmation` production URL, and `review.desktop/mobile/contentAndAssets/pluginBehavior=true`; starts publication automatically |
| `/publications/:id/reject` | POST | Decline before execution |
| `/publications/:id/reconcile` | POST | Reconcile actual journal outcome without repeated writes |
| `/publications/:id/rollback` | POST | Restore and verify the touched production resources |

`siteSetupPromotions` uses the existing persistent collection mechanism, alongside audit events, `productionApprovals`, `deployments` and `healthChecks` marked as site setup. WordPress control routes are `/setup/publication/export`, `/prepare`, `/apply`, `/rollback` and `/status/:promotionId`; all require `manage_options` and use the existing signed control transport. The generic staging-only `/setup/execute` gate remains intact.

### Additional verification

- `npm run test:setup-publication`: authenticated admin/tenant isolation, both baselines, exact publication approval, denied/incomplete approval, immediate approved execution, lost response, no duplicate publish, health failure/automatic recovery and false rollback rejection.
- `npm run test:setup-publication-ui`: real Chromium review form, mandatory confirmations, automatic approved publication, recovery and 390px fit. `SETUP_LIVE_PREVIEW=1` additionally compares real staging/production content pixels at 1440px and 390px.
- `npm run test:setup-publication-safety`: PHP export-path checks allow reviewed values and reject unreviewed sibling, nested, empty-container and scalar option data without touching a WordPress database.
- `scripts/setup-publication-wordpress-smoke.php`: refuses databases other than `aiwp_setup_test` and `aiwp_promotion_target`. Run `init`, clone the fixture database/theme/plugin files, run `source`, then `target` using WP-CLI. Source/target have distinct upload storage and staging/production environments. Requires a writable `/evidence` mount. Real Importer 0.9.6 verifies legacy OCDI/Merlin WXR imports, upstream image download, replay and rollback; remote WXR drift uses a controlled HTTP fixture. Publication covers a six-document/22-resource result, four image files, an image uploaded after cloning, shared footer, Elementor metadata, branding and fixture-plugin shortcode behavior. Injected SQL failure, interrupted journal reconciliation, targeted recovery and retained live posts/comments pass. Optional `preview` leaves an isolated publication for screenshots.
- Source conventions: [Merlin WP catalog documentation](https://github.com/richtabor/MerlinWP) and [Merlin's import-hook implementation](https://github.com/richtabor/MerlinWP/blob/master/class-merlin.php), plus the OCDI source above. The full Merlin wizard and other provider wizards were not exercised.

Source changes are uncommitted; affected app files are deployed to the local WSL development stack and PHP is loaded from the Windows bind mount. No client website was published and no new signed control-plugin release was produced.


## WooCommerce and public plugin integration

The shared setup workflow now includes WordPress.org search, installed-plugin configuration research, callback-owned page placement and WooCommerce settings/pages/simple products. See [Commerce setup](commerce-setup.md) for review steps, scoped browser verification, exact interfaces, tests and limits. WooCommerce product/option/custom-table production publication requires its own adapter; the presentation publisher does not silently transfer commerce data.

## Complete WooCommerce setup (2026-09-08)

The commerce adapter now covers catalog variations/downloads, taxonomy, delivery zones/core methods, taxes, coupons, checkout/inventory/email preferences and PayFast/Stripe/PayPal onboarding. Chat and dashboard share the business-brief proposal endpoint and exact reviewed plans. See [commerce-setup.md](commerce-setup.md) for boundaries, native adapters, bounded inspection, provider-specific manual steps and real fixture evidence. Commerce production publication remains separate from the presentation adapter.

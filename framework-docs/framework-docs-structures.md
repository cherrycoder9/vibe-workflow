```
└── 📁framework-docs
    └── 📁backend
        └── 📁hono
            └── 📁api
                └── 📁app
                    └── error-handling.md
                    └── fetch.md
                    └── fire.md
                    └── generics.md
                    └── mount.md
                    └── not-found.md
                    └── request.md
                    └── router-option.md
                    └── strict-mode.md
                └── 📁context
                    └── ContextVariableMap.md
                    └── env.md
                    └── error.md
                    └── event.md
                    └── executionCtx.md
                    └── header().md
                    └── html().md
                    └── json().md
                    └── notFound().md
                    └── redirect().md
                    └── render()setRender().md
                    └── req.md
                    └── res.md
                    └── set()get().md
                    └── status().md
                    └── text().md
                    └── var.md
                └── 📁exception
                    └── cause.md
                    └── Handling-HTTPException.md
                    └── throw-HTTPException.md
                └── 📁hono-request
                    └── arrayBuffer().md
                    └── blob().md
                    └── formData().md
                    └── header().md
                    └── json().md
                    └── matchedRoutes().md
                    └── method.md
                    └── param().md
                    └── parseBody().md
                    └── path.md
                    └── queries().md
                    └── query().md
                    └── raw.md
                    └── routePath().md
                    └── text().md
                    └── url.md
                    └── valid().md
                └── 📁presets
                    └── hono-quick.md
                    └── hono-tiny.md
                    └── hono.md
                └── 📁routing
                    └── base-path.md
                    └── chained-route.md
                    └── grouping-ordering.md
                    └── grouping-without-changing-base.md
                    └── grouping.md
                    └── including-slashes.md
                    └── optional-parameter.md
                    └── path-parameter.md
                    └── regexp.md
                    └── routing-priority.md
                    └── routing-with-host-header-value.md
                    └── routing-with-hostname.md
            └── changelog.md
            └── cheatsheet.md
            └── 📁helpers
                └── 📁accepts
                    └── accepts().md
                    └── import.md
                    └── options.md
                └── 📁adapter
                    └── env().md
                    └── getRuntimeKey().md
                    └── import.md
                └── 📁conninfo
                    └── import.md
                    └── type-definitions.md
                    └── usage.md
                └── 📁cookie
                    └── import.md
                    └── options.md
                    └── prefix.md
                    └── usage.md
                └── 📁css
                    └── css.md
                    └── cx.md
                    └── import.md
                    └── keyframes.md
                    └── usage.md
                └── 📁dev
                    └── getRouterName().md
                    └── import.md
                    └── showRoutes().md
                └── 📁factory
                    └── createFactory().md
                    └── createMiddleware().md
                    └── factory.createApp().md
                    └── factory.createHandlers().md
                    └── import.md
                └── 📁html
                    └── html.md
                    └── import.md
                    └── raw().md
                └── 📁jwt
                    └── custom-error-types.md
                    └── decode().md
                    └── import.md
                    └── payload-validation.md
                    └── sign().md
                    └── supported-algorithmTypes.md
                    └── verify().md
                └── 📁proxy
                    └── import.md
                    └── proxy().md
                    └── ProxyFetch.md
                └── 📁ssg
                    └── Generate File.md
                    └── Hook.md
                    └── Middleware.md
                    └── toSSG.md
                    └── usage.md
                    └── Using adapters for Deno and Bun.md
                └── 📁streaming
                    └── Error Handling.md
                    └── import.md
                    └── stream().md
                    └── streamSSE().md
                    └── streamText().md
                └── 📁testing
                    └── import.md
                    └── testClient().md
                └── 📁websocket
                    └── Examples.md
                    └── import.md
                    └── RPC-mode.md
                    └── upgradeWebSocket().md
            └── 📁middleware
                └── 📁Basic Auth
                    └── import.md
                    └── Options.md
                    └── Recipes.md
                    └── Usage.md
                └── 📁Bearer Auth
                    └── import.md
                    └── Options.md
                    └── Usage.md
                └── 📁Body Limit
                    └── import.md
                    └── Usage with Bun for large requests.md
                    └── Usage.md
                └── 📁Cache
                    └── import.md
                    └── Usage.md
                └── 📁Combine
                    └── import.md
                    └── Usage.md
                └── 📁Compress
                    └── compress.md
                └── 📁Context Storage
                    └── context-storage.md
                └── 📁CORS
                    └── cors.md
    └── 📁frontend
        └── 📁svelte
            └── changelog.md
            └── cheatsheet.md
            └── 📁cli
                └── 📁add-ons
                    └── drizzle.md
                    └── eslint.md
                    └── lucia.md
                    └── mdsvex.md
                    └── paraglide.md
                    └── playwright.md
                    └── prettier.md
                    └── storybook.md
                    └── sveltekit-adapter.md
                    └── tailwindcss.md
                    └── vitest.md
                └── 📁commands
                    └── sv add.md
                    └── sv check.md
                    └── sv create.md
                    └── sv migrate.md
            └── 📁svelte
                └── 📁runes
                    └── $bindable.md
                    └── $derived.md
                    └── $effect.md
                    └── $host.md
                    └── $inspect.md
                    └── $props.md
                    └── $state.md
                    └── what-are-runes.md
                └── 📁runtime
                    └── context.md
                    └── imperative-component-API.md
                    └── lifecycle-hooks.md
                    └── stores.md
                └── 📁special-elements
                    └── svelte-body.md
                    └── svelte-boundary.md
                    └── svelte-document.md
                    └── svelte-element.md
                    └── svelte-head.md
                    └── svelte-options.md
                    └── svelte-window.md
                └── 📁styling
                    └── custom-properties.md
                    └── global-styles.md
                    └── nested-style-elements.md
                    └── scoped-styles.md
                └── 📁template-syntax
                    └── {@attach ...}.md
                    └── {@const ...}.md
                    └── {@debug ...}.md
                    └── {@html ...}.md
                    └── {@render ...}.md
                    └── {#await ...}.md
                    └── {#each ...}.md
                    └── {#if ...}.md
                    └── {#key ...}.md
                    └── {#snippet ...}.md
                    └── animate.md
                    └── basic-markup.md
                    └── bind.md
                    └── class.md
                    └── in-out.md
                    └── style.md
                    └── transition.md
                    └── use.md
            └── 📁sveltekit
                └── 📁advanced
                    └── 📁advanced-routing
                        └── advanced-layouts.md
                        └── encoding.md
                        └── matching.md
                        └── optional-parameters.md
                        └── rest-parameters.md
                        └── sorting.md
                    └── 📁errors
                        └── error-objects.md
                        └── expected-errors.md
                        └── responses.md
                        └── type-safety.md
                    └── 📁hooks
                        └── handle.md
                        └── handleError.md
                        └── handleFetch.md
                        └── hooks.md
                        └── init.md
                        └── locals.md
                        └── reroute.md
                        └── transport.md
                    └── 📁link-options
                        └── data-sveltekit-keepfocus.md
                        └── data-sveltekit-noscroll.md
                        └── data-sveltekit-preload-code.md
                        └── data-sveltekit-preload-data.md
                        └── data-sveltekit-reload.md
                        └── data-sveltekit-replacestate.md
                        └── disabling-options.md
                        └── link-options.md
                    └── 📁packaging
                        └── anatomy-of-a-package-json.md
                        └── caveats.md
                        └── options.md
                        └── packaging.md
                        └── publishing.md
                        └── source-maps.md
                        └── typescript.md
                    └── 📁server-only modules
                        └── server-only modules.md
                    └── 📁service-workers
                        └── during-development.md
                        └── inside-the-service-worker.md
                        └── other-solutions.md
                        └── service-workers.md
                        └── type-safety.md
                    └── 📁shallow-routing
                        └── api.md
                        └── caveats.md
                        └── loading-data-for-a-route.md
                        └── shallow-routing.md
                    └── 📁snapshots
                        └── snapshots.md
                └── 📁best-practices
                    └── 📁accessibility
                        └── accessibility.md
                        └── focus-management.md
                        └── route-announcements.md
                        └── the-lang-attribute.md
                    └── auth.md
                    └── icons.md
                    └── 📁images
                        └── @sveltejs-enhanced-img.md
                        └── best-practices.md
                        └── loading-images-dynamically-from-a-CDN.md
                        └── vites-built-in-handling.md
                    └── 📁performance
                        └── diagnosing-issues.md
                        └── hosting.md
                        └── navigation.md
                        └── optimizing-assets.md
                        └── performance.md
                        └── reducing-code-size.md
                    └── 📁seo
                        └── manual-setup.md
                        └── out-of-the-box.md
                └── 📁core-concepts
                    └── 📁Form actions
                        └── Anatomy of an Action.md
                        └── API Endpoints with server_js.md
                        └── Custom Event Listener for Progressive Enhancement.md
                        └── Customizing use_enhance.md
                        └── GET vs POST Form Submission.md
                        └── Handling Validation Errors.md
                        └── Loading Data After Actions.md
                        └── Named Actions.md
                        └── Progressive Enhancement and use_enhance.md
                        └── Redirects and Errors in Actions.md
                        └── SvelteKit Form Actions.md
                    └── 📁page-options
                        └── csr.md
                        └── entries.md
                        └── page-options.md
                        └── prerender.md
                        └── Prerendering server routes.md
                        └── Route conflicts.md
                        └── ssr.md
                        └── SvelteKit Config.md
                        └── trailingSlash.md
                        └── Troubleshooting.md
                        └── When not to prerender.md
                    └── 📁routing
                        └── +error.md
                        └── +layout.js.md
                        └── +layout.server.js.md
                        └── +layout.svelte.md
                        └── +page.js.md
                        └── +page.server.js.md
                        └── +page.svelte.md
                        └── +server.md
                        └── $types.md
                        └── content-negotiation.md
                        └── fallback-method-handler.md
                        └── receiving-data.md
                        └── routing.md
                    └── 📁state-management
                        └── avoid-shared-state-on-the-server.md
                        └── component-and-page-state-is-preserved.md
                        └── no-side-effects-in-load.md
                        └── storing-ephemeral-state-in-snapshots.md
                        └── storing-state-in-the-URL.md
                        └── using-state-and-stores-with-context.md
```
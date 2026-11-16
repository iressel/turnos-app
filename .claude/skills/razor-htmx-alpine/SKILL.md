---
name: razor-htmx-alpine
description: Frontend standards and patterns for Turnos App - ASP.NET Core Razor Pages with htmx and Alpine.js on Bootstrap 5, mobile-first. Use when building or reviewing pages, partials, htmx fragments, forms and validation, Alpine.js components, layouts, tag helpers, CSS and design tokens.
when_to_use: Use when the task touches .cshtml files, page models, wwwroot assets, or mentions "pantalla", "página", "formulario", "wizard", "frontend", "htmx", "Alpine", "partial", "fragmento", "Bootstrap", "responsive", "mobile", or a card labelled Frontend or UX in TAREAS.md.
---

# Razor Pages + htmx + Alpine.js

The server renders HTML. htmx replaces parts of the page with HTML fragments from the server, and Alpine.js handles state that lives only in the browser. There is no jQuery, no SPA framework and no JSON API for the UI.

Check behaviour against the documentation of the versions pinned in the project (htmx.org, alpinejs.dev, getbootstrap.com) before relying on a detail from memory: htmx 2 and htmx 1 differ in several defaults.

## Who does what

| Need | Tool |
|---|---|
| Data, rules, validation, authorization | Server: page model calls a use case |
| Update part of a page after a request (filters, slots of a day, a table row, a form with errors) | htmx |
| State with no server meaning (open/closed, show a section when an option is chosen, a countdown) | Alpine.js |
| Layout, spacing, components | Bootstrap 5 plus the design tokens |

If a decision needs data or a rule, it goes to the server. Alpine.js never reimplements a business rule.

## Razor Pages

- A page model binds the request, checks authorization, calls one use case and returns the page or a partial. No business rule and no `DbContext` in it.
- Use named handlers for fragments: `OnGetSlotsAsync` answers `?handler=Slots` and returns `Partial("_Slots", model)`.
- A handler that htmx and a normal navigation can both reach checks the `HX-Request` header: return the partial for htmx and the full page otherwise. Every page must work when loaded directly from its URL.
- Shared pieces are partials (`_Name.cshtml`) or tag helpers. Use a tag helper for anything shown in many screens with one rule, such as a booking status badge, a price or a modality.
- Use the built-in tag helpers (`asp-for`, `asp-page`, `asp-page-handler`, `asp-validation-for`) so names, ids and URLs stay in sync with the model.
- In `.cshtml`, `@` starts Razor code. Write Alpine events as `x-on:click`, or escape the shorthand as `@@click`.

## htmx

- Put the behaviour on the element that triggers it: `hx-get` or `hx-post`, `hx-target`, `hx-swap`. Target a stable `id`.
- Prefer returning the smallest fragment that contains everything that changed. Update a second region (a counter, a toast) with an out-of-band element (`hx-swap-oob="true"`) in the same response.
- Use `hx-push-url="true"` when the result is a state the user should be able to bookmark or reach with the back button (search filters, wizard steps).
- Show progress with `hx-indicator`, and prevent double submission with `hx-disabled-elt="this"` on the submit button.
- Debounce inputs that query while typing: `hx-trigger="input changed delay:300ms"`.
- **Antiforgery.** A form rendered with the form tag helper already contains the token, and htmx sends the form's fields. For htmx requests that do not come from such a form, send the token in the `RequestVerificationToken` header, set once with `hx-headers` on `<body>`.
- **Validation errors.** Re-render the form partial with its errors and the values the user typed. By default htmx does not swap error responses, so either answer 200 with the form or configure the response handling for 422 once in the layout. Choose one approach for the whole application.
- **After a successful command**, redirect with the `HX-Redirect` response header or return the updated fragment. Raise a client event with `HX-Trigger` when another part of the page must refresh itself.
- **Expired session.** An htmx request that hits the login redirect must send the user to the login page, not swap the login form into a fragment. Handle it once, globally.

## Alpine.js

- Declare state next to the markup it controls with `x-data`, and keep each component small. Extract a component with `Alpine.data(...)` when the same behaviour appears in several places.
- Use `x-show` or `x-if` for conditional sections, `x-model` for local inputs, `x-on` for events, and `x-cloak` to avoid a flash of hidden content.
- Elements that htmx swaps in are initialized by Alpine automatically. Do not keep important state only in Alpine if a swap can replace the element: the server sends the state back in the fragment.
- Do not fetch data with Alpine. A request to the server is an htmx request that returns HTML.

## Layout and styles

- **Mobile-first.** Design for a 360 px screen and add breakpoints upwards. Primary actions are large, reachable with one thumb, and wizards show one step per screen on a phone.
- **Design tokens.** Colours, spacing, radii and typography come from the CSS custom properties defined for the project (card E0-08) and from Bootstrap utilities. No literal colour or pixel value in a page.
- Use the Bootstrap grid and components before writing CSS. Custom CSS goes in the project's stylesheets, not in `style` attributes.
- Every async action has a loading state, every list has an empty state, and every failure shows a message the user can act on.

## Accessibility and text

- Use semantic elements: `button` for actions, `a` for navigation, a `label` for every field.
- Keep focus visible and in a sensible place after a swap. Announce dynamic messages with `aria-live`.
- Error messages appear next to their field and are linked with `aria-describedby`.
- User-facing text is in Spanish (es-AR) and comes from resources. Dates, times and currency are formatted for the user's culture and time zone.

## Security

- Razor encodes output by default. Do not use `Html.Raw` with anything a user typed.
- Never put a secret, a token or another user's data in the HTML, in a `data-` attribute or in a URL.
- Hiding a button is not authorization: the handler checks the policy and the ownership of the resource.
- If the Content Security Policy forbids `unsafe-eval`, use Alpine's CSP build and keep expressions simple.

## Review checklist

- [ ] The page works on a phone viewport and when loaded directly from its URL.
- [ ] No business rule in the page model, the markup or Alpine.
- [ ] State-changing requests carry the antiforgery token and check authorization.
- [ ] Loading, empty and error states exist.
- [ ] Only design tokens and Bootstrap utilities for colours and spacing.
- [ ] Keyboard and screen-reader basics hold.
- [ ] Text is in Spanish from resources; identifiers and comments are in English.

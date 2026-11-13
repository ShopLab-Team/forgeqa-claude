---
name: forge-qa
description: Create, update, run and fix ForgeQA browser tests with the ForgeQA MCP tools (forge_read, forge_save, forge_run). Use when the user wants to test a flow on their website (checkout, add to cart, sign-up, login, search, contact form), write or repair an end-to-end, QA or regression test, build a test suite, or check ForgeQA test results.
---

# ForgeQA

ForgeQA runs tests in real browsers (Chromium, Firefox, WebKit, Chrome, Edge) against the user's own
websites. You work through three tools:

| Tool | Use it to |
|---|---|
| `forge_read` | See the websites you may use, existing tests and suites, and run results |
| `forge_save` | Create or update a test (`kind: "test"`) or a suite (`kind: "suite"`) |
| `forge_run` | Run a test or suite and get pass/fail plus the failing step and its error |

If these tools are missing, the ForgeQA connector is not connected: tell the user to add the MCP server
`https://app.forgeqa.io/mcp` in their assistant (it opens a browser to sign in and choose websites).

## Workflow

1. **Look first.** Call `forge_read` (no arguments). Note `websites` (the only origins you may test),
   `permissions`, and whether a matching test already exists. Update an existing test by `id` instead of
   creating a near-duplicate.
2. **Understand the journey.** If the user's request is vague ("test my checkout"), ask for the missing facts
   that change the steps: the start page, the product or form to use, and what proves success (a
   confirmation text, a URL, a cart count). Do not guess payment or login details.
3. **Write the steps.** One user journey per test, in order: `goto`, then the actions, then at least one
   assertion that proves the outcome. Give every step a short plain-language `description`.
4. **Save** with `forge_save`. It validates everything and explains problems; fix and save again.
5. **Run** with `forge_run` (Chromium unless the user asks for more browsers). Running uses the workspace's
   plan, so run once after saving rather than repeatedly without changes.
6. **Fix failures.** Read `runs[].problems[]`: the step number, its selector and the error. Typical fixes:
   a better selector, waiting for the element with `assert_visible` before clicking, or a longer
   `timeout_ms`. Save the corrected test (same `id`) and run again. Stop after three attempts and explain
   what the page is doing instead.
7. **Report** the result in plain words with the test's `url` so the user can open it in ForgeQA.

If a run is still going when `forge_run` returns, call `forge_read` with `view: "run"` and the `batch_id`.

## Steps

| action | needs | does |
|---|---|---|
| `goto` | `url` | Opens a path (`/products/shirt`) or full URL on an allowed website |
| `click` | `selector` | Clicks the element |
| `optional_click` | `selector` | Clicks it only if it appears (cookie banners, newsletter pop-ups) |
| `fill` | `selector`, `value` | Types into an input (empty `value` clears it) |
| `press` | `selector`, `value` | Presses a key in the element, such as `Enter` or `Tab` |
| `select` | `selector`, `value` | Chooses a `<select>` option by value or label |
| `check` / `uncheck` | `selector` | Ticks or unticks a checkbox |
| `assert_visible` | `selector` | Waits until the element is visible (also the right way to wait) |
| `assert_text` | `selector`, `value` | Element's text must contain `value` |
| `assert_url` | `value` | Page URL must contain `value`, e.g. `/checkout/thank-you` |
| `scroll` | `value` | JSON `{"x":0,"y":800}` |
| `wait` | `value` | Milliseconds, max 30000. Prefer `assert_visible` |
| `dismiss_popups` | none | Closes cookie banners and overlays now |
| `javascript` | `value` | Script body run in the page; throw an Error to fail the step |

Optional on any step: `description`, `timeout_ms` (default 15000, max 60000), `on_failure`
(`stop` default, `continue` to record the failure and go on, `ignore` to only warn), and up to three
`fallback_selectors`. No other fields are accepted.

## Selectors

Selectors are Playwright locators and must match **exactly one** element (two matches fail the step).
Prefer, in order:

1. Test ids: `[data-testid="add-to-cart"]`
2. Ids and form names: `#email`, `input[name="postcode"]`
3. Accessible roles and labels: `role=button[name="Add to cart"]`, `[aria-label="Search"]`
4. Visible text: `button:has-text("Checkout")`, `text=Order confirmed`
5. Narrow a repeated element: `.product-card >> nth=0`, `#cart >> text=Remove`

Avoid long CSS chains and generated class names such as `.css-1x2y3z`. When the user shares the page's HTML or
a screenshot, base selectors on it; otherwise use roles and visible text, and add one or two
`fallback_selectors`.

## Good habits

- Forge closes most cookie banners by itself; add `optional_click` for a pop-up that still gets in the way.
- After each navigation or important click, assert something the user would see before the next action.
- Never put real passwords, card numbers or personal data into steps. Use workspace secrets as
  `${TEST_SECRET_NAME}` (an owner or admin creates them in ForgeQA under Settings → Security → Test
  secrets) and Stripe or shop test cards.
- Keep tests short and focused; group journeys into a suite (`kind: "suite"`, `test_ids` in run order,
  `stop_on_failure: true` when later tests depend on earlier ones).
- If `forge_save` says the website is not allowed, ask the user to add it in ForgeQA → Websites and to
  allow it for this assistant (Integrations → AI assistants, then reconnect).
- A "may not" error means the connection lacks that permission; the user can reconnect and allow it.

## Example

"Test that a shirt can be added to the cart on shop.example.com":

```json
{
  "kind": "test",
  "name": "Add a shirt to the cart",
  "base_url": "https://shop.example.com",
  "steps": [
    {"action": "goto", "url": "/collections/shirts", "description": "Open the shirts page"},
    {"action": "click", "selector": ".product-card >> nth=0", "description": "Open the first shirt"},
    {"action": "select", "selector": "select[name=\"size\"]", "value": "M", "description": "Choose size M"},
    {"action": "click", "selector": "role=button[name=\"Add to cart\"]", "description": "Add it to the cart"},
    {"action": "assert_visible", "selector": "[data-testid=\"cart-drawer\"]", "description": "The cart opens"},
    {"action": "assert_text", "selector": "[data-testid=\"cart-count\"]", "value": "1", "description": "The cart shows one item"}
  ]
}
```

Then run it with `forge_run` (`{"kind": "test", "id": "<id from forge_save>"}`) and report the outcome.

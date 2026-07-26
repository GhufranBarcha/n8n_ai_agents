# Customer Survey Forms

Three production customer-survey flows for an e-commerce brand, built entirely with n8n's native Form nodes — no external form tool. There is an exit-intent survey (incentivized with a discount code), a returning-customer survey, and a branching new-customer survey whose page sequence adapts to how the shopper found the store, including required-field enforcement for a free-text "Other" answer. Every path ends by shaping a normalized analytics payload (`surveySlug`, `submittedAt`, `pathTaken`, `answers`), ready to wire into any datastore or webhook.

## Trigger

Three Form Triggers (public URLs by design — anyone with the link can submit):
- `customer-survey-forms-form-trigger-exit-intent`
- `customer-survey-forms-form-trigger-returning-customer`
- `customer-survey-forms-form-trigger-new-customer`

## How it works

- **Exit intent:** one required reason textarea + a required email field → payload Set → completion page offering a discount code (placeholder `YOURCODE10`).
- **Returning customer:** "why did you come back" (required) + optional feedback textarea → payload Set → thank-you completion.
- **New customer:** starts with a "How did you hear about us?" dropdown plus an optional write-in field for "Other".
- Two IF nodes validate the write-in: choosing "Other" with an empty text field routes to a completion page telling the visitor the field cannot be left blank; otherwise flow proceeds.
- A Switch routes attribution: Friend/Google skip the "What made you want to purchase?" page; Doctor/social channels get it first. Both branches then walk mirrored multi-page tracks: other solutions tried → when they first heard of the product → what almost stopped the purchase → a required Yes/No medication checkbox.
- Each branch ends in a Set node emitting `{ surveySlug, submittedAt, pathTaken, answers }` — identical shape across all three surveys — followed by a thank-you completion page. 24 nodes, no credentials required.

## Services & credentials

| Service | n8n credential type | Used for |
|---|---|---|
| — (none) | — | Runs entirely on built-in Form, Set, IF and Switch nodes |

## Placeholders to fill after import

| Placeholder | Where | Replace with |
|---|---|---|
| `customer-survey-forms-form-trigger-*` | The three Form Trigger `path` fields | Your own form paths (these become the public survey URLs) |
| `YOURCODE10` | `Form - Exit Intent Thank You` completion message | A real discount code from your store |
| Survey copy (titles, questions, brand mentions) | Form Trigger and Form page fields | Your own brand and question wording |

> Exported from a live instance with secrets replaced by placeholders. `active` is `false`; pinned test data removed. Re-select your own credentials after importing.

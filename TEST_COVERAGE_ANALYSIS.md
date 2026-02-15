# Test Coverage Analysis

## Current State

This project has **zero test coverage**. There are no test files, no testing framework, no CI/CD pipeline, and no automated validation of any kind. The project consists of three static HTML pages (`index.html`, `Contact.html`, `inputs.html`) and an image asset.

---

## Recommended Testing Areas

### 1. HTML Validation

**Priority: High**

None of the HTML files are validated for standards compliance. There are existing issues that validation tests would catch:

- `index.html:75` — Links to `contact.html` (lowercase), but the actual file is `Contact.html` (uppercase). This link is broken on case-sensitive file systems (Linux servers, most CI environments).
- `inputs.html:14` — Stray `0` character after the `<label>` for "Your Name:", which renders as visible text.
- Inconsistent use of uppercase HTML tags (`<H1>` in `index.html:13` vs `<h1>` elsewhere).

**Proposed tests:**
- Validate all `.html` files against the W3C HTML specification using a tool like `html-validate` or `vnu-jar`.
- Assert no parser errors or warnings.

### 2. Link Integrity

**Priority: High**

There are internal and external links that are not verified:

- **Internal link bug:** `index.html:75` links to `contact.html`, but the file is named `Contact.html`. This is a broken link on Linux.
- **External link:** `index.html:14` links to `https://www.multiaqua.nl` — this should be verified as reachable.
- **Mailto links:** `inputs.html:13` and `Contact.html:12` contain email addresses — format should be validated.

**Proposed tests:**
- Use a link checker (e.g., `htmlhint`, `linkinator`, or a custom script) to verify all internal links resolve to existing files.
- Validate external URLs return 2xx status codes.

### 3. Image Asset Integrity

**Priority: Medium**

- `index.html:12` references `BMN_AnneJan2.png`. There is no test confirming this file exists and is a valid image.
- The `alt` attribute is present (good), but its content is in Dutch and could be validated for non-emptiness at minimum.

**Proposed tests:**
- Assert all `<img>` `src` attributes reference files that exist in the project.
- Assert all `<img>` tags have non-empty `alt` attributes.

### 4. Contact Form Validation

**Priority: Medium**

`inputs.html` contains a contact form with:
- A text input for name
- An email input for email
- A textarea for message
- A submit button using `mailto:` action

There are no client-side or server-side validations.

**Proposed tests:**
- Verify form elements have correct `type` attributes (especially `type="email"` for email input).
- Verify all form inputs have `name` attributes.
- Verify `<label>` elements are properly associated with their inputs (currently they are not — no `for` attribute).
- Test that the `action` attribute contains a valid mailto URL.

### 5. Page Structure & SEO/Accessibility

**Priority: Medium**

None of the pages are tested for basic accessibility or structural requirements.

**Issues found:**
- No `<meta name="viewport">` tag on any page — the site won't render properly on mobile devices.
- No language consistency — `lang="en"` is declared but content is primarily in Dutch.
- No semantic HTML — uses `<table>` for layout instead of CSS.
- `<label>` elements in `inputs.html` are not associated with inputs via `for`/`id` attributes.

**Proposed tests:**
- Use an accessibility linter (e.g., `pa11y`, `axe-core`) to flag WCAG violations.
- Assert each page has exactly one `<h1>`.
- Assert `<html lang="...">` matches the actual content language.
- Assert all form labels are properly associated with inputs.

### 6. Cross-Page Consistency

**Priority: Low**

- `Contact.html` and `inputs.html` both serve as contact pages with different information and different email addresses (`ajholtland@hotmail.com` vs `annejan@multiaqua.nl`). This is potentially confusing.
- Page titles are inconsistent ("Contact" vs "Contact Me").

**Proposed tests:**
- Assert contact information is consistent across pages.
- Verify all pages follow a consistent title naming convention.

---

## Suggested Implementation Plan

### Step 1: Set up a testing framework

Initialize the project with `npm` and install lightweight HTML testing tools:

```bash
npm init -y
npm install --save-dev html-validate linkinator pa11y
```

### Step 2: Add HTML validation tests

Create a test script that runs `html-validate` against all `.html` files:

```json
{
  "scripts": {
    "test:html": "html-validate *.html",
    "test:links": "linkinator *.html --recurse",
    "test:a11y": "pa11y Contact.html && pa11y index.html && pa11y inputs.html",
    "test": "npm run test:html && npm run test:links && npm run test:a11y"
  }
}
```

### Step 3: Add CI/CD

Add a GitHub Actions workflow to run tests on every push and pull request.

### Step 4: Fix existing bugs

Before writing tests, fix the known issues identified above:
1. Rename `Contact.html` → `contact.html` (or fix the link in `index.html`)
2. Remove the stray `0` in `inputs.html:14`
3. Add `for`/`id` associations to form labels
4. Add viewport meta tags for mobile support

---

## Summary of Bugs Found

| File | Line | Issue | Severity |
|------|------|-------|----------|
| `index.html` | 75 | Broken link: `contact.html` vs `Contact.html` | **High** |
| `inputs.html` | 14 | Stray `0` character after label | **Medium** |
| `index.html` | 13 | Inconsistent tag casing (`<H1>` vs `<h1>`) | **Low** |
| `inputs.html` | 14-19 | Labels not associated with inputs (`for`/`id` missing) | **Medium** |
| All files | — | Missing viewport meta tag | **Medium** |
| `index.html` | 2 | `lang="en"` but content is Dutch | **Low** |
| Both contact pages | — | Inconsistent contact information | **Low** |

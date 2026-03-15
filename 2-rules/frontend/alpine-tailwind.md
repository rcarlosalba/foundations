---
description: Frontend ruleset for Django templates using Tailwind CSS, Alpine.js, and Vanilla JS. Enforces utility-first styling, strict separation of concerns, accessibility (a11y), and specific patterns for forms and UI components.
globs: "**/*.html, **/*.js, tailwind/base.css, constants/form_styles.py"
---

# Alpine.js & Tailwind CSS Frontend Rules

**Core Goal:** Build clean, modular, and accessible user interfaces. Maintain a strict boundary between structure (HTML), styling (Tailwind CSS), simple UI state (Alpine.js), and complex logic (Vanilla JS).

## 1. Core Principles & Boundaries
1. **Strict Separation of Logic:** Django templates MUST NOT contain business logic or complex conditional rendering. Data must be pre-processed in the Django view.
2. **No Inline Styles:** The `style="..."` attribute is STRICTLY FORBIDDEN in standard HTML. 
    - *Exception:* Email templates (table-based) and PDF generation templates may use inline styles where Tailwind is unsupported.
3. **No Native HTML5 Validation:** Do NOT use attributes like `required`, `minlength`, or `pattern`. ALWAYS add `novalidate` to `<form>` tags. Validation is handled server-side (Django) and client-side via custom JS/Alpine.
4. **Local Assets Only:** NEVER use CDNs for libraries. Assume FontAwesome, Alpine.js, and Tailwind are served locally from the `static/` directory.

## 2. Tailwind CSS Styling Rules
1. **Utility-First & No Magic Values:** Always use standard Tailwind utility classes. 
    - ❌ **WRONG:** Arbitrary values like `w-[23px]` or `top-[117px]`.
    - ✅ **CORRECT:** Theme values like `w-6`, `top-32`. If a specific value is missing, instruct the user to add it to `tailwind.config.js`.
2. **Restricted `@apply` Usage:** Only use `@apply` in `tailwind/base.css` for highly repetitive semantic components (e.g., `.btn-primary`, `.card`). Do not use it to recreate traditional CSS stylesheets.
3. **Mobile-First:** Always design for small screens first, using `sm:`, `md:`, `lg:` modifiers to scale up.

## 3. Django Form Styling Pattern (CRITICAL)
NEVER style Django form fields directly in the HTML template or use complex template tags for styling.

- **The Constants Pattern:** All Tailwind classes for form widgets MUST be defined in `constants/form_styles.py` and applied in the `forms.py` file.
    - ✅ **CORRECT (`forms.py`):**
      ```python
      from constants import form_styles
      username = forms.CharField(widget=forms.TextInput(attrs={'class': form_styles.TEXT_INPUT}))
      ```

## 4. Interactivity: Alpine.js vs Vanilla JS
1. **Alpine.js (UI State):** Use Alpine.js ONLY for declarative, self-contained UI state management (e.g., dropdowns, modals, tabs, toggling visibility).
    - ✅ **CORRECT:** `<div x-data="{ open: false }"> <button @click="open = !open">Toggle</button> </div>`
    - ❌ **WRONG:** Complex API fetching or heavy data transformations inside `x-data` or `x-init`.
2. **Vanilla JS (Complex Logic):** For complex logic, event delegation, or API calls, use modern Vanilla JS (ES6+) in dedicated files inside `static/js/`. 
3. **Global UI Components:**
    - **Toasts/Messages:** Must use the `templates/partials/messages.html` and `static/js/messages.js` pattern.
    - **Modals:** Must use the `templates/partials/modal.html` and `static/js/modal.js` pattern.

## 5. Modularity and Accessibility (a11y)
1. **Partials:** Extract reusable UI blocks (navbars, footers) into `templates/partials/` and use `{% include 'partials/component.html' %}`.
2. **Semantic HTML:** ALWAYS use semantic tags (`<main>`, `<nav>`, `<article>`) instead of generic `<div>` wrappers.
3. **ARIA Attributes:** Interactive Alpine.js components MUST include correct accessibility attributes (e.g., `aria-expanded`, `aria-hidden`, `role="dialog"`).

---
### ✅ LLM Pre-Generation Checklist
Before generating frontend code, verify:
1. Are there any inline `style="..."` attributes? (Remove them unless it's an email/PDF).
2. Are form fields styled using the `constants/form_styles.py` pattern?
3. Is HTML5 validation disabled (`novalidate` on forms)?
4. Is Alpine.js used strictly for UI state, and complex JS moved to `static/js/`?
5. Are arbitrary Tailwind values (`w-[15px]`) avoided?
6. Are semantic HTML tags and ARIA attributes present for accessibility?
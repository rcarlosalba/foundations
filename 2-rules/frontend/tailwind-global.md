---
description: Global ruleset for Tailwind CSS styling. Enforces the strict utility-first methodology, mobile-first responsive design, design system adherence (no magic values), and class organization, agnostic of the underlying frontend framework.
globs: "**/*.{html,jsx,tsx,vue,js,ts}"
---

# Global Tailwind CSS Rules

**Core Goal:** Build highly consistent, responsive, and maintainable user interfaces using pure Tailwind CSS utility classes. Custom CSS must be virtually non-existent.

## 1. The Utility-First Mandate
1. **Zero Custom CSS:** You MUST style elements exclusively using Tailwind's utility classes directly in the markup (HTML/JSX). Do not write custom CSS classes unless instructed to build a base design system layer.
2. **No Inline Styles:** The `style="..."` attribute is STRICTLY FORBIDDEN. Every visual aspect (including dynamic widths or background images) should be handled via Tailwind classes or framework-specific style bindings if absolutely necessary.
3. **Component Abstraction over `@apply`:** Do NOT use `@apply` to clean up long class lists in CSS files. Instead, abstract the repetition by creating reusable components in the underlying framework (e.g., a React `<Button>` or a Django `{% include 'button.html' %}`). 

## 2. Strict Design System Adherence
1. **No Arbitrary Values (No Magic Numbers):** You are STRICTLY FORBIDDEN from using arbitrary values with square brackets (e.g., ❌ `w-[23px]`, `text-[17px]`, `bg-[#ff2233]`).
2. **Use the Theme:** ALWAYS use the predefined design tokens from the Tailwind theme (e.g., ✅ `w-6`, `text-lg`, `bg-red-500`).
3. **Missing Values:** If a design requires a specific color or spacing that is not in the default scale, assume it will be added to `tailwind.config.js` via `theme.extend`. Do not hardcode it in the markup.

## 3. Responsive & State Design
1. **Mobile-First is Mandatory:** ALWAYS design for mobile screens first (unprefixed classes). Use breakpoints (`sm:`, `md:`, `lg:`, `xl:`) strictly to apply changes for larger screens.
    - ❌ **WRONG:** `w-1/2 sm:w-full` (Desktop first)
    - ✅ **CORRECT:** `w-full sm:w-1/2` (Mobile first)
2. **State Modifiers:** Use standard modifiers for interactive states (`hover:`, `focus:`, `active:`, `disabled:`). Ensure interactive elements always have clear focus states for accessibility (`focus:ring`, `focus:outline-none`).

## 4. Class Sorting and Readability
To maintain readable markup, group Tailwind classes logically. While strict ordering is usually handled by Prettier plugins, aim to generate classes in this general logical order:
1. Layout & Positioning (`block`, `flex`, `absolute`, `top-0`)
2. Box Model (`w-full`, `h-auto`, `p-4`, `m-2`)
3. Typography (`text-lg`, `font-bold`, `text-center`)
4. Visuals (`bg-blue-500`, `rounded-md`, `shadow-sm`)
5. Modifiers (`hover:bg-blue-600`, `md:w-1/2`)

---
### ✅ LLM Pre-Generation Checklist
Before finalizing the markup, mentally verify:
1. Did I use any arbitrary values like `[14px]` or `[#334455]`? (If yes, replace with theme utility classes).
2. Is the design mobile-first? (Are base classes for mobile, and `md:`/`lg:` for desktop?)
3. Did I completely avoid `style="..."` attributes?
4. Did I avoid writing custom CSS or abusing `@apply`?
5. Do interactive elements have `hover:` and `focus:` states?
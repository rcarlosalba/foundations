# Skill: Form Handling

**When to use:** Any time you build a form that requires validation, loading state, field-level error display, and success feedback.

---

## The 4 States Every Form Must Handle

| State | What the user sees |
|-------|--------------------|
| **Idle** | The empty form, ready to fill |
| **Pending** | Submit button disabled/spinner — request in flight |
| **Error** | Field-level messages (validation) + global message (server failure) |
| **Success** | Confirmation message or redirect |

Never build a form that handles fewer than these 4 states.

---

## Django + Alpine.js

Two valid approaches depending on the UX requirement:

- **Traditional (full-page)** — form submits, Django processes, redirects or re-renders. Simpler. Correct for most cases.
- **AJAX (inline feedback)** — submits via `fetch`, shows errors/success without a page reload. Use when a redirect would break the UX (e.g., a form inside a modal or sidebar).

### Approach 1 — Traditional (Full-Page)

```python
# apps/contact/forms.py
from django import forms
from constants import form_styles

class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={
            'class': form_styles.TEXT_INPUT,
            'placeholder': 'Your name',
        }),
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={'class': form_styles.TEXT_INPUT}),
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={'class': form_styles.TEXTAREA, 'rows': 4}),
    )
```

```python
# apps/contact/views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import ContactForm
from .services import send_contact_message

def contact_view(request):
    form = ContactForm(request.POST or None)
    if request.method == 'POST' and form.is_valid():
        send_contact_message(form.cleaned_data)
        messages.success(request, 'Message sent successfully.')
        return redirect('contact')
    return render(request, 'contact/contact.html', {'form': form})
```

```html
<!-- templates/contact/contact.html -->
{% include 'partials/messages.html' %}

<form method="POST" novalidate>
    {% csrf_token %}

    <!-- Name field -->
    <div class="mb-4">
        <label for="{{ form.name.id_for_label }}"
               class="block text-sm font-medium text-gray-700">
            Name
        </label>
        {{ form.name }}
        {% if form.name.errors %}
            <p class="mt-1 text-sm text-red-600">{{ form.name.errors.0 }}</p>
        {% endif %}
    </div>

    <!-- Email field -->
    <div class="mb-4">
        <label for="{{ form.email.id_for_label }}"
               class="block text-sm font-medium text-gray-700">
            Email
        </label>
        {{ form.email }}
        {% if form.email.errors %}
            <p class="mt-1 text-sm text-red-600">{{ form.email.errors.0 }}</p>
        {% endif %}
    </div>

    <!-- Message field -->
    <div class="mb-6">
        <label for="{{ form.message.id_for_label }}"
               class="block text-sm font-medium text-gray-700">
            Message
        </label>
        {{ form.message }}
        {% if form.message.errors %}
            <p class="mt-1 text-sm text-red-600">{{ form.message.errors.0 }}</p>
        {% endif %}
    </div>

    <button type="submit"
            class="w-full rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
        Send Message
    </button>
</form>
```

---

### Approach 2 — AJAX with Alpine.js state + Vanilla JS fetch

Use when the form lives inside a modal or the page must not reload.

> **Rule:** Alpine.js manages visual state only (`loading`, `errors` display). The `fetch` call lives in `static/js/`.

```python
# apps/contact/views.py  (API endpoint — add alongside the traditional view)
from django.http import JsonResponse
from django.views.decorators.http import require_POST
from .forms import ContactForm
from .services import send_contact_message

@require_POST
def contact_submit_api(request) -> JsonResponse:
    form = ContactForm(request.POST)
    if form.is_valid():
        send_contact_message(form.cleaned_data)
        return JsonResponse({'status': 'ok'})
    return JsonResponse({'status': 'error', 'errors': form.errors}, status=400)
```

```javascript
// static/js/contact-form.js
function contactForm(submitUrl) {
    return {
        loading: false,
        success: false,
        globalError: '',
        fieldErrors: {},

        async submit(event) {
            this.loading = true;
            this.globalError = '';
            this.fieldErrors = {};

            try {
                const res = await fetch(submitUrl, {
                    method: 'POST',
                    body: new FormData(event.target),
                    headers: { 'X-Requested-With': 'XMLHttpRequest' },
                });
                const data = await res.json();

                if (data.status === 'ok') {
                    this.success = true;
                    event.target.reset();
                } else {
                    this.fieldErrors = data.errors;
                    this.globalError = 'Please fix the errors below.';
                }
            } catch {
                this.globalError = 'An unexpected error occurred. Please try again.';
            } finally {
                this.loading = false;
            }
        },
    };
}
```

```html
<!-- templates/contact/contact_form.html -->
<div x-data="contactForm('{% url 'contact:submit' %}')" x-cloak>

    <!-- Success state -->
    <div x-show="success"
         role="alert"
         class="rounded-md bg-green-50 p-4 text-green-800">
        Your message has been sent successfully.
    </div>

    <!-- Form (hidden on success) -->
    <form x-show="!success"
          @submit.prevent="submit"
          novalidate>

        <!-- Global error -->
        <div x-show="globalError"
             role="alert"
             class="mb-4 rounded-md bg-red-50 p-4 text-sm text-red-800"
             x-text="globalError">
        </div>

        <!-- Name field -->
        <div class="mb-4">
            <label for="name" class="block text-sm font-medium text-gray-700">Name</label>
            <input type="text" name="name" id="name"
                   :class="fieldErrors.name ? 'border-red-500' : 'border-gray-300'"
                   class="mt-1 block w-full rounded-md border px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
            <p x-show="fieldErrors.name"
               x-text="fieldErrors.name?.[0]"
               class="mt-1 text-sm text-red-600">
            </p>
        </div>

        <!-- Submit button -->
        <button type="submit"
                :disabled="loading"
                class="w-full rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50">
            <span x-show="!loading">Send Message</span>
            <span x-show="loading" aria-live="polite">Sending…</span>
        </button>
    </form>
</div>

<script src="{% static 'js/contact-form.js' %}"></script>
```

---

## Next.js (App Router — Server Actions)

The canonical pattern uses `useActionState` for form state and a dedicated `SubmitButton` with `useFormStatus` for the pending state. The `SubmitButton` must be a separate component so it can call `useFormStatus` without making the entire form a Client Component.

### 1. Server Action

```typescript
// app/actions/contact.ts
'use server'
import { z } from 'zod'

const ContactSchema = z.object({
    name: z.string().min(1, 'Name is required').max(100, 'Name is too long'),
    email: z.string().email('Enter a valid email address'),
    message: z.string().min(10, 'Message must be at least 10 characters'),
})

export type ContactFormState = {
    success: boolean
    message: string
    errors: Record<string, string[]> | null
}

const initialState: ContactFormState = {
    success: false,
    message: '',
    errors: null,
}

export { initialState as contactFormInitialState }

export async function submitContact(
    _prev: ContactFormState,
    formData: FormData,
): Promise<ContactFormState> {
    const result = ContactSchema.safeParse({
        name: formData.get('name'),
        email: formData.get('email'),
        message: formData.get('message'),
    })

    if (!result.success) {
        return {
            success: false,
            message: 'Please fix the errors below.',
            errors: result.error.flatten().fieldErrors,
        }
    }

    try {
        await sendContactMessage(result.data)   // your service call
        return { success: true, message: 'Message sent successfully!', errors: null }
    } catch {
        return {
            success: false,
            message: 'An unexpected error occurred. Please try again.',
            errors: null,
        }
    }
}
```

### 2. Submit Button (isolated Client Component)

```typescript
// app/components/client/SubmitButton.tsx
'use client'
import { useFormStatus } from 'react-dom'

interface SubmitButtonProps {
    label: string
    pendingLabel: string
}

export function SubmitButton({ label, pendingLabel }: SubmitButtonProps) {
    const { pending } = useFormStatus()

    return (
        <button
            type="submit"
            disabled={pending}
            aria-disabled={pending}
            className="w-full rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50"
        >
            {pending ? pendingLabel : label}
        </button>
    )
}
```

### 3. Form Component (Client Component)

```typescript
// app/components/client/ContactForm.tsx
'use client'
import { useActionState } from 'react'
import { submitContact, contactFormInitialState } from '@/app/actions/contact'
import { SubmitButton } from './SubmitButton'

export function ContactForm() {
    const [state, action] = useActionState(submitContact, contactFormInitialState)

    if (state.success) {
        return (
            <div role="alert" className="rounded-md bg-green-50 p-4 text-green-800">
                {state.message}
            </div>
        )
    }

    return (
        <form action={action} noValidate>
            {/* Global error */}
            {state.message && !state.success && (
                <div role="alert" className="mb-4 rounded-md bg-red-50 p-4 text-sm text-red-800">
                    {state.message}
                </div>
            )}

            {/* Name field */}
            <div className="mb-4">
                <label htmlFor="name" className="block text-sm font-medium text-gray-700">
                    Name
                </label>
                <input
                    type="text"
                    name="name"
                    id="name"
                    className={`mt-1 block w-full rounded-md border px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
                        state.errors?.name ? 'border-red-500' : 'border-gray-300'
                    }`}
                    aria-describedby={state.errors?.name ? 'name-error' : undefined}
                />
                {state.errors?.name && (
                    <p id="name-error" className="mt-1 text-sm text-red-600">
                        {state.errors.name[0]}
                    </p>
                )}
            </div>

            {/* Email field */}
            <div className="mb-4">
                <label htmlFor="email" className="block text-sm font-medium text-gray-700">
                    Email
                </label>
                <input
                    type="email"
                    name="email"
                    id="email"
                    className={`mt-1 block w-full rounded-md border px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
                        state.errors?.email ? 'border-red-500' : 'border-gray-300'
                    }`}
                    aria-describedby={state.errors?.email ? 'email-error' : undefined}
                />
                {state.errors?.email && (
                    <p id="email-error" className="mt-1 text-sm text-red-600">
                        {state.errors.email[0]}
                    </p>
                )}
            </div>

            {/* Message field */}
            <div className="mb-6">
                <label htmlFor="message" className="block text-sm font-medium text-gray-700">
                    Message
                </label>
                <textarea
                    name="message"
                    id="message"
                    rows={4}
                    className={`mt-1 block w-full rounded-md border px-3 py-2 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
                        state.errors?.message ? 'border-red-500' : 'border-gray-300'
                    }`}
                    aria-describedby={state.errors?.message ? 'message-error' : undefined}
                />
                {state.errors?.message && (
                    <p id="message-error" className="mt-1 text-sm text-red-600">
                        {state.errors.message[0]}
                    </p>
                )}
            </div>

            <SubmitButton label="Send Message" pendingLabel="Sending…" />
        </form>
    )
}
```

### 4. Usage in a Server Component page

```typescript
// app/(routes)/contact/page.tsx  ← Server Component, no "use client"
import { ContactForm } from '@/app/components/client/ContactForm'

export default function ContactPage() {
    return (
        <main className="mx-auto max-w-lg px-4 py-12">
            <h1 className="mb-8 text-2xl font-bold text-gray-900">Contact Us</h1>
            <ContactForm />
        </main>
    )
}
```

---

## Accessibility Checklist (Both Stacks)

- [ ] `novalidate` on the `<form>` — validation is handled server-side, not by the browser.
- [ ] Every input has a `<label>` associated via `for`/`htmlFor` or wrapping.
- [ ] Error messages are linked to their input via `aria-describedby`.
- [ ] Global messages use `role="alert"` so screen readers announce them.
- [ ] The submit button uses `aria-disabled` (not just `disabled`) when pending, so it remains focusable.
- [ ] Loading text uses `aria-live="polite"` or `role="status"` to announce state changes.

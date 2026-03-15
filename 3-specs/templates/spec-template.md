# Spec: [Module Name]

## 1. Context & Objective
- **User Story:** As a [role], I want to [action] so that [benefit].
- **Success Criteria:** What defines this task as "Done"?

## 1b. Out of Scope
> Explicit list of what this module does NOT include. This is mandatory — it prevents gold-plating and "but what about X?" scope creep.
- This module does NOT handle [X].
- [Feature Y] will be addressed in a separate spec.

## 2. Data Contract (The Interface)
- **Input Model:** (Fields, types, constraints).
- **Output Model/Response:** (Expected JSON or return object).
- **API/Function Signature:** `function_name(params: Type) -> ReturnType`

## 3. Business Logic & Rules
- **Rule 1:** [Specific logic]
- **Edge Case A:** [What happens if X is null?]
- **Edge Case B:** [Error handling preference]

## 4. UI/UX Requirements (Optional)
- **Component:** [E.g., Modal, Page, Form]
- **State:** [E.g., Loading, Success, Error states]
- **Interactivity:** [Alpine.js/React specific behavior]

## 5. Testing Requirements
- **Unit Tests:** What specific functions must be tested?
- **Integration:** How does this module connect with existing parts?
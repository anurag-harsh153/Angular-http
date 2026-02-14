# Template-Driven Forms Machine Coding Challenges

These challenges are designed to test your understanding and practical application of Angular's Template-Driven Forms, from basic form creation and validation to more advanced scenarios.

---

### Level 1 (Junior): Basic User Registration Form

**Scenario**: You need to create a simple user registration form that collects username, email, and password. The form should include basic validation and display feedback to the user.

**Task**:
1.  Create an Angular component `RegistrationFormComponent`.
2.  In `RegistrationFormComponent`'s template, build a form using template-driven form directives.
3.  Include three input fields for:
    *   `username`: required, minimum 3 characters.
    *   `email`: required, must be a valid email format (use `type="email"` HTML5 validation).
    *   `password`: required, minimum 6 characters.
4.  Bind these inputs using `[(ngModel)]` to properties in the component's model (e.g., `user.username`, `user.email`, `user.password`).
5.  Display real-time validation error messages for each field (e.g., "Username is required," "Email is invalid") when the field is invalid and has been touched.
6.  The submit button should be disabled until the entire form is valid.
7.  On form submission, log the form's `value` and its `valid` status to the console.

**Expected Output**:
*   A functional registration form with inputs, submit button.
*   Validation messages appear/disappear correctly as user interacts.
*   Submit button enables only when all fields are valid.
*   Console log shows submitted data and form validity.

---

### Level 2 (Mid): Product Editing Form with Custom Validator

**Scenario**: You need to create a form to edit product details. This form should include standard inputs and a custom validation rule: the product name cannot be "Test Product" (case-insensitive).

**Task**:
1.  Create an Angular component `ProductEditFormComponent`.
2.  Define a `Product` interface with `id: number`, `name: string`, `price: number`, and `description: string`.
3.  Initialize a `product` object in the component (e.g., `product = { id: 1, name: 'Angular Widget', price: 29.99, description: 'A powerful widget for Angular apps.' }`).
4.  In the template, create a form to edit these product details.
    *   `name`: required, and apply a **custom validator directive** that forbids the name "Test Product".
    *   `price`: required, must be a positive number (use `type="number"` and `min="0"`).
    *   `description`: optional, multiline input.
5.  Display validation error messages for each field, including the custom "forbidden name" error.
6.  Add a "Save" button that is disabled when the form is invalid. On save, log the updated product data to the console.
7.  Add a "Reset" button that clears the form or reverts to the initial product data.

**Expected Output**:
*   A form pre-filled with product data.
*   Custom validation message appears if "Test Product" (or "test product", etc.) is entered for the name.
*   Price validation works for negative or empty values.
*   Save button enables/disables correctly.
*   Console logs updated product data on successful save.
*   Reset button functionality.

---

### Level 3 (Senior): Dynamic Survey Form with `ngModelGroup` and `updateOn`

**Scenario**: You need to build a dynamic survey form where users can answer a series of questions. Some questions might have sub-questions that appear based on the answer to a previous question. You also want to optimize performance for text areas by only validating on blur.

**Task**:
1.  Create an Angular component `SurveyFormComponent`.
2.  Define a `Question` interface (e.g., `id: number`, `text: string`, `type: 'text' | 'radio'`, `options?: string[]`, `subQuestions?: Question[]`).
3.  In the component, define an array of `Question` objects representing your survey. Include at least one question that, when answered in a certain way, reveals a sub-question.
    *   Example: "Are you interested in advanced features?" (radio: Yes/No). If Yes, reveal "Which advanced feature?" (text input).
4.  In the template, dynamically render the survey questions.
    *   Use `ngModelGroup` to group related form controls (e.g., a question and its sub-questions).
    *   For text input questions (especially `textarea`s), use `[ngModelOptions]="{ updateOn: 'blur' }"` to only trigger validation and model updates when the field loses focus, improving performance.
    *   Apply appropriate validation (`required`) to all questions.
5.  Implement logic to conditionally display sub-questions based on the parent question's answer.
6.  The "Submit Survey" button should be disabled until all visible and required questions are valid.
7.  On submission, log the complete survey answers (including dynamically revealed questions) to the console.

**Expected Output**:
*   A survey form that dynamically shows/hides sub-questions.
*   Validation messages work for dynamically added fields.
*   Submit button respects the validity of all *visible* required fields.
*   The `updateOn: 'blur'` functionality should be demonstrable (e.g., type in a textarea, validation only appears after clicking outside).
*   Console logs the full, valid survey data structure upon submission.

---
**Considerations for Level 3**:
*   How to handle `name` attributes for dynamically generated inputs to ensure uniqueness within `ngModelGroup`.
*   Structuring your `question` data model to easily manage nested questions and their answers.
*   Ensuring that form validity accurately reflects only the currently visible and required fields.

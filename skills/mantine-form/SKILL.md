---
name: mantine-form
description: >
  Build forms using @mantine/form. Use this skill when: (1) setting up a form with useForm,
  (2) adding validation rules, schema validation (zod, valibot, arktype) or async validation,
  (3) working with nested object or array fields, (4) sharing form state across components
  with createFormContext, (5) choosing between controlled and uncontrolled mode, (6) reading
  form values during render with form.useWatchValue, (7) using the standalone useField hook, or
  (8) any task involving useForm, getInputProps, onSubmit, insertListItem, or form validation.
---

# Mantine Form Skill

Written for Mantine 9.x.

## Core Workflow

### 1. Set up the form

Uncontrolled mode is the recommended mode for all forms. The default is `'controlled'`, so set it explicitly.

```tsx
const form = useForm({
  mode: 'uncontrolled',
  initialValues: {
    email: '',
    age: 0,
  },
  validate: {
    email: isEmail('Invalid email'),
    age: isInRange({ min: 18 }, 'Must be at least 18'),
  },
});
```

### 2. Wire inputs with getInputProps

In uncontrolled mode every input needs `key={form.key('path')}`. Without it the input does not
update after `form.setFieldValue`, `form.setValues` or `form.reset`.

```tsx
<TextInput label="Email" key={form.key('email')} {...form.getInputProps('email')} />
<NumberInput label="Age" key={form.key('age')} {...form.getInputProps('age')} />
```

Checkboxes and switches need `{ type: 'checkbox' }`, individual radios need `{ type: 'radio', value }`:

```tsx
<Checkbox
  label="I agree"
  key={form.key('agreed')}
  {...form.getInputProps('agreed', { type: 'checkbox' })}
/>
<Radio label="Red" {...form.getInputProps('color', { type: 'radio', value: 'red' })} />
```

### 3. Handle submission

```tsx
<form onSubmit={form.onSubmit((values) => console.log(values))}>
  ...
  <Button type="submit">Submit</Button>
</form>
```

`onSubmit` only calls the handler when validation passes. To handle failures:

```tsx
form.onSubmit(
  (values, event) => save(values),
  (errors, values, event) => console.log('Validation failed', errors)
);
```

## Validation

### Rules object (most common)

```tsx
validate: {
  name: isNotEmpty('Required'),
  email: isEmail('Invalid email'),
  password: hasLength({ min: 8 }, 'Min 8 chars'),
  confirmPassword: matchesField('password', 'Passwords do not match'),
}
```

### Schema (zod, valibot, arktype and other Standard Schema libraries)

```tsx
import { z } from 'zod/v4';
import { schemaResolver, useForm } from '@mantine/form';

const schema = z.object({
  email: z.email({ error: 'Invalid email' }),
  age: z.number().min(18, { error: 'Must be at least 18' }),
});

const form = useForm({
  mode: 'uncontrolled',
  initialValues: { email: '', age: 0 },
  validate: schemaResolver(schema, { sync: true }),
});
```

`schemaResolver` is built in: no resolver package is needed. Pass `{ sync: true }` for
synchronous schemas so that `form.validate()` returns a plain result instead of a `Promise`.

### Function (for cross-field logic)

```tsx
validate: (values) => ({
  endDate: values.endDate < values.startDate ? 'End must be after start' : null,
});
```

### When to validate

```tsx
validateInputOnChange: true,        // validate all fields on every change
validateInputOnChange: ['email'],   // validate specific fields only
validateInputOnBlur: true,          // validate on blur instead
```

## Modes

|                                  | `'uncontrolled'` (recommended)      | `'controlled'` (default) |
| -------------------------------- | ----------------------------------- | ------------------------ |
| Values storage                   | Ref                                 | React state              |
| `form.values`                    | Not updated, use `form.getValues()` | Updated on every change  |
| Re-renders on value change       | No                                  | Yes                      |
| Input props                      | `defaultValue` + `onChange`         | `value` + `onChange`     |
| `key={form.key(path)}` on inputs | Required                            | Not needed               |

`form.errors`, `form.submitting` and `form.validating` are React state in both modes.

## Reading values during render

`form.getValues()` does not rerender the component in uncontrolled mode. To show or hide
part of the form based on a value, use `form.useWatchValue`:

```tsx
const shipsInternationally = form.useWatchValue('shipsInternationally');
```

## Looking things up

This skill covers the common API. For anything else, do not guess:

- If the Mantine MCP server (`@mantine/mcp-server`) is connected, use `search_docs` and `get_item_doc`
- Otherwise fetch `https://mantine.dev/llms.txt` and open the linked form pages

## References

- **[`references/api.md`](references/api.md)** — Full API: `useForm` options, complete return value, `useField`, `createFormContext`, `createFormActions`, `schemaResolver`, all built-in validators, key types
- **[`references/patterns.md`](references/patterns.md)** — Code examples: nested objects, array fields, list validation with `formRootRule`, async validation, conditional fields, form context across components, `transformValues`, `useField` standalone, server errors

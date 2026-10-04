# @mantine/form Patterns

## Table of Contents
- [Basic form with validation](#basic-form-with-validation)
- [Nested object fields](#nested-object-fields)
- [Array / list fields](#array--list-fields)
- [Async validation](#async-validation)
- [Conditional fields](#conditional-fields)
- [Form context across components](#form-context-across-components)
- [transformValues](#transformvalues)
- [Uncontrolled mode](#uncontrolled-mode)
- [Standalone useField](#standalone-usefield)
- [Server errors after submission](#server-errors-after-submission)

---

## Basic form with validation

```tsx
import { useForm, isEmail, isNotEmpty, hasLength } from '@mantine/form';

function BasicForm() {
  const form = useForm({
    mode: 'uncontrolled',
    initialValues: { name: '', email: '', password: '' },
    validate: {
      name: isNotEmpty('Name is required'),
      email: isEmail('Invalid email'),
      password: hasLength({ min: 8 }, 'Password must be at least 8 characters'),
    },
  });

  return (
    <form onSubmit={form.onSubmit((values) => console.log(values))}>
      <TextInput label="Name" key={form.key('name')} {...form.getInputProps('name')} />
      <TextInput label="Email" key={form.key('email')} {...form.getInputProps('email')} />
      <PasswordInput label="Password" key={form.key('password')} {...form.getInputProps('password')} />
      <Button type="submit">Submit</Button>
    </form>
  );
}
```

---

## Nested object fields

Use dot notation to address nested fields.

```tsx
const form = useForm({
  mode: 'uncontrolled',
  initialValues: {
    user: {
      name: '',
      address: {
        city: '',
        zip: '',
      },
    },
  },
  validate: {
    user: {
      name: isNotEmpty('Required'),
      address: {
        city: isNotEmpty('City is required'),
        zip: matches(/^\d{5}$/, 'Must be 5 digits'),
      },
    },
  },
});

// Access nested fields with dot notation
<TextInput key={form.key('user.name')} {...form.getInputProps('user.name')} />
<TextInput key={form.key('user.address.city')} {...form.getInputProps('user.address.city')} />
<TextInput key={form.key('user.address.zip')} {...form.getInputProps('user.address.zip')} />
```

---

## Array / list fields

Give every item a stable `key` value (`randomId` from `@mantine/hooks`) and use it as the React key
of the row. Use `formRootRule` to validate the list itself next to its items.

```tsx
import { formRootRule, isNotEmpty, useForm } from '@mantine/form';
import { randomId } from '@mantine/hooks';

const form = useForm({
  mode: 'uncontrolled',
  initialValues: {
    employees: [{ name: '', role: '', key: randomId() }],
  },
  validate: {
    employees: {
      [formRootRule]: isNotEmpty('At least one employee is required'),
      name: isNotEmpty('Name required'),
      role: isNotEmpty('Role required'),
    },
  },
});

// Render the list
const fields = form.getValues().employees.map((item, index) => (
  <Group key={item.key}>
    <TextInput
      placeholder="Name"
      key={form.key(`employees.${index}.name`)}
      {...form.getInputProps(`employees.${index}.name`)}
    />
    <TextInput
      placeholder="Role"
      key={form.key(`employees.${index}.role`)}
      {...form.getInputProps(`employees.${index}.role`)}
    />
    <ActionIcon aria-label="Remove employee" onClick={() => form.removeListItem('employees', index)}>
      <IconTrash />
    </ActionIcon>
  </Group>
));

return (
  <form onSubmit={form.onSubmit((values) => console.log(values))}>
    {fields}
    {form.errors.employees && <Text c="red" size="sm">{form.errors.employees}</Text>}
    <Button
      onClick={() => form.insertListItem('employees', { name: '', role: '', key: randomId() })}
    >
      Add employee
    </Button>
    <Button type="submit">Submit</Button>
  </form>
);
```

**List methods:**
```tsx
form.insertListItem('employees', { name: '', role: '' });       // append
form.insertListItem('employees', { name: '', role: '' }, 0);    // prepend
form.removeListItem('employees', index);
form.reorderListItem('employees', { from: 2, to: 0 });
form.replaceListItem('employees', index, { name: 'New', role: 'Dev' });
```

---

## Async validation

Return a `Promise` from any validator. Use the provided `AbortSignal` to avoid stale results.

```tsx
const form = useForm({
  mode: 'uncontrolled',
  initialValues: { username: '' },
  validate: {
    username: async (value, _values, _path, signal) => {
      if (!value) return 'Username is required';
      const response = await fetch(`/api/check-username?q=${value}`, { signal });
      if (signal?.aborted) return null;
      const { taken } = await response.json();
      return taken ? 'Username is already taken' : null;
    },
  },
  validateInputOnChange: ['username'],
  validateDebounce: 500,   // debounce on-change validation
});
```

`form.validating` is `true` while any async validation runs, `form.isValidating('username')` checks one field.
With async rules, `form.validate()` and `form.isValid()` return a `Promise`.

---

## Conditional fields

Use `form.useWatchValue` to read a value during render. `form.getValues()` does not rerender the
component in uncontrolled mode.

```tsx
const form = useForm({
  mode: 'uncontrolled',
  initialValues: { hasCompany: false, companyName: '' },
});

const hasCompany = form.useWatchValue('hasCompany');

<Checkbox
  label="I represent a company"
  key={form.key('hasCompany')}
  {...form.getInputProps('hasCompany', { type: 'checkbox' })}
/>
{hasCompany && (
  <TextInput
    label="Company name"
    key={form.key('companyName')}
    {...form.getInputProps('companyName')}
  />
)}
```

---

## Form context across components

Share one form instance across a component tree without prop drilling.

```tsx
// 1. Create typed context once
import { createFormContext, isNotEmpty } from '@mantine/form';

interface ProfileValues {
  bio: string;
  website: string;
}

const [FormProvider, useFormContext, useProfileForm] = createFormContext<ProfileValues>();

// 2. Wrap your form tree with FormProvider
function ProfileForm() {
  const form = useProfileForm({
    mode: 'uncontrolled',
    initialValues: { bio: '', website: '' },
    validate: {
      bio: isNotEmpty('Bio is required'),
    },
  });

  return (
    <FormProvider form={form}>
      <form onSubmit={form.onSubmit((values) => save(values))}>
        <BioField />
        <WebsiteField />
        <Button type="submit">Save</Button>
      </form>
    </FormProvider>
  );
}

// 3. Access form in any child — no prop drilling
function BioField() {
  const form = useFormContext();
  return <Textarea label="Bio" key={form.key('bio')} {...form.getInputProps('bio')} />;
}

function WebsiteField() {
  const form = useFormContext();
  return <TextInput label="Website" key={form.key('website')} {...form.getInputProps('website')} />;
}
```

---

## transformValues

Shape the values before they reach `onSubmit`. The transform is applied transparently — `onSubmit` receives `TransformedValues`, not `Values`.

```tsx
const form = useForm({
  mode: 'uncontrolled',
  initialValues: {
    price: '',        // stored as string in input
    tags: 'a, b, c', // stored as comma-separated string
  },
  transformValues: (values) => ({
    price: Number(values.price),
    tags: values.tags.split(',').map((t) => t.trim()),
  }),
});

// handler receives { price: number, tags: string[] }
form.onSubmit((values) => console.log(values));
```

---

## Uncontrolled mode

The recommended mode for all forms. Values are stored in a ref, so typing does not rerender the form.

```tsx
const form = useForm({
  mode: 'uncontrolled',
  initialValues: { name: '', email: '' },
  validate: { email: isEmail('Invalid email') },
});

// Every input needs key={form.key(path)}: inputs receive defaultValue, and the key is what
// updates them after form.setFieldValue, form.setValues and form.reset
<TextInput label="Name" key={form.key('name')} {...form.getInputProps('name')} />
<TextInput label="Email" key={form.key('email')} {...form.getInputProps('email')} />

// Read current values in event handlers:
const current = form.getValues();
```

- `form.values` is not updated in uncontrolled mode. Use `form.getValues()` in handlers and `form.useWatchValue(path)` during render.
- `form.errors`, `form.submitting` and `form.validating` are React state in both modes and can be used during render.

---

## Standalone useField

Manage a single field without a full form — useful for isolated inputs or custom field components.

```tsx
import { useField, isEmail } from '@mantine/form';

function EmailField() {
  const field = useField({
    initialValue: '',
    validate: isEmail('Invalid email'),
    validateOnBlur: true,
  });

  return (
    <TextInput label="Email" {...field.getInputProps()} />
  );
}
```

---

## Server errors after submission

Set server-side errors on fields after a failed API call.

```tsx
const form = useForm({ mode: 'uncontrolled', initialValues: { email: '', password: '' } });

const handleSubmit = async (values: typeof form.values) => {
  try {
    await login(values);
  } catch (error) {
    if (error instanceof ApiValidationError) {
      // Map server field errors onto form: { email: 'Already registered' }
      form.setErrors(error.fields);
    } else {
      form.setFieldError('password', 'Invalid email or password');
    }
  }
};

<form onSubmit={form.onSubmit(handleSubmit)}>
  <TextInput label="Email" key={form.key('email')} {...form.getInputProps('email')} />
  <PasswordInput label="Password" key={form.key('password')} {...form.getInputProps('password')} />
  <Button type="submit" loading={form.submitting}>Sign in</Button>
</form>
```

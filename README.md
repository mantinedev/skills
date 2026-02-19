# Mantine Skills for AI Agents

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) for building with [Mantine](https://mantine.dev/).
Compatible with other agents as well via [skills.sh](https://skills.sh/).

## Available Skills

### `mantine-combobox`

Build custom dropdown/select/autocomplete/multiselect components using Mantine's Combobox primitives.

Use when: creating a custom select-like component, building a searchable dropdown, implementing multi-select or tags input, customizing option rendering, adding custom filtering logic, or working with `useCombobox`, `Combobox.Target`, `Combobox.Option`, or `Combobox.Dropdown`.

```sh
npx skills add https://github.com/mantinedev/skills --skill mantine-combobox
```

---

### `mantine-form`

Build forms using `@mantine/form`.

Use when: setting up a form with `useForm`, adding validation rules or async validation, working with nested object or array fields, sharing form state across components with `createFormContext`, using uncontrolled mode for performance, using the standalone `useField` hook, or any task involving `useForm`, `getInputProps`, `onSubmit`, `insertListItem`, or form validation.

```sh
npx skills add https://github.com/mantinedev/skills --skill mantine-form
```

---

### `mantine-custom-components`

Build custom components that integrate with Mantine's theming, Styles API, and core features.

Use when: creating a new component using `factory()`, `polymorphicFactory()`, or `genericFactory()`, adding Styles API support (`classNames`, `styles`, `vars`, `unstyled`), implementing CSS variables via `createVarsResolver`, building compound components, registering a component with `MantineProvider` via `Component.extend()`, or working with `Factory`, `useProps`, `useStyles`, `BoxProps`, `StylesApiProps`, or `ElementProps`.

```sh
npx skills add https://github.com/mantinedev/skills --skill mantine-custom-components
```

## Install All Skills

```sh
npx skills add https://github.com/mantinedev/skills --skill mantine-combobox
npx skills add https://github.com/mantinedev/skills --skill mantine-form
npx skills add https://github.com/mantinedev/skills --skill mantine-custom-components
```

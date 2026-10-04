---
name: mantine-combobox
description: >
  Build custom dropdown/select/autocomplete/multiselect components using Mantine's Combobox
  primitives. Use this skill when: (1) creating a new custom select-like component with
  Combobox primitives, (2) building a searchable dropdown, (3) implementing a multi-select
  or tags input variant, (4) customizing option rendering, (5) adding custom filtering logic,
  or (6) any task involving useCombobox, Combobox.Target, Combobox.Option, or Combobox.Dropdown.
---

# Mantine Combobox Skill

Written for Mantine 9.x.

## Overview

`Combobox` provides low-level primitives for building any select-like UI. The built-in
`Select`, `MultiSelect`, `Autocomplete`, and `TagsInput` components are all built on top of it.

## Check for a ready-made component first

Build with `Combobox` primitives only when none of these fit:

| Need | Use |
|---|---|
| Standard select, multi-select, autocomplete or tags input | `Select`, `MultiSelect`, `Autocomplete`, `TagsInput` (all support `renderOption`) |
| Options dropdown attached to a button or any other element, no input | `ComboboxPopover` (data-driven: `data`, `value`, `onChange`, `searchable`, `multiple`) |
| Hierarchical options | `TreeSelect` |

```tsx
<ComboboxPopover data={['React', 'Angular', 'Vue']} value={value} onChange={setValue}>
  <ComboboxPopover.Target>
    <Button variant="default">{value || 'Select framework'}</Button>
  </ComboboxPopover.Target>
</ComboboxPopover>
```

## Core Workflow

### 1. Create the store

```tsx
const combobox = useCombobox({
  onDropdownClose: () => combobox.resetSelectedOption(),
  onDropdownOpen: () => combobox.selectFirstOption(),
});
```

### 2. Render structure

```tsx
<Combobox store={combobox} onOptionSubmit={handleSubmit}>
  <Combobox.Target targetType="button">
    <InputBase
      component="button"
      type="button"
      pointer
      rightSection={<Combobox.Chevron />}
      onClick={() => combobox.toggleDropdown()}
    >
      {value || <Input.Placeholder>Pick value</Input.Placeholder>}
    </InputBase>
  </Combobox.Target>
  <Combobox.Dropdown>
    <Combobox.Options>
      {options.map((item) => (
        <Combobox.Option value={item} key={item}>{item}</Combobox.Option>
      ))}
    </Combobox.Options>
  </Combobox.Dropdown>
</Combobox>
```

### 3. Handle submit

```tsx
const handleSubmit = (val: string) => {
  setValue(val);
  combobox.closeDropdown();
};
```

## Target Types

| Scenario | Use |
|---|---|
| Button trigger (no text input) | `<Combobox.Target targetType="button">` |
| Input trigger | `<Combobox.Target>` (default) |
| Pills + separate input (multi-select) | `<Combobox.DropdownTarget>` + `<Combobox.EventsTarget>` |

## Looking things up

This skill covers the common API. For anything else, do not guess:

- If the Mantine MCP server (`@mantine/mcp-server`) is connected, use `search_docs` and `get_item_doc`
- Otherwise fetch `https://mantine.dev/llms.txt` and open the Combobox page. More than 50 complete examples are at `https://mantine.dev/combobox/`

## References

- **[`references/api.md`](references/api.md)** — Full API: `useCombobox` options and store, `useVirtualizedCombobox`, all sub-component props, CSS variables, Styles API selectors
- **[`references/patterns.md`](references/patterns.md)** — Code examples: searchable select, search inside the dropdown, multi-select with pills, groups, custom rendering, clear button, form integration, dropdown that fits the viewport

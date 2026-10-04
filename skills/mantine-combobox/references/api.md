# Combobox API Reference

## Table of Contents
- [useCombobox hook](#usecombobox-hook)
- [Combobox (root)](#combobox-root)
- [Sub-components](#sub-components)
- [CSS variables & Styles API](#css-variables--styles-api)

---

## useCombobox hook

```tsx
const combobox = useCombobox(options?: UseComboboxOptions);
```

**Options:**
```ts
interface UseComboboxOptions {
  defaultOpened?: boolean;
  opened?: boolean;
  onOpenedChange?: (opened: boolean) => void;
  onDropdownClose?: (eventSource: 'keyboard' | 'mouse' | 'unknown') => void;
  onDropdownOpen?: (eventSource: 'keyboard' | 'mouse' | 'unknown') => void;
  loop?: boolean;           // Default: true — keyboard nav wraps at boundaries
  scrollBehavior?: ScrollBehavior; // Default: 'instant'
}
```

**Returned store:**
```ts
interface ComboboxStore {
  // Dropdown state
  dropdownOpened: boolean;
  openDropdown(eventSource?: 'keyboard' | 'mouse' | 'unknown'): void;
  closeDropdown(eventSource?: 'keyboard' | 'mouse' | 'unknown'): void;
  toggleDropdown(eventSource?: 'keyboard' | 'mouse' | 'unknown'): void;

  // Option keyboard navigation
  selectedOptionIndex: number;
  getSelectedOptionIndex(): number;       // -1 when nothing is selected
  selectOption(index: number): void;
  selectActiveOption(): string | null;
  selectFirstOption(): string | null;
  selectNextOption(): string | null;
  selectPreviousOption(): string | null;
  resetSelectedOption(): void;
  clickSelectedOption(): void;
  updateSelectedOptionIndex(
    target?: 'active' | 'selected' | number,
    options?: { scrollIntoView?: boolean }
  ): void;

  // Programmatic focus
  searchRef: React.RefObject<HTMLInputElement | null>;  // Combobox.Search input
  targetRef: React.RefObject<HTMLElement | null>;
  focusSearchInput(): void;
  focusTarget(): void;
}
```

"Selected" option means the option highlighted by keyboard navigation (`data-combobox-selected`),
not the option that holds the current value.

### useVirtualizedCombobox

Store for virtualized option lists (`@tanstack/react-virtual`, `react-virtuoso`). Option indexing
does not rely on the DOM: keep the selected option index in React state and pass it to the hook
together with `totalOptionsCount`, `getOptionId`, `selectedOptionIndex`, `setSelectedOptionIndex`
and `onSelectedOptionSubmit`. See the virtualized examples in the Combobox documentation.

---

## Combobox (root)

```tsx
<Combobox
  store={combobox}              // required — ComboboxStore from useCombobox()
  onOptionSubmit={fn}           // (value: string, optionProps) => void
  size="sm"                     // MantineSize | string, default: 'sm'
  dropdownPadding={4}           // any CSS padding value, 4px when not set
  resetSelectionOnOptionHover   // boolean
  readOnly                      // boolean — blocks keyboard interactions on the target only;
                                // option clicks still call onOptionSubmit
  floatingHeight="viewport"     // dropdown fills the available viewport height, disables flip
  // + all Popover props (position, offset, width, withinPortal, etc.)
/>
```

---

## Sub-components

### Combobox.Target
```tsx
<Combobox.Target
  targetType="input"          // 'button' | 'input', default: 'input'
  withKeyboardNavigation      // boolean, default: true
  withAriaAttributes          // boolean, default: true
  withExpandedAttribute       // boolean, default: false
  autoComplete="off"          // string
  refProp="ref"               // prop name used to pass the ref to the child
>
  {/* single child — the trigger element */}
</Combobox.Target>
```

Use `targetType="button"` when the trigger is a button: Space and Enter open the dropdown.
With the default `input` type they do not.

### Combobox.DropdownTarget
Marks the element used for dropdown positioning when separate from the keyboard events target. Used together with `Combobox.EventsTarget` in multi-select/pills patterns.

### Combobox.EventsTarget
Receives keyboard events for dropdown navigation. Used alongside `Combobox.DropdownTarget` when the typing input is nested inside the trigger (e.g. inside `PillsInput`).

### Combobox.Dropdown
```tsx
<Combobox.Dropdown hidden={false}>
  {/* dropdown content */}
</Combobox.Dropdown>
```

Use `hidden` to hide the dropdown without unmounting it, for example when there are no options to show.

### Combobox.Options
```tsx
<Combobox.Options labelledBy="some-label-id">
  {/* Combobox.Option or Combobox.Group elements */}
</Combobox.Options>
```

### Combobox.Option
```tsx
<Combobox.Option
  value="react"       // string | number | boolean | bigint, required
  active={false}      // marks the option that holds the current value: sets data-combobox-active,
                      // used by selectActiveOption(). Has no styles by default — render a check icon
                      // or style [data-combobox-active] yourself
  selected={false}    // sets data-combobox-selected (keyboard highlight) manually
  disabled={false}
>
  React
</Combobox.Option>
```

### Combobox.Search
Built-in search input for the dropdown, wired to keyboard navigation. Render it before `Combobox.Options`. Focus it with `combobox.focusSearchInput()` in `onDropdownOpen`.

```tsx
<Combobox.Search
  value={search}
  onChange={(e) => setSearch(e.currentTarget.value)}
  placeholder="Search..."
  withAriaAttributes      // boolean, default: true
  withKeyboardNavigation  // boolean, default: true
  // + all Input props
/>
```

### Combobox.Empty
```tsx
<Combobox.Empty>Nothing found</Combobox.Empty>
```

### Combobox.Group
```tsx
<Combobox.Group label="Frontend">
  <Combobox.Option value="react">React</Combobox.Option>
</Combobox.Group>
```

### Combobox.Header / Combobox.Footer
```tsx
<Combobox.Header>Custom header</Combobox.Header>
<Combobox.Footer>Custom footer</Combobox.Footer>
```

### Combobox.Chevron
```tsx
<Combobox.Chevron size="sm" error={error} color="blue" />
```

### Combobox.ClearButton
```tsx
<Combobox.ClearButton onClear={() => setValue(null)} />
```

### Combobox.HiddenInput
```tsx
<Combobox.HiddenInput
  value={value}           // primitive, array of primitives, or null
  valuesDivider=","       // string, default: ','
  name="myField"
  form="myForm"
/>
```

---

## CSS variables & Styles API

### CSS variables (set on the `dropdown` element)
| Variable | Description |
|---|---|
| `--combobox-option-fz` | Option font size (driven by `size` prop) |
| `--combobox-option-padding` | Option padding (driven by `size` prop) |
| `--combobox-padding` | Dropdown padding (driven by `dropdownPadding`, default 4px) |
| `--combobox-floating-options-max-height` | Available height for options when `floatingHeight="viewport"` is set |

### Styles API selectors
| Selector | Element |
|---|---|
| `dropdown` | Dropdown container |
| `options` | Options listbox |
| `option` | Individual option |
| `search` | Search input |
| `empty` | Nothing found message |
| `header` | Dropdown header |
| `footer` | Dropdown footer |
| `group` | Group container |
| `groupLabel` | Group label text |

### Option data attributes
| Attribute | Meaning |
|---|---|
| `data-combobox-selected` | Option highlighted by keyboard navigation (styled by default) |
| `data-combobox-active` | Option with `active` prop (no default styles) |
| `data-combobox-disabled` | Disabled option |

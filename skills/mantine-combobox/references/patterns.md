# Combobox Implementation Patterns

## Table of Contents
- [Basic select (button trigger)](#basic-select-button-trigger)
- [Searchable select (input trigger)](#searchable-select-input-trigger)
- [Multi-select with pills](#multi-select-with-pills)
- [Options with groups](#options-with-groups)
- [Custom option rendering](#custom-option-rendering)
- [Clear button](#clear-button)
- [Form integration (hidden input)](#form-integration-hidden-input)
- [Highlight the current value on open](#highlight-the-current-value-on-open)
- [Creatable option](#creatable-option)
- [Async search](#async-search)
- [Search inside the dropdown](#search-inside-the-dropdown)
- [Dropdown that fits the viewport](#dropdown-that-fits-the-viewport)
- [Nothing found message](#nothing-found-message)

---

## Basic select (button trigger)

```tsx
function CustomSelect({ data }: { data: string[] }) {
  const [value, setValue] = useState<string | null>(null);

  const combobox = useCombobox({
    onDropdownClose: () => combobox.resetSelectedOption(),
  });

  const options = data.map((item) => (
    <Combobox.Option value={item} key={item} active={item === value}>
      {item}
    </Combobox.Option>
  ));

  return (
    <Combobox
      store={combobox}
      onOptionSubmit={(val) => {
        setValue(val);
        combobox.closeDropdown();
      }}
    >
      <Combobox.Target targetType="button">
        <InputBase
          component="button"
          type="button"
          pointer
          rightSection={<Combobox.Chevron />}
          rightSectionPointerEvents="none"
          onClick={() => combobox.toggleDropdown()}
        >
          {value || <Input.Placeholder>Pick value</Input.Placeholder>}
        </InputBase>
      </Combobox.Target>

      <Combobox.Dropdown>
        <Combobox.Options>{options}</Combobox.Options>
      </Combobox.Dropdown>
    </Combobox>
  );
}
```

---

## Searchable select (input trigger)

Input acts as both trigger and search field. Filter options based on the typed value.

```tsx
function SearchableSelect({ data }: { data: string[] }) {
  const [value, setValue] = useState<string | null>(null);
  const [search, setSearch] = useState('');

  const combobox = useCombobox({
    onDropdownClose: () => combobox.resetSelectedOption(),
    onDropdownOpen: () => combobox.selectFirstOption(),
  });

  const shouldFilterOptions = data.every((item) => item !== search);
  const filtered = shouldFilterOptions
    ? data.filter((item) => item.toLowerCase().includes(search.toLowerCase().trim()))
    : data;

  const options = filtered.length > 0
    ? filtered.map((item) => (
        <Combobox.Option value={item} key={item}>{item}</Combobox.Option>
      ))
    : <Combobox.Empty>Nothing found</Combobox.Empty>;

  return (
    <Combobox
      store={combobox}
      onOptionSubmit={(val) => {
        setValue(val);
        setSearch(val);
        combobox.closeDropdown();
      }}
    >
      <Combobox.Target>
        <InputBase
          rightSection={<Combobox.Chevron />}
          value={search}
          onChange={(e) => {
            combobox.openDropdown();
            combobox.updateSelectedOptionIndex();
            setSearch(e.currentTarget.value);
          }}
          onClick={() => combobox.openDropdown()}
          onFocus={() => combobox.openDropdown()}
          onBlur={() => {
            combobox.closeDropdown();
            setSearch(value || '');
          }}
          placeholder="Search value"
          rightSectionPointerEvents="none"
        />
      </Combobox.Target>

      <Combobox.Dropdown>
        <Combobox.Options>{options}</Combobox.Options>
      </Combobox.Dropdown>
    </Combobox>
  );
}
```

---

## Multi-select with pills

Use `Combobox.DropdownTarget` for the outer container and `Combobox.EventsTarget` around the text input inside it.

```tsx
function MultiSelect({ data }: { data: string[] }) {
  const combobox = useCombobox({ onDropdownClose: () => combobox.resetSelectedOption() });
  const [search, setSearch] = useState('');
  const [value, setValue] = useState<string[]>([]);

  const handleValueSelect = (val: string) =>
    setValue((current) =>
      current.includes(val) ? current.filter((v) => v !== val) : [...current, val]
    );

  const handleValueRemove = (val: string) =>
    setValue((current) => current.filter((v) => v !== val));

  const options = data
    .filter((item) => item.toLowerCase().includes(search.trim().toLowerCase()))
    .map((item) => (
      <Combobox.Option value={item} key={item} active={value.includes(item)}>
        <Group gap="sm">
          {value.includes(item) ? <CheckIcon size={12} /> : null}
          <span>{item}</span>
        </Group>
      </Combobox.Option>
    ));

  const pills = value.map((item) => (
    <Pill key={item} withRemoveButton onRemove={() => handleValueRemove(item)}>
      {item}
    </Pill>
  ));

  return (
    <Combobox store={combobox} onOptionSubmit={handleValueSelect}>
      <Combobox.DropdownTarget>
        <PillsInput onClick={() => combobox.openDropdown()}>
          <Pill.Group>
            {pills}
            <Combobox.EventsTarget>
              <PillsInput.Field
                onFocus={() => combobox.openDropdown()}
                onBlur={() => combobox.closeDropdown()}
                value={search}
                placeholder="Search values"
                onChange={(e) => {
                  combobox.updateSelectedOptionIndex();
                  setSearch(e.currentTarget.value);
                }}
                onKeyDown={(e) => {
                  if (e.key === 'Backspace' && search.length === 0) {
                    e.preventDefault();
                    handleValueRemove(value[value.length - 1]);
                  }
                }}
              />
            </Combobox.EventsTarget>
          </Pill.Group>
        </PillsInput>
      </Combobox.DropdownTarget>

      <Combobox.Dropdown>
        <Combobox.Options>
          {options.length > 0 ? options : <Combobox.Empty>Nothing found</Combobox.Empty>}
        </Combobox.Options>
      </Combobox.Dropdown>
    </Combobox>
  );
}
```

---

## Options with groups

```tsx
const groups = data.map((group) => (
  <Combobox.Group label={group.group} key={group.group}>
    {group.items.map((item) => (
      <Combobox.Option value={item.value} key={item.value}>
        {item.label}
      </Combobox.Option>
    ))}
  </Combobox.Group>
));

// Inside Combobox.Dropdown:
<Combobox.Options>{groups}</Combobox.Options>
```

---

## Custom option rendering

Render any content inside `Combobox.Option`:

```tsx
const options = data.map((item) => (
  <Combobox.Option value={item.value} key={item.value}>
    <Group>
      <Avatar src={item.avatar} size="sm" />
      <div>
        <Text size="sm">{item.label}</Text>
        <Text size="xs" c="dimmed">{item.email}</Text>
      </div>
    </Group>
  </Combobox.Option>
));
```

---

## Clear button

```tsx
const rightSection = value ? (
  <Combobox.ClearButton onClear={() => { setValue(null); setSearch(''); }} />
) : (
  <Combobox.Chevron />
);

<InputBase
  rightSection={rightSection}
  rightSectionPointerEvents={value ? 'all' : 'none'}
/>
```

---

## Form integration (hidden input)

```tsx
// Single value
<Combobox.HiddenInput value={value} name="framework" />

// Multiple values (joined by comma by default)
<Combobox.HiddenInput value={selectedValues} name="frameworks" valuesDivider="," />
```

---

## Highlight the current value on open

Mark the option that holds the value with `active`, then highlight it and scroll it into view when
the dropdown opens:

```tsx
const combobox = useCombobox({
  onDropdownClose: () => combobox.resetSelectedOption(),
  onDropdownOpen: () => {
    combobox.selectActiveOption(); // highlight
    combobox.updateSelectedOptionIndex('active', { scrollIntoView: true }); // scroll to it
  },
});

<Combobox.Option value={item} key={item} active={item === value}>
  <Group gap="xs">
    {item === value && <CheckIcon size={12} />}
    {item}
  </Group>
</Combobox.Option>
```

`updateSelectedOptionIndex('active')` alone does not show a highlight.

---

## Creatable option

Add a regular option with a reserved value and handle it in `onOptionSubmit`:

```tsx
const exactMatch = data.some((item) => item.toLowerCase() === search.trim().toLowerCase());

<Combobox
  store={combobox}
  onOptionSubmit={(val) => {
    if (val === '$create') {
      onCreate(search.trim());
    } else {
      onChange(val);
    }
    setSearch('');
  }}
>
  {/* target */}
  <Combobox.Dropdown>
    <Combobox.Options>
      {options}
      {!exactMatch && search.trim().length > 0 && (
        <Combobox.Option value="$create">+ Create "{search.trim()}"</Combobox.Option>
      )}
    </Combobox.Options>
  </Combobox.Dropdown>
</Combobox>
```

---

## Async search

Keep the request state next to the search value. Highlight the first option in an effect when
results arrive, hide the dropdown for an empty query with `hidden`, and keep focus in the input
when the user clicks Retry.

```tsx
const [search, setSearch] = useState('');
const [debounced] = useDebouncedValue(search, 300);
const [state, setState] = useState<{ status: 'idle' | 'loading' | 'error' | 'done'; items: User[] }>({
  status: 'idle',
  items: [],
});
const requestId = useRef(0);

const load = (query: string) => {
  const id = ++requestId.current;
  setState((current) => ({ ...current, status: 'loading' }));
  searchUsers(query).then(
    (items) => id === requestId.current && setState({ status: 'done', items }),
    () => id === requestId.current && setState({ status: 'error', items: [] })
  );
};

useEffect(() => {
  if (debounced.trim()) {
    load(debounced);
  }
}, [debounced]);

useEffect(() => {
  combobox.selectFirstOption(); // after results render; skips disabled options
}, [state.items]);

<Combobox store={combobox} onOptionSubmit={handleSubmit}>
  <Combobox.Target>
    <TextInput
      value={search}
      rightSection={state.status === 'loading' ? <Loader size="xs" /> : null}
      onChange={(event) => {
        setSearch(event.currentTarget.value);
        combobox.openDropdown();
      }}
      onBlur={() => combobox.closeDropdown()}
    />
  </Combobox.Target>

  <Combobox.Dropdown hidden={search.trim() === '' || state.status === 'idle'}>
    <Combobox.Options>
      {state.status === 'error' ? (
        <Combobox.Empty>
          Could not load users{' '}
          <Anchor
            component="button"
            onMouseDown={(event) => event.preventDefault()}
            onClick={() => load(debounced)}
          >
            Retry
          </Anchor>
        </Combobox.Empty>
      ) : state.status === 'done' && state.items.length === 0 ? (
        <Combobox.Empty>No users found</Combobox.Empty>
      ) : (
        state.items.map((user) => (
          <Combobox.Option value={user.id} key={user.id} disabled={!user.active}>
            {user.name}
          </Combobox.Option>
        ))
      )}
    </Combobox.Options>
  </Combobox.Dropdown>
</Combobox>
```

---

## Search inside the dropdown

Button trigger with `Combobox.Search` in the dropdown. The button opens the dropdown with its own
`onClick` and the search input handles the keyboard, so `targetType="button"` is not needed here and
`withAriaAttributes={false}` keeps combobox ARIA attributes off the button. Focus the search input when the dropdown
opens and return focus to the target when it closes. Call `combobox.updateSelectedOptionIndex()`
whenever the options list changes, otherwise keyboard navigation keeps the old index.

```tsx
const [search, setSearch] = useState('');
const [value, setValue] = useState<string | null>(null);

const combobox = useCombobox({
  onDropdownClose: () => {
    combobox.resetSelectedOption();
    combobox.focusTarget();
    setSearch('');
  },
  onDropdownOpen: () => combobox.focusSearchInput(),
});

const options = data
  .filter((item) => item.toLowerCase().includes(search.toLowerCase().trim()))
  .map((item) => (
    <Combobox.Option value={item} key={item}>
      {item}
    </Combobox.Option>
  ));

<Combobox
  store={combobox}
  width={250}
  position="bottom-start"
  onOptionSubmit={(val) => {
    setValue(val);
    combobox.closeDropdown();
  }}
>
  <Combobox.Target withAriaAttributes={false}>
    <Button onClick={() => combobox.toggleDropdown()}>{value || 'Pick item'}</Button>
  </Combobox.Target>

  <Combobox.Dropdown>
    <Combobox.Search
      value={search}
      onChange={(event) => {
        combobox.updateSelectedOptionIndex();
        setSearch(event.currentTarget.value);
      }}
      placeholder="Search"
    />
    <Combobox.Options>
      {options.length > 0 ? options : <Combobox.Empty>Nothing found</Combobox.Empty>}
    </Combobox.Options>
  </Combobox.Dropdown>
</Combobox>
```

If no custom option markup is needed, `<ComboboxPopover searchable data={data} />` does the same
without any of this code.

---

## Dropdown that fits the viewport

For long lists, let the dropdown take the available viewport height and scroll inside it:

```tsx
<Combobox store={combobox} floatingHeight="viewport" onOptionSubmit={handleSubmit}>
  <Combobox.Target>{/* ... */}</Combobox.Target>
  <Combobox.Dropdown>
    <Combobox.Options>
      <ScrollArea.Autosize mah="var(--combobox-floating-options-max-height)" type="scroll">
        {options}
      </ScrollArea.Autosize>
    </Combobox.Options>
  </Combobox.Dropdown>
</Combobox>
```

---

## Nothing found message

```tsx
<Combobox.Dropdown>
  <Combobox.Options>
    {filteredOptions.length > 0
      ? filteredOptions
      : <Combobox.Empty>Nothing found for "{search}"</Combobox.Empty>}
  </Combobox.Options>
</Combobox.Dropdown>
```

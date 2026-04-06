# UI Patterns for DHIS2 Apps

Always use `@dhis2/ui` for UI components, or build custom components using the primitives provided by the library. Don't use generic libraries (MUI, Chakra, Ant Design) unless the user specifically requests it. DHIS2 apps run inside the platform shell alongside other
apps, and `@dhis2/ui` implements the DHIS2 design system so everything looks consistent.
The library is included automatically when you scaffold a DHIS2 app.

Import components directly:

```tsx
import {
    Button,
    Input,
    SingleSelect,
    SingleSelectOption,
    Modal,
    ModalTitle,
    ModalContent,
    ModalActions,
    CircularLoader,
    NoticeBox,
    DataTable,
    DataTableHead,
    DataTableBody,
    DataTableRow,
    DataTableCell,
    DataTableColumnHeader,
} from '@dhis2/ui'
```

The library has more components than you'd expect — `Transfer`, `SelectorBar`,
`OrganisationUnitTree`, `Pagination`, `Tag`, `Chip`, `Tooltip`, `Popover`, `Menu`,
`Tab`, `SplitButton`, `DropdownButton`, and many more. Check before building a custom one.

To see every available component, read `node_modules/@dhis2/ui/build/es/index.js` — it
re-exports everything the library provides. Always import from `@dhis2/ui` directly.

## Custom styling

Use CSS Modules (`.module.css`) with DHIS2 CSS variables for colors, spacing, and elevation:

```css
.container {
    padding: var(--spacers-dp16);
    background: var(--colors-white);
}

.header {
    margin-block-end: var(--spacers-dp12);
    color: var(--colors-grey900);
}
```

## Forms

Use **React Hook Form** for form state management, **Zod** for schema validation, and
wire `@dhis2/ui` inputs via `Controller` (they use `onChange({ value })` not `onChange(event)`,
so they need the Controller wrapper rather than `register`). `SingleSelectField` uses
`selected` / `onChange({ selected })` instead.

For the full example and detailed key points, read `references/ui-patterns/forms.md`.

## Tables

Use **TanStack Table** (`@tanstack/react-table`) for column definitions, sorting, filtering,
and pagination state — then render with `@dhis2/ui` `DataTable` components for the visual
layer. Important: `@dhis2/ui` `Pagination` is 1-indexed while TanStack Table is 0-indexed,
so offset by 1 when bridging them. Data is fetched at the parent level and passed as props.

For the full example and detailed key points, read `references/ui-patterns/tables.md`.

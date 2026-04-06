# Tables

Use **TanStack Table** (`@tanstack/react-table`) for column definitions, sorting, filtering,
and pagination state — then render with `@dhis2/ui` `DataTable` components for the visual
layer. Data is fetched at the parent level and passed in as props (see
`references/data-fetching.md`).

```tsx
import { useState } from 'react';
import {
    DataTable,
    DataTableHead,
    DataTableBody,
    DataTableFoot,
    DataTableRow,
    DataTableCell,
    DataTableColumnHeader,
    Pagination,
} from '@dhis2/ui';
import i18n from '@dhis2/d2-i18n';
import {
    createColumnHelper,
    flexRender,
    getCoreRowModel,
    getSortedRowModel,
    getFilteredRowModel,
    getPaginationRowModel,
    useReactTable,
    Column,
    SortingState,
} from '@tanstack/react-table';

type DataElement = {
    id: string;
    name: string;
    shortName: string;
    valueType: string;
    lastUpdated: string;
};

const columnHelper = createColumnHelper<DataElement>();

const columns = [
    columnHelper.accessor('name', {
        header: i18n.t('Name'),
        filterFn: 'includesString',
    }),
    columnHelper.accessor('shortName', {
        header: i18n.t('Short name'),
    }),
    columnHelper.accessor('valueType', {
        header: i18n.t('Value type'),
        enableSorting: false,
    }),
    columnHelper.accessor('lastUpdated', {
        header: i18n.t('Last updated'),
        cell: (info) => new Date(info.getValue()).toLocaleDateString(),
    }),
];

const getSortDirection = (column: Column<DataElement>) => {
    return column.getIsSorted() || 'default';
};

type DataElementTableProps = {
    dataElements: DataElement[];
    search?: string;
};

const DataElementTable = ({ dataElements, search }: DataElementTableProps) => {
    const [sorting, setSorting] = useState<SortingState>([
        { id: 'lastUpdated', desc: true },
    ]);
    const [pagination, setPagination] = useState({ pageIndex: 0, pageSize: 10 });

    const table = useReactTable({
        data: dataElements,
        columns,
        state: {
            sorting,
            pagination,
            columnFilters: search ? [{ id: 'name', value: search }] : [],
        },
        getRowId: (row) => row.id,
        getCoreRowModel: getCoreRowModel(),
        getSortedRowModel: getSortedRowModel(),
        getFilteredRowModel: getFilteredRowModel(),
        getPaginationRowModel: getPaginationRowModel(),
        onSortingChange: setSorting,
    });

    const rows = table.getRowModel().rows;

    return (
        <DataTable>
            <DataTableHead>
                {table.getHeaderGroups().map((headerGroup) => (
                    <DataTableRow key={headerGroup.id}>
                        {headerGroup.headers.map((header) => (
                            <DataTableColumnHeader
                                key={header.id}
                                fixed
                                {...(header.column.getCanSort()
                                    ? {
                                          sortDirection: getSortDirection(header.column),
                                          sortIconTitle: i18n.t('Sort'),
                                          onSortIconClick: () => header.column.toggleSorting(),
                                      }
                                    : {})}
                            >
                                {flexRender(header.column.columnDef.header, header.getContext())}
                            </DataTableColumnHeader>
                        ))}
                    </DataTableRow>
                ))}
            </DataTableHead>

            <DataTableBody>
                {rows.length > 0 ? (
                    rows.map((row) => (
                        <DataTableRow key={row.id}>
                            {row.getVisibleCells().map((cell) => (
                                <DataTableCell key={cell.id}>
                                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                                </DataTableCell>
                            ))}
                        </DataTableRow>
                    ))
                ) : (
                    <DataTableRow>
                        <DataTableCell colSpan={String(columns.length)} align="center">
                            {i18n.t('No data elements found')}
                        </DataTableCell>
                    </DataTableRow>
                )}
            </DataTableBody>

            <DataTableFoot>
                <DataTableRow>
                    <DataTableCell colSpan={String(columns.length)}>
                        <Pagination
                            page={pagination.pageIndex + 1}
                            pageSize={pagination.pageSize}
                            pageCount={table.getPageCount()}
                            total={table.getRowCount()}
                            isLastPage={!table.getCanNextPage()}
                            onPageChange={(page: number) =>
                                setPagination((prev) => ({ ...prev, pageIndex: page - 1 }))
                            }
                            onPageSizeChange={(size: number) =>
                                setPagination({ pageIndex: 0, pageSize: size })
                            }
                        />
                    </DataTableCell>
                </DataTableRow>
            </DataTableFoot>
        </DataTable>
    );
};
```

## Key points

- **Column definitions live outside the component** — `createColumnHelper<T>()` gives type-safe accessors. Use `columnHelper.accessor` for data columns and `columnHelper.display` for non-data columns (selection checkboxes, action menus).
- **Sorting** — spread sort props conditionally onto `DataTableColumnHeader` only when `header.column.getCanSort()` is true. Disable sorting on specific columns with `enableSorting: false`. Default sort should be descending on date columns.
- **Filtering** — pass `columnFilters` in state as an array of `{ id, value }` objects. Define `filterFn` on columns that need filtering (e.g. `'includesString'` for text search, `'equals'` for exact match).
- **Pagination** — `@dhis2/ui` `Pagination` is 1-indexed while TanStack Table is 0-indexed, so offset by 1 when bridging them. Wrap pagination inside `DataTableFoot > DataTableRow > DataTableCell` with `colSpan`.
- **Empty state** — render a single row with `colSpan` covering all columns when there are no rows.
- **Custom cells** — use the `cell` property on a column definition to render links, status pills, tooltips, or action menus. Access the full row object via `info.row.original`.
- **Row selection** — add a `display` column with `Checkbox` and enable `enableRowSelection: true` + `onRowSelectionChange` on the table. Use `table.getSelectedRowModel().rows` to get selected items for batch actions.
- **Actions column** — use a `display` column rendering a `FlyoutMenu` with `MenuItem` entries for row-level actions (edit, delete, etc.).

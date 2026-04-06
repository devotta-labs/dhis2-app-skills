# Forms

Use **React Hook Form** for form state management and **Zod** for schema validation.
Mutations go through TanStack Query with `useDataEngine` (see `references/data-fetching.md`
for the mutation pattern). Wire `@dhis2/ui` inputs to React Hook Form via `Controller`
since they don't expose a standard `ref`.

```tsx
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button, ButtonStrip, InputField, SingleSelectField, SingleSelectOption } from '@dhis2/ui';
import i18n from '@dhis2/d2-i18n';

const schema = z.object({
    name: z.string().min(1, { message: i18n.t('Name is required') }),
    shortName: z.string().min(1, { message: i18n.t('Short name is required') }),
    valueType: z.string().min(1, { message: i18n.t('Select a value type') }),
});

type FormValues = z.infer<typeof schema>;

type DataElementFormProps = {
    onSubmit: (values: FormValues) => void;
    isPending?: boolean;
};

const DataElementForm = ({ onSubmit, isPending }: DataElementFormProps) => {
    const {
        control,
        handleSubmit,
    } = useForm<FormValues>({
        resolver: zodResolver(schema),
        defaultValues: { name: '', shortName: '', valueType: '' },
    });

    return (
        <form onSubmit={handleSubmit(onSubmit)}>
            <Controller
                name="name"
                control={control}
                render={({ field, fieldState }) => (
                    <InputField
                        {...field}
                        label={i18n.t('Name')}
                        onChange={({ value }) => field.onChange(value)}
                        error={!!fieldState.error}
                        validationText={fieldState.error?.message}
                    />
                )}
            />
            <Controller
                name="shortName"
                control={control}
                render={({ field, fieldState }) => (
                    <InputField
                        {...field}
                        label={i18n.t('Short name')}
                        onChange={({ value }) => field.onChange(value)}
                        error={!!fieldState.error}
                        validationText={fieldState.error?.message}
                    />
                )}
            />
            <Controller
                name="valueType"
                control={control}
                render={({ field, fieldState }) => (
                    <SingleSelectField
                        label={i18n.t('Value type')}
                        selected={field.value}
                        onChange={({ selected }) => field.onChange(selected)}
                        error={!!fieldState.error}
                        validationText={fieldState.error?.message}
                    >
                        <SingleSelectOption label="Text" value="TEXT" />
                        <SingleSelectOption label="Number" value="NUMBER" />
                        <SingleSelectOption label="Boolean" value="BOOLEAN" />
                    </SingleSelectField>
                )}
            />
            <ButtonStrip>
                <Button type="submit" primary loading={isPending} disabled={isPending}>
                    {i18n.t('Save')}
                </Button>
            </ButtonStrip>
        </form>
    );
};
```

## Key points

- Spread `{...field}` then override `onChange` — `@dhis2/ui` uses `onChange({ value })`, not `onChange(event)`. `SingleSelectField` uses `selected` / `onChange({ selected })` instead.
- Use `fieldState` from Controller render props for error display.
- `InputField` bundles label + validation text. For simple modal forms, `Input` + `Label` with manual error display works too.
- Use `FormProvider` + `useFormContext` when a form spans multiple components.
- Pass `loading` and `disabled` to submit buttons from mutation pending state.

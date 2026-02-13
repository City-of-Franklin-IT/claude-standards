## Component File Structure
- Components are organized in PascalCase directories with the following files:
```
ComponentName/
  index.tsx          # Main component (function declaration + default export)
  components.tsx     # Sub-components (arrow functions, named exports)
  hooks.ts           # Custom hooks, may also export types
  utils.ts           # Utility/helper functions, may also export types
```
- Not all files are required -- only include what the component needs
- Types are typically exported from `hooks.ts` or `utils.ts` rather than a separate `types.ts` file

## Folder Organization
- Features are organized under `src/components/` by domain with common subdirectories:
```
src/components/
  feature-name/
    containers/        # Most features will have this
    forms/             # When the feature has forms
      create/
      update/
    tables/            # When the feature has tables
    context.tsx        # When feature-level state management is needed
```
- Not all features require every subdirectory -- use only what the feature needs
- `containers/` is the most common and typically present in every feature
- `context.tsx` is optional and only added when the feature requires its own context + reducer
- Pages are always organized under `src/pages/`:
```
src/pages/
  PageName/
    index.tsx
    hooks.ts
```

## Components
- For components exported from index.tsx files use function definition with default export:
```
function Foo = () => {

  return (
    <></>
  )
}

export default Foo
```
- For child components exported from components.tsx files use arrow functions:
```
const Foo = () => {

  return (
    <></>
  )
}
```
- Components should be a minimum of 5 lines:
```
const Foo = () => {

  return (
    <></>
  )
}
```
not this:
```
const Foo = () => {
  return (
    <></>
  )
}
```
- Hooks called at top of component body
- Early returns should be null
```
const Foo = ({ visible }: { visible: boolean }) => {
  if(!visible) return null

  return (
    <></>
  )
}
```

## Tailwind
- Responsive modifiers (e.g. `lg:static`, `md:flex`) should be placed at the end of the className string:
```
<div className="flex flex-col mt-10 lg:flex-row lg:mt-0"></div>
```

## State Management
- useState uses [state, setState] convention by default and state is always an object:
```
const [state, setState] = useState<{ foo: boolean }>({ foo: false })
```
- Server state: TanStack React Query (`@tanstack/react-query`)
  - Query hooks use object syntax with `queryKey`, `queryFn`, `enabled`, `staleTime`:
  ```
  export const useGetFoo = () => {
    const { enabled, token } = useEnableQuery()

    return useQuery({
      queryKey: ['getFoo'],
      queryFn: () => AppActions.getFoo(authHeaders(token)),
      enabled: enabled && !!token,
      staleTime: Infinity
    })
  }
  ```
- UI/page state: React Context + `useReducer`
  - Each feature defines its own context in `context.tsx`
  - Context type includes `dispatch` plus state fields
  - State type derived from context type: `Omit<CtxType, "dispatch">`
  - Actions use a discriminated union with SCREAMING_SNAKE_CASE `type` values
  - Context exported as default, Provider as named export
  ```tsx
  type FooCtx = {
    dispatch: React.Dispatch<FooAction>
    searchValue: string
    currentPage: number
    fooUUID: string
  }

  type FooState = Omit<FooCtx, "dispatch">

  type FooAction =
    | { type: "SET_FOO_UUID", payload: string }
    | { type: "SET_SEARCH_VALUE", payload: string }
    | { type: "SET_CURRENT_PAGE", payload: number }
    | { type: "RESET_CTX" }

  const initialState: FooState = {
    searchValue: "",
    currentPage: 1,
    fooUUID: ""
  }

  const FooCtx = createContext<FooCtx>({
    ...initialState,
    dispatch: () => null
  })

  const FooReducer = (state: FooState, action: FooAction) => {
    switch(action.type) {
      case "SET_FOO_UUID":
        return { ...state, fooUUID: action.payload }
      case "RESET_CTX":
        return initialState
      default:
        return state
    }
  }

  export const FooProvider = ({ children }: { children: React.ReactNode }) => {
    const [state, dispatch] = useReducer(FooReducer, initialState)
    return (
      <FooCtx.Provider value={{ ...state, dispatch }}>
        {children}
      </FooCtx.Provider>
    )
  }

  export default FooCtx
  ```

## Forms
- Forms use React Hook Form (`react-hook-form`) with `FormProvider` + `useFormContext`
- Forms are organized under `forms/create/` and `forms/update/` directories
- Reusable form elements live in `src/components/form-elements/` (e.g., `FormContainer`, `FormLabel`, `FormError`, `RequiredIcon`, `FormBtns`)
- Nested form data uses `_dirtied` and `_deleted` flags to track modifications:
```ts
interface FooCreateInterface {
  name: string
  items: {
    value: string
    _dirtied?: boolean | null
    _deleted?: boolean | null
  }[]
}
```
- Full form pattern:
```tsx
// index.tsx
function CreateFooForm() {

  const { methods, handleFormSubmit, onCancelBtnClick } = useHandleCreateFoo()

  return (
    <FormProvider { ...methods }>
      <form onSubmit={methods.handleSubmit(handleFormSubmit)}>
        <Components.Header>Create Foo</Components.Header>
        <div className="flex flex-col gap-6">
          <Components.FooInput />
          <FormBtns onCancelBtnClick={onCancelBtnClick} />
        </div>
      </form>
    </FormProvider>
  )
}

export default CreateFooForm
```
```tsx
// components.tsx (sub-components use useFormContext to access form state)
export const FooInput = () => {

  const { register, formState: { errors } } = useFormContext<AppTypes.FooCreateInterface>()

  return (
     <div className="flex-1 flex flex-col gap-2">
      <FormLabel
        name={'foo'}
        required={true}>
          Foo:
      </FormLabel>
      <input
        type="text"
        className="input w-full"
        { ...register('foo', {
          required: "Foo is required",
          maxLength: {
            value: 50,
            message: "Foo must be 50 characters or less"
          },
          onChange: () => setValue('_dirtied', true)
        }) } />
      <FormError error={errors.foo?.message} />
    </div>
  )
}
```

## Error Handling
- Pages are wrapped with `ErrorBoundary` (`react-error-boundary`) and `HandleLoading`:
```tsx
function FooPage() {

  const { data, isSuccess } = useGetAllFoo()

  return (
    <Layout>
      <ErrorBoundary href={"/"}>
        <HandleLoading isSuccess={isSuccess}>
          <FooProvider>
            <FooContainer foo={data?.data || []} />
          </FooProvider>
        </HandleLoading>
      </ErrorBoundary>
    </Layout>
  )
}

export default FooPage
```
- `HandleLoading` shows a `<Loading />` spinner until data is ready:
```tsx
function HandleLoading(props: HandleLoadingProps) {

  if(!props.isSuccess) return <Loading />

  return <>{props.children}</>
}
```
- Toast notifications via `react-toastify` for user feedback:
  - `savedPopup(msg)` -- success messages
  - `errorPopup(msg)` -- error messages
  - `authPopup(msg)` -- authentication messages
  - `infoPopup(msg)` -- informational messages
- `<ToastContainer />` rendered in `App.tsx`

## Imports
- For React applications, import React module first (only when React is referenced in the file)
- Order the remaining external libraries imports by router library, query library, and form library, then remaning external libraries
- Import app utilities next with more globally relevant imports first and imports that are more tightly coupled to the component the last imports
```
import { useState } from 'react'
import { useLocation } from 'react-router'
import { useQuery } from '@tanstack/react-query'
import { useFooHook } from '@/utils/hooks.ts'
import { fooFunction } from './utils.ts'
```
- Import types next with globally used types first and types coupled to components next (and always use // Types heading)
```
// Types
import * as AppTypes from '@/context/App/AppTypes.ts'
import { FooType } from './types.ts'
```
- When importing components from components.tsx use Components namespace
```
import * as Components from './components.tsx'
```

## TypeScript Standards

### Code Style
- No semicolons
- Double quotes for all strings and imports
- Spaces inside template literal braces: `${ variable }` not `${variable}`
- No space before `if` parentheses, no spaces inside: `if(!condition) {`
- Blank line after function/component opening brace before logic

### TypeScript Configuration
- Path aliases defined in both `tsconfig.json` and resolved via `vite-tsconfig-paths`:
```json
{
  "@/*": ["src/*"],
  "@/components/*": ["src/components/*"],
  "@/config/*": ["src/config/*"],
  "@/context/*": ["src/context/*"],
  "@/helpers/*": ["src/helpers/*"],
  "@/pages/*": ["src/pages/*"],
  "@/utils/*": ["src/utils/*"],
  "@/assets/*": ["src/assets/*"]
}
```

### Type Definitions
- Global types defined in `src/context/App/AppTypes.ts`
- Import as namespace: `import * as AppTypes from "@/context/App/AppTypes"`
- Interfaces use PascalCase with `Interface` suffix: `SupportInterface`, `PersonnelCreateInterface`
- Type unions use PascalCase with `Type` suffix: `SupportType`, `ShiftType`
- Base entity interface:
```
interface BaseIterface {
  uuid: string
  createdBy: string
  createdAt: string
  updatedBy: string
  updatedAt: string
}
```
- Base server response:
```
interface ServerResponse {
  success: boolean
  msg?: string
} & { data: T }
```
- Create interfaces:  
```
Omit<EntityInterface, "uuid" | "createdBy" | "createdAt" | "updatedBy" | "updatedAt">{
  uuid?: string
}
``` 

### API Functions
- Defined in `src/context/App/AppActions.ts`
- Import as namespace: `import * as AppActions from "@/context/App/AppActions"`
- Accept form data and a `Headers` object as parameters (if headers are required)
- Return typed `Promise<ServerResponse & { data: T }>`
- Use native `fetch()` -- no axios
- JSDoc comments document HTTP method and endpoint:
```ts
/**
* Get foo
*
* GET /api/v1/foo/bar
**/
export const getFoo = async (headers: Headers): Promise<AppTypes.ServerResponse & { data: AppTypes.FooInterface[] }> => {
  const res = await fetch(`${ baseUrl }/bar`, { headers })
  return await res.json()
}
```
- POST/PUT requests append Content-Type header: `headers.append("Content-Type", "application/json")`

### Naming Conventions
| Category | Convention | Example |
|---|---|---|
| Components | PascalCase | `SupportContainer`, `CreateSupportForm` |
| Hooks | camelCase with `use` prefix | `useGetToken`, `useHandleForm` |
| Utils/helpers | camelCase | `authHeaders`, `formatDate` |
| Interfaces | PascalCase + `Interface` | `SupportInterface` |
| Type unions | PascalCase + `Type` | `SupportType` |
| Context | PascalCase + `Ctx` | `SupportCtx`, `MissionsCtx` |
| Provider | PascalCase + `Provider` | `SupportProvider` |
| Reducer | PascalCase + `Reducer` | `SupportReducer` |
| Reducer actions | SCREAMING_SNAKE_CASE | `SET_SEARCH_VALUE` |
| Config constants | SCREAMING_SNAKE_CASE | `APP_BASE`, `NODE_ENV` |
| API functions | camelCase, verb-first | `getAllSupport`, `createSupport` |
| Icon assets | kebab-case directories | `icons/sad-face/sad-face.svg` |

### Props
- Props should be written inline rather than each on a new line
- Inline type annotations preferred for simple props: `({ visible }: { visible: boolean })`

### Other
- Strict mode enabled (noUnusedLocals, noUnusedParameters)
- Ternary operations should use multiline formatting with `?` on the condition line:
    ```ts
    const value = condition ?
      trueValue :
      falseValue
    ```

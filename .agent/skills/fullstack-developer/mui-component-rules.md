# Material UI — Component-First Rules

> **Core Principle:** Before building ANY UI component from scratch, check if Material UI (MUI) already provides it. Only build custom when MUI genuinely cannot satisfy the requirement.

---

## ⚠️ MANDATORY — MUI Lookup Before Implementation

This checklist MUST be executed **before writing any component code**, including inside the TDD implementation loop (Step 2 of the main SKILL.md).

### Pre-Build Checklist

- [ ] **Step A — Identify the UI element:** What is the test expecting? (Button, Table, Dialog, TextField, Select, etc.)
- [ ] **Step B — Search MUI:** Check if a matching component exists in the MUI library:
  - Core components: `@mui/material` (Button, TextField, Select, Dialog, Table, etc.)
  - Data grid: `@mui/x-data-grid`
  - Date/time pickers: `@mui/x-date-pickers`
  - Icons: `@mui/icons-material`
- [ ] **Step C — Decision:**
  - ✅ **MUI component exists →** Use it. Wrap it if you need project-specific defaults (see Wrapping Pattern below).
  - ⚠️ **MUI component exists but needs heavy modification →** Wrap the MUI component and extend via `sx` prop, `styled()`, or theme overrides.
  - ❌ **No MUI equivalent →** Build custom. Document why in a code comment.

---

## MUI Component Catalog — Quick Reference

Before coding, consult this map of common UI needs to MUI components:

| UI Need | MUI Component | Import |
|---|---|---|
| Button (any variant) | `Button` | `@mui/material/Button` |
| Text input | `TextField` | `@mui/material/TextField` |
| Dropdown/select | `Select`, `Autocomplete` | `@mui/material/Select` |
| Checkbox | `Checkbox` | `@mui/material/Checkbox` |
| Radio | `RadioGroup`, `Radio` | `@mui/material/RadioGroup` |
| Toggle/switch | `Switch` | `@mui/material/Switch` |
| Dialog/modal | `Dialog` | `@mui/material/Dialog` |
| Data table | `DataGrid` | `@mui/x-data-grid` |
| Simple table | `Table`, `TableHead`, `TableBody`, `TableRow`, `TableCell` | `@mui/material/Table` |
| Tabs | `Tabs`, `Tab` | `@mui/material/Tabs` |
| Navigation drawer | `Drawer` | `@mui/material/Drawer` |
| App bar / Header | `AppBar`, `Toolbar` | `@mui/material/AppBar` |
| Card | `Card`, `CardContent`, `CardActions` | `@mui/material/Card` |
| Snackbar / Toast | `Snackbar`, `Alert` | `@mui/material/Snackbar` |
| Loading spinner | `CircularProgress`, `LinearProgress` | `@mui/material/CircularProgress` |
| Tooltip | `Tooltip` | `@mui/material/Tooltip` |
| Breadcrumbs | `Breadcrumbs` | `@mui/material/Breadcrumbs` |
| Stepper | `Stepper` | `@mui/material/Stepper` |
| Chip / Tag | `Chip` | `@mui/material/Chip` |
| Accordion | `Accordion` | `@mui/material/Accordion` |
| Avatar | `Avatar` | `@mui/material/Avatar` |
| Menu / Context menu | `Menu`, `MenuItem` | `@mui/material/Menu` |
| Pagination | `Pagination` | `@mui/material/Pagination` |
| Skeleton loader | `Skeleton` | `@mui/material/Skeleton` |
| Icon | Any icon from `@mui/icons-material` | `@mui/icons-material/IconName` |

> If the component you need is **not** in this table, check the full MUI docs: https://mui.com/material-ui/all-components/

---

## Wrapping Pattern (Project-Specific Defaults)

When using MUI components, **wrap** them in a project-level component to enforce consistent defaults. This keeps project standards centralized.

### Example — Wrapped Button

```tsx
// src/components/app-button/AppButton.tsx
import Button, { type ButtonProps } from '@mui/material/Button'

interface AppButtonProps extends ButtonProps {
  // Add any project-specific props here
}

function AppButton({ variant = 'contained', ...rest }: AppButtonProps) {
  return <Button variant={variant} {...rest} />
}

export default AppButton
```

### Why Wrap?

| Reason | Explanation |
|---|---|
| Consistent defaults | All buttons share the same default `variant`, `color`, `size` |
| Single point of change | Updating a default across the whole app = 1 file change |
| Design handoff adherence | Map `design_handoff.md` tokens to MUI theme/props in one place |
| Test compatibility | Wrapped components can forward `data-testid` and other test attributes |

---

## Theming — Use MUI Theme, Not Ad-Hoc Styles

When `design_handoff.md` defines colors, typography, or spacing:

1. **Map those values into a MUI theme** (`createTheme` / `ThemeProvider`).
2. Use the `sx` prop or `styled()` API referencing theme tokens — do NOT hardcode hex values inline.

```tsx
// src/theme.ts
import { createTheme } from '@mui/material/styles'

const theme = createTheme({
  palette: {
    primary: { main: '#007BFF' },
    error: { main: '#DC3545' },
  },
  typography: {
    fontFamily: 'Inter, sans-serif',
  },
  spacing: 8, // 8px grid from design_handoff.md
})

export default theme
```

---

## Extension Pattern — When MUI Needs Customization

If MUI covers 80% of the need but you need to extend:

1. **Prefer `sx` prop** for one-off overrides.
2. **Use `styled()`** for reusable styled variants.
3. **Use theme `components` overrides** for global defaults.

```tsx
// Global override via theme
const theme = createTheme({
  components: {
    MuiButton: {
      defaultProps: { variant: 'contained', disableElevation: true },
      styleOverrides: {
        root: { borderRadius: 8, textTransform: 'none' },
      },
    },
  },
})
```

---

## Decision Tree Summary

```
Need a UI component?
│
├─ Does MUI have it? ──── YES ──→ Use MUI component directly (or wrap it)
│
├─ MUI has something close? ──→ Wrap + customize with sx / styled / theme
│
└─ MUI has nothing? ──→ Build custom. Add comment: // Custom: MUI has no equivalent for [X]
```

---

## Rules Enforcement

| Rule | Consequence of Violation |
|---|---|
| Build a custom `<Button>` when `@mui/material/Button` exists | ❌ Rejected. Use MUI. |
| Build a custom `<DataTable>` when `@mui/x-data-grid` exists | ❌ Rejected. Use MUI. |
| Use MUI component without wrapping in project component | ⚠️ Warning. Wrap for consistent defaults. |
| Hardcode hex colors inline instead of using theme | ❌ Rejected. Use theme tokens. |
| Skip Step A-C before building any component | ❌ Violation. Pre-Build Checklist is mandatory. |

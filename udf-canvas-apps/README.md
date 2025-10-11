## Canvas App User-defined Function Snippets

User-defined function (UDF) snippets for Microsoft Power Apps Canvas Apps by [Jan Chlebek](https://www.linkedin.com/in/jan-chlebek/).

These lightweight helpers live in the `source` folder as individual `.fx.yaml` files so you can:
1. Inspect the formula easily in source control
2. Copy/paste into a Power Apps Canvas App (App object > Formulas, or a dedicated PowerFX function)
3. Keep your app formulas DRY and readable

### Contents

| Function | Purpose | Return Type | Quick Example |
|----------|---------|-------------|---------------|
| `fmIsStudioMode` | Detect if the app is currently open in Power Apps Studio (edit mode) | Boolean | `If(fmIsStudioMode, Notify("Dev only"))` |
| `udfGetEnvironmentVariable(SchemaName)` | Retrieve an Environment Variable value (overridden value or default) by schema name | Text | `udfGetEnvironmentVariable("crd_BaseApiUrl")` |
| `udfGetInitials(DisplayName)` | Extract the first letter of up to the first two words of a display name | Text | `udfGetInitials(User().FullName)` |
| `udfSafeDivision(Numerator, Denominator)` | Divide safely, returning 0 on divide-by-zero/error | Number | `udfSafeDivision(Value(txtA.Text), Value(txtB.Text))` |

---

## 1. `fmIsStudioMode`

```powerfx
fmIsStudioMode = StartsWith(Host.Version, "PowerApps-Studio")
```

### Use Cases
* Hide dev-only controls
* Short-circuit API calls while editing
* Show layout / debug helpers

### Example
```powerfx
Visible: fmIsStudioMode
```

---

## 2. `udfGetEnvironmentVariable`

```powerfx
udfGetEnvironmentVariable('Schema Name': Text):Text = IfError(Coalesce(
    LookUp('Environment Variable Values', 'Schema Name' = 'Schema Name', Value),
    LookUp('Environment Variable Definitions', 'Schema Name' = 'Schema Name', 'Default Value')
), Blank());
```

### Parameters
* `Schema Name` (Text) – The schema (logical) name of the Environment Variable (e.g. `crd_BaseApiUrl`).

### Logic
1. Attempts to find an overridden value in `Environment Variable Values`.
2. Falls back to the default in `Environment Variable Definitions`.
3. Returns blank if neither exists.

### Example
```powerfx
Set(baseApiUrl, udfGetEnvironmentVariable("crd_BaseApiUrl"));
If(IsBlank(baseApiUrl), Notify("Missing environment variable BaseApiUrl", NotificationType.Error));
```

> Ensure Dataverse tables are connected: `Environment Variable Definitions` and `Environment Variable Values`.

---

## 3. `udfGetInitials`

```powerfx
udfGetInitials('Display Name': Text): Text =
	Concat(FirstN(Split('Display Name', " "), 2), Left(Value, 1))
```

### Behavior
* Splits on a single space
* Takes up to the first 2 tokens
* Grabs the first character of each
* Concatenates without separator (e.g. `"John Paul Doe" -> "JP"`)

### Examples
```powerfx
udfGetInitials("Jane Doe")          // "JD"
udfGetInitials("Prince")            // "P"
udfGetInitials(Upper(User().FullName)) // STILL works; casing preserved from source
```

> If you need punctuation trimming or multi‑space handling, wrap the input with `Trim` or enhance the splitter.

---

## 4. `udfSafeDivision`

```powerfx
udfSafeDivision(Numerator: Number, Denominator: Number): Number =
	IfError(Numerator / Denominator, 0)
```

### Why
Power Fx raises an error on division by zero or invalid inputs. Wrapping with `IfError` avoids scattered error handling.

### Example
```powerfx
udfSafeDivision(Sum(colSales, Amount), CountRows(colSales))
```

### Variation (Optional)
If you prefer returning `Blank()` instead of 0:
```powerfx
IfError(Numerator / Denominator, Blank())
```

---

## Import / Usage Instructions

1. Open your Canvas App in Power Apps Studio.
1. Paste source code files into `App.Forumlas` object
1. Call the function anywhere just like a built‑in formula.

---

Happy building! If these help you, a ⭐ on the repo is appreciated.


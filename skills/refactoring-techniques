---
name: refactoring-techniques
description: Use when code has long methods, complex or unreadable expressions, switch/if-else chains on type, repeating parameter groups, magic numbers, deeply nested conditionals, or primitive obsession (strings/ints representing domain concepts). Quick reference for seven core refactoring patterns with C# and TypeScript examples.
---

# Refactoring Techniques

Seven battle-tested refactoring patterns from [refactoring.guru](https://refactoring.guru/refactoring). Each section: problem signal, recipe, before/after in C# and TypeScript, and pitfalls.

## When to Use

```dot
digraph pick_technique {
    "Code smell detected" [shape=doublecircle];
    "Complex or unreadable expression?" [shape=diamond];
    "Extract Variable" [shape=box];
    "Long method or comment-separated blocks?" [shape=diamond];
    "Extract Method" [shape=box];
    "Switch/if-else on type?" [shape=diamond];
    "Replace Conditional with Polymorphism" [shape=box];
    "Same params repeated across methods?" [shape=diamond];
    "Introduce Parameter Object" [shape=box];
    "Bare numeric literals?" [shape=diamond];
    "Replace Magic Numbers" [shape=box];
    "Deeply nested conditionals?" [shape=diamond];
    "Guard Clauses" [shape=box];
    "String/int representing a domain concept?" [shape=diamond];
    "Replace Type Code with Class" [shape=box];
    "None of these" [shape=ellipse];

    "Code smell detected" -> "Complex or unreadable expression?";
    "Complex or unreadable expression?" -> "Extract Variable" [label="yes"];
    "Complex or unreadable expression?" -> "Long method or comment-separated blocks?" [label="no"];
    "Long method or comment-separated blocks?" -> "Extract Method" [label="yes"];
    "Long method or comment-separated blocks?" -> "Switch/if-else on type?" [label="no"];
    "Switch/if-else on type?" -> "Replace Conditional with Polymorphism" [label="yes"];
    "Switch/if-else on type?" -> "Same params repeated across methods?" [label="no"];
    "Same params repeated across methods?" -> "Introduce Parameter Object" [label="yes"];
    "Same params repeated across methods?" -> "Bare numeric literals?" [label="no"];
    "Bare numeric literals?" -> "Replace Magic Numbers" [label="yes"];
    "Bare numeric literals?" -> "Deeply nested conditionals?" [label="no"];
    "Deeply nested conditionals?" -> "Guard Clauses" [label="yes"];
    "Deeply nested conditionals?" -> "String/int representing a domain concept?" [label="no"];
    "String/int representing a domain concept?" -> "Replace Type Code with Class" [label="yes"];
    "String/int representing a domain concept?" -> "None of these" [label="no"];
}
```

## Quick Reference

| Technique | Signal | Result |
|---|---|---|
| Extract Variable | Complex expression, long boolean chain, unreadable one-liner | Named intermediate variables that document intent |
| Extract Method | Long method, comment-separated blocks | Small focused methods named by intent |
| Replace Conditional with Polymorphism | switch/if-else branching on type | Subclass per variant, shared interface |
| Introduce Parameter Object | Same 3+ params repeated across methods | Single object grouping related params |
| Replace Magic Numbers | Bare `9.81`, `0.1`, `86400` in code | Named constants with clear intent |
| Guard Clauses | Nested if/else pyramid | Flat early-returns for edge cases |
| Replace Type Code with Class | Strings/ints representing domain concepts (status, role, category) | Type-safe class with validation and no invalid states |

---

## 1. Extract Variable

**Signal:** A complex expression — long boolean chain, compound calculation, or dense one-liner — that requires mental parsing to understand. Often the reader has to trace sub-expressions to figure out what the code is checking or computing.

**Recipe:**
1. Identify a sub-expression whose meaning isn't immediately obvious
2. Declare a new variable with a name that describes the **intent** of that sub-expression
3. Replace the sub-expression with the variable
4. Repeat for remaining complex parts
5. The final expression should read like natural language

### C#

```csharp
// BEFORE — compound boolean requires tracing each clause
public bool IsEligibleForDiscount(Customer customer, Order order)
{
    return customer.MembershipLevel != "none"
        && customer.AccountAgeDays > 365
        && (order.Items.Sum(i => i.Price * i.Quantity) > 200m
            || (customer.TotalPurchases > 1000m
                && order.Items.Any(i => i.Category == "premium")))
        && !customer.HasOutstandingBalance;
}

// AFTER — each business rule named by intent
public bool IsEligibleForDiscount(Customer customer, Order order)
{
    var isActiveMember = customer.MembershipLevel != "none";
    var isEstablishedCustomer = customer.AccountAgeDays > 365;
    var hasNoOutstandingBalance = !customer.HasOutstandingBalance;

    var orderTotal = order.Items.Sum(i => i.Price * i.Quantity);
    var isLargeOrder = orderTotal > 200m;
    var isLoyalPremiumBuyer =
        customer.TotalPurchases > 1000m && order.Items.Any(i => i.Category == "premium");
    var meetsOrderThreshold = isLargeOrder || isLoyalPremiumBuyer;

    return isActiveMember && isEstablishedCustomer && meetsOrderThreshold && hasNoOutstandingBalance;
}
```

### TypeScript

```typescript
// BEFORE — dense template literal with inline logic
function formatTransactionSummary(txn: Transaction): string {
  return `${txn.date.getFullYear()}-${String(txn.date.getMonth() + 1).padStart(2, "0")}-${String(txn.date.getDate()).padStart(2, "0")} | ${txn.amount < 0 ? "DEBIT" : "CREDIT"} | ${Math.abs(txn.amount).toFixed(2)} | ${txn.description.length > 30 ? txn.description.substring(0, 27) + "..." : txn.description}`;
}

// AFTER — each column extracted to a named variable
function formatTransactionSummary(txn: Transaction): string {
  const year = txn.date.getFullYear();
  const month = String(txn.date.getMonth() + 1).padStart(2, "0");
  const day = String(txn.date.getDate()).padStart(2, "0");
  const date = `${year}-${month}-${day}`;

  const direction = txn.amount < 0 ? "DEBIT" : "CREDIT";
  const amount = Math.abs(txn.amount).toFixed(2);
  const description =
    txn.description.length > 30
      ? txn.description.substring(0, 27) + "..."
      : txn.description;

  return `${date} | ${direction} | ${amount} | ${description}`;
}
```

**When NOT to apply:** If the expression is already short and readable, extracting a variable adds noise. `var x = a + b;` doesn't need `var sum = a + b; var x = sum;`. Also consider **Extract Method** instead if the sub-expression is reused across multiple methods or represents a cohesive concept worth its own function.

---

## 2. Extract Method

**Signal:** A method does multiple things, often separated by comments like `// validate`, `// calculate`, `// save`.

**Recipe:**
1. Create a new method named after its **intent** (what, not how)
2. Move the code fragment into it
3. Variables declared inside the fragment become locals; variables from outer scope become parameters
4. If a local is modified and needed afterward, return it
5. Replace the original fragment with a call

### C#

```csharp
// BEFORE
public decimal ProcessOrder(Order order)
{
    // Validate
    if (order.Items is not { Count: > 0 })
        throw new ArgumentException("Order must have items");

    // Calculate total
    var subtotal = order.Items.Sum(i => i.Price * i.Quantity);
    var tax = subtotal * 0.1m;
    var shipping = subtotal > 100 ? 0m : 10m;
    return subtotal + tax + shipping;
}

// AFTER
public decimal ProcessOrder(Order order)
{
    ValidateOrder(order);
    return CalculateTotal(order);
}

private static void ValidateOrder(Order order)
{
    if (order.Items is not { Count: > 0 })
        throw new ArgumentException("Order must have items");
}

private static decimal CalculateTotal(Order order)
{
    var subtotal = order.Items.Sum(i => i.Price * i.Quantity);
    var tax = subtotal * 0.1m;
    var shipping = subtotal > 100 ? 0m : 10m;
    return subtotal + tax + shipping;
}
```

### TypeScript

```typescript
// BEFORE
function processOrder(order: Order): number {
  // Validate
  if (!order.items?.length) {
    throw new Error("Order must have items");
  }

  // Calculate total
  const subtotal = order.items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  const tax = subtotal * 0.1;
  const shipping = subtotal > 100 ? 0 : 10;
  return subtotal + tax + shipping;
}

// AFTER
function processOrder(order: Order): number {
  validateOrder(order);
  return calculateTotal(order);
}

function validateOrder(order: Order): void {
  if (!order.items?.length) {
    throw new Error("Order must have items");
  }
}

function calculateTotal(order: Order): number {
  const subtotal = order.items.reduce((sum, i) => sum + i.price * i.quantity, 0);
  const tax = subtotal * 0.1;
  const shipping = subtotal > 100 ? 0 : 10;
  return subtotal + tax + shipping;
}
```

**When NOT to apply:** If the method is already short and clear, extracting creates indirection without value. Don't extract a single line into its own method unless it's reused elsewhere.

---

## 3. Replace Conditional with Polymorphism

**Signal:** A switch/if-else chain branches on object type or a type discriminator field, and the same branching appears in multiple places.

**Recipe:**
1. Create a base class or interface with the method that varies
2. Create a subclass for each branch
3. Move each branch's logic into the corresponding subclass
4. Replace the conditional with a polymorphic method call
5. Delete the empty conditional

### C#

```csharp
// BEFORE
public decimal CalculateShipping(Order order)
{
    return order.ShippingType switch
    {
        "standard" => order.Weight * 1.5m,
        "express" => order.Weight * 3.0m + 5.0m,
        "overnight" => order.Weight * 5.0m + 15.0m,
        _ => throw new ArgumentException($"Unknown shipping: {order.ShippingType}")
    };
}

// AFTER
public interface IShippingStrategy
{
    decimal Calculate(decimal weight);
}

public class StandardShipping : IShippingStrategy
{
    public decimal Calculate(decimal weight) => weight * 1.5m;
}

public class ExpressShipping : IShippingStrategy
{
    public decimal Calculate(decimal weight) => weight * 3.0m + 5.0m;
}

public class OvernightShipping : IShippingStrategy
{
    public decimal Calculate(decimal weight) => weight * 5.0m + 15.0m;
}

// Usage via DI or factory
var cost = shippingStrategy.Calculate(order.Weight);
```

### TypeScript

```typescript
// BEFORE
function calculateShipping(order: Order): number {
  switch (order.shippingType) {
    case "standard":
      return order.weight * 1.5;
    case "express":
      return order.weight * 3.0 + 5.0;
    case "overnight":
      return order.weight * 5.0 + 15.0;
    default:
      throw new Error(`Unknown shipping: ${order.shippingType}`);
  }
}

// AFTER
interface ShippingStrategy {
  calculate(weight: number): number;
}

const shippingStrategies: Record<string, ShippingStrategy> = {
  standard: { calculate: (w) => w * 1.5 },
  express: { calculate: (w) => w * 3.0 + 5.0 },
  overnight: { calculate: (w) => w * 5.0 + 15.0 },
};

function calculateShipping(order: Order): number {
  const strategy = shippingStrategies[order.shippingType];
  if (!strategy) throw new Error(`Unknown shipping: ${order.shippingType}`);
  return strategy.calculate(order.weight);
}
```

**When NOT to apply:** If the conditional appears in only one place, is simple (2-3 short branches), and the types are unlikely to grow, polymorphism adds complexity without benefit. A simple switch expression may be clearer.

---

## 4. Introduce Parameter Object

**Signal:** The same group of 3+ parameters appears together across multiple method signatures.

**Recipe:**
1. Create a new class/record/interface for the parameter group
2. Add it as a parameter alongside the existing ones
3. Remove old parameters one at a time, replacing with object field access
4. Consider moving related validation or behavior into the new object

### C#

```csharp
// BEFORE — date range params repeated across 3 methods
public Report Generate(string customer, DateTime startDate, DateTime endDate, string format) { ... }
private ReportData Fetch(string customer, DateTime startDate, DateTime endDate) { ... }
private void Validate(string customer, DateTime startDate, DateTime endDate) { ... }

// AFTER
public sealed record DateRange(DateTime Start, DateTime End)
{
    public void Validate()
    {
        if (Start > End)
            throw new ArgumentException("Start must precede End");
    }
}

public Report Generate(string customer, DateRange period, string format)
{
    period.Validate();
    var data = Fetch(customer, period);
    return Format(data, format);
}

private ReportData Fetch(string customer, DateRange period) { ... }
```

### TypeScript

```typescript
// BEFORE — date range params repeated
function generate(customer: string, startDate: Date, endDate: Date, format: string): Report { ... }
function fetch(customer: string, startDate: Date, endDate: Date): ReportData { ... }
function validate(customer: string, startDate: Date, endDate: Date): void { ... }

// AFTER
interface DateRange {
  readonly start: Date;
  readonly end: Date;
}

function validateDateRange(range: DateRange): void {
  if (range.start > range.end) {
    throw new Error("Start must precede End");
  }
}

function generate(customer: string, period: DateRange, format: string): Report {
  validateDateRange(period);
  const data = fetch(customer, period);
  return formatReport(data, format);
}

function fetch(customer: string, period: DateRange): ReportData { ... }
```

**When NOT to apply:** If the parameters only appear together in one method, grouping them adds a type without reducing duplication. Also beware creating a "Data Class" (object with only data and no behavior) — consider whether validation or logic belongs in the new type.

---

## 5. Replace Magic Numbers with Symbolic Constants

**Signal:** Bare numeric literals like `9.81`, `0.1`, `86400`, `1024` appear in calculations without explanation of what they represent.

**Recipe:**
1. Declare a named constant with a descriptive name
2. Find all occurrences of the magic number
3. For each occurrence, verify it represents the same concept before replacing
4. Replace with the constant

### C#

```csharp
// BEFORE
public decimal CalculateTotal(decimal subtotal)
{
    var tax = subtotal * 0.1m;
    var shipping = subtotal > 100m ? 0m : 10m;
    return subtotal + tax + shipping;
}

// AFTER
private const decimal TaxRate = 0.1m;
private const decimal FreeShippingThreshold = 100m;
private const decimal StandardShippingFee = 10m;

public decimal CalculateTotal(decimal subtotal)
{
    var tax = subtotal * TaxRate;
    var shipping = subtotal > FreeShippingThreshold ? 0m : StandardShippingFee;
    return subtotal + tax + shipping;
}
```

### TypeScript

```typescript
// BEFORE
function calculateTotal(subtotal: number): number {
  const tax = subtotal * 0.1;
  const shipping = subtotal > 100 ? 0 : 10;
  return subtotal + tax + shipping;
}

// AFTER
const TAX_RATE = 0.1;
const FREE_SHIPPING_THRESHOLD = 100;
const STANDARD_SHIPPING_FEE = 10;

function calculateTotal(subtotal: number): number {
  const tax = subtotal * TAX_RATE;
  const shipping = subtotal > FREE_SHIPPING_THRESHOLD ? 0 : STANDARD_SHIPPING_FEE;
  return subtotal + tax + shipping;
}
```

**When NOT to apply:** Universally understood values like `0`, `1`, `-1` as loop bounds or identity values don't need constants. `for (let i = 0; i < items.length; i++)` is already clear. The test: would a reader have to pause and wonder "what does this number mean?"

---

## 6. Guard Clauses

**Signal:** Deeply nested if/else forming an arrow or pyramid shape. The "happy path" is buried inside multiple indentation levels.

**Recipe:**
1. Identify edge cases and special conditions
2. Convert each into an early return (or throw) at the top of the method
3. The main logic follows naturally at the base indentation level
4. If multiple guards produce the same result, consider consolidating them

### C#

```csharp
// BEFORE
public decimal GetPayAmount(Employee employee)
{
    decimal result;
    if (employee.IsSeparated)
    {
        result = SeparatedAmount(employee);
    }
    else
    {
        if (employee.IsRetired)
        {
            result = RetiredAmount(employee);
        }
        else
        {
            result = NormalPayAmount(employee);
        }
    }
    return result;
}

// AFTER
public decimal GetPayAmount(Employee employee)
{
    if (employee.IsSeparated) return SeparatedAmount(employee);
    if (employee.IsRetired) return RetiredAmount(employee);

    return NormalPayAmount(employee);
}
```

### TypeScript

```typescript
// BEFORE
function getPayAmount(employee: Employee): number {
  let result: number;
  if (employee.isSeparated) {
    result = separatedAmount(employee);
  } else {
    if (employee.isRetired) {
      result = retiredAmount(employee);
    } else {
      result = normalPayAmount(employee);
    }
  }
  return result;
}

// AFTER
function getPayAmount(employee: Employee): number {
  if (employee.isSeparated) return separatedAmount(employee);
  if (employee.isRetired) return retiredAmount(employee);

  return normalPayAmount(employee);
}
```

**When NOT to apply:** If the conditional branches are truly symmetric (equal weight, no "special case"), guard clauses can obscure the symmetry. In those cases, a standard if/else or pattern match may communicate intent better.

---

## 7. Replace Type Code with Class

**Signal:** A string or integer field represents a domain concept (status, role, category, priority) — passed around as a primitive, validated with `if (role == "admin")` checks scattered across the codebase. No compile-time safety prevents invalid values like `"admni"`.

**Recipe:**
1. Create a new class representing the type concept
2. Add static factory methods or instances for each valid value
3. Replace the primitive field with the new type in the owning class
4. Update all references to use the type class instead of raw strings/ints
5. Remove old constants

### C#

```csharp
// BEFORE — string type code, no compile-time safety
public class User
{
    public string Role { get; set; } // "admin", "editor", "viewer"
}

// Scattered validation
if (user.Role == "admin") { /* ... */ }
if (user.Role == "editor" || user.Role == "admin") { /* ... */ }

// AFTER — type-safe class, invalid values impossible
public sealed class UserRole : IEquatable<UserRole>
{
    public static readonly UserRole Admin = new("admin");
    public static readonly UserRole Editor = new("editor");
    public static readonly UserRole Viewer = new("viewer");

    private static readonly Dictionary<string, UserRole> All =
        new[] { Admin, Editor, Viewer }.ToDictionary(r => r.Value);

    public string Value { get; }

    private UserRole(string value) => Value = value;

    public static UserRole FromString(string value) =>
        All.TryGetValue(value, out var role)
            ? role
            : throw new ArgumentException($"Unknown role: {value}");

    public bool CanEdit => this == Admin || this == Editor;

    public bool Equals(UserRole? other) => Value == other?.Value;
    public override bool Equals(object? obj) => Equals(obj as UserRole);
    public override int GetHashCode() => Value.GetHashCode();
    public override string ToString() => Value;
}

public class User
{
    public UserRole Role { get; set; } = UserRole.Viewer;
}

// Usage — type-safe, discoverable
if (user.Role == UserRole.Admin) { /* ... */ }
if (user.Role.CanEdit) { /* ... */ }
```

### TypeScript

```typescript
// BEFORE — string literal, no safety beyond type alias
type Role = string;

interface User {
  role: Role; // "admin", "editor", "viewer"
}

if (user.role === "admin") { /* ... */ }

// AFTER — class with restricted construction
class UserRole {
  static readonly Admin = new UserRole("admin");
  static readonly Editor = new UserRole("editor");
  static readonly Viewer = new UserRole("viewer");

  private static readonly all = new Map<string, UserRole>([
    ["admin", UserRole.Admin],
    ["editor", UserRole.Editor],
    ["viewer", UserRole.Viewer],
  ]);

  private constructor(public readonly value: string) {}

  static fromString(value: string): UserRole {
    const role = UserRole.all.get(value);
    if (!role) throw new Error(`Unknown role: ${value}`);
    return role;
  }

  get canEdit(): boolean {
    return this === UserRole.Admin || this === UserRole.Editor;
  }

  equals(other: UserRole): boolean {
    return this.value === other.value;
  }

  toString(): string {
    return this.value;
  }
}

interface User {
  role: UserRole;
}

// Usage
if (user.role === UserRole.Admin) { /* ... */ }
if (user.role.canEdit) { /* ... */ }
```

**When NOT to apply:** If the type code drives conditional behavior (switch/if-else that does different things per type), use **Replace Conditional with Polymorphism** or **Replace Type Code with Subclasses** instead. This technique is for codes that are purely data — they classify but don't alter behavior. Also skip if the domain has only 2 values that are unlikely to grow (e.g., `true`/`false` is just a boolean).

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Extracting variables for already-clear expressions (`var sum = a + b`) | Only extract when the sub-expression's meaning isn't immediately obvious |
| Jumping to Extract Method when Extract Variable would suffice | Use Extract Variable first for readability within one method; escalate to Extract Method only when the logic is reused or cohesive enough for its own function |
| Extracting methods that are too granular (single-line wraps) | Only extract when the fragment has a distinct **intent** worth naming |
| Using polymorphism for a 2-branch conditional that won't grow | Keep simple conditionals simple; refactor when a third branch appears |
| Creating parameter objects with no behavior (pure Data Class) | Move validation and related logic into the parameter object |
| Replacing ALL numbers including obvious ones (0, 1, loop bounds) | Only replace numbers whose meaning isn't immediately obvious |
| Applying guard clauses to symmetric conditions | Guards are for edge cases; symmetric branches deserve if/else or match |
| Using type class when the code drives branching behavior | Use Polymorphism or Subclasses instead; type class is for classification only |
| Over-engineering: applying multiple techniques at once | Refactor one smell at a time; build, test, then move to the next |

## References

- [refactoring.guru/refactoring](https://refactoring.guru/refactoring) — Full catalog of refactoring techniques
- [Refactoring (Martin Fowler)](https://refactoring.com/) — Definitive reference

# path-scan

`path-scan` is a lightweight procedural macro library for parsing paths (e.g., URLs)
with static variable binding and compile-time checks.

## Overview

This crate provides two main macros:

- **`path_scan!`**
  Returns a closure that, when given an input string, sequentially applies the defined
  patterns and returns the result of the first matching arm.

- **`path_scan_val!`**
  Immediately matches an input string against the provided patterns and returns the result
  directly.

Both macros allow you to define patterns with placeholders (`:identifier` or `{identifier}`)
that capture parts of the input string into variables. Multiple patterns can be combined
using the `|` separator. An optional `if` condition may follow the pattern(s) (e.g.,
`"pattern" if condition => expression`), and the arm only matches if the condition is true.

The default arm is specified as `_ => expression`.

**Note:**
If an if-less default arm is not provided—that is, if a default arm without an `if` condition is missing—
the macros return `None` on failure, making the overall expression type `Option<T>`.

## Examples

### Using `path_scan!` Macro

```rust
use path_scan::path_scan;

// With a default arm, the closure returns a concrete type.
let scanner = path_scan! {
    // Matches "blog/anything/index" and binds `slug`
    "blog/:slug/index" => format!("blog: {}", slug),
    // Matches "other/anything" if the captured `slug` has exactly 5 characters
    "other/{slug}" if slug.len() == 5 => format!("short blog: {}", slug),
    // Default arm
    _ => format!("default")
};

assert_eq!(scanner("blog/hello/index"), "blog: hello");
assert_eq!(scanner("other/short"), "short blog: short");
assert_eq!(scanner("unknown/path"), "default");
```

### Using `path_scan!` Without a Default Arm

```rust
use path_scan::path_scan;

// Without a default arm, the closure returns `Option<T>`.
let scanner = path_scan! {
    "product/:id" => format!("product id: {}", id)
};

assert_eq!(scanner("product/123"), Some("product id: 123".to_string()));
assert_eq!(scanner("other/path"), None);
```

### Using `path_scan_val!` Macro (Comma Form)

```rust
use path_scan::path_scan_val;

let result = path_scan_val!("user/john/profile",
    "user/{name}/profile" => format!("User: {}", name),
    _ => format!("unknown user")
);
assert_eq!(result, "User: john");
```

### Using `path_scan_val!` Macro (Brace Form) with an `if` Condition

```rust
use path_scan::path_scan_val;

let result = path_scan_val!("admin/jane/dashboard" {
    "admin/:name/dashboard" if name.starts_with("j") => format!("Admin: {}", name),
    _ => format!("not an admin")
});
assert_eq!(result, "Admin: jane");
```

### Using `path_scan_val!` Without a Default Arm (returns `Option<T>`)

```rust
use path_scan::path_scan_val;

let result = path_scan_val!("order/98765",
    "order/{id}" => format!("Order ID: {}", id)
);
assert_eq!(result, Some(format!("Order ID: 98765")));

let result_none = path_scan_val!("unknown/path",
    "order/{id}" => format!("Order ID: {}", id)
);
assert_eq!(result_none, None);
```

## License

This project is licensed under the [MIT license](LICENSE).
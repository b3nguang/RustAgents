# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Rust Coding Guidelines

- Prioritize code correctness and clarity. Speed and efficiency are secondary priorities unless otherwise specified.
- Do not write organizational or comments that summarize the code. Comments should only be written in order to explain "why" the code is written in some way in the case there is a reason that is tricky / non-obvious.
- Prefer implementing functionality in existing files unless it is a new logical component. Avoid creating many small files.
- Avoid using functions that panic like `unwrap()`, instead use mechanisms like `?` to propagate errors.
- Be careful with operations like indexing which may panic if the indexes are out of bounds.
- Never silently discard errors with `let _ =` on fallible operations. Always handle errors appropriately:
    - Propagate errors with `?` when the calling function should handle them
    - Use `.log_err()` or similar when you need to ignore errors but want visibility
    - Use explicit error handling with `match` or `if let Err(...)` when you need custom logic
    - Example: avoid `let _ = client.request(...).await?;` - use `client.request(...).await?;` instead
- When implementing async operations that may fail, ensure errors propagate to the UI layer so users get meaningful feedback.
- When using `format!` (and similar macros like `println!`, `write!`, `tracing::info!`), inline variables into `{}` whenever possible instead of positional arguments. Example: prefer `format!("hello {name}")` over `format!("hello {}", name)`.
- Collapse nested `if` statements when possible (see clippy `collapsible_if`).
- Prefer method references over closures when possible (see clippy `redundant_closure_for_method_calls`). Example: `.map(String::from)` over `.map(|s| String::from(s))`.
- Avoid bool or ambiguous `Option` parameters that force callers to write hard-to-read callsites such as `foo(false)` or `bar(None)`. Prefer enums, named methods, or newtypes that keep the callsite self-documenting.
- Prefer exhaustive `match` statements; avoid wildcard `_` arms so that new variants force a compile-time review.
- Newly added traits should include doc comments that explain their role and how implementations are expected to use them.
- Prefer private modules with an explicitly exported public crate API over making everything `pub`.
- Do not create small helper methods that are referenced only once — inline them at the call site instead.
- Never create files with `mod.rs` paths - prefer `src/some_module.rs` instead of `src/some_module/mod.rs`.
- When creating new crates, prefer specifying the library root path in `Cargo.toml` using `[lib] path = "...rs"` instead of the default `lib.rs`, to maintain consistent and descriptive naming (e.g., `gpui.rs` or `main.rs`).
- Avoid creative additions unless explicitly requested
- Use full words for variable names (no abbreviations like "q" for "queue")
- Use variable shadowing to scope clones in async contexts for clarity, minimizing the lifetime of borrowed references.
  Example:
    ```rust
    executor.spawn({
        let task_ran = task_ran.clone();
        async move {
            *task_ran.borrow_mut() = true;
        }
    });
    ```

## Module Size & Structure

- Prefer adding new modules instead of growing existing ones.
- Target Rust modules under ~500 LoC (excluding tests).
- If a file exceeds roughly 800 LoC, add new functionality in a new module instead of extending the existing file, unless there is a strong documented reason not to.
- When extracting code from a large module, move the related tests and module/type docs along with the implementation so invariants stay close to the code that owns them.
- Resist bloating any single "core"/catch-all crate. Before adding code there, consider whether an existing crate fits better, or whether it is time to introduce a new crate to the workspace and refactor accordingly.

## Build & Formatting Commands

- Always run `cargo fmt` and `cargo sort -w` after editing code
- Always run `cargo build` after completing all tasks
- Be patient with Rust commands (e.g. `cargo build`, `cargo test`, `cargo clippy`). Rust's build lock can make execution slow — this is expected. Never kill them by PID just because they seem stuck.

## Testing

- Prefer deep equality comparisons: `assert_eq!` on entire objects rather than field-by-field checks.
- Use `pretty_assertions::assert_eq` in tests for clearer diffs. Import it at the top of the test module if it isn't already.
- Avoid mutating process environment (`std::env::set_var`, etc.) inside tests; prefer passing environment-derived flags or dependencies in from above to keep tests isolated and parallel-safe.

## Changelog

- Update `CHANGELOG.md` after every meaningful change (new features, bug fixes, breaking changes, deprecations, removals)
- Follow the [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) format
- Add entries under the `[Unreleased]` section
- Keep each changelog entry to around 100 characters (soft limit)

## Documentation

- Update the mdBook docs under `docs/book/src/` when adding or changing user-facing features, configuration options, CLI behavior, etc.
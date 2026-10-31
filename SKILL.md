---
name: neotype
description: Design in neotype (.nt)—column-0 class pseudocode with no tabs or indentation—BEFORE implementing, then transpile it to executable JavaScript or export it as a multi-page printable PDF. Always use when the user asks to build, implement, program, create, or add a non-trivial JavaScript app, module, class, feature, system, or refactor, or mentions "neotype", ".nt", "pseudocode", "class design", "before coding", "paper design", or "print the design", even if they do not explicitly request a preliminary design. Also use to convert .nt designs to JSON/JSONL datasets. Do not use for one-line fixes, theoretical questions, or non-JavaScript code.
---

# neotype

Before writing code, create the design in neotype, get user approval, and only then implement **from that file**. The `.nt` file is the source of truth: it can be read on screen, printed (portrait A4, multiple pages if needed), and executed.

The core idea: there is no indentation, so the design grows downward instead of to the right. This makes it easy to write by hand on paper and keeps the PDF layout consistent.

## Workflow

1. **Design.** Write `design.nt` with one class per block. Unresolved methods can be executable stubs, for example `- calculate: (x) => null`.
2. **Validate.** Run `python3 scripts/neotype.py validate design.nt`. It prints `file:line: error: ...`; fix issues and repeat until it reports `✓`. In addition to formatting, it runs `node --check` on the generated JS.
3. **Show and wait for approval.** Include the `.nt` content in the response (code block), summarize the design decisions that need confirmation in two lines, and **stop**. If the user wants a paper copy, or the design exceeds about 40 lines, also generate the PDF: `python3 scripts/neotype.py pdf design.nt` (add `--letter` for Letter paper) and provide it.
4. **Implement from the file.** Complete the method bodies in `.nt`, validate, and generate with `python3 scripts/neotype.py js design.nt`. To execute, run `python3 scripts/neotype.py run design.nt`, which calls `Main.main()` if present (`- static main: () => {...}`). `build` performs validation + `.js` + `.pdf` generation.
5. **Deliver** both `.nt` and `.js`. If changes are needed, edit `.nt` and regenerate; never edit the generated `.js` by hand. This keeps the approved design and implementation in sync. Writing the `.nt` file *after* the code just to satisfy the workflow is not acceptable.

The design step may be skipped for trivial changes (renaming, fixing a one-line bug). If unsure whether a task is small, design it: 30 lines of planning can prevent a rewrite.

## Format

Everything starts in column 0. **No tabs, no leading spaces, maximum 80 columns.**

```
- Dog
- extends: Animal
- legs: 4
- constructor: (name) => {
this.name = name;
}
- bark: (times) => {
for (let i = 0; i < times; i++) {
console.log("woof");
}
}
- static create: (n) => new Dog(n)

- Cat
- extends: Animal
- meow: () => "meow"
```

Rules:

1. `- ` (hyphen + space) in column 0 starts a **property**.
2. **A blank line followed by `- ` starts a new class.** (Two consecutive `\n` characters followed by `- `.) The first line of each block contains only the class name: `- ClassName`.
3. Any other line **continues** the previous property. This is how long bodies are written without indentation.
4. `- extends: Other` declares inheritance. `- key: value` is a field, unless the value is a JS function (`(a) => ...`, `x => ...`, `async (a) => ...`, `function(a) {...}`), in which case it is a method. Put `static` before the key.
5. A blank line inside a body does not end the class as long as the next line **does not** start with `- `.

Main pitfall: a continuation line can never start with `- ` (it would be parsed as a new property). Put the operator at the end of the previous line (`x -` / `1`) or write `-x` without a space. See `references/format.md` for details and more edge cases.

## Commands

| Command | Description |
|---|---|
| `validate F` | Checks formatting (tabs, indentation, 80-column limit, names, duplicates, cycles, braces) and JS syntax with Node.js. |
| `js F [-o OUT]` | Generates JS classes, orders parents before children, and exports with `module.exports`. |
| `run F` | Transpiles and runs with Node.js (`Main.main()`). |
| `pdf F [-o OUT] [--letter]` | Monospaced portrait A4 PDF, multiple pages; a class that fits on a page is never split. |
| `json F [--jsonl]` | Exports `{name, extends, fields, methods}` per class for datasets. |
| `build F` | Runs validation and generates `.js` + `.pdf`. |

All commands use the form `python3 scripts/neotype.py <command> <file.nt>`.

## Notes

- If `validate` reports errors, fix the `.nt` file and validate again; do not patch the generated `.js`.
- Without Node.js, `validate` skips the JS syntax check and prints a warning, but `run` will not work.
- PDF output uses DejaVu Sans Mono (or another system monospace font); Courier only guarantees Latin-1 support.

# neotype

Indentation-free class pseudocode (`.nt`) for designing before implementation. Write it by hand on paper, print it as a multi-page PDF, let both people and language models read it (it looks like a Markdown list), transpile it into executable JavaScript, or export it to JSON/JSONL datasets.

Includes a **Claude skill** that requires designing in `.nt` and waiting for approval *before* writing code, then implementing from that same file.

## The idea

On paper, indentation consumes horizontal space: every level pushes text closer to the right edge. In neotype, everything stays in column 0, so the design grows downward instead.

Rules:

1. `- ` (hyphen + space) in column 0 starts a **property**.
2. **A blank line followed by `- ` starts a new class.** The first line of each block is the class name only.
3. Any other line **continues** the previous property (this is how long bodies are written without indentation).
4. No tabs, no leading spaces, maximum 80 columns.

```
- Animal
- name: "unnamed"
- constructor: (name) => {
this.name = name;
}
- speak: () => "..."

- Dog
- extends: Animal
- speak: () => "woof"
- static create: (n) => new Dog(n)
```

- `- extends: Other` declares inheritance.
- `- key: value` is a field, unless the value is a JS function (`(a) => ...`, `x => ...`, `async (a) => ...`, `function(a) {...}`), in which case it is a method.
- `static` goes before the key.
- A blank line inside a body does not end the class, as long as the next line does not start with `- `.

Grammar and edge cases: [`references/format.md`](references/format.md).

## Requirements

- Python 3.8+
- `reportlab` for PDF export (`pip install reportlab`)
- Node.js (optional): required for `run` and for JavaScript syntax checks during `validate`

## Usage

```bash
python3 scripts/neotype.py validate examples/example.nt
python3 scripts/neotype.py js       examples/example.nt        # -> example.js
python3 scripts/neotype.py run      examples/example.nt        # runs Main.main()
python3 scripts/neotype.py pdf      examples/example.nt        # portrait A4, multiple pages
python3 scripts/neotype.py json     examples/example.nt --jsonl
python3 scripts/neotype.py build    examples/example.nt        # validate + .js + .pdf
```

| Command | Description |
|---|---|
| `validate` | Checks formatting (tabs, indentation, 80-column limit, names, duplicates, inheritance cycles, braces) and JS syntax with `node --check`. Errors are reported as `file:line: error: ...`. |
| `js` | Generates JS classes, orders parents before children, and exports with `module.exports`. |
| `run` | Transpiles and runs with Node.js, calling `Main.main()` if it exists. |
| `pdf` | Monospaced portrait A4 PDF (`--letter` for Letter size). A class that fits on one page is never split. |
| `json` | One entry per class: `{name, extends, fields, methods}`; use `--jsonl` for one entry per line. |
| `build` | Runs `validate` and generates `.js` and `.pdf` files. |

## Install as a Claude skill

The skill is the `neotype/` directory (`SKILL.md` defines when it activates).

- **Claude.ai:** Upload the `neotype.skill` file (or the folder ZIP) from the Skills section in settings.
- **Claude Code:** Copy the folder to `~/.claude/skills/neotype/` (global) or `.claude/skills/neotype/` (project-local).

With the skill enabled, when asked to implement a non-trivial JavaScript feature, Claude writes `design.nt`, validates it, shows it to you, and waits for approval. Only then does it generate code from that file.

## Tests

```bash
python3 tests/test_neotype.py
```

## Project structure

```
neotype/
├── SKILL.md                 # skill instructions and activation triggers
├── README.md
├── LICENSE                  # MIT
├── scripts/neotype.py       # parser, validator, transpiler, JSON, PDF, CLI
├── references/format.md     # grammar and edge cases
├── examples/example.nt
└── tests/test_neotype.py
```

## Known limitations

- JavaScript is the only target language.
- A continuation line can never start with `- ` (it would be parsed as a property). Put the operator at the end of the previous line or write `-x` without a space.
- The bracket checker does not understand regex literals containing `(`, `[` or quotes.
- There is no second nesting level: the format is deliberately flat.

## License

[MIT](LICENSE)

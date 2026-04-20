# UZON Conformance Test Suite

Conformance tests for [UZON](https://uzon.dev) parsers, evaluators, and type-checkers.

Targets **UZON Specification v0.10**. See the [spec](https://github.com/uzon-dev/spec) for the full language definition.

## Conformance levels

Per spec §11.1, implementations declare one of four levels:

| Level            | Requirements                                                                |
| ---------------- | --------------------------------------------------------------------------- |
| **Parser**       | MUST parse all syntactically valid documents per §9.                        |
| **Evaluator**    | MUST evaluate all expressions per §5. MUST resolve identifiers and `env`.   |
| **Type-checker** | MUST validate `as` annotations, `called`/`as` type reuse, `to` conversions. |
| **Full**         | Parser + Evaluator + Type-checker.                                          |

Conforming implementations **MUST** be deterministic: the same document with the same `env` values **MUST** always produce the same result.

## Structure

```
conformance/
├── parse/
│   ├── valid/              131 files — one feature per file
│   │   ├── cross/           89 files — multi-feature interactions
│   │   └── starship/         3 files — full integration (diamond imports, all keywords, std sample)
│   └── invalid/            127 files — one error condition per file
│       ├── cross/           60 files — errors from feature interactions
│       └── starship/        43 files — realistic subtle errors
├── eval/                   142 input/expected pairs
└── roundtrip/             3000 files — parse → serialize → parse identity
```

### `parse/valid/`

Documents that conformant implementations **MUST** accept. Each file tests one spec feature.

- **`cross/`** — Tests interactions between two or more spec sections (e.g., `with` + `or else`, `case type` + branch narrowing). These are the most valuable tests: implementations are most likely to diverge at feature boundaries.
- **`starship/`** — A multi-file integration test (`shared.uzon`, `crew.uzon`, `starship.uzon`) exercising all 35 keywords (including 1 reserved) and 17 of 24 `std` functions with diamond imports, struct overrides, extensions, tagged unions, functions, and chained `std` operations.

### `parse/invalid/`

Documents that conformant implementations **MUST** reject. Each file tests one error condition.

- **`cross/`** — Errors arising from feature interactions (e.g., circular dependency through `plus`, `undefined` propagation into interpolation via functions).
- **`starship/`** — Realistic documents with exactly one subtle error each: type mismatches through conversion chains, nominal identity conflicts, indirect recursion through `std.map`, `null` passing through `or else` into concatenation, and similar edge cases.

Per spec §11.2, rejections **MUST** include a location (file, line, column). Error priority: syntax → circular dependency → type → runtime.

### `eval/`

Documents with expected evaluation results. Each test consists of a pair:

- `<name>.uzon` — input document
- `<name>.expected.uzon` — the fully evaluated result (all expressions resolved, all references inlined)

Implementations evaluate the input and compare the output against the expected file.

### `roundtrip/`

Documents for testing parse-serialize-parse identity. An implementation should be able to parse each file, serialize the AST back to UZON, and parse the result again to produce an identical AST.

## Test file conventions

Each test file begins with a comment identifying the spec section(s) under test:

```uzon
// §3.2.1 Struct override (with)
```

```uzon
// §5.9 + §5.3 + §5.4 + §5.11 + §4.4.1 + §5.7
// if → arithmetic → comparison → to conversion → interpolation → or else
```

Invalid test files include an `ERROR:` prefix:

```uzon
// ERROR: §3.8 + §5.16.2
// indirect recursion: function references itself through std.map
```

## License

[MIT](LICENSE)

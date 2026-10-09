# Changelog

## 0.2.2 — large expressions no longer hit the recursion limit

### Fixed

- **Expressions with a few hundred terms were rejected.** The parser recurses once
  per nesting level and once per right-hand operand, and the AST helpers recurse
  over the tree, so CPython's default recursion limit of 1000 put the practical
  ceiling at roughly 300 terms or 100 levels of nesting: a 500-term sum, a
  300-level nesting and a 2,000-term quantified claim all came back as a bare
  `RecursionError` ("maximum recursion depth exceeded"). `rigor/__init__.py` now
  raises the limit to 20,000 on import — pure-Python frames in CPython 3.11+ live
  on the heap, so this costs memory rather than stack safety. Measured after the
  change: a 6,000-term sum and 1,000 levels of nesting parse, and the 3,000-term
  simplify that used to fail completes in under a second.
- **The remaining failure mode is now actionable.** Anything beyond that still
  cannot be parsed, but the server turns `RecursionError` into an
  `ExpressionTooLarge` result that names the practical limit and suggests a
  factored or closed form, instead of leaking the interpreter's message.

### Tests

- `tests/test_parse.py` parses a 1,000-term and a 3,000-term sum, a 1,000-level
  nesting, and a 2,000-term quantified claim.
- `tests/test_mcp_stdio.py` simplifies a 2,000-term expression over the real
  stdio transport, handles a 300-level nesting, and asserts that an absurd
  60,000-term expression produces the sized `ExpressionTooLarge` error rather
  than a recursion message.

## 0.2.1 — corrected build

This repository is the corrected build of the `math-rigor` plugin
(`bauerelizabeth07139/math-rigor`), renamed to match its package name.
Nothing about the tool surface changed: the same 23 MCP tools, the same
`math-rigor` skill, the same two commands.

### Fixed

- **`find_counterexample` ignored a domain declared in the quantifier.**
  Freeing the leading `forall` binders also dropped the sorts they carried, so
  the negation was handed to the SMT solver with the variables sorted by
  inference alone. A claim about the integers was therefore decided over the
  reals, and `forall n in Z: n^2 >= n` — true for every integer — came back
  `refuted` with the witness `n = 1/2`. The domains now travel with the freed
  variables into the SMT search, the explicit-range scan and the sampler, so
  `find_counterexample` agrees with `verify_forall` on the same statement.
- **`number_theory` rejected JSON numbers.** `numbers`, `residues` and `moduli`
  were declared as arrays of strings, so the natural call
  `number_theory("gcd", numbers=[48, 18])` failed with a raw pydantic
  validation error instead of computing `6`. Integers are now accepted
  alongside strings.

### Tests

- `tests/test_verify.py` covers the binder-only domains (integer, natural,
  real), asserts the reported sort is `int`, and asserts a genuine integer
  failure still yields an integral witness.
- `tests/test_mcp_stdio.py` covers both defects through the real MCP stdio
  transport, and the previously wrong expectation
  (`find_counterexample("forall n in Z: n^2 >= n") == "refuted"`) is corrected
  to `proven`.

## 0.2.0 and earlier

See the git history of `bauerelizabeth07139/math-rigor`.

# Changelog

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

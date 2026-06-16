# Traceability Matrix: <feature / release>

> Owner: **Quality Engineer** · Gate 4 (Test → Review) isn't met until every requirement / AC
> here is traced to ≥1 test **and** a source location. For projects with a formal spec/FS this is
> CI-enforceable: fail the build if any requirement row has no linked test or source ref.

## Requirement → test → source

| Requirement / AC | Test(s) | Source ref | Status |
|------------------|---------|------------|--------|
| PRD R1 / AC1 ("shall …") | `tests/…::case` | `src/…:fn` | ☐ traced |

- **Requirement / AC:** every "shall" / acceptance criterion from the PRD (and formal spec, if any).
- **Test(s):** the test(s) that verify it — at least one.
- **Source ref:** the file/symbol that implements it.
- **Status:** ☐ until both a test and a source ref are present.

## Untraced register
Any requirement with no linked test or source — the gate is **not met** while this is non-empty.

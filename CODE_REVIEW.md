# Warmup

- [ ] Jupyter Notebook tutorial: https://github.com/stefan-jansen/zipline-reloaded/blob/main/docs/notebooks/tutorial.ipynb
- [ ] Quickstart: https://github.com/quantsummaries/zipline-reloaded
- [ ] getting started guide: https://zipline.ml4trading.io/beginner-tutorial
- [ ] zipline/examples: https://github.com/stefan-jansen/zipline-reloaded/tree/main/src/zipline/examples

# Package dependencies for code review

## Level 0 — no zipline dependencies

1. - [ ] `zipline.country`
2. - [ ] `zipline.currency`
3. - [ ] `zipline.extensions`
4. - [ ] `zipline.protocol`
5. - [ ] `zipline.zipline_warnings`
6. - [ ] `zipline.api`

## Level 1 — depends on Level 0

7. - [ ] `zipline.errors` → `zipline.utils`
8. - [ ] `zipline.assets` → `zipline.errors`, `zipline.utils`
9. - [ ] `zipline.lib` → `zipline.errors`, `zipline.utils`
10. - [ ] `zipline.sources` → `zipline.assets`, `zipline.errors`
11. - [ ] `zipline.finance` → `zipline.assets`, `zipline.errors`, `zipline.gens`, `zipline.utils`
12. - [ ] `zipline.gens` → `zipline.finance`, `zipline.utils`
13. - [ ] `zipline.data` → `zipline.assets`, `zipline.errors`, `zipline.gens`, `zipline.lib`, `zipline.utils`

## Level 2 — depends on Level 0-1

14. - [ ] `zipline.algorithm` → `zipline.assets`, `zipline.errors`, `zipline.finance`, `zipline.gens`, `zipline.pipeline`, `zipline.sources`, `zipline.utils`, `zipline.zipline_warnings`
15. - [ ] `zipline.pipeline` → `zipline.assets`, `zipline.country`, `zipline.currency`, `zipline.data`, `zipline.errors`, `zipline.lib`, `zipline.utils`
16. - [ ] `zipline.utils` → `zipline.algorithm`, `zipline.api`, `zipline.data`, `zipline.errors`, `zipline.extensions`, `zipline.finance`, `zipline.pipeline`, `zipline.sources`, `zipline.zipline_warnings`

## Level 3 — integration and test packages

17. - [ ] `zipline.__main__` → `zipline.data`, `zipline.extensions`, `zipline.utils`
18. - [ ] `zipline.examples` → `zipline.api`, `zipline.finance`, `zipline.pipeline`
19. - [ ] `zipline.dispatch`
20. - [ ] `zipline.test_algorithms` → `zipline.algorithm`, `zipline.api`, `zipline.errors`, `zipline.finance`
21. - [ ] `zipline.testing` → `zipline.algorithm`, `zipline.assets`, `zipline.data`, `zipline.dispatch`, `zipline.finance`, `zipline.lib`, `zipline.pipeline`, `zipline.utils`
22. - [ ] `zipline._version`

Key observations:

- `zipline.algorithm`, `zipline.assets`, `zipline.data`, `zipline.errors`, `zipline.finance`, `zipline.gens`, `zipline.lib`, `zipline.pipeline`, `zipline.sources`, and `zipline.utils` form a coupled core cluster with mutual imports; they should be reviewed as a group rather than strictly linearly.
- `zipline.country`, `zipline.currency`, `zipline.extensions`, `zipline.protocol`, and `zipline.zipline_warnings` are low-level packages with no internal zipline package dependencies.
- `zipline.__main__`, `zipline.examples`, `zipline.test_algorithms`, and `zipline.testing` are later-stage integration or validation layers and belong near the end of the review sequence.

# Starsim testing style guide

It's very dull to write a test for a completed feature that you know will pass. So use test-driven development (TDD) instead: before you write the feature, start with the test and use it to brainstorm the feature's API. Then you can trick yourself that you're doing cool design architecture instead of writing a boring test. We're not militant about using TDD, but it really, truly is a good idea. 

Starsim uses `pytest` for its test suite, run in parallel via `pytest-xdist`. (Do not use `unittest`.) Tests live in the `tests/` folder and follow a consistent set of conventions described below. The overriding principle is: **tests should be runnable both by pytest and as standalone scripts**, so that developers can interactively explore and debug results. All tests should be run on GitHub Actions for every pull request, and PRs should only be merged if all tests pass.


## File structure

Test files typically follow this skeleton:

```python
"""
Test <topic>
"""
import sciris as sc
import numpy as np
import starsim as ss
import matplotlib.pyplot as plt
import pytest

n_agents = 1_000
do_plot = False
sc.options(interactive=False) # Assume not running interactively


def make_sim():
    """ Helper to create a default sim for this test file """
    ...
    return sim


@sc.timer()
def test_feature(do_plot=do_plot):
    """ One-line description of what this test checks """
    sc.heading('Testing feature...')
    sim = make_sim()
    sim.run()
    assert sim.results.something > 0, 'Expected something to be positive'
    if do_plot:
        sim.plot()
    return sim


if __name__ == '__main__':
    do_plot = True
    sc.options(interactive=do_plot)
    T = sc.timer()

    sim = test_feature(do_plot=do_plot)

    T.toc()

    if do_plot:
        plt.show()
```

An example of a complete test:
```python
@sc.timer()
def test_simple_vax(do_plot=do_plot):
    """ Create and run a sim with vaccination """
    sc.heading('Testing simple vaccination...')
    pars = make_sim_pars()

    sim_base = ss.Sim(pars=pars).run()
    sim_intv = ss.Sim(pars=pars, interventions=intv).run()

    assert sim_intv.summary.sir_cum_infections < sim_base.summary.sir_cum_infections, 'Vaccine should avert infections'

    if do_plot:
        plt.figure()
        plt.plot(sim_base.timevec, sim_base.results.sir.prevalence, label='Baseline')
        plt.plot(sim_intv.timevec, sim_intv.results.sir.prevalence, label='Vax')
        plt.legend()

    return sim_base, sim_intv
```

Key elements, in order:

1. **Module docstring** -- a brief description of what the file tests.
2. **`sc.options(interactive=False)`** -- can set at module level so pytest doesn't open plot windows, to avoid the need for lots of `plot=False` arguments.
3. **Helper functions** (e.g. `make_sim()`, `make_sim_pars()`) -- shared setup logic used by multiple tests.
4. **Test functions** -- each decorated with `@sc.timer()` (showing how long it took) and named `test_*`.
5. **`if __name__ == '__main__':`** block -- enables standalone script execution with plotting.


## Module-level constants

Define constants at the top of the file so they can be easily tuned:

```python
n_agents = 1_000
do_plot = False
sc.options(interactive=False)
```

Use underscore separators for large numbers (`1_000`, `10_000`). Some files use named size tiers for clarity:

```python
small  = 100
medium = 1000
```

Use the smallest agent count that still produces meaningful test results. Integration tests (`test_baselines.py`) use larger populations (e.g. `10e3`); unit-level tests use `100`--`1000`.


## Test functions

### Naming

Name test functions `test_<feature>`, where `<feature>` describes what is being tested:

```python
def test_sir():         # Yes
def test_parallel():    # Yes
def test_api():         # No: not descriptive
def test_sir_model():   # No: "model" adds nothing
```

### Decorators

Test functions are typically decorated with `@sc.timer()`:

```python
@sc.timer()
def test_demo(do_plot=do_plot):
    ...
```

This prints timing information, which is useful for catching performance regressions. Use `pytest.mark.skip(reason="...")` for tests that are temporarily disabled.

### Returns

Tests typically return "the most interesting" object they generate. If a sim is run, then typically they would return the sim. This allows the test file to be run as a script and the results inspected without needing to set breakpoints and use the debugger.

### Assertions

Always include a descriptive message in assertions. The message should say what was *expected*, not just what happened:

```python
# Yes
assert v0 <= v1, f'Expected prevalence to be lower with {par}={lo} than {par}={hi}, but {v0} > {v1}'
assert len(log) > 900, 'Expect almost everyone to be infected'

# No: no message
assert v0 <= v1
assert sim.results.n_alive[0] > 0
```

For float comparisons, use `np.isclose` or `np.allclose` with explicit tolerances:

```python
assert np.isclose(data.n_alive.values[-1], sim.results.n_alive[-1], rtol=0.05), 'Final pop not within 5% of data'
assert np.allclose(s1.summary[:], s2.summary[:], rtol=0, atol=0, equal_nan=True), "Sims don't match"
```

For testing expected exceptions, use `pytest.raises`:
```python
with pytest.raises(TypeError):
    sim = ss.Sim(diseases=True)
    sim.init()
```

### Testing scientific correctness

Many Starsim tests check that epi dynamics go in the expected direction. The key pattern is: vary one parameter and confirm the result changes as expected:

```python
par_effects = dict(
    beta     = [0.01, 0.99],
    dur_inf  = [1, 8],
    p_death  = [0.01, 0.1],
)

for par, par_val in par_effects.items():
    s0 = ss.Sim(pars0).run()
    s1 = ss.Sim(pars1).run()
    assert v0 <= v1, f'Expected higher {par} to increase prevalence'
```

Use generous tolerances (`rtol=0.05` or more) for stochastic comparisons to avoid flaky tests. Comment the tolerance choice:

```python
rtol = 0.2  # Generous to avoid random failures with small populations
```


## Dual-mode execution

The most distinctive Starsim testing convention is that every test file is designed to run *both* as a pytest module and as a standalone script.

### pytest mode

When run via pytest (`pytest test_*.py -n auto`), tests execute with `do_plot=False` and `sc.options(interactive=False)`. Functions are discovered by name (`test_*`). Plots are suppressed.

### Script mode

When run directly (`python test_sim.py`), the `__main__` block sets `do_plot=True`, runs all tests, and shows plots.

## Plotting in tests

Plotting should always be conditional, unless the plot itself is being tested:

```python
@sc.timer()
def test_feature(do_plot=do_plot):
    ...
    if do_plot:
        sim.plot()
    return sim
```

Use `plt.figure()` before any manual plotting to avoid accidentally drawing on a previous test's figure. Always ensure `sc.options(interactive=False)` is set at the top of the file to prevent plot windows in CI.

The `SCIRIS_BACKEND=agg` environment variable is set in the test runner scripts (`run_tests`, `check_coverage`) to ensure no display backend is needed.


## Test infrastructure

### Running tests

- **`./tests/run_tests`** -- runs all `test_*.py` files in parallel with timing.
- **`./tests/check_coverage`** -- runs tests with coverage reporting and generates an HTML report.
- **`./tests/check_style`** -- runs `pylint` on the main `starsim` package.

### Configuration files

- **`pytest.ini`** -- configures pytest warnings and environment variables (e.g. `SCIRIS_BACKEND=agg`).
- **`.coveragerc`** -- configures coverage.py to measure branch coverage of the `starsim` package, excluding pragmas, defensive raises, and `__main__` blocks.
- **`.gitignore`** -- excludes the `temp/` directory used for test artifacts.


## Parting words

Good tests should:

- **Be scientific**: test that the model behaves as epidemiologically expected, not just that code runs without errors.
- **Be debuggable**: return objects, support plotting, and print informative output so failures can be investigated interactively.
- **Be stable**: use tolerances appropriate for stochastic models; a test that fails 1% of the time due to randomness is worse than no test.
- **Be fast**: use the smallest agent count that produces meaningful results; save large-scale tests for benchmarks.

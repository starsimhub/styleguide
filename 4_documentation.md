# Starsim documentation style guide

People are probably (...hopefully!) going to be spending much more time reading the docs you write than reading the code you write. So make them good! Good docs make the difference between a tool that people adopt versus one they try out but quickly abandon. And just because LLMs are now reading your docs too, that isn't an excuse to be lazy and write something a human would find frustrating to wade through. Good docs should be succinct but comprehensive -- a devilish tradeoff to say the least!


## Documentation tools

Build your docs site using either:

- **[MkDocs](https://www.mkdocs.org/)** (with the [Material](https://squidfundamentals.com/mkdocs-material/) theme) -- slightly easier to set up, less control over layout and rendering.
- **[Quarto](https://quarto.org/)** -- slightly higher startup cost, but more flexibility.

Starsim uses Quarto, but most Starsim models use MkDocs. Either way, the docs should live in `docs/` and produce a static website. (Note: Please do not use Sphinx or rST unless you hate yourself and others.)


## Readmes

The repo-level **README.md** is one of the most important entry points for the project. It should typically include:

- **What the package does** -- a clear, concise description.
- **Installation** -- `pip install`, `uv add`, and local/dev install instructions.
- **Quick usage example** -- a minimal "hello world" that runs in under 10 lines.
- **Project structure** -- a one-line description of each submodule/folder, so newcomers can orient themselves.
- **Links** -- to the full docs site, contributing guide, and issue tracker.

Repos should also include:

- **LICENSE** -- we use the MIT license.
- **What's new / changelog** -- release-by-release summary of changes.
- **Contributing** -- how to set up a dev environment, run tests, and submit PRs.
- **Code of conduct** -- how to not be a jerk (e.g, by writing rST files).

Except for the license, these should be standalone markdown files in the repo root.

### Folder-level readmes

Every folder in the repo should have a `README.md` file -- even if it's just one or two sentences. It should explain why the folder exists, what it contains, and/or how its contents are organized. Someone browsing the repo on GitHub should never have to guess what a folder is for.


## 2. API reference

The API reference should be **auto-generated from docstrings** -- use [quartodoc](https://machow.github.io/quartodoc/) (for Quarto) or [mkdocstrings](https://mkdocstrings.github.io/) (for MkDocs).

Expectations for docstrings:

- Every public module, class, and function should have a docstring.
- Use [Google-style](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings) docstrings.
- Docstrings should include at least: a one-line summary, a description of parameters and return values, and ideally **one usage example**.
- Configure interlinks so that cross-references to standard libraries (Python, NumPy, Pandas, Sciris, etc.) resolve automatically.


## 3. Tutorials

Jupyter notebook-based tutorials are strongly encouraged even for small projects. At minimum, try to provide:

- A **"hello world" tutorial** -- the simplest possible working example, with explanation.
- A **slightly more advanced tutorial** -- demonstrating the core workflow end-to-end.

For larger projects, a progressive series of tutorials works well. Bonus points for including exercises/solutions.


## 4. User guide

For complex projects where tutorials + API reference aren't enough, add a **user guide** with deeper coverage of each topic. The user guide is explicitly framed as "more in-depth" than the corresponding tutorials.

A good structure progresses from concepts to practice:

1. **Introduction** -- what the package is, how models/workflows are structured.
2. **Basics** -- the core objects and how they fit together.
3. **Modules/Components** -- detailed coverage of each major subsystem.
4. **Workflows** -- common end-to-end tasks (running, calibrating, deploying).
5. **Advanced topics** -- performance, edge cases, internals.

Like tutorials, user guide pages should be Jupyter notebooks with executable examples.

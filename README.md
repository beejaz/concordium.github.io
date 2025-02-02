# Concordium Documentation

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.0-4baaaa.svg)](https://github.com/Concordium/.github/blob/main/.github/CODE_OF_CONDUCT.md)

Before contributing please read and follow the principles outlined in
- the [Divio documentation guide](https://documentation.divio.com/)
- our [style guide](#style-guide).

Other good documentation references include:
- the [Microsoft Style Guide](https://docs.microsoft.com/en-us/style-guide/welcome/)
- the [Google developer documentation style guide](https://developers.google.com/style)
The documentation is written in reStructuredText ([Link to the
basics](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html)).

For building the documentation we use
[Sphinx](https://www.sphinx-doc.org/en/master/index.html) and the [theme from
pydata-sphinx-theme](https://pydata-sphinx-theme.readthedocs.io/en/stable/) with minor
design tweaks.

Sphinx supplies a number of useful
["directives"](https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html)
(The sphinx equivalent of LaTeX commands) for stuff like code highlighting,
remarks, warnings and so on.

Additionally, we have enabled the [extension for
todo](https://www.sphinx-doc.org/en/master/usage/extensions/todo.html)
directives
```
.. todo::

   Write a todo here
```

To show TODOs as warnings when building the docs you need to uncomment the
following line in `/source/mainnet/conf.py`:

```
# todo_emit_warnings = True
```

NB: This will only show TODOs for the files being built; as opposed to TODOs
from all files.

To generate SVG graphics, we use the [Graphviz
extension](https://www.sphinx-doc.org/en/master/usage/extensions/graphviz.html).

## Development

All of the documentation lives in the `source` directory; here there is a subdirectory for Mainnet. General content such as site images, stylesheets, and other templates are in the `source` directory.

## Installation


### Linux

Install `python3` and the python package manager `pip3`.

To install the python dependencies run:
```
pip3 install -r requirements.txt
```

Install `graphviz`:

- Ubuntu: `sudo apt install graphviz`

### macOS

On macOS
```
brew install python3 graphviz
pip3 install -r requirements.txt
```

### Windows

Install [python3](https://www.python.org/downloads/windows/)
and select a python installer, e.g. [this one](https://www.python.org/ftp/python/3.9.1/python-3.9.1-amd64.exe).
Download and run the launcher. Make sure to select "Add Python to PATH" at the bottom before proceeding with the install.

After that from a terminal run
```
pip3 install -r requirements.txt
```
from the root of this repository.

If you want the graphs to render properly you also need to install the `dot` tool, which is part of the [graphviz package](https://graphviz.org/download/).

## Development

To watch the doc files and automate the build run:

### macOS and Linux
**Mainnet**

```
make dev-mainnet
```
and navigate to [localhost:8000/mainnet](http://localhost:8000/net).

Before committing, make sure to run the linter and fix all the errors reported:
```
make lint
```

### Windows

**Note:**
The exact command depends on which terminal type you are using. For example, in Powershell, you write .\make.bat dev-XXX where XXX is your target to build.

**Mainnet**

```
make.bat dev-mainnet
```
and navigate to [localhost:8000/mainnet](http://localhost:8000/net).

Before committing, make sure to try to build and fix any warnings that are reported.

```
./make.bat html
```


> **Note**:
> When working on changes to the design it can be beneficial to disable
> caching, as it can cause UI problems. To disable it, add the `-E` flag to the
> `dev` command in the appropriate make file.

## Building the docs
Run the build script from project root:

```
./script/build.sh
```

To check for dead links (can also be done by the CI), run:
```
make linkcheck-mainnet
```

### Building the gRPC JSON schemas

The folder `./source/mainnet/net/references/grpc-json-schemas/` contains the JSON
schemas used in the gRPC documentation.

To generate the schemas:

1. Check out the [`derive-schema`
branch of Concordium Rust
SDK](https://github.com/Concordium/concordium-rust-sdk/tree/derive-schema).
2. Run `cargo run generate --output_folder <path-to-grpc-json-schema-folder>`.

# Contributing

The `source/mainnet` directory contains documentation for the current Mainnet, and should always be compatible with the current Mainnet.

With only a couple of exceptions (Downloads and Ubuntu node), Testnet documentation is handled either via a note at the beginning of the topic (where the feature is not yet released on mainnet) or in a tab if it is an update to an existing feature where mainnet and testnet differ. Very small differences are handled inline in the text.

For information about how to use tabs, see `Sphinx tabs <https://sphinx-tabs.readthedocs.io/en/latest/#basic-tabs>`_.

## Style guide

### Language

* For a user guide, use **second person** ("you")
* Structure sentences, especially numbered procedures, using the **imperative** ("click on X").
* Keep sentences **short**.
* Prefer **present** continuous ("is") over future ("will be").
* Prefer **active voice** ("the baker adds a block") over passive voice ("a block is added").

### Formatting

#### Headers
Use *sentence-style capitalization*, i.e., only capitalize the first letter of a
header.

Be consistent in the use of characters for creating headers; use the following
for each level of header:

``` restructuredtext
========
Header 1
========

Header 2
========

Header 3
--------

Header 4
^^^^^^^^

Header 5
~~~~~~~~
```

#### Terminal commands

Use `code-block:: console` to show content from a terminal and prepend commands
with `$` without a space in between.

A space is added between `$` and the command through CSS.
This solution makes only the command itself copyable, thereby improving the user-experience.

Example:

``` restructuredtext
.. code-block:: console

   $echo Hello, world!
   Hello, world!
```

#### Buttons and clickable elements

Use **bold** to highlight keyboard buttons and clickable elements (e.g., "Press **Enter**", "Select **Next**").

#### Emphasis

- Use *italics* for text emphasis (e.g., when introducing a new term: "Obtain an identity from an *identity provider*.").
- After the term is introduced avoid emphasising it again in the same text.

### Code formatting for this repository

#### Indentation
Use three spaces for indentation.
This aligns the directive name (`note::`) with the content of the directive (`This line...`).

Add an empty line between a directive and its content.

Example that follows both rules:

``` restructuredtext
.. note::

   This line has three spaces in front of it and it has an empty line above it.
```

### Variables

Use variables when it makes sense.

Add new variables in the file `source/variables.rst`.

Use the variables by:

- Including a relative path to `variables.rst`, for example
  `../../variables.rst`.
- Then using the variable net, for example `|Net|`.

### Images

Save any images that you add in the Images folder under mainnet and testnet. Create sub-folders as needed to store images.

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License][cc-by-sa].

[![CC BY-SA 4.0][cc-by-sa-image]][cc-by-sa]

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-image]: https://licensebuttons.net/l/by-sa/4.0/88x31.png

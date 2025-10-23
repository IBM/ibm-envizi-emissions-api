# IBM Envizi Emissions API

## Overview

The IBM Envizi Emissions API is a comprehensive solution for organizations to calculate, track, and report greenhouse gas (GHG) emissions across their operations and value chains. This API provides standardized methodologies for emissions calculations aligned with the Greenhouse Gas Protocol, enabling businesses to meet regulatory requirements and sustainability goals.

## Repository Structure

- **sphinx-build/**: Contains the Sphinx documentation source files and build configuration
- **docs/**: Contains the built HTML documentation for GitHub Pages
- **notebooks/**: Contains Jupyter notebooks demonstrating API usage and examples
  - **tutorials/**: Step-by-step guides for using the API
  - **samples/**: Real-world application examples

## Documentation

The documentation is built using Sphinx with nbsphinx for integrating Jupyter notebooks. The main documentation is organized into:

- **Overview**: General information about the API
- **Tutorials**: Step-by-step guides for using different API features
- **Samples**: Real-world examples demonstrating API applications

## Building the Documentation

### Prerequisites

- Python ≥3.8 and ≤3.11
- Sphinx
- nbsphinx
- Jupyter
- Other dependencies listed in requirements.txt


### Makefile Targets

#### `make clean`

Removes all build artifacts and temporary files.

```bash
cd sphinx-build
make clean
```

#### `make notebooks`

Copies notebooks from the `notebooks` directory to the Sphinx source directory for inclusion in the documentation.

```bash
cd sphinx-build
make notebooks
```

#### `make html`

Builds the HTML documentation.

```bash
cd sphinx-build
make html
```

#### `make all`

Runs `clean`, `notebooks`, and `html` targets in sequence to perform a complete documentation rebuild.

```bash
cd sphinx-build
make all
```

#### `make pages`

Builds the documentation and copies the output to the `docs` directory for GitHub Pages publishing.

```bash
cd sphinx-build
make pages
```

### Viewing the Documentation Locally

After building the documentation, you can view it locally using:

```bash
# Open the locally built documentation
open sphinx-build/build/html/index.html

# Or view the GitHub Pages version
open docs/index.html
```

## Development Workflow

1. Update the repository content, could be documentation and/or notebooks
2. Build the documentation with `make all`
3. Test the documentation locally
4. When ready to publish, run `make pages`
5. Commit and push changes

Note: Don't commit the /notebooks that gets copied over within the /sphinx-build directory on running the `make all` target
# Makefile for IBM Envizi Emissions API documentation

# Variables
PYTHON_ENV := python
SPHINX_BUILD := sphinx-build
SOURCEDIR := source
BUILDDIR := build
NOTEBOOKS_SRC := ../notebooks
NOTEBOOKS_DEST := $(SOURCEDIR)/notebooks
DOCS_DIR := ../docs

# Default target
.PHONY: help
help:
	@echo "Available targets:"
	@echo "  html       - Build HTML documentation"
	@echo "  clean      - Remove build directory"
	@echo "  notebooks  - Copy notebooks to source directory"
	@echo "  all        - Clean, copy notebooks, and build HTML"
	@echo "  pages      - Copy built HTML to docs directory for GitHub Pages"

# Clean build directory
.PHONY: clean
clean:
	@echo "Cleaning build directory..."
	@rm -rf $(BUILDDIR)
	@rm -rf $(NOTEBOOKS_DEST)
	@echo "Done."

# Copy notebooks to source directory
.PHONY: notebooks
notebooks:
	@echo "Copying notebooks to source directory..."
	@mkdir -p $(NOTEBOOKS_DEST)/tutorials/general
	@mkdir -p $(NOTEBOOKS_DEST)/tutorials/scope1
	@mkdir -p $(NOTEBOOKS_DEST)/tutorials/scope2
	@mkdir -p $(NOTEBOOKS_DEST)/tutorials/scope3
	@mkdir -p $(NOTEBOOKS_DEST)/samples/scope1/environmental_impact_of_power_plant
	@cp $(NOTEBOOKS_SRC)/introduction.ipynb $(NOTEBOOKS_DEST)
	@cp $(NOTEBOOKS_SRC)/getting_started.ipynb $(NOTEBOOKS_DEST)
	@cp $(NOTEBOOKS_SRC)/tutorials/general/*.ipynb $(NOTEBOOKS_DEST)/tutorials/general/
	@cp $(NOTEBOOKS_SRC)/tutorials/scope1/*.ipynb $(NOTEBOOKS_DEST)/tutorials/scope1/
	@cp $(NOTEBOOKS_SRC)/tutorials/scope2/*.ipynb $(NOTEBOOKS_DEST)/tutorials/scope2/
	@cp $(NOTEBOOKS_SRC)/tutorials/scope3/*.ipynb $(NOTEBOOKS_DEST)/tutorials/scope3/
	@cp $(NOTEBOOKS_SRC)/samples/scope1/environmental_impact_of_power_plant/*.ipynb $(NOTEBOOKS_DEST)/samples/scope1/environmental_impact_of_power_plant/
	@cp $(NOTEBOOKS_SRC)/samples/scope1/environmental_impact_of_power_plant/*.png $(NOTEBOOKS_DEST)/samples/scope1/environmental_impact_of_power_plant/
	@echo "Done."

# Build HTML documentation
.PHONY: html
html:
	@echo "Building HTML documentation..."
	@$(SPHINX_BUILD) -b html $(SOURCEDIR) $(BUILDDIR)/html
	@echo "Done. Output is in $(BUILDDIR)/html"

# Clean, copy notebooks, and build HTML
.PHONY: all
all: clean notebooks html
	@echo "Documentation build complete."

# Copy built HTML to docs directory for GitHub Pages
.PHONY: pages
pages: all
	@echo "Copying documentation to docs directory..."
	@rm -rf $(DOCS_DIR)/*
	@mkdir -p $(DOCS_DIR)
	@cp -r $(BUILDDIR)/html/* $(DOCS_DIR)/
	@echo "Done. GitHub Pages content is in $(DOCS_DIR)"
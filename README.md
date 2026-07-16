# Documentation Repository

[![Built with MkDocs](https://img.shields.io/badge/Built%20with-MkDocs-blue.svg)](https://www.mkdocs.org/)
[![Theme: Material](https://img.shields.io/badge/Theme-Material-indigo.svg)](https://squidfunk.github.io/mkdocs-material/)

This repository contains the source files for our documentation website, powered by **MkDocs** (configured with the Brane theme/setup)[cite: 1].
--

## Sources & References

This documentation is based on the official information published in the following source:

* **[Brane Specification](https://braneframework.github.io/specification/)** 
* **[Brane users guide](https://braneframework.github.io/manual/)**
* **[Brane Tutorials](https://braneframework.github.io/tutorials/)**

---
## Repository Structure

Understanding where files live helps when editing or configuring the site:

```text
.
├── docs/                 # Markdown source files (.md)
├── mkdocs.yml            # MkDocs site configuration & navigation
├── requirements.txt      # Python dependencies
└── README.md             # This guide
```

## Prerequisites & Installation

To run and preview the documentation website locally, you need Python installed on your system. 

1. **Clone the repository** (if you haven't already):
```bash
git clone <repository-url>
cd <repository-folder>
```
## Install MkDocs and required dependencies:
Run the following command to install MkDocs along with any theme dependencies (such as mkdocs-material):

```Bash
pip install mkdocs mkdocs-material
```
(`Note`: If a requirements.txt file is present in the repository, you can run pip install -r requirements.txt instead.)

## Local Development & Testing
Once the dependencies are installed, follow these steps to build and preview the site:

## 1. Navigate to the Documentation Directory
If your configuration file (mkdocs.yml) is inside a specific subfolder (e.g., docs), navigate to that directory:

```Bash
cd docs
```

## 2. Build the Static Site
To compile the Markdown source files into static HTML pages, run:

```Bash
mkdocs build
```

This will generate a site/ directory containing the static website.

## 3. Serve the Site Locally (Live Preview)
To start a local development server and preview the website in your browser with real-time reload capability, run:

```Bash
mkdocs serve
```
After running the serve command, open your browser and navigate to:
`http://127.0.0.1:8000`

---


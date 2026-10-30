This should not be renamed README.md to avoid being part of the published docs.

The site is built automatically on pushing to the main branch as part of a github action.  To contribute, just change the relevant markdown file in the docs folder and push.

For manual / local builds, install the pinned toolchain:

    python -m venv .venv
    . .venv/bin/activate
    pip install -r requirements-docs.txt

Then, from the root of this repo:

    mkdocs build
    mkdocs serve

Deployment is automatic on being merged to the main branch.

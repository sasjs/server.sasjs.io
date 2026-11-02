This should not be renamed README.md to avoid being part of the published docs.

The site is built automatically on pushing to the main branch as part of a github action.  To contribute, just change the relevant markdown file in the docs folder and push.

For manual / local builds, install the pinned toolchain:

    python -m venv .venv
    . .venv/bin/activate
    pip install -r requirements.txt

Then, from the root of this repo:

    ./build.sh          # mkdocs build --strict - the same build CI runs
    mkdocs serve        # a live preview on http://127.0.0.1:8000

Deployment is automatic on being merged to the main branch.

## Checks

`scripts/check-server-docs.mjs` verifies the pages against the server release
they document - every page in the nav, every endpoint registered, every setting
read, and `docs/swagger.yaml` identical to the release's
`api/public/swagger.yaml`:

    node scripts/check-server-docs.mjs --server-ref v3.9.0

CI runs it on every pull request against the pinned release, and weekly against
the server's main branch. When the weekly run fails, update the pages, refresh
the spec (`cp <server>/api/public/swagger.yaml docs/swagger.yaml`) and move the
`--server-ref` in `.github/workflows/build.yml`.

## API reference

The API page renders `docs/swagger.yaml`, a copy of the specification the server
publishes at `/swagger.yaml`. The copy is deliberate: the site must not depend on
a network call to build, and the check above is what keeps the copy honest.

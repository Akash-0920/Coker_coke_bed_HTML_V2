# Piecewise Throughput Coker Solver

Single-file, dependency-free web version of the piecewise-throughput coking
kinetic solver. Runs entirely in the browser (no backend, no build step) -
same K1/K2/K3 correlations and coke-chamber formulas as the Python model.

## Publish on GitHub Pages (make it public)

1. Create a new **public** GitHub repository (or use an existing one).
2. Upload `index.html` to the repository root (Add file -> Upload files).
3. Commit to the `main` branch.
4. In the repo: **Settings -> Pages**.
5. Under "Build and deployment" -> Source, choose **Deploy from a branch**.
6. Branch: `main`, folder: `/ (root)` -> **Save**.
7. Wait ~1 minute, then your page is live at:
   `https://<your-username>.github.io/<repo-name>/`

No further configuration needed - it's a static HTML file.

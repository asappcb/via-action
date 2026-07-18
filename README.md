# via-action

Run **via** board checks on the KiCad boards changed in a pull request, and gate the merge — as a GitHub check + a PR comment.

Checks: DRC · connectivity · schematic-parity · DFM · BOM · cost (per your repo's `.via-ci.yml`).

## Usage

```yaml
# .github/workflows/via.yml
name: via board review
on: pull_request
permissions:
  contents: read
  pull-requests: write
jobs:
  via:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: asappcb/via-action@v1
        with:
          parts-csv: parts/jlc.csv   # optional (enables BOM/cost)
```

## Inputs

| input | default | notes |
|---|---|---|
| `boards` | *(auto)* | glob; empty = the `.kicad_pcb` files changed in the PR |
| `parts-csv` | `''` | JLC/LCSC parts CSV → enables BOM/cost |
| `comment` | `true` | post/update a PR comment |
| `github-token` | `${{ github.token }}` | for the comment |

## How it works

For each changed `*.kicad_pcb`, it checks **head** (your PR) and **base** (a git worktree) in **full project context** — your `.kicad_pro` severities, schematic, and libraries are respected — then runs the hybrid differential and gates on *new* errors. Your board never leaves your runner.

Behavior is driven by a **`.via-ci.yml`** at your repo root (checks + gate policy); absent → sensible defaults.

## Notes

Runs a prebuilt container (`ghcr.io/asappcb/via-action`). The KiCad engine ships in the image.

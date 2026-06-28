# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`a-liner` is a Python/Matplotlib command-line tool that draws linear alignment visualizations for comparative genomics. Each track is a horizontal line representing a sequence region (optionally stranded); alignments between tracks are drawn as colored trapezoids shaded by percent identity, with optional gene arrows (GFF3/GenBank/Excel), region highlights, and per-position scatter tracks above each line. It is published in *Bioinformatics* (2026; btag408) — see the `Citation` section of `README.md`.

## Commands

This project has no automated test suite or linter configuration. Development is driven by running the two sample datasets and inspecting the generated PDFs.

### Install (editable / development)
```
pip install -e .
```
Dependencies are declared in [pyproject.toml](pyproject.toml): matplotlib, pandas, biopython, numpy, openpyxl, bcbio-gff. Python `>=3.8`.

### CLI entry point
After install, the console script `a-liner` is exposed (see `[project.scripts]` in `pyproject.toml`). Equivalent direct invocation:
```
python -m a_liner.cli --help
```

### Run the two bundled sample datasets
```
cd sample_data/a_Stx-phages   && bash runme.sh    # → Stx-prophage-regions.pdf
cd sample_data/b_ostrich-emu  && bash runme.sh    # → ostrich-emu_sex-chromosomes.pdf
```
Each `runme.sh` writes a `.log` of the chosen CLI flags — read it first to see a real working invocation.

### Programmatic use
```python
from a_liner.cli import run
from a_liner.common import get_args
args = get_args(["-i", "input.txt", "--gff3", "annot.gff3", "--feature", "CDS"])
run(args)
```
`common.get_args(argv=None)` accepts a list (same as CLI argv) or `None` to parse `sys.argv`.

## High-Level Architecture

The package is a flat set of single-purpose modules under [a_liner/](a_liner/). The data flow is strictly linear: parse CLI → load sequences → compute figure size → draw layers onto one matplotlib Axes → save PDF.

### Module responsibilities

- [a_liner/cli.py](a_liner/cli.py) — Orchestrator. `main()` parses argv and calls `run(args)`. `run(args)` performs, in order: configure PDF font embedding (`rcParams['pdf.fonttype']=42`), load sequences via `seqs`, build `common.Size`, plot scaffold lines + names, plot scalebar, plot alignments, plot genes, plot highlights, plot scatter, save PDF.
- [a_liner/common.py](a_liner/common.py) — `get_args()` defines the full argparse spec grouped into sections (General, Sequence layout/drawing/scale, Alignment files/options, Gene annotation files, Gene legend/drawing, Highlight, Scatter). All argument validators (`validate_range`, `validate_left_margin`, `validate_color`, `validate_marker_style`, `validate_float`) live here. Also defines `Size` (figure layout math), `Text`, `Color`, and the shared `detect_index_update(ID, start, end, seqs)` helper used by every other module to map a (sequence ID, range) to the matching `Scaffold` instance(s).
- [a_liner/seqs.py](a_liner/seqs.py) — `Scaffold` data class (one sequence region). `input_scaffold_integrated_file()` loads either the Excel config (`--xlsx`) or the TSV config (`-i`); both must have columns `n, ID, start, end, strand, name` and `n` must be consecutive integers starting at 0. `Scalebar` class draws either a bar legend (`--scale legend`), per-track ticks (`tick`), or both. `seqs` is a list-of-lists `seqs[track][index]`.
- [a_liner/alignment.py](a_liner/alignment.py) — `Colormap` (6 options indexed by `--colormap` 0–5; index 5 is the original multi-hue palette), `Colorbox` (identity legend drawn only when an alignment file was successfully loaded), and one loader per alignment format. All loaders normalize to internal coordinates and call the shared `plot_alignments()`. The `include_nonadjacent` flag controls whether alignments span non-adjacent tracks.
- [a_liner/genes.py](a_liner/genes.py) — `Gene` arrow drawing, `Feature_color_legend` for keyword→color mapping via regex (patterns in `--feature_color_map` are separated by `/`), `deal_gff()` shared by `plot_genes_from_gff` (9- or 10-column GFF; the optional 10th column is a per-feature color override) and `plot_genes_from_gff_excel`, `plot_genes_from_gb` for GenBank via Biopython. The `RNA` helper resolves exon/CDS strand by matching the parent mRNA's outer coordinates.
- [a_liner/highlight.py](a_liner/highlight.py) — Region highlights. `n=0` draws a highlight on the scatter background only; otherwise on the sequence track. `detect_index_for_highlights()` clips highlight intervals to scaffold boundaries.
- [a_liner/scatterplot.py](a_liner/scatterplot.py) — `plot_background()` paints the scatter track background plus y-axis ticks and dashed reference lines (`--scatter_ylines`); `plot_scatterplot()` plots points scaled to `[--scatter_min, --scatter_max]`.

### Coordinate convention

All input coordinates are 1-based, inclusive. Internally, every loader converts to half-open matplotlib coordinates by adding 1 to the end before calling `plot_alignments()`/`Gene`/`Highlight` — do not "fix" the `+1` offsets; they are the boundary between genomic and plotting coordinate spaces.

### Strand handling

`scaffold.convert_position2xcoord(pos)` flips x for `-` strand scaffolds. `Gene.__convert_position2coord()` additionally flips the gene arrow's strand sign when the parent scaffold is reversed, so arrowheads always point in the transcriptional direction on the screen.

## Key Conventions

- **CLI flag naming**: every option is a snake_case keyword that matches a Python attribute on the argparse Namespace (e.g. `--min_identity` → `args.min_identity`). New flags should follow this and be added inside the appropriate `parser.add_argument_group(...)` so `--help` stays organized.
- **`detect_index_update` returns `[[i,j], ...]`**: a hit of `[-1, j]` means the ID/range was not found in any scaffold. Loaders must filter `i == -1`.
- **`Scaffold.name == 'BLANK'`**: scaffolds named `BLANK` are drawn as empty space (no line, no name, no highlight detection). Use this for visual gaps inside a track.
- **Two `output_parameters*` calls per module**: each visual module prints a `## … paramenters` block (note the misspelling — preserved across modules) so users can reproduce a figure from the log. New modules should follow the same pattern.
- **Identity color**: percent identity is mapped linearly into the chosen colormap from `min_identity` to 100. `--min_identity` therefore acts as the floor, not a filter; identity < `--min_identity` is dropped, identity ≥ `--min_identity` is colored with the lowest color in the gradient.
- **`zorder` layers** (bottom→top): scatter background = 0, scaffold line = 2, highlight = 1 or 3, gene fill = 4, gene edge = 6. Insert new layers consistent with this ordering.

## Input File Formats (quick reference)

All required column headers for the scaffold config are documented in [README.md](README.md) §"Prepare input files". The three optional file types use these schemas:

- **Alignment** (`--alignment`): `seq_ID1<TAB>start1<TAB>end1<TAB>seq_ID2<TAB>start2<TAB>end2<TAB>identity%` (1-based, inclusive).
- **Highlight** (`--highlight`, `--sp_highlight`): `seq_ID<TAB>start<TAB>end<TAB>color` (no header).
- **Scatter** (`--scatter`): `seq_ID<TAB>position<TAB>value` (no header).
- **Feature color map** (`--feature_color_map`): TSV with `label<TAB>pattern(s)<TAB>color`. Multiple regex patterns in column 2 are separated by `/`.
- **GFF3**: standard 9 columns, or 10 columns where column 10 is a per-feature color.

## Verified Environment

`README.md` §"Requirements" pins the tested versions: Python 3.13.5, matplotlib 3.10.3, pandas 2.3.1, numpy 2.3.2, biopython 1.85, bcbio-gff 0.7.1, openpyxl 3.1.5. There is no CI configuration in this repo.
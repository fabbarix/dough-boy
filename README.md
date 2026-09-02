# Biga &amp; Rinfresco

A baker's-percentage calculator for Neapolitan pizza dough made by the indirect method: a stiff
**biga** built on a 100 % hydration sourdough starter, refreshed into the final dough with a
**rinfresco**.

Live at **<https://pizza.orksu.com/>** — repo `fabbarix/dough-boy`.

## What it does

Enter the batch and the formula; every gram is derived from the panetto weight you actually want.

- **Panetto weight follows the rule of 22** — diameter in inches × 22 g (12″ → 264 g). The
  grams-per-inch factor is editable, or type a weight directly to override it.
- **Total hydration, salt and malt** are baker's percentages against *total* flour.
- **The biga** takes a chosen share of the flour at its own hydration. The starter's flour and water
  are counted *inside* the biga's totals, so both the biga's stated hydration and the formula's total
  hydration hold true once it is folded in.
- **Salt and malt go into the rinfresco only** — salt in the biga would slow the ferment.
- Warnings appear when a starter weight or a biga hydration makes a component go negative.
- **Timetable** counts back from the bake time through appretto, staglio, bulk, rinfresco and the biga.
- **Share link** writes every setting into the address as query parameters.
- **Print recipe as PDF** composes a two-page sheet — formula, method, timetable — as a real vector
  PDF with IBM Plex subset and embedded, generated entirely in the browser.

## The arithmetic

With total dough `W`, hydration `H`, salt `S`, malt `M` (fractions of total flour), biga flour share
`B`, biga hydration `Hb`, and `G` grams of 100 % hydration starter:

```
F        = W / (1 + H + S + M)      total flour, starter flour included
water    = F·H      salt = F·S      malt = F·M

biga flour = B·F                    biga water = Hb·B·F
  add flour = B·F  − G/2              add water = Hb·B·F − G/2

rinfresco flour = (1−B)·F           rinfresco water = F·H − Hb·B·F
```

## Running it

One self-contained `index.html` — no build step, no framework, no bundler. Fonts for the PDF are
subset TTFs embedded as base64; [jsPDF](https://github.com/parallax/jsPDF) is loaded from cdnjs.
Open the file, or serve the directory:

```sh
python3 -m http.server -d . 8000
```

## Deployment

GitHub Pages, served from the default branch at the repository root. `CNAME` points the site at
`pizza.orksu.com`; add a DNS `CNAME` record for `pizza` pointing at `fabbarix.github.io`.

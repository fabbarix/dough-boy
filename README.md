# Biga &amp; Rinfresco

A baker's-percentage calculator for Neapolitan pizza dough made by the indirect method: a stiff
**biga** built on a 100 % hydration sourdough starter, refreshed into the final dough with a
**rinfresco**.

Live at **<https://pizza.orksu.com/>** — repo `fabbarix/dough-boy`.

## What it does

Enter the batch and the formula; every gram is derived from the weight of the piece you actually
want — a round panetto, or a teglia sized by the Italian pan rule.

- **Tonda — the rule of 22.** Diameter in inches × 22 g (12″ → 264 g). The grams-per-inch factor is
  editable, or type a weight directly to override it.
- **Teglia — the Italian pan rule.** Width × length in centimetres ÷ 2, so a 30 × 40 cm *teglia
  casalinga* takes 600 g. Presets cover the two Italian sizes (30 × 40, 40 × 60) and the American sheet
  pans — full 18 × 26″ at 1,510 g, half 13 × 18″ at 755 g, quarter 9.5 × 13″ at 398 g, eighth
  6.5 × 9.5″ at 199 g. Sides can be given in centimetres or inches; inches are converted before the rule
  is applied, and the divisor is editable for a thicker or thinner pan. Typing a weight overrides the rule
  and leaves the pan alone — one number cannot back-solve two sides.
- **Each shape keeps its own batch count** — 20 panetti, 2 teglie — so switching between them does
  not ask for 20 sheet pans of dough.
- **Total hydration, salt and malt** are baker's percentages against *total* flour.
- **The biga** takes a chosen share of the flour at its own hydration. The starter's flour and water
  are counted *inside* the biga's totals, so both the biga's stated hydration and the formula's total
  hydration hold true once it is folded in.
- **Salt and malt go into the rinfresco only** — salt in the biga would slow the ferment.
- Warnings appear when a starter weight or a biga hydration makes a component go negative.
- **Method and timetable follow the shape** — a tonda is opened by hand, a teglia is tipped into an oiled
  pan and pressed to the corners.
- **Fermentation times are solved, not typed.** Each stage — biga, bulk, appretto — names whether it
  rises in the room or the fridge, and its hours come from the dough's temperature and its surroundings.
  Type an hour figure yourself and that stage stays put until the next temperature change.
- **Timetable** counts back from the bake time through appretto, staglio, bulk, rinfresco and the biga,
  and adds a bench-tempering step when the balls come out of the fridge.
- **Share link** writes every setting into the address as query parameters, and every setting — shape,
  pan, unit, counts and formula — is remembered in `localStorage` for the next visit.
- **Print recipe as PDF** composes a two-page sheet — formula, method, timetable — as a real vector
  PDF with IBM Plex subset and embedded, generated entirely in the browser.

## Fermentation timing

Rate follows Ratkowsky's square-root law rather than a flat Q10, normalised to 1 at `Tref = 20 °C`:

```
r(T) = ((T − Tmin) / (Tref − Tmin))²            Tmin = −5 °C
```

Both agree where the usual rule works — 20→30 °C comes out at 1.96, near the classic Q10 of 2 — but
Q10 = 2 makes a 4 °C fridge only 3× slower, which would retard a 2 h bulk to 6 h. The square root
steepens in the cold and reaches zero near freezing, putting a cold ball proof around 30 h.

Dough does not arrive at fridge temperature, it decays towards it, and that lag is most of why flat cold
math misses. With a quadratic rate the accumulated fermentation integrates in closed form, so a stage
spanning bench and fridge is one expression rather than a simulation:

```
T(s) = Tenv + (T₀ − Tenv)·e^(−s/τ)
a = Tenv − Tmin,  u = T₀ − Tenv,  D = (Tref − Tmin)²

F(t) = [ a²t + 2auτ(1 − e^(−t/τ)) + (u²τ/2)(1 − e^(−2t/τ)) ] / D
```

`F` is in *reference-hours* — hours at `Tref` that ferment as much — and `τ` scales as the cube root of
the mass, about 1.1 h for a 264 g ball and 3 h for a 5 kg bulk. `F` is monotonic, so the hours come from
bisecting `F(t) = target`.

Each stage's target allows for how much is already alive in it. Gas output tracks population and
population grows exponentially, so time to a fixed target goes as a logarithm — doubling the starter
subtracts hours rather than halving them:

```
target = τμ · ln(1 + K / I)                       τμ = 8 reference-hours
```

`I` is the starter against biga flour for the biga, and the biga's share of the flour for the bulk and
appretto, since an 80 %-prefermented dough moves far faster than a 40 % one. `K` is fixed per stage by
running the page's own defaults through the model, so 20 panetti mixed at 18 °C in an 18 °C room still
give 18 h, 2 h and 5 h.

`Tmin` and `τμ` are the two knobs worth tuning against your own bakes. The model tracks yeast activity,
not flavour: hitting the same target warm will taste more acidic, which the page warns about above 28 °C
rather than pretending the curve captures it. The bench-tempering time after a cold appretto is thermal,
not fermentative, so the fermentation it contributes is not counted.

## The arithmetic

Each piece is sized by its own rule, and everything else follows from `n` pieces of that weight:

```
tonda    weight = diameter(in) × 22          12″          -> 264 g
teglia   weight = w(cm) × h(cm) / 2          30 × 40 cm   -> 600 g
```

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

GitHub Pages, published by `.github/workflows/pages.yml` on every push to the default branch: the
repository root is uploaded as the Pages artifact and deployed as-is. Set **Settings → Pages →
Source** to *GitHub Actions* for the workflow to have somewhere to deploy to.

The site answers on `pizza.orksu.com`; add a DNS `CNAME` record for `pizza` pointing at
`fabbarix.github.io`, then set the custom domain under Settings → Pages and enable *Enforce HTTPS*.
`.nojekyll` keeps Pages from running the tree through Jekyll.

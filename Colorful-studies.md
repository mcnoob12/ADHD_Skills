---
name: "adhd-chapter-notes"
description: "Turn a textbook chapter PDF into neurodivergent-friendly study notes PDFs (dark and/or light), with colour-coded families, multi-line reaction schemes and drawn chemical structures; asks reading preferences first."
---

# ADHD / neurodivergent-friendly chapter notes

Converts a dense textbook chapter into study notes built for readers who lose their place in walls of text (ADHD, dyslexia, autism and anyone else). It was built for school chemistry (e.g. NCERT Class 12), and it works for any science chapter. Chemistry-specific features (reaction schemes, drawn structures) are simply skipped for subjects that don't need them.

Output: one or two PDFs, `<Chapter>_ADHD_Notes_Dark.pdf` (screen) and `<Chapter>_ADHD_Notes_<Light|Print>.pdf` (paper), depending on the reader's choices.

## Step 0: ask preferences first (before reading the whole chapter)
First skim only the opening 1-2 pages (title, unit name, objectives) to identify the subject. Do this before asking, so the questions fit the chapter.
Ask with AskUserQuestion (two calls if needed). If nobody is there to answer, use the defaults marked (Recommended) and say which you used.

Call 1:
1. **Bionic reading** (bold first part of each word in prose)? Off (Recommended; evidence for it is weak and many readers find it distracting) / On.
2. **Which versions?** Both dark and light (Recommended) / Dark only / Light only.
3. **Name for the light version?** Print edition / Light edition. This sets the cover label ("Print edition" vs "Light edition") and the filename suffix (`_Print.pdf` vs `_Light.pdf`).
4. **Spacing?** Comfortable (Recommended; roomy, larger text, more pages) / Compact (denser, fewer pages).

Call 2:
1. **Depth?** Full rigor (Recommended; every reaction, condition, mechanism, worked example and answer key, minus fluff) / Condensed revision notes (key facts, main reactions, hooks).
2. **Draw ring structures?** Chemistry chapters only: ask this question only when the chapter is chemistry (organic, or inorganic/physical with structural formulas). Options: Yes, skeletal structures (Recommended) / No, text formulas only. For any other subject (physics, biology, maths, history and so on), don't ask it. Skip drawn structures, and use the reaction-scheme notation only if the chapter has process or step chains that benefit from it.
3. **Extras to include?** (multiSelect) Memory hooks / Exam-trap boxes / Cheat sheet / Self-check quiz. Default: all.

Remember the answers for the rest of the session.

## Fixed principles (regardless of answers)
- Cut story, history trivia, repetition and rhetorical fluff; keep every number exactly as printed (temperatures, pressures, %, pKa, bond data, b.p.).
- Chunk everything: TL;DR box at the top of each section, one idea per box, colour per topic family.
- Reactions as multi-line schemes, one step per line, reagent above the arrow, conditions below. Mechanisms one line per step with slow/fast marked.
- Anything not in the textbook (e.g. equations the book asks readers to derive, extra facts) is allowed, but tell the reader it was added.
- Tell the reader that curly electron-pushing arrows are not drawn, so they should practise drawing them.
- Suggest active recall (cover-and-write, quizzing from the cheat sheet), since reading alone doesn't make it stick.

## Workflow
1. Skim the opening pages to identify the subject, then ask preferences (Step 0).
2. Read the whole chapter PDF (Read with `pages`, max 20 per call). List every equation, condition, mechanism, worked example, intext/exercise answer key.
3. Install tools: `pip install --break-system-packages -q weasyprint rdkit beautifulsoup4` (poppler-utils for previews). Inter font is used if present (/usr/share/fonts/opentype/inter/); otherwise CSS falls back to sans-serif. Install Inter if missing for best results.
4. Write the support files below into a work dir: `notes_build.py`, `dark.css`, `light.css`, `compact.css`.
5. Choose colour families: up to three topic families plus one general. CSS slot names are fixed, meaning is per chapter: `alc` = blue (family 1), `phe` = orange (family 2), `eth` = green (family 3), `gen` = purple (general / applies to all). Examples: Aldehydes / Ketones / Carboxylic acids; Haloalkanes / Haloarenes; Amines / Diazonium salts. Set the `@page` footer text in dark.css to the chapter name.
6. Write `notes.html` (structure below). `<style>` = dark.css (+ compact.css if Compact).
7. Build:
   - Dark: `NOTES_THEME=dark NOTES_BIONIC=<0|1> python3 notes_build.py notes.html <Chapter>_ADHD_Notes_Dark.pdf`
   - Light: copy notes.html to light.html; append light.css (then compact.css again if Compact) at the end of `<style>`; set cover kicker to "Print edition" or "Light edition"; change the cover how-to wording "Yellow text above an arrow" to "Amber". Build with `NOTES_THEME=light`.
8. Verify visually: `pdftoppm -r 45 -png out.pdf p`, stitch a montage with PIL and Read it; zoom (-r 75) into pages with schemes and structures. Fix orphaned boxes, overflowing scheme lines, and a cheat sheet spilling onto a second page (it has `table.cheat`; shrink further if needed).
9. Deliver the PDF(s) with SendUserFile. Brief reply: what's inside, page count, any additions beyond the textbook, the curly-arrow note.

## Document structure
Cover (`.cover`, full-bleed): kicker line (course · unit · edition), big title with family-coloured words (`<span class="a|p|e">`), one-line subtitle, `.legend` of the families (`lg-a lg-p lg-e`), `.howto` box explaining the tags plus a small demo `.rx` scheme.

Numbered sections: `<section class="sec FAMILY">` (first one `style="margin-top:0"`):
- `.sec-head` with `.n` ("04 · PREPARATION OF ALCOHOLS"), `h2`, `.q` subtitle.
- First child: `<div class="tldr bio"><span class="tag t-tldr">TL;DR</span> 2-3 sentences</div>`.
- Then `h3` sub-headings, `.card` (title in `.ct`), `.rx` schemes, tables, `.ladder` stat tiles (`<div><span class="big">351 K</span>Ethanol</div>`), `ol/ul.steps` numbered lists, `.grid2`/`.grid3`.
- Callouts (if chosen): `.hook.bio` (`t-hook` HOOK: short mnemonic), `.trap.bio` (`t-trap` TRAP), `.ex.bio` (`t-ex` EXAMPLE / INTEXT with answers). A `t-mech` MECH tag goes inside `.rx .rt` for mechanisms.
- Inline family colours: `.A .P .E`.
Add `class="bio"` to every prose container. Bionic reading (if on) applies only there, never to formulas, tags or schemes.

Typical order: big picture & classification; nomenclature (tables + structure galleries); structure & physical properties; preparation per family; the chapter's key concept (e.g. acidity) with resonance rows; reactions per family; real-world/commercial; cheat sheet (`<section class="sec gen pb">`, `<table class="gen cheat">`, reagent → result with family-coloured result column); self-check (`<section class="sec gen pb">`, `ul.chk` of ~10 questions, answers in `.ans` which renders upside-down, a closing hook).

## Reaction notation (inside `<div class="rx FAMILY"><div class="rt">Title</div><rx> ... </rx></div>`)
One line = one step.
- Species separated by ` + ` (spaces required). Label under a species: `Species{label}`.
- Subscript `_2` / `_{N}`; superscript `^+`, `^-` (renders −), `^{δ+}`, `^‡`. Write `&gt;` `&lt;` for > <.
- Arrows: `-->`, `<=>` (equilibrium), `<->` (resonance ⟷). Reagent above / condition below: `-->[reagent|condition]`, e.g. `-->[(i) LiAlH_4|(ii) H_2O]`.
- Step tag at line start: `#Step 1#`, `#(ii)#`. Grey side note at end: ` // text`.
- ` ;; ` = spacing instead of + (galleries; use `class="rx gal FAMILY"` to centre).
- Drawn structure: `$SMILES$` (RDKit; NO2, SO3H, COOH, CHO, OCH3, OC2H5, COCH3, OCOCH3, CHCl2 auto-condensed to labels). Coefficient: `2 $SMILES$`. `*` draws as R (`$*Oc1ccccc1$`).
- Exact Kekulé / resonance form: `$!SMILES$`, non-aromatic with explicit charges, e.g. phenoxide I–V:
  `$![O-]C1=CC=CC=C1${I} <-> $!O=C1[CH-]C=CC=C1${II} <-> $!O=C1C=C[CH-]C=C1${III} <-> $!O=C1C=CC=C[CH-]1${IV} <-> $![O-]C1C=CC=CC=1${V}`
- Salts: `[O-]c1ccccc1.[Na+]`, or textbook-style O–Na bond `[Na]Oc1ccccc1`.
Rings are auto-oriented with the key group (O > OR > N > others) on top, so o/m/p positions are consistent. If "No" to structures, write ring compounds as text (C_6H_5OH, o-O_2N–C_6H_4–OH). Chains stay text either way.
```
#Step 1# CH_3–CH_2–OH + H^+ <=>[fast] CH_3–CH_2–O^+H_2{ethyl oxonium ion} // protonated alcohol
$Oc1ccccc1$ -->[CHCl_3 + aq NaOH] $[O-]c1ccccc1C(Cl)Cl.[Na+]${intermediate} -->[NaOH|hydrolysis] $[O-]c1ccccc1C=O.[Na+]$ -->[H^+] $Oc1ccccc1C=O${salicylaldehyde}
```

## notes_build.py
```python
#!/usr/bin/env python3
"""Build study notes: expand <rx> reaction notation, draw structures, optional bionic text, render PDF.
Usage: NOTES_THEME=dark|light NOTES_BIONIC=0|1 python3 notes_build.py notes.html out.pdf
"""
import hashlib, math, os, re, sys
from rdkit import Chem, RDLogger
from rdkit.Chem import rdAbbreviations, rdDepictor
from rdkit.Chem.Draw import rdMolDraw2D
from rdkit.Geometry import Point2D
from weasyprint import HTML
from bs4 import BeautifulSoup, NavigableString

RDLogger.DisableLog("rdApp.*")
rdDepictor.SetPreferCoordGen(False)
THEME = os.environ.get("NOTES_THEME", "dark")
BIONIC = os.environ.get("NOTES_BIONIC", "0") == "1"
src, out = sys.argv[1], sys.argv[2]
MOLDIR = os.path.join(os.path.dirname(os.path.abspath(src)), "mol")
os.makedirs(MOLDIR, exist_ok=True)

ABBR = rdAbbreviations.ParseAbbreviations("""NO2 [N+](=O)[O-] NO<sub>2</sub> O<sub>2</sub>N
SO3H S(=O)(=O)[OH] SO<sub>3</sub>H HO<sub>3</sub>S
COOH C(=O)[OH] COOH HOOC
CHO [CH]=O CHO OHC
OCOCH3 OC(=O)[CH3] OCOCH<sub>3</sub> H<sub>3</sub>COCO
COCH3 C(=O)[CH3] COCH<sub>3</sub> H<sub>3</sub>COC
OCH3 O[CH3] OCH<sub>3</sub> H<sub>3</sub>CO
OC2H5 O[CH2][CH3] OC<sub>2</sub>H<sub>5</sub> H<sub>5</sub>C<sub>2</sub>O
CHCl2 [CH](Cl)Cl CHCl<sub>2</sub> Cl<sub>2</sub>HC""")

if THEME == "light":
    W = (0.07, 0.09, 0.15)
    PAL = {6: W, 1: W, 0: (0.55, 0.33, 0.0), 8: (0.78, 0.13, 0.13), 7: (0.11, 0.30, 0.85),
           17: (0.08, 0.50, 0.24), 35: (0.70, 0.33, 0.04), 16: (0.60, 0.42, 0.02),
           11: (0.43, 0.16, 0.85), 12: (0.43, 0.16, 0.85), 53: (0.50, 0.15, 0.70), 13: (0.3, 0.3, 0.3)}
else:
    W = (0.95, 0.96, 0.99)
    PAL = {6: W, 1: W, 0: (0.98, 0.85, 0.45), 8: (1.0, 0.56, 0.56), 7: (0.56, 0.76, 1.0),
           17: (0.45, 0.92, 0.58), 35: (0.99, 0.66, 0.36), 16: (1.0, 0.86, 0.36),
           11: (0.80, 0.72, 1.0), 12: (0.80, 0.72, 1.0), 53: (0.86, 0.58, 1.0), 13: (0.8, 0.8, 0.8)}

FONT = "/usr/share/fonts/opentype/inter/Inter-SemiBold.otf"

def _prio(a):
    lab = a.GetProp("_displayLabel") if a.HasProp("_displayLabel") else a.GetSymbol()
    if a.GetAtomicNum() == 8: return 0
    if lab.startswith("O"): return 1
    if a.GetAtomicNum() == 7 or lab.startswith("N"): return 2
    return 5

def _orient(m):
    """First 6-ring upright, most important substituent (O > OR > N > rest) pointing up."""
    rings = [r for r in m.GetRingInfo().AtomRings() if len(r) == 6]
    if not rings:
        rdDepictor.Compute2DCoords(m); return
    ring = list(rings[0]); rs = set(ring); best = None
    for i in ring:
        for nb in m.GetAtomWithIdx(i).GetNeighbors():
            if nb.GetIdx() in rs: continue
            cand = (_prio(nb), ring.index(i), i, nb.GetIdx())
            if best is None or cand < best: best = cand
    start = best[2] if best else ring[0]
    k = ring.index(start); ring = ring[k:] + ring[:k]
    L = 1.5; cm = {}
    for n, idx in enumerate(ring):
        ang = math.radians(90 - 60 * n)
        cm[idx] = Point2D(L * math.cos(ang), -L + L * math.sin(ang))
    if best: cm[best[3]] = Point2D(0, L)
    rdDepictor.Compute2DCoords(m, coordMap=cm)

def mol_svg(smi, resonance=False):
    if resonance:  # keep the Kekulé form exactly as written
        m = Chem.MolFromSmiles(smi, sanitize=False); m.UpdatePropertyCache(strict=False)
        Chem.SanitizeMol(m, Chem.SanitizeFlags.SANITIZE_ALL
                         ^ Chem.SanitizeFlags.SANITIZE_SETAROMATICITY
                         ^ Chem.SanitizeFlags.SANITIZE_KEKULIZE)
    else:
        m = Chem.MolFromSmiles(smi)
        if m is None: raise ValueError(f"bad SMILES: {smi}")
        m = rdAbbreviations.CondenseMolAbbreviations(m, ABBR, maxCoverage=0.7)
    for a in m.GetAtoms():
        if a.GetAtomicNum() == 0 and not a.HasProp("_displayLabel"):
            a.SetProp("atomLabel", "R")
    _orient(m)
    d = rdMolDraw2D.MolDraw2DSVG(-1, -1); o = d.drawOptions()
    o.clearBackground = False; o.fixedBondLength = 27; o.bondLineWidth = 1.7
    o.minFontSize = 12; o.maxFontSize = 14; o.padding = 0.08; o.additionalAtomLabelPadding = 0.1
    if os.path.exists(FONT): o.fontFile = FONT
    o.updateAtomPalette(PAL); o.setSymbolColour(W); o.setAnnotationColour(W); o.multipleBondOffset = 0.18
    rdMolDraw2D.PrepareAndDrawMolecule(d, m, kekulize=not resonance); d.FinishDrawing()
    return re.sub(r"<\?xml[^>]*\?>", "", d.GetDrawingText())

def mol_file(smi, resonance):
    key = hashlib.md5(f"{smi}|{resonance}|{THEME}".encode()).hexdigest()[:12]
    p = os.path.join(MOLDIR, f"m_{key}.svg")
    if not os.path.exists(p): open(p, "w").write(mol_svg(smi, resonance))
    return p

def chem(s):
    s = s.replace("S_N2", "S_{N}2").replace("S_N1", "S_{N}1")
    s = re.sub(r"_\{([^}]*)\}", r"<sub>\1</sub>", s)
    s = re.sub(r"_(\d+)", r"<sub>\1</sub>", s)
    s = re.sub(r"\^\{([^}]*)\}", r"<sup>\1</sup>", s)
    s = re.sub(r"\^-", "<sup>−</sup>", s)
    return re.sub(r"\^([+−‡])", r"<sup>\1</sup>", s)

def species(tok):
    tok = tok.strip(); lab = ""
    m = re.search(r"(?<![_^])\{([^{}]*)\}\s*$", tok)
    if m: lab, tok = m.group(1), tok[:m.start()]
    mm = re.fullmatch(r"(\d+)?\s*\$(!?)([^$]+)\$", tok.strip())
    if mm:
        coef = f'<span class="coef">{mm.group(1)}</span>' if mm.group(1) else ""
        h = coef + f'<span class="sp mol"><img src="file://{mol_file(mm.group(3), bool(mm.group(2)))}"/>'
    else:
        h = f'<span class="sp"><span>{chem(tok)}</span>'
    if lab: h += f'<span class="lb">{chem(lab)}</span>'
    return h + "</span>"

def segment(seg):
    groups = []
    for grp in re.split(r"\s;;\s", seg.strip()):
        parts = [p for p in re.split(r"\s\+\s", grp.strip()) if p.strip()]
        groups.append('<span class="pl">+</span>'.join(species(p) for p in parts))
    return '<span class="gap"></span>'.join(groups)

def arrow(kind, cond):
    if kind == "<->": return '<span class="res">⟷</span>'
    bits = (cond or "").split("|"); top = bits[0]; bot = bits[1] if len(bits) > 1 else ""
    h = f'<span class="ar{" eq" if kind == "<=>" else ""}"><span class="c">{chem(top) or " "}</span><span class="l"></span>'
    if kind == "<=>": h += '<span class="l2"></span>'
    return h + f'<span class="c b">{chem(bot) or " "}</span></span>'

ARR = re.compile(r"\s+(-->|<=>|<->)(?:\[([^\]]*)\])?\s+")

def line(l):
    l = l.strip()
    if not l: return ""
    step = ""; m = re.match(r"#([^#]+)#\s*", l)
    if m: step = f'<span class="stp">{m.group(1)}</span>'; l = l[m.end():]
    note = ""
    if " // " in l:
        l, n = l.split(" // ", 1); note = f'<span class="nt">{chem(n)}</span>'
    toks = ARR.split(" " + l + " "); html = [step]
    for i in range(0, len(toks), 3):
        html.append(segment(toks[i]))
        if i + 1 < len(toks): html.append(arrow(toks[i + 1], toks[i + 2]))
    return f'<div class="ln">{"".join(html)}{note}</div>'

raw = open(src, encoding="utf-8").read()
raw = re.sub(r"<rx>(.*?)</rx>", lambda m: "".join(line(x) for x in m.group(1).split("\n")), raw, flags=re.S)

if BIONIC:  # bold word-starts in .bio prose only
    soup = BeautifulSoup(raw, "html.parser")
    SKIP_TAGS = {"sub", "sup", "b", "strong", "style", "script", "th"}
    SKIP_CLS = {"rx", "ln", "tag"}
    WORD = re.compile(r"[A-Za-z][A-Za-z'’]*")
    esc = lambda t: t.replace("&", "&amp;").replace("<", "&lt;").replace(">", "&gt;")
    def bion(text):
        res, last = [], 0
        for m in WORD.finditer(text):
            w = m.group()
            if sum(c.isupper() for c in w) >= 2 or len(w) < 2 or (len(w) == 2 and w[0].isupper()):
                continue  # formulas / element symbols (HCl, NaOH, Br)
            k = 1 if len(w) <= 3 else (2 if len(w) == 4 else (len(w) + 1) // 2)
            res.append(esc(text[last:m.start()])); res.append(f'<b class="bi">{w[:k]}</b>{w[k:]}')
            last = m.end()
        res.append(esc(text[last:]))
        return "".join(res)
    for el in soup.select(".bio"):
        for node in list(el.descendants):
            if not isinstance(node, NavigableString): continue
            bad = False
            for p in node.parents:
                if p is el: break
                if p.name in SKIP_TAGS or set(p.get("class") or []) & SKIP_CLS: bad = True; break
            if node.parent is not None and node.parent.name in SKIP_TAGS: bad = True
            if not bad: node.replace_with(BeautifulSoup(bion(str(node)), "html.parser"))
    raw = str(soup)

HTML(string=raw, base_url=os.path.dirname(os.path.abspath(src))).write_pdf(out)
print("ok", out)
```

## dark.css
```css
@page { size: A4; margin: 13mm 13mm 15mm 13mm; background:#0b1120;
  @bottom-center { content: "CHAPTER NAME  —  " counter(page); font: 600 8pt Inter, sans-serif; color: #8494ab; } }
@page:first { margin: 0; @bottom-center { content: none; } }
:root{
  --alc:#60a5fa; --alc-bg:#102142; --alc-mid:#3563a8;
  --phe:#fb923c; --phe-bg:#2a190b; --phe-mid:#9a5520;
  --eth:#34d399; --eth-bg:#0b271f; --eth-mid:#21805f;
  --gen:#a78bfa; --gen-bg:#1d1840; --gen-mid:#6151b0;
  --ink:#f1f5f9; --body:#e2e8f0; --mute:#a5b4c8; --line:#2a3650;
  --trap:#f87171; --trap-bg:#33141a; --hook:#facc15; --hook-bg:#2b2308;
}
*{box-sizing:border-box}
html,body{background:#0b1120}
body{font-family:Inter, sans-serif; color:var(--body); font-size:10.6pt; line-height:1.65; margin:0}
.bio{font-weight:400; color:var(--body)}
.bio b:not(.bi), .bio strong{font-weight:800; color:var(--c,#f8fafc)}
b.bi{font-weight:700; color:#f8fafc}
sub,sup{font-size:70%; line-height:0}
h1,h2,h3{font-family:"Inter Display",Inter,sans-serif; margin:0}
i{color:#e2e8f0}
.cover{height:297mm; padding:24mm 20mm; background:#060a14; color:#f8fafc; page-break-after:always; position:relative}
.cover .kicker{font:700 9.5pt Inter, sans-serif; letter-spacing:.2em; color:#94a3b8; text-transform:uppercase}
.cover h1{font-size:48pt; line-height:1.02; margin:6mm 0 4mm; font-weight:800; color:#f8fafc}
.cover h1 .a{color:var(--alc)} .cover h1 .p{color:var(--phe)} .cover h1 .e{color:var(--eth)}
.cover .sub{font-size:12.5pt; max-width:155mm}
.legend{margin-top:12mm; display:flex; gap:4mm}
.legend div{border-radius:4mm; padding:4mm 5mm; flex:1; font-size:9.5pt; color:var(--body); border:1.5px solid}
.legend .t{font:800 12pt "Inter Display", sans-serif; display:block; margin-bottom:1mm}
.lg-a{background:var(--alc-bg); border-color:var(--alc-mid)!important} .lg-a .t{color:var(--alc)}
.lg-p{background:var(--phe-bg); border-color:var(--phe-mid)!important} .lg-p .t{color:var(--phe)}
.lg-e{background:var(--eth-bg); border-color:var(--eth-mid)!important} .lg-e .t{color:var(--eth)}
.howto{margin-top:10mm; border:1px solid #33415c; background:#0b1120; border-radius:4mm; padding:5mm 6mm; font-size:10pt}
.howto h3{font-size:11pt; color:#f8fafc; margin-bottom:2mm}
.howto ul{margin:0; padding-left:5mm} .howto li{margin:1.4mm 0}
.demo{margin-top:6mm}
.cover .foot{position:absolute; bottom:16mm; left:20mm; right:20mm; font-size:9pt; color:#8494ab}
.tag{display:inline-block; font:800 7.3pt Inter, sans-serif; letter-spacing:.08em; padding:.6mm 2mm; border-radius:2mm; color:#0b1120; vertical-align:1px; margin-right:1.5mm}
.tag.t-hook{background:var(--hook)} .tag.t-trap{background:var(--trap)} .tag.t-tldr{background:#e2e8f0}
.tag.t-check{background:var(--gen)} .tag.t-ex{background:#38bdf8} .tag.t-mech{background:#f472b6}
.sec{margin-top:11mm}
.sec-head{border-radius:4mm; padding:4.5mm 6mm; margin-bottom:5.5mm; color:#0b1120; break-after:avoid; break-inside:avoid}
.sec-head .n{font:800 8.5pt Inter, sans-serif; letter-spacing:.18em; opacity:.75}
.sec-head h2{font-size:21pt; font-weight:800; line-height:1.1; color:#0b1120}
.sec-head .q{font-size:9.8pt; opacity:.85; margin-top:.8mm; font-weight:600}
.alc .sec-head{background:var(--alc)} .phe .sec-head{background:var(--phe)} .eth .sec-head{background:var(--eth)} .gen .sec-head{background:var(--gen)}
.alc{--c:var(--alc);--cb:var(--alc-bg);--cm:var(--alc-mid)} .phe{--c:var(--phe);--cb:var(--phe-bg);--cm:var(--phe-mid)}
.eth{--c:var(--eth);--cb:var(--eth-bg);--cm:var(--eth-mid)} .gen{--c:var(--gen);--cb:var(--gen-bg);--cm:var(--gen-mid)}
h3{font-size:13pt; font-weight:700; margin:8mm 0 3mm; padding-left:3mm; border-left:4px solid var(--c); color:#f1f5f9; break-after:avoid}
p{margin:2.2mm 0}
ul{margin:2mm 0; padding-left:5mm} li{margin:1.8mm 0}
.A{color:var(--alc); font-weight:700} .P{color:var(--phe); font-weight:700} .E{color:var(--eth); font-weight:700}
.tldr{background:#16213a; border:1px solid #3a4a6b; border-left:5px solid var(--c); border-radius:3mm; padding:4mm 5mm; margin:0 0 5.5mm; break-inside:avoid; break-before:avoid}
.tldr.bio b:not(.bi){color:#fde68a}
.hook{background:var(--hook-bg); border:1.5px dashed #d4ab12; border-radius:3mm; padding:3.8mm 5mm; margin:4.5mm 0; break-inside:avoid}
.hook.bio b:not(.bi){color:#fde047}
.trap{background:var(--trap-bg); border-left:5px solid var(--trap); border-radius:0 3mm 3mm 0; padding:3.8mm 5mm; margin:4.5mm 0; break-inside:avoid}
.trap.bio b:not(.bi){color:#fca5a5}
.ex{background:#0c2232; border:1.5px solid #2b8fb0; border-radius:3mm; padding:3.8mm 5mm; margin:4.5mm 0; break-inside:avoid}
.ex.bio b:not(.bi){color:#7dd3fc}
.card{background:var(--cb); border:1.5px solid var(--cm); border-radius:3mm; padding:3.8mm 5mm; margin:4mm 0; break-inside:avoid}
.card .ct{font:800 11pt "Inter Display", sans-serif; color:var(--c); margin-bottom:2mm}
.grid2{display:flex; gap:4.5mm} .grid2>*{flex:1; margin:0}
.grid3{display:flex; gap:4.5mm} .grid3>*{flex:1; margin:0}
.rx{background:#050913; border:1.2px solid var(--cm); border-radius:2.5mm; padding:3.2mm 4.5mm; margin:3.5mm 0; break-inside:avoid}
.rx .rt{font:800 7.5pt Inter, sans-serif; letter-spacing:.04em; color:var(--c); margin-bottom:1mm}
.ln{display:block; margin:2.4mm 0; font-weight:600; color:#f1f5f9; font-size:10pt; line-height:1.3}
.ln + .ln{border-top:1px dashed #2a3a5c; padding-top:2.4mm}
.ln > span{vertical-align:middle; margin:0.6mm 0.6mm}
.sp{display:inline-block; text-align:center}
.sp > span{display:block}
.sp .lb{font-size:7.3pt; font-weight:700; color:var(--c); margin-top:.5mm; line-height:1.1}
.pl{color:var(--mute); font-weight:700; display:inline-block; padding:0 .4mm}
.ar{display:inline-block; text-align:center; min-width:15mm; padding:0 1mm}
.ar .c{display:block; font-size:7.2pt; font-weight:700; color:#fbbf24; line-height:1.15; white-space:nowrap}
.ar .c.b{color:#c3cede}
.ar .l, .ar .l2{display:block; height:0; border-top:1.6px solid var(--c); position:relative; margin:.9mm 0}
.ar .l2{margin-top:-0.1mm}
.ar.eq .l{margin-bottom:.9mm}
.ar .l::after{content:""; position:absolute; right:-1px; top:-3.4px; border-left:5px solid var(--c); border-top:2.6px solid transparent; border-bottom:2.6px solid transparent}
.ar .l2::before{content:""; position:absolute; left:-1px; top:-3.4px; border-right:5px solid var(--c); border-top:2.6px solid transparent; border-bottom:2.6px solid transparent}
.stp{font:800 7.3pt Inter, sans-serif; color:#0b1120; background:var(--c); border-radius:1.5mm; padding:.6mm 1.6mm; margin-right:1mm; letter-spacing:.05em}
.nt{font-size:8.4pt; color:var(--mute); font-weight:500; margin-left:1.5mm}
.steps{counter-reset:s; list-style:none; padding:0}
.steps li{counter-increment:s; position:relative; padding-left:9mm; margin:2.6mm 0}
.steps li::before{content:counter(s); position:absolute; left:0; top:-.2mm; width:5.6mm; height:5.6mm; border-radius:50%; background:var(--c); color:#0b1120; font:800 8.5pt Inter, sans-serif; text-align:center; line-height:5.6mm}
table{width:100%; border-collapse:separate; border-spacing:0; margin:4mm 0; font-size:10pt; break-inside:avoid; color:#dbe3ee}
th{background:var(--c); color:#0b1120; text-align:left; padding:2.2mm 3mm; font-weight:800}
th:first-child{border-top-left-radius:2.5mm} th:last-child{border-top-right-radius:2.5mm}
td{padding:2.2mm 3mm; border-bottom:1px solid var(--line); vertical-align:top}
tr:nth-child(even) td{background:var(--cb)}
.big{font:800 15pt "Inter Display", sans-serif; color:var(--c)}
.ladder{display:flex; gap:3mm; margin:3.5mm 0}
.ladder div{flex:1; text-align:center; border-radius:3mm; padding:3.5mm 2mm; background:var(--cb); border:1.5px solid var(--cm); font-size:9.8pt; color:#dbe3ee}
.ladder .big{display:block; font-size:17pt}
.chk{list-style:none; padding:0} .chk li::before{content:"☐ "; color:var(--gen); font-weight:800}
.ans{font-size:9pt; color:var(--mute); transform:rotate(180deg); text-align:right; margin-top:2mm}
.note{font-size:9.2pt; color:var(--mute)}
.avoid{break-inside:avoid}
.pb{break-before:page; margin-top:0}
.sp.mol img{display:block; margin:0 auto}
.sp.mol{vertical-align:middle}
.coef{display:inline-block; vertical-align:middle; font-weight:700; color:#f1f5f9; margin-right:1mm}
.gap{display:inline-block; width:7mm}
.res{display:inline-block; vertical-align:middle; font-family:"DejaVu Sans", sans-serif; font-size:15pt; color:var(--c); padding:0 1mm}
.gal .ln{text-align:center}
table.cheat{font-size:9.4pt; line-height:1.35} table.cheat td{padding:1.45mm 3mm}
```

## light.css (append after dark.css for the light / print version)
```css
@page { background:#ffffff; @bottom-center { color:#64748b; } }
:root{
  --alc:#1e40af; --alc-bg:#dce8ff; --alc-mid:#5b8def;
  --phe:#b23a06; --phe-bg:#ffe6cf; --phe-mid:#e98a45;
  --eth:#046c4e; --eth-bg:#d3f3e4; --eth-mid:#3fb488;
  --gen:#5b21b6; --gen-bg:#e9e2ff; --gen-mid:#9478e6;
  --ink:#0b1220; --body:#111827; --mute:#3f4b5e; --line:#b8c2d1;
  --trap:#b91c1c; --trap-bg:#fde0e0; --hook:#a16207; --hook-bg:#fff3c4;
}
html,body{background:#ffffff}
.bio b:not(.bi), .bio strong{color:var(--c,#0f172a)}
b.bi{color:#0b1220}
i{color:#0f172a}
.cover{background:#ffffff; color:#0f172a; border-bottom:6mm solid var(--gen)}
.cover .kicker{color:#526077}
.cover h1{color:#0f172a}
.howto{border:1.5px solid #94a3b8; background:#eef2f7}
.howto h3{color:#0f172a}
.cover .foot{color:#64748b}
.tag{color:#ffffff}
.tag.t-tldr{background:#0f172a} .tag.t-ex{background:#0e7490} .tag.t-mech{background:#be185d}
.sec-head{background:var(--cb)!important; color:var(--c); border:2px solid var(--cm); border-left:9px solid var(--c)}
.sec-head h2{color:#0f172a}
.sec-head .n{color:var(--c); opacity:1}
.sec-head .q{color:#334155; opacity:1}
h3{color:#0f172a}
.tldr{background:#e2e8f0; border:1.5px solid #94a3b8; border-left:6px solid var(--c)}
.tldr.bio b:not(.bi){color:#0f172a}
.hook{border:2px dashed #a16207}
.hook.bio b:not(.bi){color:#854d0e}
.trap.bio b:not(.bi){color:#991b1b}
.ex{background:#d9f3f9; border:2px solid #2596b8}
.ex.bio b:not(.bi){color:#0e5a70}
.rx{background:#ffffff; border:2px solid var(--cm)}
.card{border-width:2px}
.ladder div{border-width:2px}
.trap{border-left-width:6px}
.ln{color:#0f172a}
.ln + .ln{border-top:1px dashed #a9b4c4}
.ar .c{color:#9a4a00}
.ar .c.b{color:#334155}
.stp{color:#ffffff}
.steps li::before{color:#ffffff}
table{color:#1e293b}
th{color:#ffffff}
.ladder div{color:#1e293b}
.coef{color:#0f172a}
.legend div{color:#1e293b}
```

## compact.css (append last, only if the reader chose Compact)
```css
body{font-size:10.2pt; line-height:1.45}
p{margin:1.5mm 0} ul{margin:1mm 0} li{margin:1mm 0}
.sec{margin-top:7mm} .sec-head{margin-bottom:4mm}
h3{margin:5mm 0 2mm}
.tldr{padding:3.2mm 4.5mm; margin:0 0 4mm}
.hook,.trap,.ex{padding:2.8mm 4.2mm; margin:3mm 0}
.card{padding:2.8mm 4.2mm; margin:2.5mm 0}
.rx{padding:2.2mm 3.2mm; margin:2mm 0}
.ln{margin:1.6mm 0} .ln + .ln{padding-top:1.6mm}
.steps li{margin:1.8mm 0}
table{margin:2.5mm 0; font-size:9.5pt} td,th{padding:1.5mm 2.5mm}
.ladder{gap:2mm; margin:2mm 0}
```

## Gotchas
- WeasyPrint: schemes rely on inline-block layout (flex with width:100% arrows breaks). `.sec-head` and `.tldr` avoid page breaks so headers never orphan; `.pb` forces the cheat sheet and self-check onto fresh pages.
- Never use Unicode sub/superscript digits; the notation emits `<sub>`/`<sup>`.
- Check every SMILES against the textbook drawing (o/m/p positions, charges). Invalid SMILES raise an error. Resonance (`$!`) strings need explicit `[CH-]`, `[O+]`, `[OH+]` and a Kekulé ring.
- A light-theme build needs its own molecule colours: always pass `NOTES_THEME=light` (cached SVGs are keyed by theme).
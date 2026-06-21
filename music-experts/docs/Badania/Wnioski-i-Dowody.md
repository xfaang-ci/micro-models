---
type: nauka-wnioski
title: "Wnioski i dowody — micro-models (muzyka + kompozycja)"
status: zywy-dokument
data: 2026-06-21
created_at: 2026-06-21
author: Arkadiusz Słota
---

# Wnioski i dowody

> Skonsolidowany rejestr: **twierdzenie → dowód (liczba/źródło) → werdykt.** Zasada: żadnej tezy bez dowodu. Żywy dokument.

## ✅ Udowodnione (z dowodem)
| Twierdzenie | Dowód | Werdykt |
|---|---|---|
| Char-LM uczy się struktury muzycznej (metrum, tonacje, kadencje) z samej predykcji następnego znaku | 4 modele generują poprawne ABC; respektują metrum i sygnatury (np. `^F` w G-dur) — bez teorii w kodzie | ✅ |
| Czyszczenie danych obniża perplexity | jig: ppl **3,88 → 3,80** po usunięciu szumu akordowego | ✅ zmierzone |
| `[ace]` musi zostać z nawiasami, by grało jako akord | test: 2 akordy w ABC → 2 akordy w MIDI | ✅ |
| Mechanizm stitchu jest bezstratny (E0) | g=identyczność == baseline co do cyfry; g losowy (ppl **122**) → trening 16K param → ppl **3,83** (Δ+0,03) | ✅ |
| **Ensemble ekspertów bije pojedyncze modele** (zadanie mieszane) | zbalansowany held-out: ensemble **5,15** < reel 5,87 < walc 6,20 | ✅ zmierzone |

## ❌ NIE udowodnione (jeszcze)
| Hipoteza | Co wyszło | Status |
|---|---|---|
| **Stitch reprezentacji bije ensemble** (KMS1) | E1: stitch **5,18 ≈ ensemble 5,15** (liniowy g, post-hoc, bez kontraktu) | ❌ niepotwierdzone tą drogą |
| Most n-gram→transformer (NPLM, kNN-LM jako ogniwa) | eksperymenty niezrobione | ⏳ planowane |
| Mała skala (Bach 79K, Slayer ~150 utw.) wystarcza na mocny model | za mało danych | ⏳ |

## 🔬 Lekcje metodologiczne
- **Pomiar trzeba sprawdzać:** pierwszy E1 dał „stitch bije ensemble" — **artefakt** (val = czysty reel ze sklejenia korpusów). Po zbalansowaniu: ensemble nie gorszy. *Setup ewaluacji = część dowodu.*
- **Negatywny wynik to wynik:** prosty baseline (ensemble) jest mocny; złożoność (stitch) musi **zasłużyć** na siebie.
- **Bach ppl 2,09 nieporównywalne** z jig 3,80 (mniejszy słownik + bardzo powtarzalne dane).

## 📚 Referencje zweryfikowane u źródeł (deep-research: 13/13 charakterystyk trafnych)
ZipIt! (ICLR'24) · Git Re-Basin (ICLR'23) · model stitching (Lenc-Vedaldi CVPR'15; Bansal NeurIPS'21) · SN-Net (CVPR'23) · VQ-VAE (NeurIPS'17) · SAE / Towards Monosemanticity (Bricken, Cunningham '23) · Rosetta Neurons (ICCV'23) · AoANet (ICCV'19) · deep ensembles (NeurIPS'17) · MEMIT (ICLR'23) + Hase i in. (NeurIPS'23) · NPLM (JMLR'03) · kNN-LM (ICLR'20, ppl 15,79) · Infini-gram (COLM'24).

## ➡️ Następne dowody do zdobycia
1. **E0.5** — czy niezależne modele (inny seed, ta sama architektura) są liniowo wyrównywalne (geometria).
2. **Wymuszony kontrakt (KMS2)** — trenować oba przeciw **wspólnej zamrożonej głowicy** → czy wtedy stitch bije ensemble.
3. **Bogatszy mapper (KMT3)** — słownik/SAE zamiast liniowego.

## Powiązania
[[Kompozycja-Eksperymenty]] · [[Kompozycja-Malych-Modeli]] · [[Capstone-Muzyka-LOG-Realizacja]] · [[Research-NGram-vs-MiniTransformer]] · [[Badania-INDEX]]

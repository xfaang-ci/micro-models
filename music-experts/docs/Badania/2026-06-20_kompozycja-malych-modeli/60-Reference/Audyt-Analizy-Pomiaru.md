---
type: nauka-reference
id: KM-AUDYT
title: "Dialektyka audytu — czy poprawnie analizujemy wyniki i pobieramy dane?"
status: zywy-dokument
data: 2026-06-21
created_at: 2026-06-21
author: Arkadiusz Słota
---

# Dialektyka audytu — czy poprawnie analizujemy i pobieramy dane?

> Meta-warstwa: nie audytujemy *wyników*, tylko **sposób, w jaki je mierzymy, pobieramy i interpretujemy**. Tu siedzą ciche błędy. Adwersarialny recenzent próbuje OBALIĆ naszą analizę (jak „najmocniejszy recenzent ma exit code", zastosowane do siebie). Dokument jest **wielokrotnego użytku** — przepuszczamy przez niego każdy wynik.

## Dialektyka
- **Teza:** analizujemy rzetelnie — liczby są prawdziwe, metryka mierzy to, co myślimy, interpretacja jest uprawniona.
- **Antyteza:** pomiar/pobieranie/interpretacja mogą kłamać na wiele sposobów (artefakty danych, zły null, słaba statystyka, metryka skażona, framing ≠ to-co-zmierzono).
- **Synteza:** każdy wynik przechodzi **checklistę adwersarialną** (niżej) ZANIM go ogłosimy; znaleziska triażujemy (krytyczny/istotny/drobny) i albo naprawiamy, albo dopisujemy zastrzeżenie. Werdykt = co przeżyło, co osłabione, co wycofane.

## ♻️ Checklista adwersarialna (szablon — pytaj o KAŻDY wynik)
1. **Artefakt danych/probe:** czy dane wejściowe nie mają duplikatów / wycieku / sztucznej struktury, która daje wynik „za darmo"?
2. **Właściwy null:** z czym porównujemy? Czy null jest skalibrowany (np. losowy-vs-losowy, permutacja), a nie przypadkowy punkt?
3. **Statystyka:** ile niezależnych próbek (nie pozornych)? std próbkowe czy populacyjne? czy „pary" nie są zależne? czy jest test istotności/CI, czy tylko „na oko"?
4. **Skażenie metryki:** czy metryka nie mierzy czegoś trywialnego (struktura wejścia, architektura) zamiast efektu? **Porównaj metrykę z jej własnym nullem.**
5. **Framing = pomiar:** czy słowa opisu odpowiadają temu, co FAKTYCZNIE zmierzono (np. „konwergencja" vs „stabilność względem seeda")?
6. **Pułapki:** ceiling/floor, label leakage, single-domain, wąski zakres, confirmation bias („mierzyłem aż znalazłem metrykę, która rośnie"), porównania wielokrotne.

## 📋 Przypadek 1 — audyt E_CKA (2026-06-21)
Niezależny agent-adwersarz przejrzał `src/tools/cka.py` + dokumenty E_CKA. Znaleziska:

| # | Problem | Severity | Status |
|---|---|---|---|
| K1 | Probe = jeden blok **×3** → sztuczne sąsiedztwa zawyżały mutual-kNN | krytyczny | ✅ naprawione (probe realny, N=11 712) |
| K3 | „rośnie ponad szum" przy **n=3 zależnych parach** + std **populacyjne** | krytyczny | ✅ std próbkowe; ⏳ ≥5 niezależnych seedów + test nachylenia (GPU) |
| K2 | N małe + autoskorelowane; różne między skalami | istotny | ✅ N raportowane (11 712, wspólny dla skal); ⏳ subsampling pozycji |
| I1 | same-data+seed = **stabilność**, nie „platońska konwergencja" (PRH = różne dane) | istotny | ✅ przeformułowane |
| I2 | null „trenowany-vs-losowy" źle skalibrowany | istotny | ✅ dodany null losowy-vs-losowy |
| I3 | średnia po warstwach ukrywa profil (czy nie tylko L0?) | istotny | ✅ raport per-warstwa (L0–L3 równomierne) |
| I4 | best-val checkpoint na różnych krokach miesza się z efektem skali | istotny | ⏳ fixed-step snapshot (dalej) |
| D1 | wzór CKA poprawny (centrowanie + HSIC) | — | ✅ OK (nie błąd) |

**Co audyt ZMIENIŁ (wniosek się odwrócił):**
- Przed: „mutual-kNN rośnie → kierunek PRH; CKA saturuje".
- Po (właściwy null + realny probe): **CKA jest czystym sygnałem** (trenowane 0,94 ≫ null losowy-losowy 0,44), a **mutual-kNN jest skażony** (losowy-vs-losowy 0,68 ≈ trenowane 0,66 → mierzy strukturę wejścia, nie wyuczoną zgodność). **kNN wycofany jako miara konwergencji.**
- Reframe: jig–jig-v2 = stabilność względem seeda; „konwergencja" tylko dla cross-styl (0,84 ≫ 0,44), i tak słaby surogat PRH.

**Werdykt:** twierdzenie o wspólnej geometrii **przeżyło na CKA** (i to mocniej — czysty null), twierdzenie o „kNN/PRH-trend" **wycofane jako artefakt**. To jest sukces metody: adwersarz zabił nasz headline, zostało to, co prawdziwe.

## Powiązania
[[Kompozycja-Eksperymenty]] · [[Emergencja-i-Wspolna-Reprezentacja]] · [[Wnioski-i-Dowody]] · [[Kompozycja-INDEX]]

# Slayer Micro-Models — raport stanu prac

> Wejście dla osób z zewnątrz. Prosto, krok po kroku, w kolejności jak idą prace.
> Zasada przez całość: **żadnej tezy bez dowodu; wyniki negatywne też publikujemy.**

## 1. Cel projektu
Budujemy bardzo małe sieci neuronowe (~0,8 mln parametrów, trening w minuty na CPU), na tyle proste,
by w pełni je zrozumieć. Pytanie badawcze: **czy wielu małych, wyspecjalizowanych modeli da się złożyć
w jeden, działający lepiej niż pojedynczy?** Dziedziną testową jest muzyka zapisana tekstowo (notacja ABC):
dane są darmowe, a poprawność łatwo sprawdzić. Cel docelowy (osobny) to model dla budownictwa (pliki `.ifc`).

## 2. Materiał: eksperci
Kilka modeli, każdy na jednym stylu, ta sama architektura (4 warstwy, 4 głowy, `d_model=128`, kontekst 128 znaków, poziom znaków):

| Ekspert | Styl | Jakość (perplexity, niżej = lepiej) |
|---|---|---|
| jig | irlandzki taniec 6/8 | 3,80 |
| walc | 3/4 | — |
| reel | 4/4 | — |
| bach | sopran chorału | 2,09\* |

\* nieporównywalne wprost (mniejszy słownik, bardziej powtarzalne dane). Każdy uczy się sam z siebie
struktury muzycznej (metrum, tonacja, kadencje) z samej predykcji następnego znaku — bez wpisanej teorii muzyki.

## 3. Eksperymenty (w kolejności)

### E0 — kontrola mechanizmu (self-stitch)
**Pytanie:** czy „szew" łączący modele jest bezstratny.
**Metoda:** rozciąć jeden model w środku, wstawić mały liniowy łącznik (mapper), resztę zamrozić; trenować tylko łącznik.
**Wynik:** łącznik odtwarza wynik bazowy (perplexity 3,80 → 3,83; wariant „identyczność" == baseline co do cyfry).
**Wniosek:** mechanizm działa. To **nie** dowód, że kompozycja pomaga — tylko że hydraulika jest poprawna.

### Ensemble — baseline do pobicia
**Metoda:** w każdym kroku uśrednić przewidywania dwóch modeli.
**Wynik** (zadanie mieszane, walc+reel po równo): ensemble **5,15** — lepszy niż każdy z osobna (reel 5,87; walc 6,20).
**Wniosek:** łączenie ekspertów realnie pomaga; to jest poprzeczka.

### E1 — łączenie reprezentacji (właściwa hipoteza)
**Metoda:** front jednego modelu + łącznik + tył drugiego; trenowany tylko łącznik.
Po drodze złapano i poprawiono błąd pomiaru (zbiór testowy był przekrzywiony ku jednemu stylowi).
**Wynik (po poprawie):** stitch **5,18 ≈ ensemble 5,15** — **nie pobił** baseline'u. Wynik negatywny, zaraportowany uczciwie.

### E_CKA — dlaczego nie pobił? (kluczowy)
**Pytanie:** czy niezależnie trenowane małe modele mają **wspólną geometrię reprezentacji**, czy różne.
**Metoda:** zmierzyć podobieństwo reprezentacji (CKA / mutual-kNN) między modelami, z baselinem losowym. ~godzina, zero treningu.

| para | CKA | mutual-kNN |
|---|---|---|
| self (sanity) | 1,000 | 1,000 |
| jig vs jig-v2 (te same dane, inny seed) | 0,973 | 0,822 |
| walc vs reel | 0,918 | 0,762 |
| jig vs walc | 0,851 | 0,648 |
| jig vs bach (najdalszy) | 0,685 | 0,488 |
| baseline losowy (null) | ~0,35 | — |

**Wynik:** geometrie są **wysoce wspólne**. To **obaliło** wcześniejsze wyjaśnienie remisu E1: nie chodzi o „różne geometrie"
(są wspólne), tylko o **redundancję** — modele kodują w dużej części to samo, więc nad uśrednianiem nie ma czego dodać.
**Wąskie gardło kompozycji to KOMPLEMENTARNOŚĆ ekspertów, nie wyrównanie reprezentacji.**
(Uczciwość: to jeden punkt skali; pełny wniosek wymaga trendu względem rozmiaru i baseline'u — oba uwzględnione.)

## 4. Co wiemy, czego nie

**Udowodnione (z liczbą):** mały char-LM uczy się struktury muzyki; czyszczenie danych obniża perplexity (3,88 → 3,80);
mechanizm szwu jest bezstratny; ensemble bije pojedyncze modele; niezależne maluchy mają wspólną geometrię (1 punkt skali).

**Jeszcze nie:** że łączenie reprezentacji bije ensemble; most n-gram→transformer; routing.

## 5. Następne kroki (kolejność wg wartość/wysiłek)
1. Sprawdzić, czy rozbieżność (wariancja) między ekspertami koreluje z błędem — bramka przed routingiem.
2. Wymuszony wspólny kontrakt (wspólny zamrożony front + głowa) na stylach w **tym samym metrum**
   (różne metrum = model oszukuje po nagłówku — pułapka pomiaru).
3. Bogatszy, nieliniowy łącznik — dopiero jeśli (2) pokaże sygnał.
4. Most n-gram → transformer (model NPLM) — równolegle, pewny rezultat na osobny mini-paper.
- Osobny kierunek: wiele modeli „grające razem" (polifonia, synchronizacja).

## 6. Po co to
Dwa cele: (a) **budować know-how zespołu Slayer** — tani, jawny, reprodukowalny poligon;
(b) **de-ryzykować metody** pod docelowy model budowlany (klasyfikacja / tworzenie / rozumienie `.ifc`),
gdzie poprawność jest sprawdzalna kodem (walidator = weryfikowalna nagroda).

---
*Pełne rozumowanie (teza ↔ antyteza → synteza, z weryfikacją źródeł) i kod eksperymentów: `music-experts/docs/` oraz `music-experts/src/compose/`.*

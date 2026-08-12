# TIMDER Multisensory Meditation Engine (TMME)

Model do tworzenia i integracji multisensorycznych strumieni medytacyjnych opartych na protokole TIMDER.

📘 Pełny opis architektury: [MODEL.md](MODEL.md)
🌐 Dokumentacja online: https://jbackk-lang.github.io/ — TIMDR + Λ–τ–ρ na danych

## Opis projektu

TMME łączy pięć kanałów sensorycznych w jeden strumień:

| Kanał | Silnik | Reprezentuje |
|---|---|---|
| muzyka | `RhythmEngine` | rytm skrętu |
| kolor | `ColorEngine` | warstwa Λ |
| światło | `LightEngine` | modulacja τ |
| obraz | `image_engine.generate_fractal()` | defekt ρ |
| sygnał | `signal_engine.generate_pulse()` | klucz J synchronizacji |

`Integrator` łączy wszystkie pięć w jeden model strumienia — **TIMDER-FLOW**.

Model nie generuje fizycznych bodźców (dźwięku, światła) — opisuje logikę, strukturę i parametry, które można podłączyć do DAW, sterownika LED, silnika VR/AR czy innego backendu wykonawczego.

## Stan projektu (uczciwie)

Silniki nie są jednakowo rozwinięte:

- **`ImageEngine` (`core/image_engine.py`)** i **`SignalEngine` (`core/signal_engine.py`)** przeszły przez redesign "v2" i mają realną logikę: `image_engine` renderuje fraktale Mandelbrota/Julii piksel po pikselu i zapisuje PNG bez zewnętrznych bibliotek (własny enkoder z `zlib`+`struct`); `signal_engine` próbkuje sygnał w czasie w 6 trybach (`stable`/`pulse`/`wave`/`burst`/`sweep`/`off`) ze statystykami (mean/max/duty cycle). Oba są modułami z funkcjami (`generate_fractal()`, `generate_pulse()`), **nie klasami**.
- **`RhythmEngine`, `ColorEngine`, `LightEngine`, `Integrator`** to proste klasy zwracające gotowe/zahardkodowane struktury danych (np. `ColorEngine.palette()` ma dokładnie dwie zdefiniowane palety) — działający szkielet do rozbudowy, nie docelowa logika generatywna.
- Katalogi `models/` (metamodele TIMDER: skręt, Λ–τ–ρ, klucz J, flow) to proste kontenery na parametry (`__init__` + `as_dict()`), bez własnej logiki.

To jest model/prototyp architektury, nie gotowy silnik audio-wizualny.

## Naprawione błędy

`examples/flow_script.py` (jedyny działający przykład w repo) **nie uruchamiał się** z dwóch niezależnych powodów, oba naprawione w tej wersji:

1. Uruchomienie `python examples/flow_script.py` z katalogu głównego repo kończyło się `ModuleNotFoundError: No module named 'core'` — Python dodaje do `sys.path` katalog *skryptu* (`examples/`), nie katalog główny repo, więc `from core.rhythm_engine import ...` nie miał jak zadziałać. Naprawione dopisaniem katalogu głównego do `sys.path` na początku skryptu.
2. Skrypt importował `ImageEngine` i `SignalEngine` jako klasy (`ImageEngine().fractal(...)`, `SignalEngine().pulse(...)`) — ale po redesignie "v2" oba moduły eksportują funkcje (`generate_fractal()`, `generate_pulse()`), nie klasy. Import kończył się `ImportError`. Ten sam nieaktualny wzorzec API był też w `MODEL.md` (sekcja 6, przykład pseudokodu) — poprawione w obu miejscach.

Zweryfikowane działającym uruchomieniem (odtworzone lokalnie z rzeczywistych plików repo, `python3 examples/flow_script.py` z katalogu głównego i z innego katalogu roboczego — oba przypadki przechodzą).

Struktura repozytorium opisana w poprzedniej wersji tego README wymieniała katalogi `tests/`, `docs/` oraz pliki `examples/minute_demo.json`, `examples/color_palette.json`, `examples/light_sequence.json` — żadne z nich nie istnieją w repo. Usunięte z opisu struktury poniżej, żeby był zgodny ze stanem faktycznym.

## Struktura repozytorium

```
TIMDER-Multisensory-Meditation-Engine/
│
├── core/                    # Silniki sensoryczne
│   ├── rhythm_engine.py     # RhythmEngine (klasa)
│   ├── color_engine.py      # ColorEngine (klasa)
│   ├── light_engine.py      # LightEngine (klasa)
│   ├── image_engine.py      # generate_fractal() (funkcja) -- realny render
│   ├── signal_engine.py     # generate_pulse() (funkcja) -- realne próbkowanie
│   └── integrator.py        # Integrator (klasa)
│
├── models/                  # Metamodele TIMDER (kontenery na parametry)
│   ├── skręt_model.py       # SkretModel
│   ├── LTR_model.py         # LTRModel (Λ-τ-ρ)
│   ├── J_key_model.py       # JKeyModel
│   └── flow_model.py        # FlowModel
│
├── examples/
│   └── flow_script.py       # jedyny działający przykład end-to-end
│
├── MODEL.md                 # pełny opis architektury i zasad modelu
├── README.md
└── LICENSE
```

## Instalacja

```
git clone https://github.com/jbackk-lang/TIMDER-Multisensory-Meditation-Engine.git
cd TIMDER-Multisensory-Meditation-Engine
```

Wymaga Python 3.10+ (`core/image_engine.py` używa adnotacji typu `str | None`, składnia z PEP 604).

## Przykład użycia

```
python3 examples/flow_script.py
```

Wypisuje `dict` reprezentujący pełny model strumienia TIMDER-FLOW: rytm, paletę kolorów, sekwencję światła, metadane fraktala (lub piksele + zapis PNG, jeśli wywołasz `generate_fractal(..., render=True, save_path="out.png")`), oraz próbki sygnału klucza J.

## Cel projektu

TMME jest fundamentem dla przyszłych implementacji multisensorycznych: generowania muzyki, sterowania światłem, modulacji kolorów, renderowania obrazów, synchronizacji sygnałów. Model jest neutralny technologicznie — może być użyty w dowolnym środowisku wykonawczym (DAW, sterownik LED, silnik VR/AR).

## Zastosowania

- aplikacje relaksacyjne
- systemy terapeutyczne
- VR/AR
- instalacje LED
- środowiska DAW
- eksperymenty naukowe nad multisensoryczną modulacją

## Status

Projekt jest modelem do rozwijania przez: programistów, twórców muzyki, projektantów światła, badaczy, twórców VR/AR, terapeutów sensorycznych.

## Licencja

MIT — patrz `LICENSE`.

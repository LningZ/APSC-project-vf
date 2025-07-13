# L\*_SHA Variants for Thermostat Case Study

This repository hosts three branches of the L\*_SHA algorithm, each representing a different variant of the learning approach, adapted for Uppaal compatibility and parallel performance testing.

## Branches Overview

### `lsha_ov` — Original Serial Version
- This is the original version using **serial trace generation** and **random seeds**.
- Compatible only with **Uppaal v4**.
- This version may not run properly on macOS with Apple Silicon chips.

### `lsha_non_parall` — Modified Serial Version for Uppaal 5
- Adapted for **Uppaal v5** because macOS (M1/M2/Pro/Max chips) does not support Uppaal 4.
- Code structure is updated to match Uppaal 5’s format.
- Uses **fixed seed** for trace generation to enable performance comparison with the parallel version.
- Logic is still **serial**, no concurrency.

### `lsha_parall` — Parallel Version
- Parallel trace generation and processing.
- Uses **fixed seed** for reproducibility.
- Fully adapted to **Uppaal v5** format.
- Recommended as the **main branch** for performance evaluation.

---

## How to Run

### 0. Install dependencies

```bash
pip install -r requirements.txt
export PYTHONPATH="${PYTHONPATH}:$(pwd)"
```


### 1. Install Uppaal v5
Download Uppaal v5 from the official website:  
https://uppaal.org/downloads/  
(Required because Uppaal v4 is incompatible with Apple Silicon machines.)

---

### 2. Modify `config.ini` with local paths

Edit the file:

```
sha_learning/resources/config.ini
```

Set the following absolute paths to match your machine:

```ini
[TRACE GENERATION]
UPPAAL_PATH = /Applications/Dev/uppaal-bin/bin
UPPAAL_SCRIPT_PATH = /Users/yourname/.../verify.sh
UPPAAL_MODEL_PATH = /Users/yourname/.../thermostat.xml
UPPAAL_QUERY_PATH = /Users/yourname/.../thermostat.q
UPPAAL_OUT_PATH = /Users/yourname/.../upp_results/{}.txt
```

---

### 2b. Configure learning options

In `config.ini`, the following parameters control learning behavior:

```ini
[GENERAL]
CASE_STUDY = THERMO             ; Use "THERMO" for thermostat case
CS_VERSION = 1                  ; Use values from 1 to 6 (ROOM1 to ROOM6)
N_min = 100                     ; Min. number of samples before confirming a row entry
INITIAL_SEED = 1000             ; Fixed seed for deterministic trace generation
RESAMPLE_STRATEGY = UPPAAL     ; Use "UPPAAL" (default) or "SIM"


---

### 3. Run the code from terminal

```bash
python3 -m sha_learning.learn_model config.ini
```

If you encounter errors, try:

```bash
python3 -m sha_learning.learn_model config.ini ROOM1 2012-12-01 2013-01-01
```

- `ROOM1` can be replaced with `ROOM1` to `ROOM6`, depending on the test case.

---

## How to Compare Serial vs. Parallel Versions

### For successful convergence and learned models:
1. Switch to `lsha_non_parall` and `lsha_parall` branches.
2. In each branch, edit `config.ini` to:
   - Set `CS_VERSION = 1`
   - Try values for `N_min = 20, 50, 100, 150, 200, 300`
3. Run both versions.
4. Compare output `.txt` files located at:

```
resources/learned_sha/THERMO_UPPAAL_1.txt
```

You can compare:
- `--OBSERVABLE EVENTS--`
- `--LEARNED DISTRIBUTIONS--`
- `--FINAL OBSERVATION TABLE--`
- `--PERFORMANCE DATA--`

All outputs should match, except **execution time** due to parallelism

---

### ⚠️ For non-converging cases with timing focus:
- Try `CS_VERSION = 2, 3, ..., 6`
- Keep testing with increasing `N_min = 20 → 300`
- Even if `.txt` files are not generated (due to non-convergence), observe **training time directly in terminal** logs.
- This still reflects performance gain from parallelization (e.g., fewer seconds for same number of iterations).

---

### Modify Seed to Compare Effects

In both branches, open:

```ini
[GENERAL]
INITIAL_SEED = 1000
```

Change the seed to a different value **in both config files** to ensure consistent trace generation.

To **restore random seed mode**, open:

```python
sha_learning/learning_setup/trace_gen.py
```

And replace `get_traces_uppaal()` with the commented random-seed version provided in the file.

---

## Testing Other Case Studies (like HRI)

- The **HRI case** cannot be run on your Mac with Uppaal 5, because the HRI XML model is not compatible.
- If you want to test HRI:
  - Use **Uppaal 4**
  - Set in `config.ini`:
    ```ini
    CASE_STUDY = HRI
    UPPAAL_MODEL_PATH = .../hri-w_ref.xml
    UPPAAL_QUERY_PATH = .../hri-w_ref1.q
    ```
  - Note: may still encounter version format issues.

---

## Author
This repository was adapted and tested on **macOS (M1 Pro)** by **Luning Zhu**.

This implementation is based on the original method developed by Livia Lestingi (Politecnico di Milano).

The source code is available at github.com/LesLivia/lsha, and is reused in my customized branch lsha_vo.
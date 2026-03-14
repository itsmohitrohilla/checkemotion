<p align="center">
  <img src="logo.png" alt="emotion-ai logo" width="180"/>
</p>

<h1 align="center">emotion-ai</h1>

<p align="center">
  <strong>Give your Python functions feelings.</strong><br/>
  Assign emotional personalities to functions based on their code structure and runtime behaviour — then watch them talk to each other.
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT"/></a>
  <img src="https://img.shields.io/badge/python-3.11%2B-brightgreen.svg" alt="Python 3.11+"/>
  <img src="https://img.shields.io/badge/dependencies-zero-success.svg" alt="Zero dependencies"/>
  <img src="https://img.shields.io/badge/stdlib-only-informational.svg" alt="Pure stdlib"/>
  <img src="https://img.shields.io/badge/emotions-500%2B-ff69b4.svg" alt="500+ emotions"/>
</p>

---

## What Is emotion-ai?

**emotion-ai** is a pure-Python observability and developer-experience library that analyses your functions — statically at decoration time via AST, and dynamically at every call — to assign each one an **emotional personality** from a catalogue of **500+ human emotions**.

It gives you:

| Feature | Description |
|---|---|
| 🎭 **Emotion assignment** | Each function gets an emotion based on its code structure: complexity, keywords, recursion, async, error-handling, etc. |
| 🌈 **Rich colorful CLI** | Every call prints ANSI 256-color output: emotion chip, intensity bar, valence badge, wall time, CPU time, memory, error count |
| 💬 **Inter-function conversation** | When two `conversation=True` functions run concurrently, they talk to each other in the terminal |
| 📊 **Live dashboard** | `show_dashboard()` prints a full health report: metrics, call graph, social relationships, star rating |
| 🔄 **Emotion drift** | Emotions evolve every 10 calls based on error rate, latency, and frequency |
| 🤝 **Social relationships** | Functions develop feelings toward each other: friendship, jealousy, respect, anger, pity… |
| 🔬 **Deep profiling** | Wall time, CPU time, heap delta, heap peak via `tracemalloc` — O(1) overhead per call |
| 🕸️ **Call graph** | Static AST scan builds a project-wide call graph; see who calls whom |

---

## Quick Start

```python
from emotion import emotion

@emotion
def calculate_total(numbers: list) -> float:
    return sum(n for n in numbers if n > 0)

calculate_total([1, 2, 3, 4, 5])
```

**What you see in the terminal:**

```
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
  😊  calculate_total  ▸  glad  [joy]  ░····  ▲ +
    ▸ awakened  Sums positive numbers in a list and returns the total.

  │ ✓ done  😊 calculate_total  #1  ⏱ 0.1ms  cpu 0.01ms  mem +0.0KB/4200KB  cc=2  err=0
```

- The **registration banner** fires once when `@emotion` is applied — showing the assigned emotion, intensity bar, and valence
- The **metrics row** fires on every call — showing call number, timing, memory, complexity, and error count

---

## Installation

No install required. Pure Python 3.11+, zero external dependencies. Clone and run:

```bash
git clone https://github.com/your-org/emotion-ai.git
cd emotion-ai
python examples/quickstart.py
```

Or install as a package into your project:

```bash
pip install .
```

**Optional LLM descriptions** — install [Ollama](https://ollama.ai) locally and pull one of these models for AI-generated, human-readable function descriptions. The library auto-detects it and falls back to AST-based descriptions silently if Ollama is not running.

```bash
ollama pull phi3.5:mini    # recommended — fast and accurate
# or: llama3.2:1b  tinyllama  phi3:mini
```

---

## The `@emotion` Decorator

### Three call forms — all equivalent in behaviour

```python
# Form 1: bare — no parentheses
@emotion
def my_function(x):
    return x * 2

# Form 2: empty parentheses
@emotion()
def my_function(x):
    return x * 2

# Form 3: with options
@emotion(enabled=True, conversation=True, verbose=True)
def my_function(x):
    return x * 2
```

### Options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `enabled` | `bool` | `True` | `False` → function is returned unchanged, zero overhead |
| `conversation` | `bool` | `False` | Enable inter-function dialogue on concurrent execution |
| `verbose` | `bool` | `True` | `False` → silences all terminal output (metrics still collected) |

```python
# Completely bypass the library for this function
@emotion(enabled=False)
def legacy_function(data):
    return data

# Collect metrics silently — no terminal output
@emotion(verbose=False)
def high_frequency_tick():
    ...
```

---

## How Emotions Are Assigned

### Static analysis (at decoration time)

`@emotion` parses each function's source code with Python's `ast` module and extracts a **15-feature vector**:

| Index | Feature | Example signal |
|---|---|---|
| 0 | `branch_count` | `if / for / while / except` nodes |
| 1 | `line_count` | non-blank, non-decorator lines |
| 2 | `param_count` | all parameter kinds |
| 3 | `has_recursion` | self-call detected |
| 4 | `has_async` | `async def` |
| 5 | `has_try_except` | `try` block present |
| 6 | `has_yield` | `yield` or `yield from` |
| 7 | `cyclomatic_complexity` | McCabe score |
| 8 | `nesting_depth` | maximum control-flow depth |
| 9 | `return_count` | number of `return` statements |
| 10–14 | keyword flags | name hints: dramatic / humble / excited / nervous / sarcastic |

These 15 features are scored against **46 emotion families** (joy, anger, serenity, anxiety, resilience, intellectual, etc.) and the winning family picks a specific emotion at the right **intensity level** (1–5).

### Example mappings

| Function characteristics | Assigned emotion |
|---|---|
| Short, simple, named `get_*` | `restful`, `calm`, `content` |
| Many branches, high complexity | `overwhelmed`, `tense`, `complex` |
| `try/except` + recursive | `anxious`, `resilient`, `persistent` |
| Named `delete_*` or `kill_*` | `furious`, `decisive`, `destructive` |
| Named `generate_*` or `craft_*` | `creative`, `inspired`, `inventive` |
| `async def` with `yield` | `flowing`, `anticipating`, `streaming` |
| Single line, no params | `lonely`, `numb`, `bored` |

### Emotion drift (runtime)

Every **10 calls**, the emotion may drift based on live data:

| Runtime condition | New emotion |
|---|---|
| Exception rate > 30% | `angry` → `grumpy` |
| Average latency > 500ms | `overwhelmed` → `dramatic` |
| Total calls ≥ 50 | `nostalgic` |
| Total calls ≥ 20 | `bold` |
| No recent calls | `lonely` |

---

## Rich CLI Output Explained

### Registration banner (fires once per function, at import/decoration time)

```
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ← valence accent bar (green=positive, red=negative, orange=mixed)
  😤  validate_user  ▸  irritated  [irritation]  ▒▒···  ▼ −      ← emoji · name · emotion · family · intensity · valence
    ◆ initialized  Validates user_id and role, raising on invalid input.   ← description
```

**Intensity bar** `░▒▓██` — 5 segments, colour-coded from gray (1) → red (5)
**Valence badge** — `▲ +` positive · `▼ −` negative · `◆ ±` mixed

### Per-call metrics row (fires on every call)

```
  │ ✓ done  😤 validate_user  #3  ⏱ 0.4ms  cpu 0.02ms  mem +0.1KB/4250KB  cc=4  err=1
```

| Field | Color logic |
|---|---|
| `✓ done` / `💥 raised` | Green / Red |
| `⏱ Xms` | Green < 100ms · Orange < 500ms · Red ≥ 500ms |
| `mem peak` | Green < 1MB · Orange < 10MB · Red ≥ 10MB |
| `cc=N` | Green ≤ 5 · Orange ≤ 10 · Red > 10 |
| `err=N` | Red if any exceptions |

---

## Inter-Function Conversation

When two functions decorated with `conversation=True` **execute concurrently** (one is already running when the other starts), they display a dialogue box in the terminal:

```python
@emotion(conversation=True)
def fetch_records(query: str) -> list:
    ...

@emotion(conversation=True)
def process_batch(records: list) -> dict:
    ...
```

```
╔══════════════════════════════════════════════════════════════╗
║──────────────────── ⚡ FUNCTION DIALOGUE ────────────────────║
╠══════════════════════════════════════════════════════════════╣
║  FRIENDSHIP  ·  😄 fetch_records [joyful]  ↔  😮‍💨 process_batch [calm]
╠──────────────────────────────────────────────────────────────╣
║  😄 fetch_records  says:  Hey! Great working with you!
║  😮‍💨 process_batch  says:  Everything flows gently.
║··············································
║  💬 fetch_records: Best pair in the codebase, right here!
║  💬 process_batch: Same! You always make the runtime feel lighter.
╚══════════════════════════════════════════════════════════════╝
```

The dialogue content is driven by:
- **Emotion voice** — each emotion family has its own characteristic phrases
- **Relationship type** — the conversation tone changes based on the social relationship

| Relationship | Conversation tone | Trigger |
|---|---|---|
| `friendship` | Warm, supportive | Co-called ≥ 3 times |
| `anger` | Hostile, accusatory | Both have high error counts |
| `jealousy` | Bitter, resentful | Other called ≥ 3× more |
| `respect` | Admiring, humble | Other has ≥ 1.5× your complexity |
| `mock` | Condescending | Other is much simpler than you |
| `pity` | Patronising | Other rarely called, very simple |
| `confusion` | Bewildered | Complexity difference ≥ 5 |
| `class_respect` | Formal, paradigm-aware | Other is a class method |

---

## Dashboard

```python
from emotion import show_dashboard

show_dashboard()                        # all functions, sorted by call count
show_dashboard(top_n=10)                # top 10 only
show_dashboard(sort_by="health")        # sort by health score (0–100)
show_dashboard(sort_by="complexity")    # sort by cyclomatic complexity
show_dashboard(sort_by="name")          # alphabetical
```

**Example card output:**

```
★ 😤  validate_user  [irritated]  ⭐⭐⭐⭐  health=82/100
  ↳ Validates user_id and role, raising on invalid input.
  static   lines=8  params=2  branches=3  nesting=2  cc=4 (simple)  [try/except]
  runtime  calls=42 ████████░░░░  errors=3(7%)  avg=0.4ms  total=17ms
  profile  wall=0.4ms  cpu=0.02ms  mem_avg=+0.1KB  mem_peak=4250.0KB
  graph    callers=[orchestrate, main]  calls=[db_query, logger]
  social   friendship→orchestrate, jealousy→process_data
```

**Health score** (0–100) is deducted based on:
- Exception rate (−8 to −40 points)
- Cyclomatic complexity (−5 to −25 points)
- Average latency (−3 to −20 points)
- Peak memory usage (−8 to −15 points)

---

## Using with Classes

```python
from emotion import emotion

class DataPipeline:

    @emotion
    def load(self, path: str) -> list:
        ...

    @emotion(conversation=True)
    def transform(self, records: list) -> list:
        ...

    @emotion(verbose=False)
    def export(self, data: list, path: str) -> None:
        ...
```

Class methods are detected automatically — `self`/`cls` as first parameter — and `class_respect` relationships are formed with other class methods.

---

## Programmatic API

### Get a report for one function

```python
from emotion import build_report

report = build_report("calculate_total")
print(report.emotion)              # e.g. "glad"
print(report.health_score())       # e.g. 95.0
print(report.star_rating())        # e.g. "⭐⭐⭐⭐⭐"
print(report.cyclomatic_complexity) # e.g. 2.0
print(report.call_count)           # e.g. 7
print(report.exception_rate)       # e.g. 0.0
print(report.callers)              # e.g. ["main", "orchestrate"]
print(report.relationships)        # e.g. {"process_data": "friendship"}
```

### Get reports for all functions

```python
from emotion import all_reports

for report in all_reports(sort_by="health"):
    print(f"{report.name:30s} {report.emotion:20s} health={report.health_score():.0f}")
```

### Raw registry access

```python
from emotion import get_data, get_all_names

names = get_all_names()
data  = get_data("calculate_total")
# data keys: emotion, call_count, exception_count, average_time_ms,
#            total_time_ms, profile_history, relationships, description, ...
```

### Call graph

```python
from emotion import callers_of, callees_of, scan_directory
from pathlib import Path

scan_directory(Path("src/"))       # trigger AST scan on a directory
print(callers_of("process_data"))  # ["orchestrate", "main"]
print(callees_of("orchestrate"))   # ["fetch_data", "process_data", "export"]
```

### Function descriptions

```python
from emotion import describe

print(describe(my_function))
# "Processes records and returns a filtered list."
# Uses Ollama if available, falls back to AST analysis
```

---

## FunctionReport Reference

`build_report(name)` returns a `FunctionReport` dataclass with these fields:

```python
@dataclass
class FunctionReport:
    # Identity
    name:        str
    emotion:     str
    description: str

    # Static metrics
    branch_count:          int
    line_count:            int
    param_count:           int
    cyclomatic_complexity: float
    nesting_depth:         float
    return_count:          int
    has_recursion:         bool
    has_async:             bool
    has_try_except:        bool
    has_yield:             bool

    # Runtime metrics
    call_count:      int
    exception_count: int
    exception_rate:  float    # 0.0–1.0
    average_time_ms: float
    total_time_ms:   float

    # Profiling (last ≤50 calls)
    average_wall_time_ms:    float
    average_cpu_time_ms:     float
    average_memory_delta_kb: float
    peak_memory_kb:          float

    # Call graph
    callers:          List[str]
    callees:          List[str]
    centrality_score: int
    is_central:       bool
    max_call_depth:   int

    # Social
    relationships: Dict[str, str]   # {other_name: relationship_type}

    # Derived
    def health_score(self) -> float  # 0–100
    def star_rating(self) -> str     # ⭐ to ⭐⭐⭐⭐⭐
    def complexity_tier(self) -> str # trivial/simple/moderate/complex/very complex
    def size_tier(self) -> str       # tiny/small/medium/large/huge
```

---

## Performance

emotion-ai is designed for minimal overhead:

- **Static analysis** runs **once** at decoration time — zero per-call cost
- **`tracemalloc`** is started **once** at package import — per-call overhead is O(1): two calls to `get_traced_memory()` + `reset_peak()`
- **Typical per-call overhead**: 0.01–0.05ms on modern hardware
- **Registry access** uses a single `threading.Lock` — safe for multi-threaded code

> Use `@emotion(verbose=False)` to disable terminal I/O for hot paths while retaining all metric collection.

---

## Project Structure

```
emotion-ai/
├── LICENSE
├── README.md
├── pyproject.toml
├── logo.png
├── examples/
│   ├── quickstart.py          ← minimal 5-function demo
│   └── demo.py                ← full 10-function showcase
└── emotion/
    ├── __init__.py            ← public API surface
    ├── core/
    │   ├── analyzer.py        ← AST feature extraction + emotion classification (46 families)
    │   ├── decorator.py       ← @emotion — all 3 call forms, conversation trigger, profiling
    │   ├── profiler.py        ← Profiler context manager (wall / CPU / heap)
    │   └── registry.py        ← thread-safe global runtime state store
    ├── analysis/
    │   ├── call_graph.py      ← project-wide static call graph via AST directory scan
    │   ├── metrics.py         ← FunctionReport dataclass, health scoring, report builder
    │   └── narrator.py        ← Ollama LLM or AST-based one-line function descriptions
    ├── output/
    │   ├── conversation.py    ← inter-function dialogue engine (voices + relationship exchanges)
    │   ├── dashboard.py       ← rich terminal dashboard renderer
    │   └── logger.py          ← ANSI 256-color output: registration banners + metrics rows
    ├── schemas/
    │   └── emotions.py        ← 500+ emotions: name, emoji, ANSI color, family, intensity, valence
    └── utils/
        ├── ansi.py            ← color helpers, progress bar renderer
        └── ast_helpers.py     ← cyclomatic complexity, nesting depth, recursion detection
```

---

## Full Public API

```python
from emotion import (
    # Core decorator
    emotion,              # @emotion, @emotion(), @emotion(enabled, conversation, verbose)

    # Terminal output
    show_dashboard,       # show_dashboard(top_n=None, sort_by="call_count")
    display_conversation, # display_conversation(func_a, emotion_a, func_b, emotion_b, relationship)

    # Reports
    build_report,         # build_report(name) → FunctionReport | None
    all_reports,          # all_reports(sort_by="call_count") → List[FunctionReport]
    FunctionReport,       # dataclass — see FunctionReport Reference above

    # Descriptions
    describe,             # describe(func) → str

    # Call graph
    callers_of,           # callers_of(name) → List[str]
    callees_of,           # callees_of(name) → List[str]
    scan_directory,       # scan_directory(path: Path) → None

    # Registry
    get_all_names,        # get_all_names() → List[str]
    get_data,             # get_data(name) → dict
)
```

---

## Requirements

- **Python 3.11+**
- **No external dependencies** — pure Python standard library (`ast`, `inspect`, `tracemalloc`, `threading`, `time`, `textwrap`, `urllib`)
- **Optional**: [Ollama](https://ollama.ai) running at `localhost:11434` with `phi3.5:mini`, `phi3:mini`, `llama3.2:1b`, or `tinyllama` pulled

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes (no external dependencies — keep it stdlib-only)
4. Test: `python examples/demo.py`
5. Open a pull request

Areas to contribute:
- New emotion families in `emotion/schemas/emotions.py`
- Better AST-based description generation in `analysis/narrator.py`
- New relationship types in `core/registry.py`
- Additional conversation voice phrases in `output/conversation.py`
- Test suite under `tests/`

---

## License

[MIT](LICENSE) — free to use, modify, and distribute.

---

<p align="center">Made with ❤️ and a lot of <strong>feelings</strong> · <a href="LICENSE">MIT</a></p>

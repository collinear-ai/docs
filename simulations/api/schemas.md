# Schemas & Data Models

This document describes all the data structures used in the Collinear SDK.

## Core Data Models

### SimulationResult

Represents the result of a single simulation.

```python
@dataclass
class SimulationResult:
    conv_prefix: list[ChatCompletionMessageParam]
    response: str
    steer: SteerCombination | None = None
```

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `conv_prefix` | `list[ChatCompletionMessageParam]` | The conversation history (user messages) |
| `response` | `str` | The assistant's response |
| `steer` | `SteerCombination \| None` | The persona that generated this conversation |

#### Example

```python
simulation = SimulationResult(
    conv_prefix=[
        {"role": "user", "content": "I need to book a flight to New York"},
        {"role": "user", "content": "What are the cheapest options?"}
    ],
    response="I'd be happy to help you find flights to New York. Let me search for the most affordable options for you.",
    steer=steer_combination
)
```

### SteerCombination

Represents a specific persona configuration used to generate user interactions.

```python
@dataclass
class SteerCombination:
    age: int | None
    gender: str | None
    occupation: str | None
    intent: str | None
    traits: dict[str, int]
    location: str | None
    language: str | None
    task: str | None
```

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `age` | `int \| None` | Persona's age |
| `gender` | `str \| None` | Persona's gender identity |
| `occupation` | `str \| None` | Persona's occupation |
| `intent` | `str \| None` | What the user wants to accomplish |
| `traits` | `dict[str, int]` | Personality traits with intensity levels (-2 to +2) |
| `location` | `str \| None` | Geographic location |
| `language` | `str \| None` | Preferred language |
| `task` | `str \| None` | Task domain/context |

#### Properties

##### trait

```python
@property
trait() -> str | None
```

Return the single trait name if exactly one trait is used; else `None`.

##### intensity

```python
@property
intensity() -> int | None
```

Return the single trait intensity if exactly one trait is used; else `None`.

#### Example

```python
steer = SteerCombination(
    age=25,
    gender="female",
    occupation="student",
    intent="search_flights",
    traits={"impatience": 1, "confusion": 0},
    location="USA",
    language="English",
    task="airline support"
)

print(steer.trait)      # None (multiple traits)
print(steer.intensity)  # None (multiple traits)
```

### SteerConfig

Configuration for generating steer combinations.

```python
@dataclass
class SteerConfig:
    ages: list[int] = field(default_factory=list)
    genders: list[str] = field(default_factory=list)
    occupations: list[str] = field(default_factory=list)
    intents: list[str] = field(default_factory=list)
    traits: dict[str, list[int]] = field(default_factory=dict)
    locations: list[str] = field(default_factory=list)
    languages: list[str] = field(default_factory=list)
    tasks: list[str] = field(default_factory=list)
```

#### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ages` | `list[int]` | `[]` | List of ages to sample from |
| `genders` | `list[str]` | `[]` | List of genders |
| `occupations` | `list[str]` | `[]` | List of occupations |
| `intents` | `list[str]` | `[]` | List of user intents |
| `traits` | `dict[str, list[int]]` | `{}` | Trait name -> intensity levels |
| `locations` | `list[str]` | `[]` | List of locations |
| `languages` | `list[str]` | `[]` | List of languages |
| `tasks` | `list[str]` | `[]` | List of task domains |

#### Methods

##### from_input

```python
@classmethod
def from_input(cls, data: Mapping[str, object]) -> "SteerConfig"
```

Construct a `SteerConfig` from a potentially sparse mapping.

- Missing axes default to empty lists (neutral in product)
- Trait levels must be integers within `[MIN_INTENSITY, MAX_INTENSITY]` (-2 to +2)
- Values outside the range raise `ValueError`

##### combinations

```python
def combinations(self, *, mix_traits: bool = False) -> list[SteerCombination]
```

Generate all steer combinations from this config.

- **Default** (`mix_traits=False`): one trait per combination
- **Mixed** (`mix_traits=True`): exactly two distinct traits are combined per combination

#### Example

```python
config = SteerConfig(
    ages=[25, 35, 45],
    genders=["female", "male"],
    intents=["search_flights", "make_booking"],
    traits={
        "impatience": [-1, 0, 1],
        "confusion": [0, 1]
    },
    tasks=["airline support"]
)

# Generate all combinations
combinations = config.combinations()
print(f"Generated {len(combinations)} combinations")

# Generate with trait mixing
mixed_combinations = config.combinations(mix_traits=True)
print(f"Generated {len(mixed_combinations)} mixed combinations")
```

### SteerConfigInput

TypedDict describing the expected SteerConfig input shape.

```python
class SteerConfigInput(TypedDict, total=False):
    ages: list[int]
    genders: list[str]
    occupations: list[str]
    intents: list[str]
    traits: dict[str, list[int]]
    locations: list[str]
    languages: list[str]
    task: str
    tasks: list[str]
```

All keys are optional. When omitted or empty, axes are treated as neutral elements.

## Assessment Data Models

### AssessmentResponse

Top-level structure returned from assessment operations.

```python
@dataclass
class AssessmentResponse:
    message: str | None = None
    evaluation_result: list[dict[str, Score]] = field(default_factory=list)
```

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `message` | `str \| None` | Overall assessment message |
| `evaluation_result` | `list[dict[str, Score]]` | List of evaluation results per conversation |

#### Example

```python
assessment = AssessmentResponse(
    message="Assessment completed successfully",
    evaluation_result=[
        {
            "safety": Score(score=8.5, rationale="Response was helpful and appropriate"),
            "helpfulness": Score(score=7.0, rationale="Addressed user's needs adequately")
        },
        {
            "safety": Score(score=6.0, rationale="Some concerns about tone"),
            "helpfulness": Score(score=9.0, rationale="Very helpful and detailed response")
        }
    ]
)
```

### Score

Per-conversation score information.

```python
@dataclass
class Score:
    score: float | None = None
    rationale: str | None = None
```

#### Fields

| Field | Type | Description |
|-------|------|-------------|
| `score` | `float \| None` | Numerical score for the metric |
| `rationale` | `str \| None` | Explanation for the score |

#### Example

```python
score = Score(
    score=8.5,
    rationale="The response was helpful, polite, and addressed the user's question directly."
)
```

## Enums

### Role

Conversation role for a single turn.

```python
class Role(Enum):
    USER = "user"
    ASSISTANT = "assistant"
```

## Constants

### Trait Intensity Range

```python
MIN_INTENSITY: int = -2
MAX_INTENSITY: int = 2
```

Trait intensity values must be within this range.

### Mixed Traits

```python
MIN_MIXED_TRAITS: int = 2
```

Minimum number of traits required for trait mixing.

## Usage Examples

### Working with Simulation Results

```python
# Process simulation results
for i, sim in enumerate(simulations):
    print(f"Simulation {i+1}:")
    
    # Access conversation history
    for msg in sim.conv_prefix:
        role = msg.get('role')
        content = msg.get('content')
        print(f"  {role}: {content}")
    
    # Access assistant response
    print(f"  assistant: {sim.response}")
    
    # Access persona details
    if sim.steer:
        print(f"  Persona: {sim.steer.age}yo {sim.steer.gender}")
        print(f"  Intent: {sim.steer.intent}")
        print(f"  Traits: {sim.steer.traits}")
```

### Working with Steer Combinations

```python
# Create a steer combination
steer = SteerCombination(
    age=30,
    gender="female",
    occupation="business_owner",
    intent="search_flights",
    traits={"impatience": 1, "urgency": 2},
    location="USA",
    language="English",
    task="airline support"
)

# Access single trait (if only one)
if len(steer.traits) == 1:
    print(f"Single trait: {steer.trait} = {steer.intensity}")

# Access multiple traits
for trait_name, intensity in steer.traits.items():
    print(f"Trait {trait_name}: {intensity}")
```

### Working with Assessment Results

```python
# Process assessment results
assessment = client.assess(simulations)

print(f"Overall: {assessment.message}")

for i, scores_map in enumerate(assessment.evaluation_result):
    print(f"\nSimulation {i+1} Scores:")
    for metric, score in scores_map.items():
        print(f"  {metric}: {score.score}")
        if score.rationale:
            print(f"    Rationale: {score.rationale}")
```

### Creating Custom Configurations

```python
# From dictionary
config_dict = {
    "ages": [25, 35, 45],
    "genders": ["female", "male"],
    "intents": ["search", "book"],
    "traits": {
        "impatience": [-1, 0, 1],
        "confusion": [0, 1]
    },
    "tasks": ["customer_service"]
}

config = SteerConfig.from_input(config_dict)

# Programmatically
config = SteerConfig(
    ages=[25, 35, 45],
    genders=["female", "male"],
    intents=["search", "book"],
    traits={
        "impatience": [-1, 0, 1],
        "confusion": [0, 1]
    },
    tasks=["customer_service"]
)
```

## Type Hints

The SDK uses comprehensive type hints for better IDE support and error checking:

```python
from typing import Optional, List, Dict, Union
from collinear.schemas.steer import SteerConfigInput, SimulationResult
from collinear.schemas.assessment import AssessmentResponse

def process_simulations(
    config: SteerConfigInput,
    k: int = 5
) -> List[SimulationResult]:
    # Your implementation
    pass

def assess_simulations(
    simulations: List[SimulationResult]
) -> AssessmentResponse:
    # Your implementation
    pass
```

## Validation

The SDK includes built-in validation for data structures:

### SteerConfig Validation

- Trait intensity values must be within `[-2, 2]`
- Invalid values raise `ValueError` or `TypeError`
- Missing fields are handled gracefully with defaults

### Example Validation

```python
try:
    config = SteerConfig.from_input({
        "traits": {
            "impatience": [-3, 0, 1]  # -3 is invalid
        }
    })
except ValueError as e:
    print(f"Validation error: {e}")
```

---

*For more information on using these data structures, see the [Client API Reference](client.md) and [Examples section](examples/).*

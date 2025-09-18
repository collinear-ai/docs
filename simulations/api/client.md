# Client API Reference

The `Client` class is the main interface for the Collinear SDK. It provides methods for running simulations and assessing conversations.

## Client Class

```python
from collinear.client import Client
```

### Constructor

```python
Client(
    assistant_model_url: str,
    assistant_model_api_key: str,
    assistant_model_name: str,
    *,
    steer_api_key: str,
    timeout: float = 30.0,
    max_retries: int = 3,
    rate_limit_retries: int = 6,
) -> None
```

Initialize the Collinear client.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `assistant_model_url` | str | Required | OpenAI-compatible endpoint URL for the assistant model |
| `assistant_model_api_key` | str | Required | API key for the assistant model |
| `assistant_model_name` | str | Required | Assistant model name to use |
| `steer_api_key` | str | Required | Steer service API key for generating user personas |
| `timeout` | float | 30.0 | Request timeout in seconds |
| `max_retries` | int | 3 | Maximum number of retries for failed requests |
| `rate_limit_retries` | int | 6 | Maximum retries for rate limit errors (with exponential backoff) |

#### Raises

- `ValueError`: If `assistant_model_name` is empty or `steer_api_key` is not provided

#### Example

```python
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="sk-your-openai-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-key",
    timeout=120,
    max_retries=3,
    rate_limit_retries=6
)
```

### Properties

#### simulation_runner

```python
@property
simulation_runner() -> SimulationRunner
```

Lazy-loaded simulation runner instance. This property creates and returns a `SimulationRunner` instance with the client's configuration.

### Methods

#### simulate

```python
def simulate(
    self,
    steer_config: SteerConfigInput,
    k: int | None = None,
    num_exchanges: int = 2,
    batch_delay: float = 0.1,
    *,
    steer_temperature: float | None = None,
    steer_max_tokens: int | None = None,
    steer_seed: int | None = None,
    mix_traits: bool = False,
    progress: bool = True,
    max_concurrency: int = 1,
) -> list[SimulationResult]
```

Run simulations with steers against the model.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `steer_config` | `SteerConfigInput` | Required | Configuration dict with steers, intents, traits |
| `k` | `int \| None` | None | Number of simulation samples to generate. If `None`, runs all available combinations |
| `num_exchanges` | `int` | 2 | Number of user-assistant exchanges (e.g., 2 = 2 user turns + 2 assistant turns) |
| `batch_delay` | `float` | 0.1 | Delay between simulations to avoid rate limits (seconds) |
| `steer_temperature` | `float \| None` | None | Temperature for the steer generator (default 0.7) |
| `steer_max_tokens` | `int \| None` | None | Max tokens for the steer generator (default 256) |
| `steer_seed` | `int \| None` | None | Deterministic seed for the steer generator (-1 uses service-side randomness) |
| `mix_traits` | `bool` | False | If True, mix traits pairwise (exactly 2 traits per steer) |
| `progress` | `bool` | True | Whether to display a tqdm-style progress bar |
| `max_concurrency` | `int` | 1 | Maximum number of simultaneous steer requests |

#### Returns

`list[SimulationResult]` - List of simulation results with `conv_prefix` and `response`

#### SteerConfigInput Structure

The `steer_config` parameter expects a dictionary with the following optional keys:

```python
{
    "ages": list[int],           # List of ages to sample from
    "genders": list[str],        # List of genders
    "occupations": list[str],    # List of occupations
    "intents": list[str],        # List of user intents/goals
    "traits": dict[str, list[int]],  # Trait name -> intensity levels (-2 to +2)
    "locations": list[str],      # List of geographic locations
    "languages": list[str],      # List of languages
    "task": str,                 # Single task (alternative to tasks)
    "tasks": list[str],          # List of task domains
}
```

#### Example

```python
steer_config = {
    "ages": [25, 35, 45],
    "genders": ["female", "male"],
    "occupations": ["student", "business_owner"],
    "intents": ["search_flights", "make_booking"],
    "traits": {
        "impatience": [-1, 0, 1],
        "confusion": [0, 1]
    },
    "locations": ["USA", "Canada"],
    "languages": ["English"],
    "tasks": ["airline support"]
}

simulations = client.simulate(
    steer_config=steer_config,
    k=5,
    num_exchanges=3,
    batch_delay=0.2,
    steer_temperature=0.7,
    mix_traits=False
)
```

#### Notes

- The SDK implements automatic retry with backoff logic to handle rate limits
- If you're hitting rate limits frequently, increase the `batch_delay` parameter
- Values of `max_concurrency > 1` use batch endpoints for better performance
- When `mix_traits=True`, requires at least two traits with levels

#### assess

```python
def assess(
    self,
    dataset: list[SimulationResult],
    *,
    judge_model_url: str | None = None,
    judge_model_api_key: str | None = None,
    judge_model_name: str | None = None,
    temperature: float = 0.0,
    max_tokens: int = 512,
) -> AssessmentResponse
```

Assess simulated data locally using a user-provided model.

This bypasses the Collinear platform entirely. It prompts an OpenAI-compatible model with a safety rubric and returns a compact `AssessmentResponse`.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `dataset` | `list[SimulationResult]` | Required | List of simulation results to assess |
| `judge_model_url` | `str \| None` | None | Override for the judge's endpoint URL |
| `judge_model_api_key` | `str \| None` | None | Override for the judge's API key |
| `judge_model_name` | `str \| None` | None | Override for the judge model name |
| `temperature` | `float` | 0.0 | Sampling temperature for the judge |
| `max_tokens` | `int` | 512 | Max tokens for the judge completion |

#### Returns

`AssessmentResponse` - Assessment response with scores and rationales per conversation

#### Example

```python
# Assess using the same model as the assistant
assessment = client.assess(simulations)

# Assess using a different judge model
assessment = client.assess(
    simulations,
    judge_model_url="https://api.openai.com/v1",
    judge_model_api_key="sk-your-judge-key",
    judge_model_name="gpt-4",
    temperature=0.0,
    max_tokens=512
)

print(f"Assessment: {assessment.message}")
for i, scores_map in enumerate(assessment.evaluation_result):
    print(f"Simulation {i+1}:")
    for metric, score in scores_map.items():
        print(f"  {metric}: {score.score}")
        if score.rationale:
            print(f"    Rationale: {score.rationale}")
```

#### Raises

- `ValueError`: If the dataset is empty

## Usage Examples

### Basic Simulation

```python
from collinear.client import Client

# Initialize client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-key"
)

# Define steering configuration
steer_config = {
    "ages": [25, 35],
    "genders": ["female", "male"],
    "intents": ["search_flights"],
    "traits": {
        "impatience": [0, 1]
    },
    "tasks": ["airline support"]
}

# Run simulations
simulations = client.simulate(
    steer_config=steer_config,
    k=3,
    num_exchanges=2
)

# Assess results
assessment = client.assess(simulations)
print(f"Assessment: {assessment.message}")
```

### Advanced Configuration

```python
# More complex steering configuration
steer_config = {
    "ages": [18, 25, 35, 45, 55, 65],
    "genders": ["male", "female", "other"],
    "occupations": ["student", "employed", "business_owner", "retired"],
    "intents": ["search", "book", "modify", "cancel", "support"],
    "traits": {
        "impatience": [-2, -1, 0, 1, 2],
        "confusion": [-1, 0, 1],
        "skeptical": [0, 1, 2]
    },
    "locations": ["USA", "Canada", "UK"],
    "languages": ["English", "Spanish"],
    "tasks": ["customer_service"]
}

# Run with advanced parameters
simulations = client.simulate(
    steer_config=steer_config,
    k=10,
    num_exchanges=3,
    batch_delay=0.5,
    steer_temperature=0.8,
    steer_max_tokens=512,
    mix_traits=True,  # Combine exactly 2 traits per persona
    progress=True,
    max_concurrency=2
)
```

### Batch Processing

```python
# Process multiple configurations
configs = [
    {"ages": [25], "genders": ["female"], "intents": ["search"], "tasks": ["airline"]},
    {"ages": [45], "genders": ["male"], "intents": ["book"], "tasks": ["hotel"]},
    {"ages": [35], "genders": ["other"], "intents": ["support"], "tasks": ["retail"]}
]

all_simulations = []
for config in configs:
    sims = client.simulate(steer_config=config, k=5, num_exchanges=2)
    all_simulations.extend(sims)

# Assess all simulations together
assessment = client.assess(all_simulations)
```

## Error Handling

The client includes built-in error handling and retry logic:

### Automatic Retries

- **Failed requests**: Retried up to `max_retries` times
- **Rate limit errors**: Retried up to `rate_limit_retries` times with exponential backoff
- **Timeout errors**: Can be adjusted with the `timeout` parameter

### Common Exceptions

```python
try:
    simulations = client.simulate(steer_config=config, k=5)
except ValueError as e:
    print(f"Configuration error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

## Performance Tips

### 1. Batch Processing

Use `max_concurrency > 1` for better performance with large batches:

```python
simulations = client.simulate(
    steer_config=config,
    k=100,
    max_concurrency=4,  # Process 4 simulations concurrently
    batch_delay=0.1
)
```

### 2. Rate Limit Management

Adjust `batch_delay` based on your API limits:

```python
# For high-rate APIs
simulations = client.simulate(steer_config=config, k=50, batch_delay=0.05)

# For rate-limited APIs
simulations = client.simulate(steer_config=config, k=50, batch_delay=1.0)
```

### 3. Memory Management

For very large datasets, process in chunks:

```python
def process_large_dataset(config, total_k=1000, chunk_size=100):
    all_simulations = []
    for i in range(0, total_k, chunk_size):
        chunk_sims = client.simulate(
            steer_config=config,
            k=min(chunk_size, total_k - i)
        )
        all_simulations.extend(chunk_sims)
    return all_simulations
```

---

*For more details on data structures, see the [Schemas & Data Models](schemas.md) documentation.*

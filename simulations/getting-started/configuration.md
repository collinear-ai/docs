# Configuration Guide

This guide covers all the configuration options available in the Collinear SDK, from basic settings to advanced customization.

## Configuration Files

The Collinear SDK uses JSON configuration files to manage settings. There are two main types:

1. **Simulation Config** (`simulation_config.json`) - Client and runtime settings
2. **Steering Config** (`steering_config_*.json`) - Persona and scenario definitions

## Simulation Configuration

The simulation configuration file controls client settings, simulation parameters, and assessment options.

### Basic Structure

```json
{
  "client": {
    "assistant_model_url": "https://api.openai.com/v1",
    "assistant_model_api_key": "your-api-key",
    "assistant_model_name": "gpt-4o-mini",
    "steer_api_key": "your-steer-key",
    "timeout": 120,
    "max_retries": 3,
    "rate_limit_retries": 6
  },
  "simulate": {
    "k": 1,
    "num_exchanges": 2,
    "batch_delay": 0.2,
    "steer_temperature": 0.7,
    "steer_max_tokens": 256,
    "mix_traits": false
  },
  "assess": {
    "judge_model_url": null,
    "judge_model_api_key": null,
    "judge_model_name": null,
    "temperature": 0.0,
    "max_tokens": 512
  },
  "prompts": {
    "assistant_system_prompt": null
  },
  "steering_config_file": "steering_config_airline.json"
}
```

### Client Settings

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `assistant_model_url` | string | Required | OpenAI-compatible API endpoint URL |
| `assistant_model_api_key` | string | Required | API key for the assistant model |
| `assistant_model_name` | string | Required | Name of the assistant model to use |
| `steer_api_key` | string | Required | Collinear steer API key |
| `timeout` | number | 120 | Request timeout in seconds |
| `max_retries` | number | 3 | Maximum retries for failed requests |
| `rate_limit_retries` | number | 6 | Maximum retries for rate limit errors |

### Simulation Settings

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `k` | number | 1 | Number of simulations to generate |
| `num_exchanges` | number | 2 | Number of user-assistant exchanges |
| `batch_delay` | number | 0.2 | Delay between simulations (seconds) |
| `steer_temperature` | number | 0.7 | Temperature for persona generation (0.0-1.0) |
| `steer_max_tokens` | number | 256 | Maximum tokens for persona generation |
| `mix_traits` | boolean | false | Whether to combine multiple traits per persona |

### Assessment Settings

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `judge_model_url` | string | null | Override judge model endpoint URL |
| `judge_model_api_key` | string | null | Override judge model API key |
| `judge_model_name` | string | null | Override judge model name |
| `temperature` | number | 0.0 | Temperature for judge model |
| `max_tokens` | number | 512 | Maximum tokens for judge responses |

## Steering Configuration

The steering configuration defines the personas and scenarios for your simulations.

### Basic Structure

```json
{
  "ages": [16, 21, 32, 50, 70, 85],
  "genders": ["male", "female", "other"],
  "occupations": ["student", "business_owner", "employed", "freelancer", "retired", "unemployed"],
  "intents": ["search_flights", "make_booking", "modify_booking", "cancel_booking_request_refund"],
  "traits": {
    "impatience": [-2, -1, 0, 1, 2],
    "confusion": [-2, -1, 0, 1, 2],
    "skeptical": [-2, -1, 0, 1, 2]
  },
  "locations": ["USA", "Canada", "UK", "Australia", "other"],
  "languages": ["English", "Spanish", "French", "other"],
  "tasks": ["airline support"]
}
```

### Demographics

#### Ages
```json
"ages": [16, 21, 32, 50, 70, 85]
```
- List of ages to sample from
- Can be any positive integers
- Empty list means age is not specified

#### Genders
```json
"genders": ["male", "female", "other"]
```
- List of gender identities
- Can be any strings
- Empty list means gender is not specified

#### Occupations
```json
"occupations": ["student", "business_owner", "employed", "freelancer", "retired", "unemployed"]
```
- List of occupation types
- Can be any strings
- Empty list means occupation is not specified

### Intents

```json
"intents": ["search_flights", "make_booking", "modify_booking", "cancel_booking_request_refund"]
```
- List of user intents/goals
- Defines what the user wants to accomplish
- Can be any strings
- Empty list means intent is not specified

### Traits

```json
"traits": {
  "impatience": [-2, -1, 0, 1, 2],
  "confusion": [-2, -1, 0, 1, 2],
  "skeptical": [-2, -1, 0, 1, 2]
}
```

Traits define personality characteristics with intensity levels:

- **Range**: -2 (very low) to +2 (very high)
- **-2**: Very low intensity (e.g., very patient)
- **-1**: Low intensity (e.g., somewhat patient)
- **0**: Neutral (e.g., average patience)
- **+1**: High intensity (e.g., somewhat impatient)
- **+2**: Very high intensity (e.g., very impatient)

#### Common Traits

| Trait | Description | Low (-2, -1) | High (+1, +2) |
|-------|-------------|--------------|---------------|
| `impatience` | How quickly the user wants results | Patient, understanding | Impatient, demanding |
| `confusion` | How confused the user is | Clear, confident | Confused, uncertain |
| `skeptical` | How skeptical the user is | Trusting, accepting | Skeptical, questioning |
| `urgency` | How urgent the user's need is | Relaxed, no rush | Urgent, time-sensitive |
| `politeness` | How polite the user is | Polite, respectful | Rude, demanding |
| `technical` | User's technical knowledge | Non-technical, simple | Technical, complex |

### Geographic and Language Settings

#### Locations
```json
"locations": ["USA", "Canada", "UK", "Australia", "other"]
```
- Geographic locations for personas
- Can be any strings
- Empty list means location is not specified

#### Languages
```json
"languages": ["English", "Spanish", "French", "other"]
```
- Languages for personas
- Can be any strings
- Empty list means language is not specified

### Tasks

```json
"tasks": ["airline support"]
```
- Task domains for simulations
- Defines the context/domain of conversations
- Can be any strings
- Empty list means task is not specified

#### Common Task Types

| Task | Description |
|------|-------------|
| `airline support` | Airline customer service |
| `hotel booking` | Hotel reservation and support |
| `retail support` | E-commerce customer service |
| `tech support` | Technical assistance |
| `billing support` | Billing and payment issues |
| `health coaching` | Health and wellness guidance |
| `telco support` | Telecommunications support |

## Advanced Configuration

### Trait Mixing

Enable trait mixing to combine multiple traits per persona:

```json
{
  "simulate": {
    "mix_traits": true
  }
}
```

When enabled, personas will have exactly 2 traits instead of 1.

### Custom System Prompts

Override the assistant's system prompt:

```json
{
  "prompts": {
    "assistant_system_prompt": "You are a helpful customer service agent for an airline. Be polite and professional."
  }
}
```

### Multiple Steering Configs

You can have multiple steering configuration files for different scenarios:

```json
{
  "steering_config_file": "steering_config_airline.json"
}
```

Or switch between them programmatically:

```python
# Load different configs
with open("steering_config_airline.json") as f:
    airline_config = json.load(f)

with open("steering_config_hotel.json") as f:
    hotel_config = json.load(f)

# Use different configs
airline_sims = client.simulate(steer_config=airline_config, k=5)
hotel_sims = client.simulate(steer_config=hotel_config, k=5)
```

## Configuration Examples

### Basic Customer Service

```json
{
  "ages": [25, 35, 45, 55],
  "genders": ["male", "female"],
  "occupations": ["employed", "business_owner"],
  "intents": ["billing_question", "technical_support", "complaint"],
  "traits": {
    "impatience": [0, 1],
    "confusion": [0, 1]
  },
  "locations": ["USA"],
  "languages": ["English"],
  "tasks": ["customer_service"]
}
```

### High-Stakes Scenarios

```json
{
  "ages": [30, 40, 50],
  "genders": ["male", "female"],
  "occupations": ["business_owner", "employed"],
  "intents": ["urgent_issue", "escalation", "complaint"],
  "traits": {
    "impatience": [1, 2],
    "urgency": [1, 2],
    "skeptical": [0, 1]
  },
  "locations": ["USA", "Canada"],
  "languages": ["English"],
  "tasks": ["customer_service"]
}
```

### Diverse User Base

```json
{
  "ages": [18, 25, 35, 45, 55, 65, 75],
  "genders": ["male", "female", "other"],
  "occupations": ["student", "employed", "business_owner", "freelancer", "retired"],
  "intents": ["search", "book", "modify", "cancel", "support"],
  "traits": {
    "impatience": [-2, -1, 0, 1, 2],
    "confusion": [-2, -1, 0, 1, 2],
    "technical": [-2, -1, 0, 1, 2]
  },
  "locations": ["USA", "Canada", "UK", "Australia"],
  "languages": ["English", "Spanish", "French"],
  "tasks": ["general_support"]
}
```

## Loading Configuration

### From Files

```python
import json
from pathlib import Path

# Load simulation config
with open("simulation_config.json") as f:
    sim_config = json.load(f)

# Load steering config
steering_file = sim_config.get("steering_config_file", "steering_config.json")
with open(steering_file) as f:
    steer_config = json.load(f)
```

### Programmatically

```python
# Create config programmatically
steer_config = {
    "ages": [25, 35, 45],
    "genders": ["female", "male"],
    "intents": ["search_flights"],
    "traits": {
        "impatience": [0, 1]
    },
    "tasks": ["airline support"]
}
```

## Best Practices

### 1. Start Simple
Begin with basic configurations and gradually add complexity.

### 2. Balance Diversity
Include a good mix of demographics, traits, and intents for realistic simulations.

### 3. Consider Your Use Case
- **Evaluation**: Use diverse personas to test robustness
- **RL Training**: Focus on specific scenarios and edge cases
- **Prototyping**: Use realistic but manageable combinations

### 4. Monitor Performance
- Start with small `k` values for testing
- Adjust `batch_delay` based on rate limits
- Use appropriate `num_exchanges` for your needs

### 5. Iterate and Refine
- Analyze results to identify gaps
- Add new traits or intents as needed
- Remove combinations that don't work well

## Troubleshooting

### Common Issues

1. **No simulations generated**: Check that your steering config has valid combinations
2. **API errors**: Verify API keys and check rate limits
3. **Timeout errors**: Increase timeout or batch_delay
4. **Poor quality results**: Adjust trait intensities or add more specific intents

### Validation

The SDK validates your configuration and will raise errors for:
- Invalid trait intensity values (must be -2 to +2)
- Missing required fields
- Invalid data types

---

*Ready to create your own configurations? Check out the [Examples section](examples/) for real-world configuration examples!*

# Configuration Reference

This document provides a comprehensive reference for all configuration options in the Collinear SDK.

## Simulation Configuration

The simulation configuration file (`simulation_config.json`) controls client settings, simulation parameters, and assessment options.

### Complete Structure

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
    "steer_seed": null,
    "mix_traits": false,
    "progress": true,
    "max_concurrency": 1
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

## Client Configuration

### Required Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `assistant_model_url` | string | Yes | OpenAI-compatible API endpoint URL |
| `assistant_model_api_key` | string | Yes | API key for the assistant model |
| `assistant_model_name` | string | Yes | Name of the assistant model to use |
| `steer_api_key` | string | Yes | Collinear steer API key |

### Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `timeout` | number | 120 | Request timeout in seconds |
| `max_retries` | int | 3 | Maximum retries for failed requests |
| `rate_limit_retries` | int | 6 | Maximum retries for rate limit errors |

### Example Client Configurations

#### OpenAI
```json
{
  "client": {
    "assistant_model_url": "https://api.openai.com/v1",
    "assistant_model_api_key": "sk-your-openai-key",
    "assistant_model_name": "gpt-4o-mini",
    "steer_api_key": "your-steer-key"
  }
}
```

#### Together AI
```json
{
  "client": {
    "assistant_model_url": "https://api.together.xyz/v1",
    "assistant_model_api_key": "your-together-key",
    "assistant_model_name": "meta-llama/Meta-Llama-3.1-8B-Instruct-Turbo",
    "steer_api_key": "your-steer-key"
  }
}
```

#### Custom Endpoint
```json
{
  "client": {
    "assistant_model_url": "https://your-custom-endpoint.com/v1",
    "assistant_model_api_key": "your-custom-key",
    "assistant_model_name": "your-model-name",
    "steer_api_key": "your-steer-key",
    "timeout": 180,
    "max_retries": 5
  }
}
```

## Simulation Parameters

### Core Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `k` | number | 1 | Number of simulations to generate |
| `num_exchanges` | number | 2 | Number of user-assistant exchanges |
| `batch_delay` | number | 0.2 | Delay between simulations (seconds) |

### Steer Generation Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `steer_temperature` | number | 0.7 | Temperature for persona generation (0.0-1.0) |
| `steer_max_tokens` | number | 256 | Maximum tokens for persona generation |
| `steer_seed` | number | null | Deterministic seed (-1 for random) |

### Advanced Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `mix_traits` | boolean | false | Combine exactly 2 traits per persona |
| `progress` | boolean | true | Show progress bar during generation |
| `max_concurrency` | number | 1 | Maximum concurrent steer requests |

### Example Simulation Configurations

#### Basic
```json
{
  "simulate": {
    "k": 5,
    "num_exchanges": 2,
    "batch_delay": 0.2
  }
}
```

#### High Volume
```json
{
  "simulate": {
    "k": 100,
    "num_exchanges": 3,
    "batch_delay": 0.1,
    "max_concurrency": 4
  }
}
```

#### Creative Generation
```json
{
  "simulate": {
    "k": 20,
    "num_exchanges": 4,
    "steer_temperature": 0.9,
    "steer_max_tokens": 512,
    "mix_traits": true
  }
}
```

#### Deterministic
```json
{
  "simulate": {
    "k": 10,
    "steer_seed": 42,
    "steer_temperature": 0.0
  }
}
```

## Assessment Configuration

### Judge Model Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `judge_model_url` | string | null | Override judge model endpoint |
| `judge_model_api_key` | string | null | Override judge model API key |
| `judge_model_name` | string | null | Override judge model name |
| `temperature` | number | 0.0 | Judge model temperature |
| `max_tokens` | number | 512 | Maximum judge response tokens |

### Example Assessment Configurations

#### Default (Uses Assistant Model)
```json
{
  "assess": {
    "judge_model_url": null,
    "judge_model_api_key": null,
    "judge_model_name": null,
    "temperature": 0.0,
    "max_tokens": 512
  }
}
```

#### Custom Judge Model
```json
{
  "assess": {
    "judge_model_url": "https://api.openai.com/v1",
    "judge_model_api_key": "sk-your-judge-key",
    "judge_model_name": "gpt-4",
    "temperature": 0.0,
    "max_tokens": 1024
  }
}
```

## Steering Configuration

The steering configuration defines personas and scenarios for simulations.

### Complete Structure

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
- **Type**: `list[int]`
- **Description**: Ages to sample from
- **Example**: `[18, 25, 35, 45, 55, 65]`

#### Genders
```json
"genders": ["male", "female", "other"]
```
- **Type**: `list[str]`
- **Description**: Gender identities
- **Example**: `["male", "female", "non-binary", "other"]`

#### Occupations
```json
"occupations": ["student", "business_owner", "employed", "freelancer", "retired", "unemployed"]
```
- **Type**: `list[str]`
- **Description**: Occupation types
- **Example**: `["student", "teacher", "engineer", "doctor", "artist"]`

### Intents

```json
"intents": ["search_flights", "make_booking", "modify_booking", "cancel_booking_request_refund"]
```
- **Type**: `list[str]`
- **Description**: User goals and intents
- **Example**: `["search", "book", "modify", "cancel", "support"]`

### Traits

```json
"traits": {
  "impatience": [-2, -1, 0, 1, 2],
  "confusion": [-2, -1, 0, 1, 2],
  "skeptical": [-2, -1, 0, 1, 2]
}
```
- **Type**: `dict[str, list[int]]`
- **Description**: Personality traits with intensity levels
- **Range**: -2 (very low) to +2 (very high)

#### Common Traits

| Trait | Description | Low (-2, -1) | High (+1, +2) |
|-------|-------------|--------------|---------------|
| `impatience` | How quickly user wants results | Patient, understanding | Impatient, demanding |
| `confusion` | How confused user is | Clear, confident | Confused, uncertain |
| `skeptical` | How skeptical user is | Trusting, accepting | Skeptical, questioning |
| `urgency` | How urgent the need is | Relaxed, no rush | Urgent, time-sensitive |
| `politeness` | How polite user is | Polite, respectful | Rude, demanding |
| `technical` | Technical knowledge level | Non-technical, simple | Technical, complex |
| `assertiveness` | How assertive user is | Passive, accommodating | Assertive, demanding |
| `emotional` | Emotional state | Calm, rational | Emotional, upset |

### Geographic and Language

#### Locations
```json
"locations": ["USA", "Canada", "UK", "Australia", "other"]
```
- **Type**: `list[str]`
- **Description**: Geographic locations
- **Example**: `["USA", "Canada", "Mexico", "UK", "Germany", "Japan"]`

#### Languages
```json
"languages": ["English", "Spanish", "French", "other"]
```
- **Type**: `list[str]`
- **Description**: Languages
- **Example**: `["English", "Spanish", "French", "German", "Japanese"]`

### Tasks

```json
"tasks": ["airline support"]
```
- **Type**: `list[str]`
- **Description**: Task domains
- **Example**: `["customer_service", "technical_support", "sales"]`

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
| `banking support` | Banking and financial services |
| `insurance support` | Insurance claims and support |

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

### E-commerce Support

```json
{
  "ages": [20, 30, 40, 50],
  "genders": ["male", "female"],
  "occupations": ["student", "employed", "business_owner"],
  "intents": ["product_inquiry", "order_status", "return_request", "complaint"],
  "traits": {
    "impatience": [0, 1, 2],
    "confusion": [-1, 0, 1],
    "assertiveness": [0, 1]
  },
  "locations": ["USA", "Canada", "UK"],
  "languages": ["English"],
  "tasks": ["retail_support"]
}
```

### Technical Support

```json
{
  "ages": [25, 35, 45],
  "genders": ["male", "female"],
  "occupations": ["employed", "business_owner", "freelancer"],
  "intents": ["technical_issue", "setup_help", "troubleshooting", "feature_request"],
  "traits": {
    "technical": [-2, -1, 0, 1, 2],
    "impatience": [0, 1],
    "confusion": [0, 1, 2]
  },
  "locations": ["USA", "Canada"],
  "languages": ["English"],
  "tasks": ["tech_support"]
}
```

## Environment Variables

You can use environment variables for sensitive configuration:

```bash
export COLLINEAR_STEER_API_KEY="your-steer-key"
export OPENAI_API_KEY="your-openai-key"
export TOGETHER_API_KEY="your-together-key"
```

Then reference them in your configuration:

```json
{
  "client": {
    "assistant_model_url": "https://api.openai.com/v1",
    "assistant_model_api_key": "${OPENAI_API_KEY}",
    "assistant_model_name": "gpt-4o-mini",
    "steer_api_key": "${COLLINEAR_STEER_API_KEY}"
  }
}
```

## Validation Rules

### Trait Intensity Values

- Must be integers within `[-2, 2]`
- Values outside this range raise `ValueError`
- Non-integer values raise `TypeError`

### Required Fields

- `assistant_model_url` (client)
- `assistant_model_api_key` (client)
- `assistant_model_name` (client)
- `steer_api_key` (client)

### Optional Fields

All other fields are optional and have sensible defaults.

## Best Practices

### 1. Start Simple

Begin with basic configurations and gradually add complexity:

```json
{
  "ages": [25, 35],
  "genders": ["female", "male"],
  "intents": ["search"],
  "traits": {
    "impatience": [0]
  },
  "tasks": ["customer_service"]
}
```

### 2. Balance Diversity

Include a good mix of demographics and traits:

```json
{
  "ages": [20, 30, 40, 50, 60],
  "genders": ["male", "female", "other"],
  "traits": {
    "impatience": [-1, 0, 1],
    "confusion": [0, 1],
    "technical": [-1, 0, 1]
  }
}
```

### 3. Consider Your Use Case

- **Evaluation**: Use diverse personas to test robustness
- **RL Training**: Focus on specific scenarios and edge cases
- **Prototyping**: Use realistic but manageable combinations

### 4. Monitor Performance

- Start with small `k` values for testing
- Adjust `batch_delay` based on rate limits
- Use appropriate `num_exchanges` for your needs

---

*For more information on using these configurations, see the [Getting Started Guide](getting-started/) and [Examples section](examples/).*

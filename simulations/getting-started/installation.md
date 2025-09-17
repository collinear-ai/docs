# Installation & Setup

This guide will help you install the Collinear SDK and set up your environment for running simulations.

## Prerequisites

- Python 3.8 or higher
- An API key for your assistant model (OpenAI, Together, or compatible)
- A Collinear steer API key (contact info@collinear.ai)

## Installation

### Install the Collinear SDK

```bash
pip install collinear --upgrade
```

### Verify Installation

```python
import collinear
print(collinear.__version__)
```

## API Keys Setup

### 1. Assistant Model API Key

You'll need an API key for your chosen assistant model. The Collinear SDK supports any OpenAI-compatible API:

**OpenAI:**
```python
assistant_model_url = "https://api.openai.com/v1"
assistant_model_api_key = "sk-your-openai-key"
assistant_model_name = "gpt-4o-mini"
```

**Together AI:**
```python
assistant_model_url = "https://api.together.xyz/v1"
assistant_model_api_key = "your-together-key"
assistant_model_name = "meta-llama/Meta-Llama-3.1-8B-Instruct-Turbo"
```

**Custom/Other:**
```python
assistant_model_url = "https://your-custom-endpoint.com/v1"
assistant_model_api_key = "your-custom-key"
assistant_model_name = "your-model-name"
```

### 2. Collinear Steer API Key

Contact info@collinear.ai to obtain a steer API key for generating user personas and interactions.

```python
steer_api_key = "your-steer-key"
```

## Environment Variables (Optional)

You can set your API keys as environment variables to avoid hardcoding them:

```bash
export COLLINEAR_STEER_API_KEY="your-steer-key"
export OPENAI_API_KEY="your-openai-key"
export TOGETHER_API_KEY="your-together-key"
```

Then use them in your code:

```python
import os

steer_api_key = os.getenv("COLLINEAR_STEER_API_KEY")
assistant_model_api_key = os.getenv("OPENAI_API_KEY")  # or TOGETHER_API_KEY
```

## Basic Client Setup

Here's how to initialize the Collinear client:

```python
from collinear.client import Client

client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-assistant-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key",
    timeout=120,  # Optional: request timeout in seconds
    max_retries=3,  # Optional: max retries for failed requests
    rate_limit_retries=6,  # Optional: max retries for rate limits
)
```

## Configuration Files

The Collinear SDK uses JSON configuration files to manage settings. Here's a basic structure:

### simulation_config.json
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
  "steering_config_file": "steering_config_airline.json"
}
```

### steering_config.json
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
  "locations": ["USA", "Canada", "UK", "Australia"],
  "languages": ["English", "Spanish", "French"],
  "tasks": ["airline support"]
}
```

## Testing Your Setup

Run this simple test to verify everything is working:

```python
from collinear.client import Client

# Initialize client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-key"
)

# Simple steering config
steer_config = {
    "ages": [25],
    "genders": ["female"],
    "occupations": ["student"],
    "intents": ["search_flights"],
    "traits": {
        "impatience": [0]
    },
    "tasks": ["airline support"]
}

# Run a single simulation
simulations = client.simulate(
    steer_config=steer_config,
    k=1,
    num_exchanges=1
)

print("Simulation successful!")
print(f"Generated {len(simulations)} conversation(s)")
```

## Troubleshooting

### Common Issues

1. **API Key Errors**: Ensure your API keys are valid and have sufficient credits
2. **Timeout Errors**: Increase the `timeout` parameter in your client configuration
3. **Rate Limit Errors**: Increase `batch_delay` between simulations
4. **Import Errors**: Make sure you've installed the collinear package correctly

### Getting Help

- Check the [API Reference](api/client.md) for detailed parameter information
- Review the [Examples](examples/) for common usage patterns
- Contact info@collinear.ai for API key issues or technical support

## Next Steps

Once you have the Collinear SDK installed and configured:

1. Follow the [Quick Start Guide](quick-start.md) to run your first simulation
2. Explore the [Configuration Guide](configuration.md) to customize your simulations
3. Check out the [Examples](examples/) for different use cases

---

*Ready to start simulating? Head to the [Quick Start Guide](quick-start.md) to run your first simulation!*

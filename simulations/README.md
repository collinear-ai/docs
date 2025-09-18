# Collinear Simulations Documentation

Generate realistic, high-fidelity, multi-turn user interactions conditioned on user intents and user personas (traits and attributes). Simulations can be used to evaluate AI agents, create training data for long horizon RL training, or prototype product flows.

## ✨ Features

- **Personas by design**: Compose personas from personality traits and user attributes (e.g., age, gender, occupations, location)
- **Intent control**: Drive conversations with an intent taxonomy (e.g., billing dispute, cancellation, upgrade, tech support)
- **Multi-turn realism**: User personas that do not fade even after >25 turns
- **Pluggable agents**: Use your own Assistant API (open or closed source) to benchmark against or create custom evals
- **Evaluation hooks**: Intent understanding, query resolution, looping behavior, knowledge gaps, customer satisfaction, tone, empathy, etc.
- **RL-friendly**: Push to Prime Intellect hub with single line of code

## 📚 Documentation

### Getting Started
- [Installation & Setup](getting-started/installation.md)
- [Quick Start Guide](getting-started/quick-start.md)
- [Configuration Guide](getting-started/configuration.md)

### API Reference
- [Client API](api/client.md)
- [Schemas & Data Models](api/schemas.md)
- [Configuration Reference](api/configuration.md)

### Examples & Tutorials
- [Basic Simulation](examples/basic-simulation.md)
- [Agent Evaluation](examples/agent-evaluation.md)
- [Custom Personas](examples/custom-personas.md)
- [Batch Processing](examples/batch-processing.md)
- [RL Training Data](examples/rl-training.md)

## 🚀 Quick Example

```python
from collinear.client import Client

# Initialize client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-key"
)

# Load steering configuration
import json
with open("steering_config.json") as f:
    steer_config = json.load(f)

# Generate simulations
simulations = client.simulate(
    steer_config=steer_config,
    k=5,  # Generate 5 conversations
    num_exchanges=3  # 3 user-assistant exchanges each
)

# Assess the conversations
assessment = client.assess(simulations)
print(f"Assessment: {assessment.message}")
```

## 📖 What's Next?

1. **Install the SDK** with the [Installation & Setup](getting-started/installation.md) guide.
2. **Run your first workflow** using the [Quick Start Guide](getting-started/quick-start.md).
3. **Tune configurations** with the [Configuration Guide](getting-started/configuration.md).
4. **Learn from examples** in [Basic Simulation](examples/basic-simulation.md) and [Agent Evaluation](examples/agent-evaluation.md).

## 🤝 Support

- **Documentation Issues**: Open an issue in this repository
- **API Support**: Contact info@collinear.ai for API key and technical support
- **Feature Requests**: Submit via GitHub issues

---

*This documentation covers the Collinear Simulations library for creating realistic AI agent interactions and RL training data.*

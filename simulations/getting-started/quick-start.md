# Quick Start Guide

This guide will walk you through running your first simulation with the Collinear SDK in just a few minutes.

## Prerequisites

- Collinear SDK installed (see [Installation Guide](installation.md))
- API keys configured
- Basic understanding of Python

## Step 1: Basic Setup

Create a new Python file and import the necessary modules:

```python
import json
from collinear.client import Client
```

## Step 2: Initialize the Client

Set up your Collinear client with your API credentials:

```python
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)
```

## Step 3: Define Your Steering Configuration

Create a steering configuration that defines the personas and scenarios you want to simulate:

```python
steer_config = {
    "ages": [25, 45],
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
```

## Step 4: Run Your First Simulation

Generate simulated conversations:

```python
simulations = client.simulate(
    steer_config=steer_config,
    k=3,  # Generate 3 conversations
    num_exchanges=2,  # 2 user-assistant exchanges each
    batch_delay=0.2  # Small delay between requests
)

print(f"Generated {len(simulations)} simulations")
```

## Step 5: Examine the Results

Let's look at what was generated:

```python
for i, sim in enumerate(simulations, 1):
    print(f"\n=== Simulation {i} ===")
    
    # Show the persona details
    if sim.steer:
        print(f"Persona: {sim.steer.age}yo {sim.steer.gender} {sim.steer.occupation}")
        print(f"Intent: {sim.steer.intent}")
        print(f"Traits: {sim.steer.traits}")
    
    # Show the conversation
    print("\nConversation:")
    for msg in sim.conv_prefix:
        role = msg.get('role', '')
        content = msg.get('content', '')
        if content:
            print(f"{role}: {content}")
    
    print(f"assistant: {sim.response}")
    print("-" * 50)
```

## Step 6: Assess the Conversations (Optional)

Evaluate the quality of your generated conversations:

```python
assessment = client.assess(simulations)
print(f"Assessment: {assessment.message}")

for i, scores_map in enumerate(assessment.evaluation_result):
    print(f"\nSimulation {i+1} Scores:")
    for metric, score in scores_map.items():
        print(f"  {metric}: {score.score}")
        if score.rationale:
            print(f"    Rationale: {score.rationale}")
```

## Complete Example

Here's the complete script:

```python
import json
from collinear.client import Client

# Initialize client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)

# Define steering configuration
steer_config = {
    "ages": [25, 45],
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

# Run simulations
print("Generating simulations...")
simulations = client.simulate(
    steer_config=steer_config,
    k=3,
    num_exchanges=2,
    batch_delay=0.2
)

# Display results
for i, sim in enumerate(simulations, 1):
    print(f"\n=== Simulation {i} ===")
    
    if sim.steer:
        print(f"Persona: {sim.steer.age}yo {sim.steer.gender} {sim.steer.occupation}")
        print(f"Intent: {sim.steer.intent}")
        print(f"Traits: {sim.steer.traits}")
    
    print("\nConversation:")
    for msg in sim.conv_prefix:
        role = msg.get('role', '')
        content = msg.get('content', '')
        if content:
            print(f"{role}: {content}")
    
    print(f"assistant: {sim.response}")

# Assess conversations
print("\n" + "="*50)
print("ASSESSMENT")
print("="*50)

assessment = client.assess(simulations)
print(f"Overall: {assessment.message}")

for i, scores_map in enumerate(assessment.evaluation_result):
    print(f"\nSimulation {i+1}:")
    for metric, score in scores_map.items():
        print(f"  {metric}: {score.score}")
        if score.rationale:
            print(f"    {score.rationale}")

print("\nDone! 🎉")
```

## Understanding the Output

### Simulation Results

Each simulation contains:
- **`conv_prefix`**: The conversation history (user messages)
- **`response`**: The assistant's response
- **`steer`**: The persona details that generated this conversation

### Persona Details

The `steer` object contains:
- **Demographics**: `age`, `gender`, `occupation`, `location`, `language`
- **Intent**: What the user wants to accomplish
- **Traits**: Personality characteristics with intensity levels (-2 to +2)
- **Task**: The domain/context of the conversation

### Assessment Results

The assessment provides:
- **Scores**: Numerical ratings for different metrics
- **Rationale**: Explanations for the scores
- **Overall message**: Summary of the assessment

## Next Steps

Now that you've run your first simulation:

1. **Experiment with different configurations** - Try different ages, traits, and intents
2. **Increase complexity** - Add more exchanges or mix multiple traits
3. **Explore different domains** - Try different task types (customer service, e-commerce, etc.)
4. **Learn about configuration** - Check out the [Configuration Guide](configuration.md)
5. **See more examples** - Browse the [Examples section](examples/)

## Common Parameters

Here are the key parameters you can adjust:

### Simulation Parameters
- **`k`**: Number of simulations to generate
- **`num_exchanges`**: Number of user-assistant exchanges per simulation
- **`batch_delay`**: Delay between requests (to avoid rate limits)
- **`steer_temperature`**: Creativity level for persona generation (0.0-1.0)
- **`steer_max_tokens`**: Maximum tokens for persona generation
- **`mix_traits`**: Whether to combine multiple traits per persona

### Steering Configuration
- **`ages`**: List of ages to sample from
- **`genders`**: List of genders
- **`occupations`**: List of occupations
- **`intents`**: List of user intents/goals
- **`traits`**: Dictionary mapping trait names to intensity levels
- **`locations`**: List of geographic locations
- **`languages`**: List of languages
- **`tasks`**: List of task domains

## Troubleshooting

### Common Issues

1. **Empty simulations**: Check that your steering config has valid combinations
2. **API errors**: Verify your API keys and check rate limits
3. **Timeout errors**: Increase the `timeout` parameter or `batch_delay`

### Getting Help

- Check the [API Reference](api/client.md) for detailed parameter information
- Review the [Configuration Guide](configuration.md) for advanced setup
- Contact info@collinear.ai for technical support

---

*Ready for more? Check out the [Configuration Guide](configuration.md) to learn about advanced configuration options!*

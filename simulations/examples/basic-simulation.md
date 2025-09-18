# Basic Simulation Example

This example demonstrates how to run a basic simulation with the Collinear SDK.

## Overview

This example shows:
- Setting up the Collinear client
- Creating a simple steering configuration
- Running simulations
- Examining the results
- Basic assessment

## Complete Example

```python
import json
from collinear.client import Client

# Initialize the client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)

# Define a simple steering configuration
steer_config = {
    "ages": [25, 35],
    "genders": ["female", "male"],
    "occupations": ["student", "business_owner"],
    "intents": ["search_flights", "make_booking"],
    "traits": {
        "impatience": [0, 1],
        "confusion": [0]
    },
    "locations": ["USA"],
    "languages": ["English"],
    "tasks": ["airline support"]
}

# Run simulations
print("Generating simulations...")
simulations = client.simulate(
    steer_config=steer_config,
    k=4,  # Generate 4 conversations
    num_exchanges=2,  # 2 user-assistant exchanges each
    batch_delay=0.2
)

# Display results
print(f"\nGenerated {len(simulations)} simulations:")
print("=" * 50)

for i, sim in enumerate(simulations, 1):
    print(f"\n--- Simulation {i} ---")
    
    # Show persona details
    if sim.steer:
        print(f"Persona: {sim.steer.age}yo {sim.steer.gender} {sim.steer.occupation}")
        print(f"Intent: {sim.steer.intent}")
        print(f"Traits: {sim.steer.traits}")
    
    # Show conversation
    print("\nConversation:")
    for msg in sim.conv_prefix:
        role = msg.get('role', '')
        content = msg.get('content', '')
        if content:
            print(f"  {role}: {content}")
    
    print(f"  assistant: {sim.response}")

# Assess the conversations
print("\n" + "=" * 50)
print("ASSESSMENT")
print("=" * 50)

assessment = client.assess(simulations)
print(f"Overall: {assessment.message}")

for i, scores_map in enumerate(assessment.evaluation_result):
    print(f"\nSimulation {i+1} Scores:")
    for metric, score in scores_map.items():
        print(f"  {metric}: {score.score}")
        if score.rationale:
            print(f"    Rationale: {score.rationale}")

print("\nDone! 🎉")
```

## Expected Output

```
Generating simulations...

Generated 4 simulations:
==================================================

--- Simulation 1 ---
Persona: 25yo female student
Intent: search_flights
Traits: {'impatience': 0, 'confusion': 0}

Conversation:
  user: I'm looking for flights from New York to Los Angeles for next week
  user: What are the cheapest options available?
  assistant: I'd be happy to help you find affordable flights from New York to Los Angeles for next week. Let me search for the best options for you.

--- Simulation 2 ---
Persona: 35yo male business_owner
Intent: make_booking
Traits: {'impatience': 1, 'confusion': 0}

Conversation:
  user: I need to book a flight to Chicago for tomorrow
  user: I need it confirmed ASAP, this is urgent
  assistant: I understand this is urgent. Let me help you find and book a flight to Chicago for tomorrow right away.

==================================================
ASSESSMENT
==================================================

Overall: Assessment completed successfully

Simulation 1 Scores:
  safety: 8.5
    Rationale: Response was helpful and appropriate
  helpfulness: 7.0
    Rationale: Addressed user's needs adequately

Simulation 2 Scores:
  safety: 8.0
    Rationale: Good response to urgent request
  helpfulness: 8.5
    Rationale: Very helpful and responsive to urgency

Done! 🎉
```

## Step-by-Step Breakdown

### 1. Client Initialization

```python
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)
```

This creates a client that will:
- Use OpenAI's API for the assistant model
- Use Collinear's steer API for generating user personas
- Handle retries and rate limiting automatically

### 2. Steering Configuration

```python
steer_config = {
    "ages": [25, 35],
    "genders": ["female", "male"],
    "occupations": ["student", "business_owner"],
    "intents": ["search_flights", "make_booking"],
    "traits": {
        "impatience": [0, 1],
        "confusion": [0]
    },
    "locations": ["USA"],
    "languages": ["English"],
    "tasks": ["airline support"]
}
```

This configuration will generate personas with:
- Ages: 25 or 35
- Genders: female or male
- Occupations: student or business_owner
- Intents: search_flights or make_booking
- Traits: impatience (0 or 1) and confusion (0)
- Location: USA
- Language: English
- Task: airline support

### 3. Running Simulations

```python
simulations = client.simulate(
    steer_config=steer_config,
    k=4,  # Generate 4 conversations
    num_exchanges=2,  # 2 user-assistant exchanges each
    batch_delay=0.2
)
```

This generates 4 simulations, each with 2 user-assistant exchanges.

### 4. Examining Results

Each simulation contains:
- **`conv_prefix`**: The conversation history (user messages)
- **`response`**: The assistant's response
- **`steer`**: The persona details that generated this conversation

### 5. Assessment

```python
assessment = client.assess(simulations)
```

This evaluates the quality of the generated conversations using a safety rubric.

## Customizing the Example

### Different Demographics

```python
steer_config = {
    "ages": [18, 25, 35, 45, 55, 65],
    "genders": ["male", "female", "other"],
    "occupations": ["student", "employed", "business_owner", "retired"],
    # ... rest of config
}
```

### More Complex Traits

```python
steer_config = {
    # ... demographics
    "traits": {
        "impatience": [-2, -1, 0, 1, 2],
        "confusion": [-1, 0, 1],
        "skeptical": [0, 1, 2],
        "urgency": [0, 1, 2]
    },
    # ... rest of config
}
```

### Different Domains

```python
steer_config = {
    # ... demographics and traits
    "intents": ["product_inquiry", "order_status", "return_request"],
    "tasks": ["retail_support"]
}
```

### More Exchanges

```python
simulations = client.simulate(
    steer_config=steer_config,
    k=4,
    num_exchanges=5,  # 5 user-assistant exchanges
    batch_delay=0.2
)
```

## Troubleshooting

### Common Issues

1. **No simulations generated**: Check that your steering config has valid combinations
2. **API errors**: Verify your API keys and check rate limits
3. **Empty conversations**: Ensure your intents and tasks are appropriate for the domain

### Debugging

Add some debug output to see what's happening:

```python
# Check the number of possible combinations
from collinear.schemas.steer import SteerConfig
config = SteerConfig.from_input(steer_config)
combinations = config.combinations()
print(f"Possible combinations: {len(combinations)}")

# Run with progress bar
simulations = client.simulate(
    steer_config=steer_config,
    k=4,
    progress=True  # Shows progress bar
)
```

## Next Steps

Now that you've run a basic simulation:

1. **Try different configurations** - Experiment with different ages, traits, and intents
2. **Increase complexity** - Add more exchanges or mix multiple traits
3. **Explore different domains** - Try customer service, e-commerce, etc.
4. **Check out advanced examples** - See the [RL Training](rl-training.md) and [Agent Evaluation](agent-evaluation.md) examples

---

*Ready for more? Check out the [RL Training Example](rl-training.md) to see how to generate training data for reinforcement learning!*

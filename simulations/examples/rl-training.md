# RL Training Data Generation

This example demonstrates how to generate high-quality training data for reinforcement learning using the Collinear SDK.

## Overview

This example shows:
- Setting up the client for RL training
- Creating diverse persona configurations
- Generating large-scale training data
- Formatting data for RL training
- Pushing to Prime Intellect hub

## Complete Example

```python
import json
from pathlib import Path
from collinear.client import Client

# Initialize the client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)

# Define comprehensive steering configuration for RL training
steer_config = {
    "ages": [18, 25, 35, 45, 55, 65, 75],
    "genders": ["male", "female", "other"],
    "occupations": ["student", "employed", "business_owner", "freelancer", "retired", "unemployed"],
    "intents": [
        "search_flights", "make_booking", "modify_booking", "cancel_booking",
        "check_status", "seat_selection", "baggage_inquiry", "lounge_access",
        "frequent_flyer", "refund_request", "complaint", "escalation"
    ],
    "traits": {
        "impatience": [-2, -1, 0, 1, 2],
        "confusion": [-2, -1, 0, 1, 2],
        "skeptical": [-2, -1, 0, 1, 2],
        "urgency": [-2, -1, 0, 1, 2],
        "politeness": [-2, -1, 0, 1, 2],
        "technical": [-2, -1, 0, 1, 2]
    },
    "locations": ["USA", "Canada", "UK", "Australia", "Germany", "Japan"],
    "languages": ["English", "Spanish", "French", "German", "Japanese"],
    "tasks": ["airline support"]
}

# Generate training data
print("Generating RL training data...")
print(f"Configuration: {len(steer_config['ages'])} ages × {len(steer_config['genders'])} genders × {len(steer_config['intents'])} intents")

# Run simulations with high diversity
simulations = client.simulate(
    steer_config=steer_config,
    k=1000,  # Generate 1000 conversations
    num_exchanges=3,  # 3 user-assistant exchanges each
    batch_delay=0.1,  # Faster processing
    mix_traits=True,  # Combine traits for more complex personas
    progress=True,  # Show progress bar
    max_concurrency=2  # Parallel processing
)

print(f"Generated {len(simulations)} training samples")

# Format data for RL training
training_data = []
for i, sim in enumerate(simulations):
    # Extract conversation history
    conversation = []
    for msg in sim.conv_prefix:
        conversation.append({
            "role": msg.get('role', ''),
            "content": msg.get('content', '')
        })
    
    # Add assistant response
    conversation.append({
        "role": "assistant",
        "content": sim.response
    })
    
    # Extract persona metadata
    persona = {}
    if sim.steer:
        persona = {
            "age": sim.steer.age,
            "gender": sim.steer.gender,
            "occupation": sim.steer.occupation,
            "intent": sim.steer.intent,
            "traits": sim.steer.traits,
            "location": sim.steer.location,
            "language": sim.steer.language,
            "task": sim.steer.task
        }
    
    # Create training sample
    training_sample = {
        "id": f"rl_training_{i:06d}",
        "conversation": conversation,
        "persona": persona,
        "metadata": {
            "num_exchanges": len(sim.conv_prefix),
            "total_tokens": sum(len(msg.get('content', '')) for msg in conversation),
            "domain": "airline_support"
        }
    }
    
    training_data.append(training_sample)

# Save training data
output_dir = Path("rl_training_data")
output_dir.mkdir(exist_ok=True)

# Save as JSONL for easy processing
output_file = output_dir / "airline_rl_training.jsonl"
with output_file.open('w', encoding='utf-8') as f:
    for sample in training_data:
        f.write(json.dumps(sample, ensure_ascii=False) + '\n')

print(f"Saved training data to: {output_file}")

# Generate summary statistics
print("\n" + "="*50)
print("TRAINING DATA SUMMARY")
print("="*50)

# Demographics distribution
age_dist = {}
gender_dist = {}
intent_dist = {}
trait_dist = {}

for sample in training_data:
    persona = sample["persona"]
    
    # Age distribution
    age = persona.get("age")
    if age:
        age_group = f"{(age//10)*10}-{(age//10)*10+9}"
        age_dist[age_group] = age_dist.get(age_group, 0) + 1
    
    # Gender distribution
    gender = persona.get("gender")
    if gender:
        gender_dist[gender] = gender_dist.get(gender, 0) + 1
    
    # Intent distribution
    intent = persona.get("intent")
    if intent:
        intent_dist[intent] = intent_dist.get(intent, 0) + 1
    
    # Trait distribution
    traits = persona.get("traits", {})
    for trait, intensity in traits.items():
        if trait not in trait_dist:
            trait_dist[trait] = {}
        trait_dist[trait][intensity] = trait_dist[trait].get(intensity, 0) + 1

print(f"Total samples: {len(training_data)}")
print(f"Average conversation length: {sum(s['metadata']['num_exchanges'] for s in training_data) / len(training_data):.1f} exchanges")

print("\nAge distribution:")
for age_group, count in sorted(age_dist.items()):
    print(f"  {age_group}: {count} ({count/len(training_data)*100:.1f}%)")

print("\nGender distribution:")
for gender, count in sorted(gender_dist.items()):
    print(f"  {gender}: {count} ({count/len(training_data)*100:.1f}%)")

print("\nIntent distribution:")
for intent, count in sorted(intent_dist.items()):
    print(f"  {intent}: {count} ({count/len(training_data)*100:.1f}%)")

print("\nTrait intensity distribution:")
for trait, intensities in sorted(trait_dist.items()):
    print(f"  {trait}:")
    for intensity, count in sorted(intensities.items()):
        print(f"    {intensity}: {count} ({count/len(training_data)*100:.1f}%)")

# Optional: Push to Prime Intellect hub
print("\n" + "="*50)
print("PRIME INTELLECT HUB INTEGRATION")
print("="*50)

try:
    # Install prime if not already installed
    import subprocess
    import sys
    
    try:
        import prime
    except ImportError:
        print("Installing Prime Intellect CLI...")
        subprocess.check_call([sys.executable, "-m", "pip", "install", "prime"])
        import prime
    
    # Push to Prime Intellect hub
    print("Pushing to Prime Intellect hub...")
    
    # Create a simple environment file
    env_file = output_dir / "environment.yaml"
    with env_file.open('w') as f:
        f.write("""name: airline-rl-training
description: Airline customer service RL training data
version: 1.0.0
dependencies:
  - python>=3.8
  - transformers
  - torch
  - datasets
""")
    
    # Push to hub
    result = prime.push(
        path=str(output_dir),
        name="airline-rl-training",
        description="High-quality airline customer service training data for RL",
        tags=["rl", "training", "airline", "customer-service"]
    )
    
    print(f"Successfully pushed to Prime Intellect hub: {result}")
    
except Exception as e:
    print(f"Could not push to Prime Intellect hub: {e}")
    print("You can manually upload the data or install the Prime CLI")

print("\nRL training data generation complete! 🎉")
```

## Advanced RL Training Configuration

### High-Diversity Configuration

```python
# For maximum diversity in training data
steer_config = {
    "ages": list(range(18, 80, 2)),  # Every 2 years from 18-80
    "genders": ["male", "female", "non-binary", "other"],
    "occupations": [
        "student", "teacher", "engineer", "doctor", "lawyer", "artist",
        "business_owner", "manager", "consultant", "freelancer", "retired"
    ],
    "intents": [
        "search", "book", "modify", "cancel", "refund", "complaint",
        "escalation", "technical_issue", "billing_question", "status_check"
    ],
    "traits": {
        "impatience": [-2, -1, 0, 1, 2],
        "confusion": [-2, -1, 0, 1, 2],
        "skeptical": [-2, -1, 0, 1, 2],
        "urgency": [-2, -1, 0, 1, 2],
        "politeness": [-2, -1, 0, 1, 2],
        "technical": [-2, -1, 0, 1, 2],
        "assertiveness": [-2, -1, 0, 1, 2],
        "emotional": [-2, -1, 0, 1, 2]
    },
    "locations": ["USA", "Canada", "UK", "Australia", "Germany", "France", "Japan", "Brazil"],
    "languages": ["English", "Spanish", "French", "German", "Japanese", "Portuguese"],
    "tasks": ["customer_service"]
}
```

### Domain-Specific Configurations

#### E-commerce Support
```python
ecommerce_config = {
    "ages": [18, 25, 35, 45, 55, 65],
    "genders": ["male", "female", "other"],
    "occupations": ["student", "employed", "business_owner", "retired"],
    "intents": [
        "product_inquiry", "order_status", "return_request", "refund_request",
        "shipping_question", "payment_issue", "complaint", "technical_issue"
    ],
    "traits": {
        "impatience": [-1, 0, 1, 2],
        "confusion": [0, 1, 2],
        "skeptical": [0, 1, 2]
    },
    "tasks": ["retail_support"]
}
```

#### Technical Support
```python
tech_support_config = {
    "ages": [20, 30, 40, 50],
    "genders": ["male", "female"],
    "occupations": ["employed", "business_owner", "freelancer"],
    "intents": [
        "technical_issue", "setup_help", "troubleshooting", "feature_request",
        "bug_report", "integration_help", "performance_issue"
    ],
    "traits": {
        "technical": [-2, -1, 0, 1, 2],
        "impatience": [0, 1, 2],
        "confusion": [0, 1, 2]
    },
    "tasks": ["tech_support"]
}
```

## Data Formatting for Different RL Frameworks

### Hugging Face Datasets

```python
from datasets import Dataset

# Convert to Hugging Face format
hf_dataset = Dataset.from_list(training_data)

# Save as Hugging Face dataset
hf_dataset.save_to_disk("airline_rl_dataset")

# Or push to Hugging Face Hub
hf_dataset.push_to_hub("your-username/airline-rl-dataset")
```

### PyTorch Dataset

```python
import torch
from torch.utils.data import Dataset

class RLTrainingDataset(Dataset):
    def __init__(self, data):
        self.data = data
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        sample = self.data[idx]
        return {
            "conversation": sample["conversation"],
            "persona": sample["persona"],
            "metadata": sample["metadata"]
        }

# Create PyTorch dataset
pytorch_dataset = RLTrainingDataset(training_data)
dataloader = torch.utils.data.DataLoader(pytorch_dataset, batch_size=32, shuffle=True)
```

### TensorFlow Dataset

```python
import tensorflow as tf

# Convert to TensorFlow format
tf_dataset = tf.data.Dataset.from_generator(
    lambda: (sample for sample in training_data),
    output_signature={
        "conversation": tf.TensorSpec(shape=(), dtype=tf.string),
        "persona": tf.TensorSpec(shape=(), dtype=tf.string),
        "metadata": tf.TensorSpec(shape=(), dtype=tf.string)
    }
)

# Save as TFRecord
tf_dataset.save("airline_rl_dataset.tfrecord")
```

## Quality Control and Filtering

### Filter by Quality Scores

```python
# Assess all simulations
assessment = client.assess(simulations)

# Filter by quality scores
high_quality_data = []
for i, (sim, scores_map) in enumerate(zip(simulations, assessment.evaluation_result)):
    # Get average score across all metrics
    avg_score = sum(score.score for score in scores_map.values() if score.score is not None) / len(scores_map)
    
    if avg_score >= 7.0:  # Only keep high-quality samples
        high_quality_data.append(training_data[i])

print(f"Kept {len(high_quality_data)} high-quality samples out of {len(training_data)}")
```

### Filter by Conversation Length

```python
# Filter by conversation length
filtered_data = [
    sample for sample in training_data
    if sample["metadata"]["num_exchanges"] >= 2  # At least 2 exchanges
    and sample["metadata"]["num_exchanges"] <= 10  # At most 10 exchanges
]

print(f"Filtered to {len(filtered_data)} samples with appropriate length")
```

### Filter by Trait Combinations

```python
# Filter for specific trait combinations
complex_personas = [
    sample for sample in training_data
    if len(sample["persona"].get("traits", {})) >= 2  # At least 2 traits
    and any(abs(intensity) >= 1 for intensity in sample["persona"].get("traits", {}).values())  # At least one strong trait
]

print(f"Found {len(complex_personas)} samples with complex personas")
```

## Batch Processing for Large Datasets

```python
def generate_large_dataset(config, total_samples=10000, batch_size=1000):
    """Generate a large dataset in batches to avoid memory issues."""
    all_data = []
    
    for batch_start in range(0, total_samples, batch_size):
        batch_size_actual = min(batch_size, total_samples - batch_start)
        
        print(f"Generating batch {batch_start//batch_size + 1}/{(total_samples-1)//batch_size + 1}")
        
        # Generate batch
        simulations = client.simulate(
            steer_config=config,
            k=batch_size_actual,
            num_exchanges=3,
            batch_delay=0.1,
            progress=True
        )
        
        # Process batch
        batch_data = []
        for sim in simulations:
            # ... process simulation ...
            batch_data.append(processed_sample)
        
        all_data.extend(batch_data)
        
        # Save intermediate results
        if batch_start > 0:
            with open(f"rl_training_batch_{batch_start//batch_size}.jsonl", "w") as f:
                for sample in batch_data:
                    f.write(json.dumps(sample) + "\n")
    
    return all_data

# Generate large dataset
large_dataset = generate_large_dataset(steer_config, total_samples=10000, batch_size=1000)
```

## Integration with RL Training

### Reward Function Design

```python
def calculate_reward(conversation, persona):
    """Calculate reward based on conversation quality and persona alignment."""
    reward = 0.0
    
    # Base reward for completing conversation
    reward += 1.0
    
    # Reward for addressing user intent
    intent = persona.get("intent")
    if intent and any(intent.lower() in msg.get("content", "").lower() 
                     for msg in conversation if msg.get("role") == "assistant"):
        reward += 2.0
    
    # Reward for appropriate tone based on traits
    traits = persona.get("traits", {})
    if traits.get("politeness", 0) > 0:
        # Reward polite responses
        if any("please" in msg.get("content", "").lower() 
               for msg in conversation if msg.get("role") == "assistant"):
            reward += 1.0
    
    # Penalty for inappropriate responses
    if traits.get("impatience", 0) > 0:
        # Penalize overly long responses for impatient users
        assistant_responses = [msg for msg in conversation if msg.get("role") == "assistant"]
        if any(len(msg.get("content", "")) > 200 for msg in assistant_responses):
            reward -= 0.5
    
    return reward

# Add rewards to training data
for sample in training_data:
    sample["reward"] = calculate_reward(sample["conversation"], sample["persona"])
```

## Next Steps

Now that you have high-quality RL training data:

1. **Choose your RL framework** - PPO, DQN, A2C, etc.
2. **Design reward functions** - Based on conversation quality and persona alignment
3. **Implement training loop** - Use your chosen framework
4. **Evaluate performance** - Test on held-out data
5. **Iterate and improve** - Refine based on results

---

*Ready to evaluate your AI agent? Check out the [Agent Evaluation Example](agent-evaluation.md) to see how to test your trained models!*

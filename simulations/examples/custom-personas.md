# Custom Personas Example

This example demonstrates how to create and use custom persona configurations for specific use cases and domains.

## Overview

This example shows:
- Designing custom persona types
- Creating domain-specific configurations
- Using trait combinations effectively
- Testing persona behavior
- Optimizing for specific scenarios

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

# Define custom persona types
persona_types = {
    "frustrated_customer": {
        "description": "Customers who are frustrated with service issues",
        "traits": {
            "impatience": [1, 2],
            "skeptical": [1, 2],
            "urgency": [1, 2],
            "politeness": [-1, 0]
        },
        "intents": ["complaint", "escalation", "refund_request", "technical_issue"],
        "ages": [25, 35, 45, 55],
        "genders": ["male", "female"],
        "occupations": ["employed", "business_owner"]
    },
    
    "confused_elderly": {
        "description": "Elderly customers who need extra help and patience",
        "traits": {
            "confusion": [1, 2],
            "impatience": [-1, 0],
            "technical": [-2, -1],
            "politeness": [1, 2]
        },
        "intents": ["technical_issue", "billing_question", "general_inquiry"],
        "ages": [65, 75, 85],
        "genders": ["male", "female"],
        "occupations": ["retired", "unemployed"]
    },
    
    "impatient_business_traveler": {
        "description": "Business travelers who need quick, efficient service",
        "traits": {
            "impatience": [2],
            "urgency": [2],
            "technical": [1, 2],
            "politeness": [0, 1]
        },
        "intents": ["search_flights", "make_booking", "modify_booking", "check_status"],
        "ages": [30, 40, 50],
        "genders": ["male", "female"],
        "occupations": ["business_owner", "employed"]
    },
    
    "budget_conscious_student": {
        "description": "Students looking for the best deals and value",
        "traits": {
            "impatience": [0, 1],
            "confusion": [0, 1],
            "technical": [-1, 0],
            "politeness": [1, 2]
        },
        "intents": ["search_flights", "make_booking", "billing_question"],
        "ages": [18, 20, 22, 24],
        "genders": ["male", "female"],
        "occupations": ["student"]
    },
    
    "upset_vip_customer": {
        "description": "High-value customers who expect premium service",
        "traits": {
            "impatience": [1, 2],
            "assertiveness": [1, 2],
            "urgency": [1, 2],
            "politeness": [-1, 0]
        },
        "intents": ["escalation", "complaint", "refund_request", "lounge_access"],
        "ages": [35, 45, 55, 65],
        "genders": ["male", "female"],
        "occupations": ["business_owner", "employed"]
    }
}

# Generate personas for each type
print("Generating custom personas...")
print("=" * 50)

all_personas = {}
for persona_type, config in persona_types.items():
    print(f"\nGenerating {persona_type} personas...")
    print(f"Description: {config['description']}")
    
    # Create steering configuration
    steer_config = {
        "ages": config["ages"],
        "genders": config["genders"],
        "occupations": config["occupations"],
        "intents": config["intents"],
        "traits": config["traits"],
        "locations": ["USA", "Canada", "UK"],
        "languages": ["English"],
        "tasks": ["customer_service"]
    }
    
    # Generate personas
    personas = client.simulate(
        steer_config=steer_config,
        k=20,  # Generate 20 personas of this type
        num_exchanges=3,
        batch_delay=0.1,
        progress=True
    )
    
    all_personas[persona_type] = personas
    print(f"Generated {len(personas)} {persona_type} personas")

# Analyze persona characteristics
print("\n" + "=" * 50)
print("PERSONA ANALYSIS")
print("=" * 50)

for persona_type, personas in all_personas.items():
    print(f"\n{persona_type.upper().replace('_', ' ')}:")
    print("-" * 30)
    
    # Analyze trait distributions
    trait_distributions = {}
    for persona in personas:
        if persona.steer:
            traits = persona.steer.traits
            for trait, intensity in traits.items():
                if trait not in trait_distributions:
                    trait_distributions[trait] = {}
                trait_distributions[trait][intensity] = trait_distributions[trait].get(intensity, 0) + 1
    
    print("Trait distributions:")
    for trait, intensities in trait_distributions.items():
        print(f"  {trait}:")
        for intensity, count in sorted(intensities.items()):
            print(f"    {intensity}: {count} ({count/len(personas)*100:.1f}%)")
    
    # Analyze intent distributions
    intent_distributions = {}
    for persona in personas:
        if persona.steer and persona.steer.intent:
            intent = persona.steer.intent
            intent_distributions[intent] = intent_distributions.get(intent, 0) + 1
    
    print("Intent distributions:")
    for intent, count in sorted(intent_distributions.items()):
        print(f"  {intent}: {count} ({count/len(personas)*100:.1f}%)")
    
    # Show sample conversations
    print("Sample conversations:")
    for i, persona in enumerate(personas[:3]):  # Show first 3
        print(f"  Sample {i+1}:")
        if persona.steer:
            print(f"    Persona: {persona.steer.age}yo {persona.steer.gender} {persona.steer.occupation}")
            print(f"    Traits: {persona.steer.traits}")
            print(f"    Intent: {persona.steer.intent}")
        
        print("    Conversation:")
        for msg in persona.conv_prefix:
            role = msg.get('role', '')
            content = msg.get('content', '')
            if content:
                print(f"      {role}: {content}")
        print(f"      assistant: {persona.response}")
        print()

# Test persona behavior patterns
print("\n" + "=" * 50)
print("PERSONA BEHAVIOR TESTING")
print("=" * 50)

def analyze_persona_behavior(personas, persona_type):
    """Analyze behavior patterns for a specific persona type."""
    print(f"\n{persona_type.upper().replace('_', ' ')} Behavior Analysis:")
    print("-" * 40)
    
    # Analyze conversation patterns
    conversation_lengths = []
    politeness_scores = []
    urgency_indicators = []
    
    for persona in personas:
        # Conversation length
        conv_length = len(persona.conv_prefix)
        conversation_lengths.append(conv_length)
        
        # Politeness analysis (simple heuristic)
        user_messages = [msg.get('content', '') for msg in persona.conv_prefix if msg.get('role') == 'user']
        polite_words = ['please', 'thank you', 'sorry', 'excuse me']
        politeness_count = sum(1 for msg in user_messages for word in polite_words if word in msg.lower())
        politeness_scores.append(politeness_count)
        
        # Urgency indicators
        urgency_words = ['urgent', 'asap', 'immediately', 'right now', 'quickly']
        urgency_count = sum(1 for msg in user_messages for word in urgency_words if word in msg.lower())
        urgency_indicators.append(urgency_count)
    
    # Calculate statistics
    avg_length = sum(conversation_lengths) / len(conversation_lengths)
    avg_politeness = sum(politeness_scores) / len(politeness_scores)
    avg_urgency = sum(urgency_indicators) / len(urgency_indicators)
    
    print(f"Average conversation length: {avg_length:.1f} exchanges")
    print(f"Average politeness score: {avg_politeness:.1f}")
    print(f"Average urgency indicators: {avg_urgency:.1f}")
    
    # Show extreme cases
    max_length_idx = conversation_lengths.index(max(conversation_lengths))
    min_length_idx = conversation_lengths.index(min(conversation_lengths))
    
    print(f"Longest conversation: {max(conversation_lengths[max_length_idx]} exchanges")
    print(f"Shortest conversation: {min(conversation_lengths[min_length_idx]} exchanges")

# Analyze behavior for each persona type
for persona_type, personas in all_personas.items():
    analyze_persona_behavior(personas, persona_type)

# Create persona-specific evaluation scenarios
print("\n" + "=" * 50)
print("PERSONA-SPECIFIC EVALUATION")
print("=" * 50)

def create_evaluation_scenarios(persona_type, personas, num_scenarios=10):
    """Create evaluation scenarios for a specific persona type."""
    print(f"\nCreating evaluation scenarios for {persona_type}...")
    
    # Select diverse personas for evaluation
    selected_personas = personas[:num_scenarios]
    
    # Generate additional scenarios if needed
    if len(selected_personas) < num_scenarios:
        # Get the original config
        config = persona_types[persona_type]
        steer_config = {
            "ages": config["ages"],
            "genders": config["genders"],
            "occupations": config["occupations"],
            "intents": config["intents"],
            "traits": config["traits"],
            "locations": ["USA", "Canada", "UK"],
            "languages": ["English"],
            "tasks": ["customer_service"]
        }
        
        additional_personas = client.simulate(
            steer_config=steer_config,
            k=num_scenarios - len(selected_personas),
            num_exchanges=3,
            batch_delay=0.1
        )
        
        selected_personas.extend(additional_personas)
    
    return selected_personas

# Create evaluation scenarios for each persona type
evaluation_scenarios = {}
for persona_type in persona_types.keys():
    scenarios = create_evaluation_scenarios(persona_type, all_personas[persona_type])
    evaluation_scenarios[persona_type] = scenarios
    print(f"Created {len(scenarios)} evaluation scenarios for {persona_type}")

# Save persona data
output_dir = Path("custom_personas")
output_dir.mkdir(exist_ok=True)

# Save each persona type
for persona_type, personas in all_personas.items():
    output_file = output_dir / f"{persona_type}_personas.jsonl"
    with output_file.open('w', encoding='utf-8') as f:
        for persona in personas:
            # Convert to serializable format
            persona_data = {
                "persona_type": persona_type,
                "conversation": persona.conv_prefix,
                "response": persona.response,
                "persona_details": {
                    "age": persona.steer.age if persona.steer else None,
                    "gender": persona.steer.gender if persona.steer else None,
                    "occupation": persona.steer.occupation if persona.steer else None,
                    "intent": persona.steer.intent if persona.steer else None,
                    "traits": persona.steer.traits if persona.steer else {},
                    "location": persona.steer.location if persona.steer else None,
                    "language": persona.steer.language if persona.steer else None,
                    "task": persona.steer.task if persona.steer else None
                }
            }
            f.write(json.dumps(persona_data, ensure_ascii=False) + '\n')
    
    print(f"Saved {persona_type} personas to: {output_file}")

# Save evaluation scenarios
eval_output_dir = output_dir / "evaluation_scenarios"
eval_output_dir.mkdir(exist_ok=True)

for persona_type, scenarios in evaluation_scenarios.items():
    output_file = eval_output_dir / f"{persona_type}_evaluation.jsonl"
    with output_file.open('w', encoding='utf-8') as f:
        for scenario in scenarios:
            scenario_data = {
                "persona_type": persona_type,
                "conversation": scenario.conv_prefix,
                "response": scenario.response,
                "persona_details": {
                    "age": scenario.steer.age if scenario.steer else None,
                    "gender": scenario.steer.gender if scenario.steer else None,
                    "occupation": scenario.steer.occupation if scenario.steer else None,
                    "intent": scenario.steer.intent if scenario.steer else None,
                    "traits": scenario.steer.traits if scenario.steer else {},
                    "location": scenario.steer.location if scenario.steer else None,
                    "language": scenario.steer.language if scenario.steer else None,
                    "task": scenario.steer.task if scenario.steer else None
                }
            }
            f.write(json.dumps(scenario_data, ensure_ascii=False) + '\n')
    
    print(f"Saved {persona_type} evaluation scenarios to: {output_file}")

# Generate persona summary report
summary_file = output_dir / "persona_summary.md"
with summary_file.open('w', encoding='utf-8') as f:
    f.write("# Custom Personas Summary\n\n")
    f.write(f"**Generated:** {len(all_personas)} persona types\n")
    f.write(f"**Total personas:** {sum(len(personas) for personas in all_personas.values())}\n\n")
    
    f.write("## Persona Types\n\n")
    for persona_type, config in persona_types.items():
        f.write(f"### {persona_type.replace('_', ' ').title()}\n")
        f.write(f"**Description:** {config['description']}\n")
        f.write(f"**Count:** {len(all_personas[persona_type])}\n")
        f.write(f"**Traits:** {', '.join(config['traits'].keys())}\n")
        f.write(f"**Intents:** {', '.join(config['intents'])}\n\n")
    
    f.write("## Files Generated\n\n")
    f.write("- `*_personas.jsonl`: Raw persona data\n")
    f.write("- `evaluation_scenarios/*_evaluation.jsonl`: Evaluation scenarios\n")
    f.write("- `persona_summary.md`: This summary\n")

print(f"Saved persona summary to: {summary_file}")

print("\nCustom persona generation complete! 🎉")
print(f"All data saved in: {output_dir}")
```

## Advanced Persona Design Patterns

### Trait Combination Strategies

```python
# Complementary traits (work well together)
complementary_traits = {
    "confused_elderly": {
        "confusion": [1, 2],
        "impatience": [-1, 0],  # Not impatient when confused
        "politeness": [1, 2]    # Polite when confused
    },
    "frustrated_expert": {
        "impatience": [1, 2],
        "technical": [1, 2],    # Technical but impatient
        "skeptical": [1, 2]     # Skeptical of solutions
    }
}

# Conflicting traits (create interesting dynamics)
conflicting_traits = {
    "impatient_polite": {
        "impatience": [1, 2],
        "politeness": [1, 2]    # Polite but impatient
    },
    "confused_expert": {
        "confusion": [1, 2],
        "technical": [1, 2]     # Technical but confused
    }
}
```

### Domain-Specific Personas

```python
# Healthcare personas
healthcare_personas = {
    "anxious_patient": {
        "traits": {
            "impatience": [1, 2],
            "confusion": [1, 2],
            "urgency": [1, 2],
            "politeness": [1, 2]
        },
        "intents": ["appointment_booking", "test_results", "medication_question", "emergency"],
        "ages": [25, 35, 45, 55, 65, 75],
        "genders": ["male", "female"],
        "occupations": ["employed", "retired", "unemployed"]
    },
    "busy_doctor": {
        "traits": {
            "impatience": [1, 2],
            "urgency": [2],
            "technical": [2],
            "politeness": [0, 1]
        },
        "intents": ["schedule_management", "patient_consultation", "emergency_response"],
        "ages": [30, 40, 50, 60],
        "genders": ["male", "female"],
        "occupations": ["employed"]
    }
}

# Financial services personas
financial_personas = {
    "worried_investor": {
        "traits": {
            "skeptical": [1, 2],
            "confusion": [0, 1],
            "urgency": [1, 2],
            "politeness": [1, 2]
        },
        "intents": ["portfolio_review", "investment_advice", "market_concern", "account_issue"],
        "ages": [35, 45, 55, 65],
        "genders": ["male", "female"],
        "occupations": ["employed", "business_owner", "retired"]
    }
}
```

### Cultural and Regional Personas

```python
# Regional personas
regional_personas = {
    "formal_european": {
        "traits": {
            "politeness": [1, 2],
            "impatience": [-1, 0],
            "technical": [0, 1]
        },
        "locations": ["UK", "Germany", "France"],
        "languages": ["English", "German", "French"],
        "intents": ["formal_inquiry", "complaint", "escalation"]
    },
    "direct_american": {
        "traits": {
            "impatience": [0, 1],
            "assertiveness": [1, 2],
            "politeness": [0, 1]
        },
        "locations": ["USA", "Canada"],
        "languages": ["English"],
        "intents": ["direct_request", "complaint", "escalation"]
    }
}
```

## Persona Validation and Testing

### Trait Validation

```python
def validate_persona_traits(personas, expected_traits):
    """Validate that personas have the expected trait distributions."""
    validation_results = {}
    
    for trait, expected_intensities in expected_traits.items():
        actual_intensities = {}
        for persona in personas:
            if persona.steer and trait in persona.steer.traits:
                intensity = persona.steer.traits[trait]
                actual_intensities[intensity] = actual_intensities.get(intensity, 0) + 1
        
        validation_results[trait] = {
            "expected": expected_intensities,
            "actual": actual_intensities,
            "valid": all(intensity in expected_intensities for intensity in actual_intensities.keys())
        }
    
    return validation_results

# Validate personas
for persona_type, personas in all_personas.items():
    expected_traits = persona_types[persona_type]["traits"]
    validation = validate_persona_traits(personas, expected_traits)
    
    print(f"\n{persona_type} validation:")
    for trait, result in validation.items():
        status = "✓" if result["valid"] else "✗"
        print(f"  {status} {trait}: {result['actual']}")
```

### Behavior Consistency Testing

```python
def test_persona_consistency(personas, persona_type):
    """Test that personas behave consistently with their traits."""
    consistency_results = []
    
    for persona in personas:
        if not persona.steer:
            continue
        
        traits = persona.steer.traits
        user_messages = [msg.get('content', '') for msg in persona.conv_prefix if msg.get('role') == 'user']
        
        # Test impatience consistency
        if traits.get('impatience', 0) > 0:
            urgency_words = ['urgent', 'asap', 'quickly', 'immediately']
            has_urgency = any(any(word in msg.lower() for word in urgency_words) for msg in user_messages)
            consistency_results.append({
                'persona_id': id(persona),
                'trait': 'impatience',
                'expected': True,
                'actual': has_urgency,
                'consistent': has_urgency
            })
        
        # Test politeness consistency
        if traits.get('politeness', 0) > 0:
            polite_words = ['please', 'thank you', 'sorry', 'excuse me']
            has_politeness = any(any(word in msg.lower() for word in polite_words) for msg in user_messages)
            consistency_results.append({
                'persona_id': id(persona),
                'trait': 'politeness',
                'expected': True,
                'actual': has_politeness,
                'consistent': has_politeness
            })
    
    return consistency_results

# Test consistency for each persona type
for persona_type, personas in all_personas.items():
    consistency = test_persona_consistency(personas, persona_type)
    consistent_count = sum(1 for result in consistency if result['consistent'])
    total_count = len(consistency)
    
    print(f"{persona_type} consistency: {consistent_count}/{total_count} ({consistent_count/total_count*100:.1f}%)")
```

## Next Steps

Now that you have custom personas:

1. **Use them for evaluation** - Test your agents against specific persona types
2. **Create targeted training data** - Generate data for specific scenarios
3. **Develop persona-specific strategies** - Design responses for different persona types
4. **Monitor persona behavior** - Track how personas evolve over time
5. **Iterate and refine** - Improve personas based on real-world feedback

---

*Ready to process large datasets? Check out the [Batch Processing Example](batch-processing.md) to see how to handle large-scale simulations!*

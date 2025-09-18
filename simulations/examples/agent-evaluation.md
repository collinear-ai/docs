# Agent Evaluation Example

This example demonstrates how to use the Collinear SDK to evaluate AI agents across diverse personas and scenarios.

## Overview

This example shows:
- Setting up evaluation configurations
- Generating diverse test scenarios
- Running evaluations with custom agents
- Analyzing results and performance metrics
- Creating evaluation reports

## Complete Example

```python
import json
import time
from pathlib import Path
from collinear.client import Client

# Initialize the client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)

# Define comprehensive evaluation configuration
evaluation_config = {
    "ages": [18, 25, 35, 45, 55, 65, 75],
    "genders": ["male", "female", "other"],
    "occupations": ["student", "employed", "business_owner", "freelancer", "retired"],
    "intents": [
        "search_flights", "make_booking", "modify_booking", "cancel_booking",
        "check_status", "seat_selection", "baggage_inquiry", "refund_request",
        "complaint", "escalation", "technical_issue", "billing_question"
    ],
    "traits": {
        "impatience": [-2, -1, 0, 1, 2],
        "confusion": [-2, -1, 0, 1, 2],
        "skeptical": [-2, -1, 0, 1, 2],
        "urgency": [-2, -1, 0, 1, 2],
        "politeness": [-2, -1, 0, 1, 2]
    },
    "locations": ["USA", "Canada", "UK", "Australia", "Germany"],
    "languages": ["English", "Spanish", "French", "German"],
    "tasks": ["airline support"]
}

# Generate evaluation scenarios
print("Generating evaluation scenarios...")
evaluation_scenarios = client.simulate(
    steer_config=evaluation_config,
    k=200,  # Generate 200 test scenarios
    num_exchanges=3,  # 3 user-assistant exchanges each
    batch_delay=0.1,
    mix_traits=True,  # Use complex personas
    progress=True
)

print(f"Generated {len(evaluation_scenarios)} evaluation scenarios")

# Custom agent evaluation function
def evaluate_agent(conversation_prefix, agent_model_url, agent_api_key, agent_model_name):
    """Evaluate a custom agent on a conversation scenario."""
    import requests
    
    # Prepare the conversation for the agent
    messages = []
    for msg in conversation_prefix:
        messages.append({
            "role": msg.get("role", ""),
            "content": msg.get("content", "")
        })
    
    # Add system prompt
    system_prompt = {
        "role": "system",
        "content": "You are a helpful customer service agent for an airline. Be polite, professional, and helpful."
    }
    messages.insert(0, system_prompt)
    
    # Call the agent API
    try:
        response = requests.post(
            f"{agent_model_url}/chat/completions",
            headers={
                "Authorization": f"Bearer {agent_api_key}",
                "Content-Type": "application/json"
            },
            json={
                "model": agent_model_name,
                "messages": messages,
                "temperature": 0.7,
                "max_tokens": 256
            },
            timeout=30
        )
        
        if response.status_code == 200:
            result = response.json()
            return result["choices"][0]["message"]["content"]
        else:
            return f"Error: {response.status_code} - {response.text}"
    
    except Exception as e:
        return f"Error: {str(e)}"

# Evaluate multiple agents
agents = [
    {
        "name": "GPT-4o-mini",
        "url": "https://api.openai.com/v1",
        "api_key": "your-openai-api-key",
        "model": "gpt-4o-mini"
    },
    {
        "name": "GPT-3.5-turbo",
        "url": "https://api.openai.com/v1",
        "api_key": "your-openai-api-key",
        "model": "gpt-3.5-turbo"
    },
    # Add more agents as needed
]

# Run evaluations
evaluation_results = {}
for agent in agents:
    print(f"\nEvaluating {agent['name']}...")
    
    agent_results = []
    for i, scenario in enumerate(evaluation_scenarios):
        if i % 50 == 0:
            print(f"  Progress: {i}/{len(evaluation_scenarios)}")
        
        # Get agent response
        agent_response = evaluate_agent(
            scenario.conv_prefix,
            agent["url"],
            agent["api_key"],
            agent["model"]
        )
        
        # Store result
        result = {
            "scenario_id": i,
            "persona": {
                "age": scenario.steer.age if scenario.steer else None,
                "gender": scenario.steer.gender if scenario.steer else None,
                "occupation": scenario.steer.occupation if scenario.steer else None,
                "intent": scenario.steer.intent if scenario.steer else None,
                "traits": scenario.steer.traits if scenario.steer else {},
                "location": scenario.steer.location if scenario.steer else None,
                "language": scenario.steer.language if scenario.steer else None,
                "task": scenario.steer.task if scenario.steer else None
            },
            "conversation": scenario.conv_prefix,
            "agent_response": agent_response,
            "reference_response": scenario.response  # Collinear's reference response
        }
        
        agent_results.append(result)
    
    evaluation_results[agent["name"]] = agent_results
    print(f"  Completed evaluation for {agent['name']}")

# Assess all agent responses
print("\nAssessing agent responses...")
assessment_results = {}
for agent_name, results in evaluation_results.items():
    print(f"Assessing {agent_name}...")
    
    # Convert to simulation format for assessment
    simulations_for_assessment = []
    for result in results:
        from collinear.schemas.steer import SimulationResult
        sim = SimulationResult(
            conv_prefix=result["conversation"],
            response=result["agent_response"],
            steer=None  # We don't need steer for assessment
        )
        simulations_for_assessment.append(sim)
    
    # Assess the agent responses
    assessment = client.assess(simulations_for_assessment)
    assessment_results[agent_name] = assessment

# Generate evaluation report
print("\n" + "="*60)
print("EVALUATION REPORT")
print("="*60)

# Overall performance comparison
print("\nOverall Performance Comparison:")
print("-" * 40)

for agent_name, assessment in assessment_results.items():
    if assessment.evaluation_result:
        # Calculate average scores
        all_scores = []
        for scores_map in assessment.evaluation_result:
            for score in scores_map.values():
                if score.score is not None:
                    all_scores.append(score.score)
        
        if all_scores:
            avg_score = sum(all_scores) / len(all_scores)
            print(f"{agent_name:20}: {avg_score:.2f} (n={len(all_scores)})")
        else:
            print(f"{agent_name:20}: No scores available")
    else:
        print(f"{agent_name:20}: No evaluation results")

# Detailed analysis by persona traits
print("\nPerformance by Persona Traits:")
print("-" * 40)

# Group results by trait combinations
trait_groups = {}
for agent_name, results in evaluation_results.items():
    for result in results:
        traits = result["persona"].get("traits", {})
        trait_key = tuple(sorted(traits.items()))
        
        if trait_key not in trait_groups:
            trait_groups[trait_key] = {}
        
        if agent_name not in trait_groups[trait_key]:
            trait_groups[trait_key][agent_name] = []
        
        trait_groups[trait_key][agent_name].append(result)

# Analyze each trait group
for trait_key, agent_results in trait_groups.items():
    if len(trait_key) == 0:
        continue
    
    print(f"\nTrait combination: {dict(trait_key)}")
    
    for agent_name, results in agent_results.items():
        if len(results) > 0:
            print(f"  {agent_name}: {len(results)} scenarios")

# Performance by intent
print("\nPerformance by Intent:")
print("-" * 40)

intent_groups = {}
for agent_name, results in evaluation_results.items():
    for result in results:
        intent = result["persona"].get("intent")
        if intent:
            if intent not in intent_groups:
                intent_groups[intent] = {}
            
            if agent_name not in intent_groups[intent]:
                intent_groups[intent][agent_name] = []
            
            intent_groups[intent][agent_name].append(result)

for intent, agent_results in intent_groups.items():
    print(f"\nIntent: {intent}")
    
    for agent_name, results in agent_results.items():
        if len(results) > 0:
            print(f"  {agent_name}: {len(results)} scenarios")

# Save detailed results
output_dir = Path("evaluation_results")
output_dir.mkdir(exist_ok=True)

# Save raw results
for agent_name, results in evaluation_results.items():
    output_file = output_dir / f"{agent_name.lower().replace('-', '_')}_results.jsonl"
    with output_file.open('w', encoding='utf-8') as f:
        for result in results:
            f.write(json.dumps(result, ensure_ascii=False) + '\n')
    print(f"Saved {agent_name} results to: {output_file}")

# Save assessment results
assessment_file = output_dir / "assessment_results.json"
with assessment_file.open('w', encoding='utf-8') as f:
    # Convert assessment results to serializable format
    serializable_assessments = {}
    for agent_name, assessment in assessment_results.items():
        serializable_assessments[agent_name] = {
            "message": assessment.message,
            "evaluation_result": [
                {
                    metric: {
                        "score": score.score,
                        "rationale": score.rationale
                    }
                    for metric, score in scores_map.items()
                }
                for scores_map in assessment.evaluation_result
            ]
        }
    
    json.dump(serializable_assessments, f, indent=2, ensure_ascii=False)

print(f"Saved assessment results to: {assessment_file}")

# Generate summary report
summary_file = output_dir / "evaluation_summary.md"
with summary_file.open('w', encoding='utf-8') as f:
    f.write("# Agent Evaluation Summary\n\n")
    f.write(f"**Evaluation Date:** {time.strftime('%Y-%m-%d %H:%M:%S')}\n")
    f.write(f"**Total Scenarios:** {len(evaluation_scenarios)}\n")
    f.write(f"**Agents Evaluated:** {len(agents)}\n\n")
    
    f.write("## Overall Performance\n\n")
    f.write("| Agent | Average Score | Scenarios |\n")
    f.write("|-------|---------------|----------|\n")
    
    for agent_name, assessment in assessment_results.items():
        if assessment.evaluation_result:
            all_scores = []
            for scores_map in assessment.evaluation_result:
                for score in scores_map.values():
                    if score.score is not None:
                        all_scores.append(score.score)
            
            if all_scores:
                avg_score = sum(all_scores) / len(all_scores)
                f.write(f"| {agent_name} | {avg_score:.2f} | {len(all_scores)} |\n")
            else:
                f.write(f"| {agent_name} | N/A | 0 |\n")
        else:
            f.write(f"| {agent_name} | N/A | 0 |\n")
    
    f.write("\n## Detailed Results\n\n")
    f.write("See individual agent result files for detailed analysis.\n")

print(f"Saved summary report to: {summary_file}")

print("\nEvaluation complete! 🎉")
print(f"Results saved in: {output_dir}")
```

## Advanced Evaluation Configurations

### Stress Testing Configuration

```python
# High-stress scenarios for testing robustness
stress_test_config = {
    "ages": [25, 45, 65],
    "genders": ["male", "female"],
    "occupations": ["business_owner", "employed"],
    "intents": ["complaint", "escalation", "refund_request", "technical_issue"],
    "traits": {
        "impatience": [1, 2],  # High impatience
        "urgency": [1, 2],     # High urgency
        "skeptical": [1, 2],   # High skepticism
        "politeness": [-2, -1] # Low politeness
    },
    "locations": ["USA", "Canada"],
    "languages": ["English"],
    "tasks": ["customer_service"]
}
```

### Edge Case Testing

```python
# Edge cases and unusual scenarios
edge_case_config = {
    "ages": [18, 85],  # Very young and very old
    "genders": ["other"],  # Non-binary
    "occupations": ["unemployed", "retired"],
    "intents": ["escalation", "technical_issue", "billing_question"],
    "traits": {
        "confusion": [2],      # Very confused
        "technical": [-2],     # Very non-technical
        "impatience": [2],     # Very impatient
        "emotional": [2]       # Very emotional
    },
    "locations": ["other"],    # Non-English speaking countries
    "languages": ["other"],
    "tasks": ["customer_service"]
}
```

### Multi-Domain Evaluation

```python
# Evaluate across multiple domains
domains = {
    "airline": {
        "intents": ["search_flights", "make_booking", "modify_booking", "cancel_booking"],
        "tasks": ["airline support"]
    },
    "hotel": {
        "intents": ["search_hotels", "make_reservation", "modify_reservation", "cancel_reservation"],
        "tasks": ["hotel booking"]
    },
    "retail": {
        "intents": ["product_inquiry", "order_status", "return_request", "refund_request"],
        "tasks": ["retail support"]
    }
}

# Run evaluation for each domain
for domain_name, domain_config in domains.items():
    print(f"Evaluating {domain_name} domain...")
    
    # Generate domain-specific scenarios
    domain_scenarios = client.simulate(
        steer_config=domain_config,
        k=50,
        num_exchanges=3,
        progress=True
    )
    
    # Evaluate agents on this domain
    # ... evaluation logic ...
```

## Custom Evaluation Metrics

### Response Quality Metrics

```python
def calculate_response_quality(conversation, agent_response, persona):
    """Calculate custom quality metrics for agent responses."""
    metrics = {}
    
    # Response length
    metrics["response_length"] = len(agent_response)
    
    # Politeness score (simple heuristic)
    polite_words = ["please", "thank you", "sorry", "apologize", "helpful"]
    metrics["politeness_score"] = sum(1 for word in polite_words if word in agent_response.lower())
    
    # Intent alignment
    intent = persona.get("intent", "")
    if intent:
        intent_keywords = {
            "search_flights": ["flight", "search", "find", "available"],
            "make_booking": ["book", "reserve", "confirm", "booking"],
            "cancel_booking": ["cancel", "refund", "cancelation"],
            "complaint": ["apologize", "sorry", "issue", "problem"]
        }
        
        keywords = intent_keywords.get(intent, [])
        metrics["intent_alignment"] = sum(1 for keyword in keywords if keyword in agent_response.lower())
    
    # Trait alignment
    traits = persona.get("traits", {})
    if traits.get("impatience", 0) > 0:
        # For impatient users, prefer shorter responses
        metrics["impatience_alignment"] = 1 if len(agent_response) < 150 else 0
    else:
        metrics["impatience_alignment"] = 1
    
    if traits.get("confusion", 0) > 0:
        # For confused users, prefer clear, simple language
        complex_words = ["however", "furthermore", "consequently", "nevertheless"]
        metrics["clarity_score"] = 1 if not any(word in agent_response.lower() for word in complex_words) else 0
    else:
        metrics["clarity_score"] = 1
    
    return metrics

# Apply custom metrics to all results
for agent_name, results in evaluation_results.items():
    for result in results:
        result["custom_metrics"] = calculate_response_quality(
            result["conversation"],
            result["agent_response"],
            result["persona"]
        )
```

### Comparative Analysis

```python
def compare_agents(agent_results_1, agent_results_2, agent_name_1, agent_name_2):
    """Compare two agents across multiple dimensions."""
    comparison = {
        "agent_1": agent_name_1,
        "agent_2": agent_name_2,
        "total_scenarios": len(agent_results_1),
        "metrics": {}
    }
    
    # Compare response lengths
    lengths_1 = [len(r["agent_response"]) for r in agent_results_1]
    lengths_2 = [len(r["agent_response"]) for r in agent_results_2]
    
    comparison["metrics"]["avg_response_length"] = {
        agent_name_1: sum(lengths_1) / len(lengths_1),
        agent_name_2: sum(lengths_2) / len(lengths_2)
    }
    
    # Compare politeness scores
    politeness_1 = [r.get("custom_metrics", {}).get("politeness_score", 0) for r in agent_results_1]
    politeness_2 = [r.get("custom_metrics", {}).get("politeness_score", 0) for r in agent_results_2]
    
    comparison["metrics"]["avg_politeness"] = {
        agent_name_1: sum(politeness_1) / len(politeness_1),
        agent_name_2: sum(politeness_2) / len(politeness_2)
    }
    
    return comparison

# Compare all pairs of agents
agent_names = list(evaluation_results.keys())
for i in range(len(agent_names)):
    for j in range(i + 1, len(agent_names)):
        comparison = compare_agents(
            evaluation_results[agent_names[i]],
            evaluation_results[agent_names[j]],
            agent_names[i],
            agent_names[j]
        )
        print(f"\nComparison: {agent_names[i]} vs {agent_names[j]}")
        print(f"Response length: {comparison['metrics']['avg_response_length']}")
        print(f"Politeness: {comparison['metrics']['avg_politeness']}")
```

## Automated Evaluation Pipeline

```python
def run_evaluation_pipeline(config, agents, output_dir):
    """Run a complete evaluation pipeline."""
    import subprocess
    import sys
    
    # 1. Generate scenarios
    print("Step 1: Generating evaluation scenarios...")
    scenarios = client.simulate(
        steer_config=config,
        k=1000,
        num_exchanges=3,
        progress=True
    )
    
    # 2. Evaluate agents
    print("Step 2: Evaluating agents...")
    results = {}
    for agent in agents:
        # ... evaluation logic ...
        results[agent["name"]] = agent_results
    
    # 3. Assess responses
    print("Step 3: Assessing responses...")
    assessments = {}
    for agent_name, agent_results in results.items():
        # ... assessment logic ...
        assessments[agent_name] = assessment
    
    # 4. Generate report
    print("Step 4: Generating evaluation report...")
    generate_evaluation_report(results, assessments, output_dir)
    
    # 5. Send notifications (optional)
    print("Step 5: Sending notifications...")
    send_evaluation_notifications(results, assessments)
    
    return results, assessments

# Run the pipeline
results, assessments = run_evaluation_pipeline(
    evaluation_config,
    agents,
    "evaluation_results"
)
```

## Next Steps

Now that you have comprehensive evaluation results:

1. **Analyze performance patterns** - Identify where agents excel or struggle
2. **Identify improvement areas** - Focus on specific persona types or intents
3. **Iterate on agent design** - Use insights to improve your agents
4. **Set up continuous evaluation** - Automate the evaluation pipeline
5. **Monitor performance over time** - Track improvements and regressions

---

*Ready to create custom personas? Check out the [Custom Personas Example](custom-personas.md) to see how to design specific persona types!*

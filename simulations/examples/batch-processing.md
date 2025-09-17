# Batch Processing Example

This example demonstrates how to efficiently process large-scale simulations using batch processing techniques.

## Overview

This example shows:
- Processing large datasets in batches
- Memory-efficient data handling
- Parallel processing strategies
- Progress tracking and monitoring
- Error handling and recovery
- Data persistence and resumption

## Complete Example

```python
import json
import time
import logging
from pathlib import Path
from concurrent.futures import ThreadPoolExecutor, as_completed
from collinear.client import Client

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Initialize the client
client = Client(
    assistant_model_url="https://api.openai.com/v1",
    assistant_model_api_key="your-openai-api-key",
    assistant_model_name="gpt-4o-mini",
    steer_api_key="your-steer-api-key"
)

# Configuration for large-scale processing
BATCH_SIZE = 100
TOTAL_SAMPLES = 10000
MAX_WORKERS = 4
OUTPUT_DIR = Path("batch_processing_results")
OUTPUT_DIR.mkdir(exist_ok=True)

# Comprehensive steering configuration
steer_config = {
    "ages": list(range(18, 80, 2)),  # Every 2 years from 18-80
    "genders": ["male", "female", "other"],
    "occupations": [
        "student", "teacher", "engineer", "doctor", "lawyer", "artist",
        "business_owner", "manager", "consultant", "freelancer", "retired", "unemployed"
    ],
    "intents": [
        "search", "book", "modify", "cancel", "refund", "complaint",
        "escalation", "technical_issue", "billing_question", "status_check",
        "support", "inquiry", "feedback", "upgrade", "downgrade"
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

class BatchProcessor:
    """Handles large-scale batch processing of simulations."""
    
    def __init__(self, client, steer_config, batch_size=100, max_workers=4):
        self.client = client
        self.steer_config = steer_config
        self.batch_size = batch_size
        self.max_workers = max_workers
        self.processed_batches = 0
        self.failed_batches = 0
        self.start_time = None
        
    def process_batch(self, batch_id, batch_size):
        """Process a single batch of simulations."""
        try:
            logger.info(f"Processing batch {batch_id} (size: {batch_size})")
            
            # Generate simulations for this batch
            simulations = self.client.simulate(
                steer_config=self.steer_config,
                k=batch_size,
                num_exchanges=3,
                batch_delay=0.1,
                mix_traits=True,
                progress=False,  # Disable progress bar for individual batches
                max_concurrency=2
            )
            
            # Process simulations
            batch_data = []
            for sim in simulations:
                data = self._process_simulation(sim)
                batch_data.append(data)
            
            # Save batch results
            batch_file = OUTPUT_DIR / f"batch_{batch_id:06d}.jsonl"
            with batch_file.open('w', encoding='utf-8') as f:
                for data in batch_data:
                    f.write(json.dumps(data, ensure_ascii=False) + '\n')
            
            logger.info(f"Completed batch {batch_id}: {len(batch_data)} simulations")
            return {
                'batch_id': batch_id,
                'status': 'success',
                'count': len(batch_data),
                'file': str(batch_file)
            }
            
        except Exception as e:
            logger.error(f"Failed batch {batch_id}: {str(e)}")
            return {
                'batch_id': batch_id,
                'status': 'failed',
                'error': str(e)
            }
    
    def _process_simulation(self, sim):
        """Process a single simulation into the desired format."""
        # Extract conversation
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
        
        # Extract persona details
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
        
        # Calculate metadata
        metadata = {
            "num_exchanges": len(sim.conv_prefix),
            "total_tokens": sum(len(msg.get('content', '')) for msg in conversation),
            "domain": "customer_service",
            "timestamp": time.time()
        }
        
        return {
            "conversation": conversation,
            "persona": persona,
            "metadata": metadata
        }
    
    def process_all(self, total_samples):
        """Process all batches."""
        self.start_time = time.time()
        num_batches = (total_samples + self.batch_size - 1) // self.batch_size
        
        logger.info(f"Starting batch processing: {total_samples} samples in {num_batches} batches")
        
        # Process batches in parallel
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            # Submit all batch jobs
            future_to_batch = {}
            for batch_id in range(num_batches):
                remaining_samples = total_samples - batch_id * self.batch_size
                current_batch_size = min(self.batch_size, remaining_samples)
                
                future = executor.submit(self.process_batch, batch_id, current_batch_size)
                future_to_batch[future] = batch_id
            
            # Process completed batches
            for future in as_completed(future_to_batch):
                batch_id = future_to_batch[future]
                try:
                    result = future.result()
                    if result['status'] == 'success':
                        self.processed_batches += 1
                        logger.info(f"Batch {batch_id} completed successfully")
                    else:
                        self.failed_batches += 1
                        logger.error(f"Batch {batch_id} failed: {result.get('error', 'Unknown error')}")
                except Exception as e:
                    self.failed_batches += 1
                    logger.error(f"Batch {batch_id} failed with exception: {str(e)}")
        
        # Generate summary
        self._generate_summary()
    
    def _generate_summary(self):
        """Generate processing summary."""
        elapsed_time = time.time() - self.start_time
        
        logger.info("=" * 50)
        logger.info("BATCH PROCESSING SUMMARY")
        logger.info("=" * 50)
        logger.info(f"Total batches: {self.processed_batches + self.failed_batches}")
        logger.info(f"Successful batches: {self.processed_batches}")
        logger.info(f"Failed batches: {self.failed_batches}")
        logger.info(f"Success rate: {self.processed_batches / (self.processed_batches + self.failed_batches) * 100:.1f}%")
        logger.info(f"Total time: {elapsed_time:.1f} seconds")
        logger.info(f"Average time per batch: {elapsed_time / (self.processed_batches + self.failed_batches):.1f} seconds")
        
        # Save summary to file
        summary_file = OUTPUT_DIR / "processing_summary.json"
        summary_data = {
            "total_batches": self.processed_batches + self.failed_batches,
            "successful_batches": self.processed_batches,
            "failed_batches": self.failed_batches,
            "success_rate": self.processed_batches / (self.processed_batches + self.failed_batches) * 100,
            "total_time": elapsed_time,
            "average_time_per_batch": elapsed_time / (self.processed_batches + self.failed_batches),
            "timestamp": time.time()
        }
        
        with summary_file.open('w') as f:
            json.dump(summary_data, f, indent=2)
        
        logger.info(f"Summary saved to: {summary_file}")

# Run batch processing
processor = BatchProcessor(
    client=client,
    steer_config=steer_config,
    batch_size=BATCH_SIZE,
    max_workers=MAX_WORKERS
)

print("Starting large-scale batch processing...")
processor.process_all(TOTAL_SAMPLES)

# Consolidate results
print("\nConsolidating results...")
consolidate_results(OUTPUT_DIR)

def consolidate_results(output_dir):
    """Consolidate all batch files into a single dataset."""
    batch_files = list(output_dir.glob("batch_*.jsonl"))
    batch_files.sort()
    
    logger.info(f"Consolidating {len(batch_files)} batch files...")
    
    # Read all data
    all_data = []
    for batch_file in batch_files:
        with batch_file.open('r', encoding='utf-8') as f:
            for line in f:
                if line.strip():
                    data = json.loads(line)
                    all_data.append(data)
    
    # Save consolidated dataset
    consolidated_file = output_dir / "consolidated_dataset.jsonl"
    with consolidated_file.open('w', encoding='utf-8') as f:
        for data in all_data:
            f.write(json.dumps(data, ensure_ascii=False) + '\n')
    
    logger.info(f"Consolidated {len(all_data)} samples into: {consolidated_file}")
    
    # Generate dataset statistics
    generate_dataset_statistics(all_data, output_dir)

def generate_dataset_statistics(data, output_dir):
    """Generate comprehensive dataset statistics."""
    logger.info("Generating dataset statistics...")
    
    # Demographics
    age_dist = {}
    gender_dist = {}
    occupation_dist = {}
    intent_dist = {}
    trait_dist = {}
    
    for sample in data:
        persona = sample.get("persona", {})
        
        # Age distribution
        age = persona.get("age")
        if age:
            age_group = f"{(age//10)*10}-{(age//10)*10+9}"
            age_dist[age_group] = age_dist.get(age_group, 0) + 1
        
        # Gender distribution
        gender = persona.get("gender")
        if gender:
            gender_dist[gender] = gender_dist.get(gender, 0) + 1
        
        # Occupation distribution
        occupation = persona.get("occupation")
        if occupation:
            occupation_dist[occupation] = occupation_dist.get(occupation, 0) + 1
        
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
    
    # Conversation statistics
    conv_lengths = [sample["metadata"]["num_exchanges"] for sample in data]
    token_counts = [sample["metadata"]["total_tokens"] for sample in data]
    
    # Generate statistics report
    stats_file = output_dir / "dataset_statistics.json"
    statistics = {
        "total_samples": len(data),
        "demographics": {
            "age_distribution": age_dist,
            "gender_distribution": gender_dist,
            "occupation_distribution": occupation_dist,
            "intent_distribution": intent_dist
        },
        "traits": trait_dist,
        "conversation_stats": {
            "avg_length": sum(conv_lengths) / len(conv_lengths),
            "min_length": min(conv_lengths),
            "max_length": max(conv_lengths),
            "avg_tokens": sum(token_counts) / len(token_counts),
            "min_tokens": min(token_counts),
            "max_tokens": max(token_counts)
        },
        "generated_at": time.time()
    }
    
    with stats_file.open('w') as f:
        json.dump(statistics, f, indent=2)
    
    logger.info(f"Dataset statistics saved to: {stats_file}")
    
    # Print summary
    print("\n" + "=" * 50)
    print("DATASET STATISTICS")
    print("=" * 50)
    print(f"Total samples: {len(data)}")
    print(f"Average conversation length: {statistics['conversation_stats']['avg_length']:.1f} exchanges")
    print(f"Average tokens per conversation: {statistics['conversation_stats']['avg_tokens']:.1f}")
    
    print(f"\nAge distribution:")
    for age_group, count in sorted(age_dist.items()):
        print(f"  {age_group}: {count} ({count/len(data)*100:.1f}%)")
    
    print(f"\nGender distribution:")
    for gender, count in sorted(gender_dist.items()):
        print(f"  {gender}: {count} ({count/len(data)*100:.1f}%)")
    
    print(f"\nTop intents:")
    for intent, count in sorted(intent_dist.items(), key=lambda x: x[1], reverse=True)[:10]:
        print(f"  {intent}: {count} ({count/len(data)*100:.1f}%)")

print("\nBatch processing complete! 🎉")
print(f"Results saved in: {OUTPUT_DIR}")
```

## Advanced Batch Processing Strategies

### Memory-Efficient Processing

```python
class MemoryEfficientProcessor:
    """Process large datasets without loading everything into memory."""
    
    def __init__(self, client, steer_config, batch_size=100):
        self.client = client
        self.steer_config = steer_config
        self.batch_size = batch_size
    
    def process_streaming(self, total_samples, output_file):
        """Process samples in a streaming fashion."""
        with open(output_file, 'w', encoding='utf-8') as f:
            for batch_start in range(0, total_samples, self.batch_size):
                batch_size_actual = min(self.batch_size, total_samples - batch_start)
                
                # Generate batch
                simulations = self.client.simulate(
                    steer_config=self.steer_config,
                    k=batch_size_actual,
                    num_exchanges=3,
                    batch_delay=0.1,
                    progress=False
                )
                
                # Process and write immediately
                for sim in simulations:
                    data = self._process_simulation(sim)
                    f.write(json.dumps(data, ensure_ascii=False) + '\n')
                    f.flush()  # Ensure data is written immediately
                
                logger.info(f"Processed batch {batch_start//self.batch_size + 1}")
    
    def _process_simulation(self, sim):
        """Process a single simulation."""
        # ... same as before ...
        pass
```

### Error Recovery and Resumption

```python
class ResumableProcessor:
    """Process batches with error recovery and resumption capabilities."""
    
    def __init__(self, client, steer_config, batch_size=100, checkpoint_file="checkpoint.json"):
        self.client = client
        self.steer_config = steer_config
        self.batch_size = batch_size
        self.checkpoint_file = checkpoint_file
        self.checkpoint = self._load_checkpoint()
    
    def _load_checkpoint(self):
        """Load processing checkpoint."""
        if Path(self.checkpoint_file).exists():
            with open(self.checkpoint_file, 'r') as f:
                return json.load(f)
        return {
            "completed_batches": [],
            "failed_batches": [],
            "total_batches": 0,
            "start_time": None
        }
    
    def _save_checkpoint(self):
        """Save processing checkpoint."""
        with open(self.checkpoint_file, 'w') as f:
            json.dump(self.checkpoint, f, indent=2)
    
    def process_with_resumption(self, total_samples):
        """Process with resumption capability."""
        num_batches = (total_samples + self.batch_size - 1) // self.batch_size
        self.checkpoint["total_batches"] = num_batches
        
        if self.checkpoint["start_time"] is None:
            self.checkpoint["start_time"] = time.time()
        
        for batch_id in range(num_batches):
            if batch_id in self.checkpoint["completed_batches"]:
                logger.info(f"Skipping already completed batch {batch_id}")
                continue
            
            try:
                # Process batch
                result = self.process_batch(batch_id, min(self.batch_size, total_samples - batch_id * self.batch_size))
                
                if result['status'] == 'success':
                    self.checkpoint["completed_batches"].append(batch_id)
                    logger.info(f"Completed batch {batch_id}")
                else:
                    self.checkpoint["failed_batches"].append(batch_id)
                    logger.error(f"Failed batch {batch_id}: {result.get('error')}")
                
                # Save checkpoint
                self._save_checkpoint()
                
            except Exception as e:
                self.checkpoint["failed_batches"].append(batch_id)
                logger.error(f"Exception in batch {batch_id}: {str(e)}")
                self._save_checkpoint()
    
    def retry_failed_batches(self):
        """Retry failed batches."""
        logger.info(f"Retrying {len(self.checkpoint['failed_batches'])} failed batches...")
        
        for batch_id in self.checkpoint["failed_batches"][:]:  # Copy list to avoid modification during iteration
            try:
                result = self.process_batch(batch_id, self.batch_size)
                
                if result['status'] == 'success':
                    self.checkpoint["completed_batches"].append(batch_id)
                    self.checkpoint["failed_batches"].remove(batch_id)
                    logger.info(f"Retry successful for batch {batch_id}")
                else:
                    logger.error(f"Retry failed for batch {batch_id}: {result.get('error')}")
                
                self._save_checkpoint()
                
            except Exception as e:
                logger.error(f"Retry exception for batch {batch_id}: {str(e)}")
```

### Parallel Processing with Queue

```python
import queue
import threading

class QueueBasedProcessor:
    """Process batches using a queue-based approach."""
    
    def __init__(self, client, steer_config, batch_size=100, num_workers=4):
        self.client = client
        self.steer_config = steer_config
        self.batch_size = batch_size
        self.num_workers = num_workers
        self.task_queue = queue.Queue()
        self.result_queue = queue.Queue()
        self.workers = []
    
    def start_workers(self):
        """Start worker threads."""
        for i in range(self.num_workers):
            worker = threading.Thread(target=self._worker, args=(i,))
            worker.daemon = True
            worker.start()
            self.workers.append(worker)
    
    def _worker(self, worker_id):
        """Worker thread function."""
        while True:
            try:
                task = self.task_queue.get(timeout=1)
                if task is None:
                    break
                
                batch_id, batch_size = task
                result = self.process_batch(batch_id, batch_size)
                self.result_queue.put(result)
                self.task_queue.task_done()
                
            except queue.Empty:
                continue
            except Exception as e:
                logger.error(f"Worker {worker_id} error: {str(e)}")
                self.task_queue.task_done()
    
    def process_all(self, total_samples):
        """Process all batches using queue-based approach."""
        self.start_workers()
        
        num_batches = (total_samples + self.batch_size - 1) // self.batch_size
        
        # Add tasks to queue
        for batch_id in range(num_batches):
            batch_size_actual = min(self.batch_size, total_samples - batch_id * self.batch_size)
            self.task_queue.put((batch_id, batch_size_actual))
        
        # Collect results
        results = []
        for _ in range(num_batches):
            result = self.result_queue.get()
            results.append(result)
            logger.info(f"Received result for batch {result['batch_id']}")
        
        # Stop workers
        for _ in range(self.num_workers):
            self.task_queue.put(None)
        
        for worker in self.workers:
            worker.join()
        
        return results
```

## Monitoring and Progress Tracking

### Real-time Progress Monitoring

```python
class ProgressMonitor:
    """Monitor batch processing progress in real-time."""
    
    def __init__(self, total_batches):
        self.total_batches = total_batches
        self.completed_batches = 0
        self.failed_batches = 0
        self.start_time = time.time()
        self.last_update = time.time()
    
    def update(self, batch_id, status):
        """Update progress."""
        if status == 'success':
            self.completed_batches += 1
        else:
            self.failed_batches += 1
        
        # Update every 10 seconds or when complete
        current_time = time.time()
        if current_time - self.last_update > 10 or self.completed_batches + self.failed_batches >= self.total_batches:
            self._print_progress()
            self.last_update = current_time
    
    def _print_progress(self):
        """Print progress information."""
        elapsed = time.time() - self.start_time
        total_processed = self.completed_batches + self.failed_batches
        progress = total_processed / self.total_batches * 100
        
        if total_processed > 0:
            eta = elapsed / total_processed * (self.total_batches - total_processed)
            print(f"Progress: {progress:.1f}% ({total_processed}/{self.total_batches}) | "
                  f"Success: {self.completed_batches} | Failed: {self.failed_batches} | "
                  f"ETA: {eta:.1f}s")
        else:
            print(f"Progress: {progress:.1f}% ({total_processed}/{self.total_batches})")

# Use progress monitor
monitor = ProgressMonitor(num_batches)
# ... in your processing loop ...
monitor.update(batch_id, 'success' if result['status'] == 'success' else 'failed')
```

## Next Steps

Now that you have efficient batch processing:

1. **Scale up** - Process even larger datasets
2. **Optimize performance** - Tune batch sizes and worker counts
3. **Add monitoring** - Set up alerts and dashboards
4. **Implement recovery** - Add robust error handling
5. **Deploy to cloud** - Use cloud resources for massive scale

---

*Ready to explore advanced features? Check out the [Advanced Topics](advanced/) section for more sophisticated techniques!*

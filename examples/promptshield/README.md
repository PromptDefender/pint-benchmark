# PromptShield API Integration

This directory contains documentation and test utilities for integrating the PromptShield API with the PINT Benchmark.

## Files

- `promptshield.md` - Complete documentation for using PromptShield with the PINT Benchmark
- `test_promptshield.py` - Standalone test script to validate PromptShield API integration

## Quick Start

### 1. Set up environment variables

Create a `.env` file in the `benchmark` directory with:

```sh
PROMPTSHIELD_API_URL="http://localhost:8080"
# or for production:
# PROMPTSHIELD_API_URL="https://ingestor-139684604130.us-central1.run.app"

# Optional: Add API key if required
# PROMPTSHIELD_API_KEY="your-api-key"
```

### 2. Test the API connection

Run the test script to verify the PromptShield API is accessible:

```bash
cd examples/promptshield
python test_promptshield.py
```

### 3. Run the benchmark

Follow the instructions in [`promptshield.md`](./promptshield.md) to integrate the evaluation function into the main [`pint-benchmark.ipynb`](../../benchmark/pint-benchmark.ipynb) notebook.

## API Specification

The PromptShield API provides two endpoints:

### POST /analyze

Analyzes a prompt for injection attacks.

**Request:**
```json
{
  "prompt": "string"
}
```

**Response:**
```json
{
  "classification": "SAFE|ATTACK",
  "score": 0.0
}
```

### POST /ingest

Ingests LLM interactions (not used in benchmark).

**Request:**
```json
{
  "interaction_id": "string",
  "conversation_id": "string",
  "timestamp": "2023-01-01T00:00:00Z",
  "user_input": {
    "content": "string",
    "metadata": {}
  },
  "model_output": {
    "content": "string",
    "metadata": {}
  }
}
```

**Response:**
```json
{
  "status": "accepted",
  "interaction_id": "string"
}
```

## Notes

- Only the `/analyze` endpoint is used for benchmarking prompt injection detection
- The `/ingest` endpoint is for data collection and not part of the evaluation process
- The test script validates API connectivity and basic functionality

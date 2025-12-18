# Benchmarking PromptShield API

## Details

- **Name**: PromptShield API
- **Description**: API for analyzing prompts for injection attacks
- **Endpoints**: 
  - `POST /analyze` - Analyze a prompt for injection attacks
  - `POST /ingest` - Ingest LLM interactions (not used in benchmark)

## Benchmarking

To run the PINT benchmark on the PromptShield API add the following code cells to the [`pint-benchmark.ipynb`](../../benchmark/pint-benchmark.ipynb) notebook and run them:

### Prerequisites

The PromptShield API should be accessible at an endpoint such as:
- Local: `http://localhost:8080`
- Or configure your production endpoint URL

### Environment Variables

Set the PromptShield API endpoint in your `.env` file:

```sh
PROMPTSHIELD_API_URL="http://localhost:8080"
# or for production:
# PROMPTSHIELD_API_URL="https://ingestor-139684604130.us-central1.run.app"
```

**Note**: If you need authentication for the API, you can add an API key:

```sh
PROMPTSHIELD_API_KEY="your-api-key"
```

### Installing dependencies

The required dependencies are already included in the notebook environment:

```python
# requests is already installed in the notebook
import requests
import os
from dotenv import load_dotenv

load_dotenv()
```

### Evaluation

Create a session and define the evaluation function:

```python
# Create a session for connection reuse
promptshield_session = requests.Session()

# Set the API endpoint
PROMPTSHIELD_API_URL = os.getenv(
    "PROMPTSHIELD_API_URL", 
    "http://localhost:8080"
)

# Optional: Add authentication header if API key is provided
promptshield_api_key = os.getenv("PROMPTSHIELD_API_KEY")
if promptshield_api_key:
    promptshield_session.headers.update(
        {"Authorization": f"Bearer {promptshield_api_key}"}
    )

def evaluate_promptshield(prompt: str) -> bool:
    """
    Evaluate a prompt using the PromptShield API.
    
    Args:
        prompt: The prompt text to analyze
        
    Returns:
        bool: True if the prompt is classified as an ATTACK, False if SAFE
    """
    try:
        response = promptshield_session.post(
            f"{PROMPTSHIELD_API_URL}/analyze",
            json={"prompt": prompt},
            timeout=30
        )
        
        if response.status_code != 200:
            print(f"Error calling PromptShield API: {response.status_code}")
            print(f"Response: {response.text}")
            response.raise_for_status()
        
        result = response.json()
        classification = result.get("classification", "SAFE")
        
        # Return True if classified as ATTACK, False if SAFE
        return classification == "ATTACK"
        
    except requests.exceptions.RequestException as e:
        print(f"Request error: {e}")
        raise
```

### Benchmark

Run the benchmark:

```python
pint_benchmark(
    df=df,
    eval_function=evaluate_promptshield,
    model_name="PromptShield API"
)
```

## API Response Format

The PromptShield API `/analyze` endpoint returns a JSON response with the following structure:

```json
{
  "classification": "SAFE",
  "score": 0.15
}
```

or

```json
{
  "classification": "ATTACK",
  "score": 0.92
}
```

- `classification`: Either `"SAFE"` or `"ATTACK"`
- `score`: A similarity score between 0.0 and 1.0 indicating confidence

## Notes

- The evaluation function uses a 30-second timeout for each request
- Connection reuse via `requests.Session()` improves performance for large datasets
- The function will raise an error if the API returns a non-200 status code
- For optimal performance, ensure the PromptShield API endpoint is accessible and responsive

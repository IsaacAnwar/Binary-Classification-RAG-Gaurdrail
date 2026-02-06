# Classification RAG Guardrail

A two-layer classification microservice that acts as a guardrail for AI-powered finance interview systems. It ensures only relevant, safe, and compliant queries reach the main interview orchestrator.

## Overview

This microservice implements a **layered defense architecture** with two specialized BERT-based classifiers:

- **Layer 1 (Domain Classifier)**: Fast binary classification determining if a query is finance-related
- **Layer 2 (Intent Classifier)**: Multi-class classification for query intent/safety analysis

The two-layer design enables a very quick classification of a user query to be used as a router in agentic systems; allowing the control of which specialized agents handles the user query.

## Architecture

```
User Query
    │
    ▼
┌─────────────────┐
│    Layer 1      │
│ Domain Classifier│
│ (Finance/Non)   │
└────────┬────────┘
         │
    Is Finance?
    /        \
   No         Yes
   │           │
   ▼           ▼
 Flagged->┌─────────────────┐
          │    Layer 2      │
          │ Intent Classifier│
          │ (6 categories)  │
          └────────┬────────┘
                   │
                   ▼
            Classification
              Response
```

## Layer Classifications

### Layer 1 - Domain Classification
| Label | Description |
|-------|-------------|
| `finance` | Query is related to finance topics |
| `non_finance` | Query is not finance-related |

### Layer 2 - Intent Classification
| Label | Description |
|-------|-------------|
| `answer_submission` | Candidate providing an answer |
| `clarification_request` | Asking for clarification |
| `process_inquiry` | Questions about the interview process |
| `challenge_assessment` | Challenging or questioning the assessment |
| `off_topic` | Off-topic within finance context |
| `small_talk` | General conversation/small talk |

## Installation

### Prerequisites
- Python 3.11+
- pip or conda

### Setup

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd Classification-RAG-gaurdrail
   ```

2. **Create a virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On macOS/Linux
   # or
   venv\Scripts\activate     # On Windows
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Train the models** (if not already trained)
   - Run `layer1/layer1_training.ipynb` for Layer 1 model
   - Run `layer2/layer2_training.ipynb` for Layer 2 model

## Usage

### Running Locally

```bash
uvicorn main:app --reload
```

The API will be available at `http://127.0.0.1:8000`

### API Documentation

Once running, visit:
- Swagger UI: `http://127.0.0.1:8000/docs`
- ReDoc: `http://127.0.0.1:8000/redoc`

## API Endpoints

### POST `/classify`

Classify a user message through both layers.

**Request:**
```json
{
  "message": "What is the WACC formula?"
}
```

**Response:**
```json
{
  "layer1": {
    "prediction": "finance",
    "confidence": 0.9876
  },
  "layer2": {
    "prediction": "answer_submission",
    "confidence": 0.8543
  }
}
```

### GET `/health`

Health check endpoint.

**Response:**
```json
{
  "status": "healthy",
  "models_loaded": true
}
```

### Example Usage with cURL

```bash
# Health check
curl http://127.0.0.1:8000/health

# Classify a query
curl -X POST http://127.0.0.1:8000/classify \
  -H "Content-Type: application/json" \
  -d '{"message": "How do you calculate DCF?"}'
```

### Example Usage with Python

```python
import requests

response = requests.post(
    "http://127.0.0.1:8000/classify",
    json={"message": "What is the WACC formula?"}
)
print(response.json())
```

## Docker Deployment

### Build the Docker image

```bash
docker build -t guardrail-service .
```

### Run the container

```bash
docker run -p 8000:8000 guardrail-service
```

## Cloud Deployment (Google Cloud Run)

### Deploy with GPU support

```bash
# Build and push to Google Container Registry
gcloud builds submit --tag gcr.io/YOUR_PROJECT/guardrail-service

# Deploy to Cloud Run with GPU
gcloud run deploy guardrail-service \
  --image gcr.io/YOUR_PROJECT/guardrail-service \
  --gpu 1 \
  --gpu-type nvidia-l4 \
  --memory 8Gi \
  --cpu 4 \
  --region us-central1 \
  --allow-unauthenticated
```

## Project Structure

```
Classification-RAG-gaurdrail/
├── main.py                 # FastAPI application
├── Dockerfile              # Docker configuration
├── requirements.txt        # Python dependencies
├── project_overview.md     # Detailed project documentation
├── README.md               # This file
│
├── layer1/
│   ├── layer1_training.ipynb   # Layer 1 training notebook
│   ├── combined_data.csv       # Layer 1 training data
│   └── layer1_model/           # Trained Layer 1 checkpoints
│       └── checkpoint-*/
│
└── layer2/
    ├── layer2_training.ipynb   # Layer 2 training notebook
    ├── layer2_data.csv         # Layer 2 training data
    └── layer2_model/           # Trained Layer 2 checkpoints
        └── checkpoint-*/
```

## Model Details

| Property | Layer 1 | Layer 2 |
|----------|---------|---------|
| Base Model | ModernBERT-base | ModernBERT-base |
| Task | Binary Classification | Multi-class Classification |
| Classes | 2 | 6 |
| Max Sequence Length | 128 tokens | 128 tokens |

## Performance Targets

| Metric | Target |
|--------|--------|
| Layer 1 Latency | < 20ms (CPU), < 5ms (GPU) |
| Layer 2 Latency | < 20ms (CPU), < 5ms (GPU) |
| F1 Score | > 90% |
| Precision | > 95% |

## Dependencies

Core dependencies (see `requirements.txt` for versions):
- PyTorch
- Transformers (HuggingFace)
- FastAPI
- Uvicorn
- scikit-learn
- pandas
- numpy

## Development

### Training New Models

1. Prepare your training data in CSV format with `text` and `label` columns
2. Open the respective training notebook (`layer1/` or `layer2/`)
3. Run all cells to train and save the model
4. Update the model path in `main.py` if needed

### Adding New Endpoints

The FastAPI application in `main.py` can be extended with additional endpoints as needed.

## License

[Add your license here]

## Contributing

[Add contribution guidelines here]

# Groq to Mistral Migration Guide

## Changes Made

### 1. Dependencies Updated
- **Removed**: `langchain-groq==0.3.2`, `groq==0.23.1`
- **Added**: `langchain-mistralai==0.2.1`, `mistralai==1.5.1`

### 2. Environment Variables
- **Removed**: `GROQ_API_KEY`
- **Added**: `MISTRAL_API_KEY` (already configured in `.env`)

### 3. Code Changes

#### Main API File (`server/src/api/main.py`)
- Changed import: `from langchain_groq import ChatGroq` → `from langchain_mistralai import ChatMistralAI`
- Updated LLM initialization to use `mistral-large-latest` model
- Added API key parameter from environment variable

#### Analysis Files
- `src/analysis/invention_extractor.py` - Updated to use ChatMistralAI
- `src/analysis/feature_extractor.py` - Updated to use ChatMistralAI  
- `src/analysis/claim_representation.py` - Updated to use ChatMistralAI

#### Generation Files
- `src/generation/report_generator.py` - Updated to use ChatMistralAI

#### Routing Files
- `src/routing/ip_router.py` - Updated to use ChatMistralAI
- `src/routing/orchestrator.py` - Updated to use ChatMistralAI
- `src/routing/jurisdiction.py` - Updated to use ChatMistralAI

#### Formulation Files
- `src/formulation/classifier.py` - Updated to use ChatMistralAI
- `src/formulation/extractor.py` - Updated to use ChatMistralAI
- `src/formulation/analyze.py` - Updated to use ChatMistralAI

#### Multilingual Files
- `src/multilingual/provider.py` - Updated to use ChatMistralAI

#### Agent Files
- `src/agents/orchestrator.py` - Updated to use ChatMistralAI

### 4. Model Configuration
- **Old Model**: `openai/gpt-oss-20b` (Groq)
- **New Model**: `mistral-large-latest` (Mistral)

### 5. Installation Commands
```bash
cd server
.venv/Scripts/python.exe -m pip install langchain-mistralai==0.2.1 mistralai==1.5.1
```

## Testing
After migration, test the backend with:
```bash
cd server
.venv/Scripts/python.exe run_server.py
```

Then test API endpoints:
```bash
curl http://127.0.0.1:8000/health
curl -X POST http://127.0.0.1:8000/ask -H "Content-Type: application/json" -d '{"query": "test question"}'
```

## Notes
- All Groq references have been replaced with Mistral equivalents
- The Mistral API key is already configured in `.env`
- Model parameter names are compatible between LangChain implementations
- Temperature settings remain the same (0 for deterministic responses)

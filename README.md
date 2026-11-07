# REVAC: Multi-Agent System with LoRA Fine-Tuning for Social Deception Games
### Team Phoebus - MindGames Arena NeurIPS 2025 (Social Deduction Track, Efficient Division)

## Overview

This project implements **REVAC** (Review and Act), a two-stage AI agent using a PEFT-adapted Qwen3-8B large language model designed to play the social deduction game Secret Mafia (Mafia). The system achieves a **73% win rate against the base Qwen3-8B model** through a sequential review-and-act architecture with well-designed system prompts.

## Architecture

### Multi-Agent Pipeline

The system employs a **two-agent sequential architecture**:

1. **Review Agent** (Analyzer)
   - Processes raw game observations
   - Extracts facts from `[GAME]` announcements
   - Profiles players based on behavior and claims
   - Identifies hallucinations and contradictions from other agents
   - Generates mafia probability scenarios (when playing as Village)
   - Recommends strategic actions

2. **Final Action Agent** (Executor)
   - Takes the strategic review as input
   - Determines current game phase and valid actions
   - Executes the recommended strategy
   - Formats output according to game requirements
   - Handles edge cases and malformed responses

### Model Architecture

- **Base Model**: Qwen3-8B (8 billion parameter causal language model)
- **Adapter Technology**: LoRA via PEFT (Parameter-Efficient Fine-Tuning)
- **Fine-tuned LoRA Adapters**:
  - `mihirArya/qwafia_10`: Review agent adapter (default)
  - `mihirArya/qwafiaB_10`: Final action agent adapter (final_action_agent)
- **Inference**: Dynamic adapter switching during gameplay
- **Performance**: 73% win rate vs. base Qwen3-8B model

### Key Components

```
qwafia.py
├── Base model loading (Qwen3-8B)
├── PEFT adapter loading and switching
├── review_agent()      # Strategic analysis
├── final_action_agent() # Action execution
└── revac class         # Agent interface wrapper
```

The agents use specialized system prompts (`prompts.py`) that handle:
- Game state parsing and validation
- Hallucination detection from opponent LLM agents
- Role-specific strategies (Mafia vs. Village)
- Format compliance enforcement

## File Structure

```
social_detection-mindgames/
├── agent.py                 # Abstract agent classes (Agent, HumanAgent)
├── qwafia.py               # Core REVAC multi-agent implementation
├── prompts.py              # System prompts for both agents
├── run_agent.py            # Competition entry point (online play)
├── main.py                 # CLI interface for testing
├── requirements.txt        # Python dependencies
├── envs/                   # Game environment configurations
│   ├── README.md          # Environment documentation
│   ├── SecretMafia/       # Social deduction game
│   │   ├── env.py
│   │   └── renderer.py
│   ├── Codenames/         # Word association game
│   ├── ColonelBlotto/     # Resource allocation game
│   └── ThreePlayerIPD/    # Iterated Prisoner's Dilemma
```

### Core Files

- **`agent.py`**: Defines the abstract `Agent` interface that all agents must implement
- **`qwafia.py`**: Main implementation containing model loading, adapter switching, and the two-agent pipeline
- **`prompts.py`**: Contains carefully engineered system prompts for reviewer and final action agents
- **`run_agent.py`**: Connects the agent to TextArena's online competition environment
- **`main.py`**: Command-line interface for testing with custom observations

## Setup Instructions

### 1. Clone the repository
```bash
git clone https://github.com/mihiraryaa/social_detection-mindgames.git
cd social_detection-mindgames
```

### 2. Create and activate a virtual environment (recommended)
```bash
python3 -m venv venv

source venv/bin/activate     # On macOS/Linux
venv\Scripts\activate        # On Windows
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

**Note**: This project requires PyTorch with CUDA support for GPU acceleration. The models will automatically use GPU if available.

## Usage

### Online Competition Mode
```bash
python run_agent.py
```
Connects to the TextArena online environment and plays Secret Mafia games.

### Local Testing with Custom Observation
```bash
python main.py --observation "Your game observation string here"
```
Useful for debugging specific game states.

### Simple Test
```bash
python test.py
```
Runs a basic sanity check with a test observation.

## How It Works

1. **Observation Input**: The agent receives a string containing the current game state, including player messages, game announcements, and phase information.

2. **Review Phase**: The review agent (using the default adapter) analyzes the observation:
   - Parses `[GAME]` tags for authoritative information
   - Tracks player behaviors and identifies inconsistencies
   - Generates strategic recommendations

3. **Action Phase**: The final action agent (using the final_action_agent adapter) executes:
   - Receives both the original observation and the review
   - Determines the correct action format based on game phase
   - Outputs a properly formatted response

4. **Response Extraction**: Responses are extracted from `<response>` tags with fallback handling for malformed outputs.

 

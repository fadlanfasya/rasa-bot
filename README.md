# Customer Service Chatbot

A conversational AI assistant built with Rasa to handle customer service inquiries.

## Table of Contents
- [Installation](#installation)
- [Project Setup](#project-setup)
- [Project Structure](#project-structure)
- [Training the Bot](#training-the-bot)
- [Running the Bot](#running-the-bot)
- [Testing the Bot](#testing-the-bot)
- [Customization](#customization)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)

## Installation

### Prerequisites
- Python 3.7 or later
- pip (Python package manager)

### Step 1: Set up a virtual environment (recommended)
```bash
# Create a virtual environment
python -m venv ./venv

# Activate the virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

### Step 2: Install Rasa
```bash
pip install rasa
```

## Project Setup

### Create a new Rasa project
```bash
# Navigate to your desired project directory
mkdir customer-service-bot
cd customer-service-bot

# Initialize a new Rasa project
rasa init
```

This command will:
- Create a project structure with sample files
- Train an initial model
- Allow you to test the bot in command line

Answer "Y" when asked if you want to train an initial model.

## Project Structure

After initialization, your project will contain these key files:

- `data/nlu.yml`: Training examples for natural language understanding
- `data/stories.yml`: Conversation flow examples
- `data/rules.yml`: Specific conversation patterns
- `config.yml`: NLU and policy configuration
- `domain.yml`: Bot's universe (intents, entities, responses, etc.)
- `endpoints.yml`: Connection to external services
- `credentials.yml`: Authentication for messaging platforms
- `actions/actions.py`: Custom action code

## Training the Bot

After making changes to your training data or configuration:

```bash
rasa train
```

This will:
- Process the training data
- Train a new model
- Save the model in the `models/` directory

## Running the Bot

### Option 1: Command Line Interface
```bash
rasa shell
```

### Option 2: Start the Rasa API Server
```bash
# Start with API enabled
rasa run --enable-api --cors "*"

# If using custom actions, start the action server in a separate terminal
rasa run actions
```

## Testing the Bot

### Using the Command Line
```bash
rasa shell
```

Type messages directly in the terminal to interact with your bot.

### Using Interactive Learning
```bash
rasa interactive
```

This allows you to correct the bot's understanding in real-time and improve its performance.

### Using Postman or curl

With the Rasa server running (`rasa run --enable-api`), you can send POST requests:

#### Postman Setup:
1. Create a new POST request
2. URL: `http://localhost:5005/webhooks/rest/webhook`
3. Headers: `Content-Type: application/json`
4. Body (raw JSON):
   ```json
   {
     "sender": "test_user",
     "message": "Hello"
   }
   ```
5. Click Send

#### Using curl:
```bash
curl -X POST \
  http://localhost:5005/webhooks/rest/webhook \
  -H 'Content-Type: application/json' \
  -d '{"sender": "test_user", "message": "Hello"}'
```

### Running Tests
```bash
# Test the entire bot
rasa test

# Test just the NLU component
rasa test nlu
```

## Customization

### Adding New Intents and Responses

1. Add new intents and examples to `data/nlu.yml`:
```yaml
nlu:
- intent: ask_hours
  examples: |
    - What are your opening hours?
    - When are you open?
    - What time do you close?
    - Are you open on weekends?
```

2. Add responses in `domain.yml`:
```yaml
responses:
  utter_hours:
  - text: "We're open Monday to Friday, 9am to 5pm, and Saturday 10am to 4pm. We're closed on Sundays."
```

3. Register the intent in the `intents` section of `domain.yml`:
```yaml
intents:
  - ask_hours
```

4. Add conversational paths in `data/stories.yml`:
```yaml
stories:
- story: customer asks about hours
  steps:
  - intent: ask_hours
  - action: utter_hours
```

5. Retrain the model:
```bash
rasa train
```

### Adding Custom Actions

1. Edit `actions/actions.py` to add a custom action:
```python
from typing import Any, Text, Dict, List
from rasa_sdk import Action, Tracker
from rasa_sdk.executor import CollectingDispatcher

class ActionCheckOrderStatus(Action):
    def name(self) -> Text:
        return "action_check_order_status"

    def run(self, dispatcher: CollectingDispatcher,
            tracker: Tracker,
            domain: Dict[Text, Any]) -> List[Dict[Text, Any]]:
        
        # Extract order number (this is simplified)
        order_id = tracker.get_slot("order_id")
        
        if order_id:
            # In a real scenario, you would check a database here
            status = "shipped" # Placeholder status
            message = f"Your order #{order_id} is currently {status}."
        else:
            message = "I'll need your order number to check the status."
            
        dispatcher.utter_message(text=message)
        return []
```

2. Register the action in `domain.yml`:
```yaml
actions:
  - action_check_order_status
```

3. Update `endpoints.yml` to use the action server:
```yaml
action_endpoint:
  url: "http://localhost:5055/webhook"
```

4. Run the action server in a separate terminal:
```bash
rasa run actions
```

## Deployment

### Basic Server Deployment

1. Set up a server with Python and Rasa installed
2. Upload your project files
3. Run Rasa as a service:
```bash
rasa run --enable-api --cors "*" --port 5005
```

### Docker Deployment

1. Create a Dockerfile in your project directory:
```dockerfile
FROM rasa/rasa:latest

COPY . /app
WORKDIR /app
USER root

RUN rasa train

ENTRYPOINT ["rasa"]
CMD ["run", "--enable-api", "--cors", "*"]
```

2. Build and run the Docker image:
```bash
docker build -t customer-service-bot .
docker run -p 5005:5005 customer-service-bot
```

## Troubleshooting

### Common Issues

1. **Bot doesn't understand input**
   - Add more training examples in `data/nlu.yml`
   - Use `rasa interactive` to improve understanding

2. **Custom actions not working**
   - Ensure action server is running (`rasa run actions`)
   - Check `endpoints.yml` configuration
   - Verify action is registered in `domain.yml`

3. **Training errors**
   - Check YAML syntax in configuration files
   - Ensure all referenced intents and actions are defined

4. **API connection issues**
   - Verify server is running with `--enable-api` flag
   - Check for correct endpoint URLs
   - Ensure proper JSON formatting in requests

### Getting Help

- Check the [Rasa Documentation](https://rasa.com/docs/)
- Join the [Rasa Community Forum](https://forum.rasa.com/)
- Search for issues on [GitHub](https://github.com/RasaHQ/rasa/issues)

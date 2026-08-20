# Lab: Integrate an AI agent with Foundry IQ

## Test the Agent in the playground

<img width="1518" height="955" alt="image" src="https://github.com/user-attachments/assets/cac4f58c-2ed3-466b-9ac7-c9e68fef8205" />

<img width="1514" height="949" alt="image" src="https://github.com/user-attachments/assets/092f519f-f7a2-4489-be69-9327f2f7e7d9" />

## Complete the agent client code

> Note: Remember to update your project endpoint at **.env** file.

```python
import os
from dotenv import load_dotenv
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

# Load environment variables
load_dotenv()
project_endpoint = os.getenv("PROJECT_ENDPOINT")
agent_name = os.getenv("AGENT_NAME")

# Validate configuration
if not project_endpoint or not agent_name:
    raise ValueError("PROJECT_ENDPOINT and AGENT_NAME must be set in .env file")

print(f"Connecting to project: {project_endpoint}")
print(f"Using agent: {agent_name}\n")

# TODO: Connect to the project and create a conversation
# Add your code here to:
# 1. Create DefaultAzureCredential
# 2. Create AIProjectClient with endpoint
# 3. Get the OpenAI client
# 4. Get the agent by name
# 5. Create a new conversation

# Connect to the project and agent
credential = DefaultAzureCredential(
    exclude_environment_credential=True,
    exclude_managed_identity_credential=True
)
project_client = AIProjectClient(
    credential=credential,
    endpoint=project_endpoint
)

# Get the OpenAI client
openai_client = project_client.get_openai_client()

# Get the agent
agent = project_client.agents.get(agent_name=agent_name)
print(f"Connected to agent: {agent.name} (id: {agent.id})\n")

# Create a new conversation
conversation = openai_client.conversations.create(items=[])
print(f"Created conversation (id: {conversation.id})\n")  


# Conversation history for context (client-side tracking)
conversation_history = []


def send_message_to_agent(user_message):
    """
    Send a message to the agent and handle the response using the conversations API.
    """
    try:
        print("\nAgent: ", end="", flush=True)
        
        # TODO: Add user message to conversation and get response
        # Add your code here to:
        # 1. Add the user message to the conversation using conversations.items.create()
        # 2. Create a response using responses.create() with agent reference
        # 3. Extract and display the response text
        # 4. Check for and display any citations
        # Your code will go here


        # Add user message to the conversation
        openai_client.conversations.items.create(
            conversation_id=conversation.id,
            items=[{"type": "message", "role": "user", "content": user_message}],
        )

        # Store in conversation history (client-side)
        conversation_history.append({
            "role": "user",
            "content": user_message
        })

        # Create a response using the agent
        response = openai_client.responses.create(
            conversation=conversation.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
            input=""
        )

        # Loop until a response has no pending approval requests (zero, one, or many)
        while True:
            approval_requests = [
                item for item in (getattr(response, "output", None) or [])
                if getattr(item, "type", None) == "mcp_approval_request"
            ]

            if not approval_requests:
                break

            approval_items = []

            for approval_request in approval_requests:
                print(f"[Approval required for: {approval_request.name}]\n")
                print(f"Server: {approval_request.server_label}")

                # Show the tool call arguments for transparency
                import json
                try:
                    args = json.loads(approval_request.arguments)
                    print(f"Arguments: {json.dumps(args, indent=2)}\n")
                except Exception:
                    print(f"Arguments: {approval_request.arguments}\n")

                approval_input = input("Approve this action? (yes/no): ").strip().lower()
                approved = approval_input in ['yes', 'y']
                print("Approving action...\n" if approved else "Action denied.\n")

                approval_items.append({
                    "type": "mcp_approval_response",
                    "approval_request_id": approval_request.id,
                    "approve": approved
                })

                # Send the approval decisions and fetch the next response
                openai_client.conversations.items.create(
                    conversation_id=conversation.id,
                    items=approval_items
                )

                response = openai_client.responses.create(
                    conversation=conversation.id,
                    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}},
                    input=""
            )
        
        # Extract the response text
        if response and response.output_text:
            response_text = response.output_text
            
            print(f"{response_text}\n")
            
            # Check for citations if available
            if hasattr(response, 'citations') and response.citations:
                print("\nSources:")
                for citation in response.citations:
                    print(f"  - {citation.content if hasattr(citation, 'content') else 'Knowledge Base'}")
            
            # Store in conversation history (client-side)
            conversation_history.append({
                "role": "assistant",
                "content": response_text
            })
            
            return response_text
        else:
            print("No response received.\n")
            return None
    except Exception as e:
        print(f"\n\nError: {str(e)}\n")
        return None


def display_conversation_history():
    """
    Display the full conversation history.
    """
    print("\n" + "="*60)
    print("CONVERSATION HISTORY")
    print("="*60 + "\n")
    
    for turn in conversation_history:
        role = turn["role"].upper()
        content = turn["content"]
        print(f"{role}: {content}\n")
    
    print("="*60 + "\n")


def main():
    """
    Main interaction loop.
    """
    print("Contoso Product Expert Agent")
    print("Ask questions about our outdoor and camping products.")
    print("Type 'history' to see conversation history, or 'quit' to exit.\n")
    
    while True:
        try:
            user_input = input("You: ").strip()
            
            if not user_input:
                continue
                
            if user_input.lower() == 'quit':
                print("\nEnding conversation...")
                break
                
            if user_input.lower() == 'history':
                display_conversation_history()
                continue
            
            # Send message and get response
            send_message_to_agent(user_input)
            
        except KeyboardInterrupt:
            print("\n\nInterrupted by user.")
            break
        except Exception as e:
            print(f"\nUnexpected error: {str(e)}\n")
    
    print("\nConversation ended.")


if __name__ == "__main__":
    main()

```

## Test the Integration


Query 1 - Product Categories: "What types of outdoor products does Contoso offer?
<img width="1518" height="975" alt="image" src="https://github.com/user-attachments/assets/5699ef3d-3ba5-42f4-aa81-4182ce3c45ee" />


Query 2 - Specific Product Details: "Tell me about the weatherproof features of your tents."**
<img width="1518" height="975" alt="image" src="https://github.com/user-attachments/assets/9ef61d64-a605-4048-8d38-6d6254e808fc" />


Query 3 - Product Comparisons: "What's the difference between your daypacks and expedition backpacks?"


Query 4 - Accessories and Add-ons: "What camping accessories would you recommend for a weekend hiking trip?"
<img width="1489" height="979" alt="image" src="https://github.com/user-attachments/assets/25337b51-a9f8-43fe-a649-c35aa72298ff" />


Query 5 - Follow-up Question: "How much do those items typically cost?"
<img width="1489" height="973" alt="image" src="https://github.com/user-attachments/assets/6f6dd48c-b4fa-4e18-aac8-bb0c53b3136b" />


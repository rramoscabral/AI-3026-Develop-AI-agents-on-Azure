# "Develop a multi-agent solution" or "Develop a multi-agent solution with Microsoft Agent Framework"


## Create AI agents + Create a sequential orchestration

File: **agents.py**

> Note: Remember to update your project endpoint at **.env** file.

```python
import asyncio
import os
from typing import cast
from dotenv import load_dotenv

# Add references
from agent_framework import Message
from agent_framework.foundry import FoundryChatClient
from agent_framework.orchestrations import SequentialBuilder
from azure.identity import AzureCliCredential

load_dotenv()

async def main():
    # Agent instructions
    summarizer_instructions="""
    Summarize the customer's feedback in one short sentence. Keep it neutral and concise.
    Example output:
    App crashes during photo upload.
    User praises dark mode feature.
    """

    classifier_instructions="""
    Classify the feedback as one of the following: Positive, Negative, or Feature request.
    """

    action_instructions="""
    Based on the summary and classification, suggest the next action in one short sentence.
    Example output:
    Escalate as a high-priority bug for the mobile team.
    Log as positive feedback to share with design and marketing.
    Log as enhancement request for product backlog.
    """

    # Create the chat client
    credential = AzureCliCredential()
    chat_client = FoundryChatClient(
        credential=credential,
        project_endpoint=os.getenv("AZURE_AI_PROJECT_ENDPOINT"),
        model=os.getenv("AZURE_AI_MODEL_DEPLOYMENT_NAME"),
    )

    # Create agents
    summarizer_agent = chat_client.as_agent(
        name="summarizer",
        instructions=summarizer_instructions,
    )

    classifier_agent = chat_client.as_agent(
        name="classifier",
        instructions=classifier_instructions,
    )

    action_agent = chat_client.as_agent(
        name="action",
        instructions=action_instructions,
    )

    # Initialize the current feedback
    feedback="""
    I use the dashboard every day to monitor metrics, and it works well overall. 
    But when I'm working late at night, the bright screen is really harsh on my eyes. 
    If you added a dark mode option, it would make the experience much more comfortable.
    """


    # Build sequential orchestration
    workflow = SequentialBuilder(
        participants=[summarizer_agent, classifier_agent, action_agent],
        output_from="all",
    ).build()


    # Run and collect outputs
    result = await workflow.run(f"Customer feedback: {feedback}")
    outputs = result.get_outputs()


    # Display outputs
    i = 1
    for response in outputs:
        for msg in cast(list[Message], response.messages):
            name = msg.author_name or ("assistant" if msg.role == "assistant" else "user")
            print(f"{'-' * 60}\n{i:02d} [{name}]\n{msg.text}")
            i += 1

if __name__ == "__main__":
    asyncio.run(main())
```

## Test the application

- **Feedback:**
  ```
  I use the dashboard every day to monitor metrics, and it works well overall. 
  But when I'm working late at night, the bright screen is really harsh on my eyes. 
  If you added a dark mode option, it would make the experience much more comfortable.
  ```


  <img width="915" height="228" alt="image" src="https://github.com/user-attachments/assets/622da747-93f4-4446-96d4-4335f38cfac7" />


## Optionally, you can try running the code using different feedback inputs

- **Feedback:**
  ```
  I reached out to your customer support yesterday because I couldn't access my account.
  The representative responded almost immediately, was polite and professional, and fixed the issue within minutes.
  Honestly, it was one of the best support experiences I've ever had.
  ```

  <img width="1199" height="572" alt="image" src="https://github.com/user-attachments/assets/00960f62-14a0-4351-a450-86afed0d8e4f" />


## If you want to see all outputs

Replace **# Display outputs** whit this:

```python
    # Prompt from each agent
    agent_prompts = {
        "summarizer": summarizer_instructions.strip(),
        "classifier": classifier_instructions.strip(),
        "action": action_instructions.strip(),
    }


    # Display outputs
    print("-" * 60)
    print("00 [user]")
    print(f"Customer feedback: {feedback.strip()}")

    i = 1
    for response in outputs:
        for msg in cast(list[Message], response.messages):
            name = msg.author_name or (
                "assistant" if msg.role == "assistant" else "user"
            )
        
        print("-" * 60)
        
        # Show prompt from the agent
        if name in agent_prompts:
            print(f"Agent [{name}]")
            print("Instructions: [{agent_prompts[name]}]")
            print()
        
            print(f"{i:02d} [{name}]")
            print(msg.text)
        
            i += 1

```

<img width="941" height="461" alt="image" src="https://github.com/user-attachments/assets/753feda1-bbb9-41c0-8dfa-ebf69fa739f7" />




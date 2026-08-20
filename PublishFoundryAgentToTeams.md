# Publish a Foundry agent to Teams


## The subscription is not registered to use namespace 'Microsoft.BotService'.

Error "The subscription is not registered to use namespace 'Microsoft.BotService'" Status 409 - MissingSubscriptionRegistration

<img width="735" height="653" alt="image" src="https://github.com/user-attachments/assets/a8f17d93-41d6-4b8b-b619-80d53a324da8" />

You need to register **Microsoft.BotService** resource provider in Azure subscription.

<img width="1085" height="207" alt="image" src="https://github.com/user-attachments/assets/e4c8595d-b62e-49f4-b9a1-4490ce6c3c76" />

> Note: You can use Azure CLI ``az provider register --namespace Microsoft.BotService``.

<br>

## Direct publish

| Option	| Behavior	| Admin approval	| Best for |
| --- | --- | --- | ---- | 
| Just you	| Available immediately. The agent appears under Your agents in the agent store. Share it with others by sending the agent link. |	Not required	| Personal testing, small teams, pilots |
| People in your organization	| The agent is submitted for admin approval. Your Microsoft 365 admin reviews the request and assigns access. Once approved, the agent appears under Built by your org for all tenant users.	| Required |	Organization-wide distribution, production deployments |

<br>

## Download and customize

1. Download ZIP.
1. Open Microsoft Team.
1. Go to **Apps** > **Manage your apps** > **Upload an app**.
1. Select
2. **Upload a custom app** or **Submit an app to your org** and choose the downloaded ``.zip`` file.

> [Doc: Upload your app in Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/apps-upload)

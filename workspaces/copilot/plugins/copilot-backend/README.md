# GitHub Copilot Plugin

This GitHub Copilot plugin integrates with Backstage to provide metrics and insights for members of your organization or enterprise.

## Installation

### Install Dependencies

Add the `@backstage-community/plugin-copilot-backend` package to your backend:

```sh
# From your Backstage root directory
yarn --cwd packages/backend add @backstage-community/plugin-copilot-backend
```

### New Backend System

To configure the plugin using the new backend system:

1. In the `packages/backend/src/index.ts` file, add the following:

   ```typescript
   import { createBackend } from '@backstage/backend-defaults';

   const backend = createBackend();

   backend.add(import('@backstage-community/plugin-copilot-backend'));

   backend.start();
   ```

### Old System

To install the plugin using the old method:

1. In your `packages/backend/src/plugins/copilot.ts` file, add the following code:

   ```typescript
   import { SchedulerServiceTaskScheduleDefinition } from '@backstage/backend-plugin-api';
   import { createRouterFromConfig } from '@backstage-community/plugin-copilot-backend';

   export default async function createPlugin(): Promise<void> {
     const schedule: SchedulerServiceTaskScheduleDefinition = {
       frequency: { cron: '0 2 * * *' },
       timeout: { minutes: 15 },
       initialDelay: { seconds: 15 },
     };

     return createRouterFromConfig({ schedule });
   }
   ```

1. Integrate the plugin into the main backend router in `packages/backend/src/index.ts`:

   ```typescript
   import { createRouterFromConfig } from './plugins/copilot';

   async function main() {
     const env = createEnv('copilot');
     apiRouter.use('/copilot', await createRouterFromConfig(env));
   }
   ```

## Configuration

### App Config

To configure the GitHub Copilot plugin, you need to set the following values in the app-config:

- **`copilot.host`**: The host URL for your GitHub Copilot instance (e.g., `github.com` or `github.enterprise.com`).
- **`copilot.enterprise`**: The name of your GitHub Enterprise instance (e.g., `my-enterprise`).
- **`copilot.organization`**: The name of your GitHub Organization (e.g., `my-organization`).

These variables are used to configure the plugin and ensure it communicates with the correct GitHub instance.

### GitHub Credentials

GitHub supports different auth methods depending on which API you are using.

- Enterprise API - [Supports both "classic" PAT tokens and GitHub Apps](https://docs.github.com/en/enterprise-cloud@latest/rest/authentication/permissions-required-for-github-apps?apiVersion=2022-11-28#enterprise-permissions-for-enterprise-copilot-metrics)
- Org API - [Supports app tokens, "classic", and fine grained PAT tokens](https://docs.github.com/en/enterprise-cloud@latest/rest/copilot/copilot-usage?apiVersion=2022-11-28#get-a-summary-of-copilot-usage-for-organization-members)

This plugin supports both schemes and detects the best scheme based on which API(s) you have configured for use.

### GitHub Token/App Scopes

To ensure the GitHub Copilot plugin operates correctly within your organization or enterprise, your GitHub access token must include specific scopes. These scopes grant the plugin the necessary permissions to interact with your GitHub organization and manage Copilot usage.

#### Required Scopes

1. **List Teams Endpoint**

   - **Scope Required:** `read:org`
   - **Purpose:** Allows the plugin to list all teams within your GitHub organization.

2. **Copilot Usage**
   - **Scopes Required - enterprise (PAT):** `manage_billing:copilot`, `read:enterprise`
   - **Scopes Required - enterprise (GitHub App):** `Enterprise Copilot metrics: read` (enterprise-level permission)
   - **Scopes Required - organization:** `manage_billing:copilot`, `read:org`, or `read:enterprise`
   - **Purpose:** Enables the plugin to manage and monitor GitHub Copilot usage within your organization or/and enterprise.

#### How to Configure Token Scopes

**Generate a Personal Access Token (PAT)**

- Navigate to [GitHub Personal Access Tokens](https://github.com/settings/tokens).
- Click on **Generate new token**.
- Select the scopes according to your needs

**Using a GitHub App**

- Create or reuse an existing GitHub App that you own.
- Navigate to the app permissions
- For organization-level access: Select permissions to read the org and manage billing for copilot
- For enterprise-level access: Select the "Enterprise Copilot metrics" permission with `read` access
- Install the app at the organization level (for org metrics) or enterprise level (for enterprise metrics)

### YAML Configuration Example

```yaml
copilot:
  schedule:
    frequency:
      hours: 2
    timeout:
      minutes: 2
    initialDelay:
      seconds: 15
  host: YOUR_GITHUB_HOST_HERE
  enterprise: YOUR_ENTERPRISE_NAME_HERE
  organization: YOUR_ORGANIZATION_NAME_HERE

# Using a PAT (works for both enterprise and organization)
integrations:
  github:
    - host: YOUR_GITHUB_HOST_HERE
      token: YOUR_GENERATED_TOKEN

# Using a GitHub App (works for both enterprise and organization)
# For enterprise: The app must be installed at the enterprise level
# For organization: The app must be installed at the organization level
integrations:
  github:
    - host: github.com
      apps:
        - appId: YOUR_APP_ID
          allowedInstallationOwners:
            - YOUR_ORG_OR_ENTERPRISE_NAME
          clientId: CLIENT_ID
          clientSecret: CLIENT_SECRET
          webhookSecret: WEBHOOK_SECRET
          privateKey: PRIVATE_KEY
```

[You can find more about the integrations config in the official docs](https://backstage.io/docs/integrations/github/locations/)

### API Documentation

For more details on using the GitHub Copilot and Teams APIs, refer to the following documentation:

- [GitHub Teams API - List Teams](https://docs.github.com/en/rest/teams/teams?apiVersion=2022-11-28#list-teams)
- [GitHub Copilot API - Usage](https://docs.github.com/en/rest/copilot/copilot-usage?apiVersion=2022-11-28)

## Run

1. Start the backend:

   ```sh
   yarn start-backend
   ```

2. Verify the plugin is running by accessing `http://localhost:7007/api/copilot/health`.

## Links

- [GitHub Copilot Plugin Frontend](https://github.com/backstage/community-plugins/tree/main/workspaces/copilot/plugins/copilot)
- [Backstage Homepage](https://backstage.io)

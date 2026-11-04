# My First AI Agent

A JavaScript starter project for an AI assistant in Slack. The intended flow is: receive a Slack message, send it to an AI model through LangChain, and reply in Slack.

**Current status:** this is an early scaffold. `index.js` contains imports, environment loading, a logger, and a commented-out agent constructor. It does not yet handle messages or start a server. `db.js` is empty.

## Step 1: Prepare the development environment

The first step is to make sure your computer can run JavaScript and install this project's packages. A **package** is reusable code that provides a capability, such as connecting to Slack or making an HTTP request.

### 1. Check Node.js and npm

Node.js runs JavaScript outside a browser. npm installs packages and runs the commands defined in `package.json`.

Use Node.js 22 or newer for the dependencies currently recorded in the lockfile. In Windows PowerShell, run:

```powershell
node --version
npm.cmd --version
```

Using `npm.cmd` avoids the PowerShell script-policy error that can occur with `npm.ps1`. In other shells, use `npm` in place of `npm.cmd`.

### 2. Open the project

If you already have this folder open, skip cloning. Otherwise:

```powershell
git clone https://github.com/klm2109/My-First-AI-Agent.git
cd My-First-AI-Agent
```

### 3. Understand and install dependencies

`package.json` describes the project, its commands, and the packages it needs. `package-lock.json` records the resolved dependency tree so installations can be repeated. `node_modules/` holds the downloaded package code.

The current files need reconciliation: the lockfile's root dependencies differ from `package.json`, including different major versions for Slack packages and dotenv. Decide which versions the project should use, align the manifest and lockfile, and review the resulting changes before treating the installation as reproducible.

For an initial install using the version ranges in `package.json`:

```powershell
npm.cmd install
```

This can update `package-lock.json`; review that diff. Once both files agree, fresh checkouts can use `npm.cmd ci` for an installation based on the lockfile. Installation downloads dependencies; it does not build a working bot by itself.

## What each package does

These are the direct dependencies declared in `package.json`. The last column distinguishes the intended role from what the scaffold currently implements.

| Package | Function | Role in this project |
| --- | --- | --- |
| `@langchain/core` | Provides shared LangChain building blocks, including prompt templates, messages, and runnable components. | Intended to format instructions and messages for the model. The prompt import currently points to the wrong package path; see below. |
| `@langchain/openai` | Provides LangChain integrations for OpenAI models, including `ChatOpenAI`. | Imported for generating AI responses; the example model setup is commented out. |
| `@slack/bolt` | A framework for handling Slack events, commands, and interactive actions. | Intended to receive Slack activity through a Bolt `App`; its initialization is commented out. |
| `@slack/web-api` | Provides `WebClient` for calling Slack API methods, such as posting a message. | Imported for direct Slack API calls. Bolt also provides a client, so a separate instance may be unnecessary later. |
| `axios` | Makes outgoing HTTP requests to APIs and other web services. | Imported but not used yet; it could let the agent retrieve external information. |
| `dotenv` | Reads values from a local `.env` file into `process.env`. | `dotenv.config()` is already called to load configuration. |
| `express` | Creates an HTTP server with routes and middleware. | Imported for a possible API or health endpoint; no server is currently started. Slack Socket Mode itself does not require a separate Express server. |

Official references: [LangChain core](https://reference.langchain.com/javascript/modules/_langchain_core.html), [ChatOpenAI](https://docs.langchain.com/oss/javascript/integrations/chat/openai), [Slack Bolt](https://docs.slack.dev/tools/bolt-js/), [Slack Web API](https://docs.slack.dev/tools/node-slack-sdk/web-api/), [Axios](https://axios-http.com/docs/intro), [dotenv](https://github.com/motdotla/dotenv), and [Express](https://expressjs.com/).

## Prepare local configuration

The scaffold references these environment variables. Create or edit `.env` locally, keeping any existing values you need:

```dotenv
OPENAI_API_KEY=replace_with_your_api_key
SLACK_BOT_TOKEN=replace_with_your_bot_token
SLACK_SIGNING_SECRET=replace_with_your_signing_secret
SLACK_APP_TOKEN=replace_with_your_socket_mode_app_token
NODE_ENV=development
```

`OPENAI_API_KEY` authenticates model requests. `SLACK_BOT_TOKEN` authenticates the bot's Slack API calls. `SLACK_SIGNING_SECRET` verifies incoming HTTP requests from Slack; the sample constructor includes it, though Socket Mode uses its app token for the connection. `SLACK_APP_TOKEN` enables that Socket Mode connection. `NODE_ENV=development` enables the scaffold's debug logger.

These placeholders do not create credentials or configure a Slack app. Slack app permissions, event subscriptions, and workspace installation still need to be set up when implementing the bot.

The repository currently tracks `.env` and files under `node_modules/`. Adding ignore rules does not untrack existing files. Before publishing further changes, remove these paths from Git tracking while retaining local copies, and ignore `.env` and `node_modules/`. If real credentials have already been committed, rotate them with their provider; deleting a file does not remove it from Git history. A committed `.env.example` should contain placeholders only.

## Setup issues to fix before running the agent

1. **Module format:** `package.json` sets `"type": "commonjs"`, while `index.js` uses ES module `import` syntax. Change the type to `"module"` to match the existing code, or rewrite the imports as CommonJS.
2. **Prompt import:** `index.js` imports from `langchain/prompts`, but `langchain` is not a declared direct dependency. Use the installed core package's prompt entry point:

   ```javascript
   import {
     ChatPromptTemplate,
     HumanMessagePromptTemplate,
     SystemMessagePromptTemplate,
   } from '@langchain/core/prompts';
   ```

3. **Start script:** rename the current script key `` starts` `` to `start`, keeping its value `node index.js`.
4. **Dependency versions:** align `package.json` and `package-lock.json` as described above.
5. **Agent behavior:** implement and instantiate the agent, register Slack message handlers, invoke the model, send replies, and start the Slack connection. The commented constructor alone will not do this.

After correcting the module format and imports, the existing development command is:

```powershell
npm.cmd run dev
```

It runs `node --watch index.js`, restarting when the source changes. After correcting the start script, `npm.cmd start` runs the entry point once. Neither command will connect a functioning bot until the agent behavior is implemented. The current `test` script is an npm placeholder that intentionally exits with an error; there are no implemented tests yet.

Step 1 is complete when Node.js and npm work, dependencies install with a consistent manifest and lockfile, and you understand the configuration and remaining scaffold issues. The next milestone is a minimal Slack message handler that replies successfully, followed by connecting that handler to the AI model.

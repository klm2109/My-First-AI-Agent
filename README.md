# My First AI Agent

A JavaScript starter project for an AI assistant in Slack. The intended flow is: receive a Slack message, send it to an AI model through LangChain, and reply in Slack.

**Current status:** commit `07bcaf4` defines a `SlackAIAgent` class with a constructor that configures Express, Slack, and an AI model. The class is not instantiated, and its two setup methods are not implemented. It does not yet handle messages or start a server. `db.js` is empty. See the code walkthrough below for detailed study notes.

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
| `@langchain/openai` | Provides LangChain integrations for OpenAI models, including `ChatOpenAI`. | The constructor configures a model client, but no model request is made yet. |
| `@slack/bolt` | A framework for handling Slack events, commands, and interactive actions. | The constructor configures a Bolt `App` in Socket Mode; no event handlers or connection startup exist yet. |
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

`OPENAI_API_KEY` authenticates model requests. `SLACK_BOT_TOKEN` authenticates the bot's Slack API calls. `SLACK_SIGNING_SECRET` verifies incoming HTTP requests from Slack; the constructor includes it, though Socket Mode uses its app token for the connection. `SLACK_APP_TOKEN` enables that Socket Mode connection. `NODE_ENV=development` enables the scaffold's debug logger.

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
5. **Missing setup methods:** the constructor calls `this.setupSlackEvents()` and `this.setupExpress()`, but neither method exists. Instantiating the class after resolving the import/configuration issues will fail when it reaches these calls.
6. **Agent behavior:** instantiate the agent, register Slack message handlers, invoke the model, send replies, and start the Slack connection. Defining the constructor alone does not do this.

After correcting the module format and imports, the existing development command is:

```powershell
npm.cmd run dev
```

It runs `node --watch index.js`, restarting when the source changes. After correcting the start script, `npm.cmd start` runs the entry point once. Neither command will connect a functioning bot until the agent behavior is implemented. The current `test` script is an npm placeholder that intentionally exits with an error; there are no implemented tests yet.

Step 1 is complete when Node.js and npm work, dependencies install with a consistent manifest and lockfile, and you understand the configuration and remaining scaffold issues. The next milestone is a minimal Slack message handler that replies successfully, followed by connecting that handler to the AI model.

## Study notes: what changed in commit `07bcaf4`

Reviewed on 5 November 2026 using the local checkout. Remote access was unavailable during this review, so this identifies the exact version these notes describe rather than confirming the latest GitHub state.

The commit turns the previously commented constructor into a real class definition, changes the example model from `gpt-4o-mini` to `gpt-4`, adds `temperature` and an explicit API key, and adds calls to two planned setup methods. It also adds this README and a `.gitignore` rule for `node_modules`. It does not add event handlers, routes, a model invocation, or startup code.

### File map

| File | Purpose and current contents |
| --- | --- |
| `index.js` | Entry point containing imports, configuration loading, logging helpers, and the agent class. |
| `db.js` | Empty placeholder. No database, persistence, or conversation storage is implemented. |
| `package.json` | Project metadata, direct dependency ranges, module format, and npm scripts. `main` names an entry file for consumers; it does not run that file automatically. |
| `package-lock.json` | Resolved dependency versions, download locations, integrity hashes, and dependency requirements. It currently disagrees with the manifest. |
| `.env` | Local configuration values loaded by dotenv. This file is currently tracked; keep actual credentials out of future commits. |
| `.gitignore` | Currently contains only `node_modules`. It prevents new matching files from being added by default, but does not remove already tracked files. |
| `README.md` | Setup instructions and these explanations. |

### 1. Imports: bringing package code into the file

```javascript
import pkg from '@slack/bolt';
const { App } = pkg;
```

The first line imports the package's default export into the local variable `pkg`. The second uses **destructuring** to extract its `App` property. This is the Slack application class; it is different from the Express application stored later as `this.app`.

Imports such as `import { WebClient } from '@slack/web-api'` select a named export. Imports such as `import express from 'express'` select a default export. The package name tells Node where to look; braces indicate which exported values to select.

The prompt imports are intended to build structured model input: a system message describes the assistant's instructions, and a human message carries the user's input. `ChatPromptTemplate` combines message templates. None of these templates is used yet, and their current `langchain/prompts` import needs the correction listed above. Axios is also imported but unused.

### 2. Configuration: `dotenv.config()` and `process.env`

```javascript
dotenv.config();
```

This loads `.env` configuration before the constructor reads it. `process` is a Node.js global, and `process.env` exposes environment variables as strings (or `undefined` when absent). For example, `process.env.SLACK_BOT_TOKEN` reads the value named `SLACK_BOT_TOKEN`; it does not create a token.

Loading a file is different from validating configuration. The current code does not check that required variables exist or contain valid credentials. Later startup code should report missing variable names without printing their secret values.

### 3. Logging: the `log` object

```javascript
const log = {
  info: (msg, ...args) => console.log(`[INFO] ${msg}`, ...args),
  error: (msg, ...args) => console.log(`[ERROR] ${msg}`, ...args),
  debug: (msg, ...args) => process.env.NODE_ENV === 'development' && console.log(`[DEBUG] ${msg}`, ...args)
}
```

This object groups three arrow functions. `msg` is the main message. `...args` collects extra arguments, then spreads them into `console.log`. Backticks create a template string, where `${msg}` inserts the message value.

`info` and `error` always print when called. Despite its name, `error` currently uses `console.log`, so it writes to standard output rather than the standard error stream used by `console.error`. `debug` prints only when `NODE_ENV` equals `development`: `&&` evaluates the logging call only if the condition is true. These helpers are defined but never called in the current file.

### 4. The class, constructor, and `this`

```javascript
class SlackAIAgent {
  constructor() {
    // Create and configure this instance's components.
  }
}
```

A **class** describes how to create an object. Its **constructor** executes when code calls `new SlackAIAgent()`. `this` refers to that new instance, allowing later methods to reuse properties such as `this.slack` and `this.openai`.

The current file only defines the class. There is no `new SlackAIAgent()` call, so none of the constructor's statements executes. Even after adding that call, creating clients alone does not start the agent.

### 5. `this.app = express()`

This would create an Express application and save it on the instance. Express handles incoming HTTP requests through routes such as `GET /health`. No routes or `listen()` call exist yet. The property name `app` belongs to this class; it is not the same object as the Bolt `App` constructor.

The intended `setupExpress()` method could register routes. Starting an HTTP listener would still need explicit code. A separate Express server is optional for a bot that receives Slack events through Socket Mode.

### 6. `this.slack = new App({...})`

The options object configures the Slack Bolt client:

| Option | Meaning in this code |
| --- | --- |
| `token` | Reads the bot token used to call Slack APIs. |
| `signingSecret` | Supplies the secret used for HTTP request verification; Socket Mode uses a different connection mechanism. |
| `socketMode: true` | Selects a WebSocket connection for Slack events instead of receiving them through a public HTTP event endpoint. |
| `appToken` | Reads the app-level token used for the Socket Mode connection. |

Constructing `App` configures it. A later `await this.slack.start()` call is needed to start the Slack connection. Event handlers must also be registered before the bot can respond to messages.

### 7. `this.WebClient = new WebClient(...)`

This would create a second client for direct Slack Web API calls. For example, a future handler could call `chat.postMessage` to send a reply to a channel. The bot token determines its identity and permissions; creating this object does not automatically grant access to a channel.

`this.WebClient` is an instance property despite its capitalized name. `this.webClient` would be a more conventional instance name. Bolt already exposes Web API client functionality, so later code can choose whether the separate client is useful.

### 8. `this.openai = new ChatOpenAI({...})`

The constructor configures the LangChain model client with these values:

| Option | Meaning in this code |
| --- | --- |
| `model: 'gpt-4'` | Requests this model identifier when a model call is eventually made. Account access and model availability must be checked when implementing that call. |
| `temperature: 0.3` | Requests relatively low sampling randomness for a model that supports temperature. It does not guarantee correctness or identical answers. |
| `apiKey` | Reads `OPENAI_API_KEY` for authentication. |

Creating the client does not send a prompt or generate an answer. A future asynchronous call such as `await this.openai.invoke(messages)` would request a model response. The handler would then need to extract the response content and send it to Slack. There is currently no memory, tool execution, or autonomous decision loop; the class name expresses the intended project direction.

### 9. The two missing setup methods

```javascript
this.setupSlackEvents();
this.setupExpress();
```

These are method calls, not method definitions. The names describe planned work, but the class has no corresponding methods. Once construction gets past configuration and client creation, the first call will fail with a `TypeError` because `this.setupSlackEvents` is not a function; the second call will not be reached.

`setupSlackEvents()` should register the chosen event handlers, ignore bot messages to avoid reply loops, and handle model or Slack errors. `setupExpress()` should register any HTTP routes the project actually needs. Starting connections and listeners should be explicit, with startup failures handled too.

### Execution order and intended message flow

At present, Node fails while parsing `index.js` because its imports conflict with `"type": "commonjs"`. This was reproduced with `node --check index.js`. After correcting that and the prompt import, the top-level code would load configuration, create the logger, and define the class, then finish without starting a bot.

Once an instance and the missing behavior are implemented, the intended sequence is:

1. Load and validate configuration.
2. Construct the Express, Slack, and model clients.
3. Register Slack handlers and any HTTP routes.
4. Start the Slack connection and any required HTTP listener.
5. Receive a user message and filter out messages the bot should ignore.
6. Build model input, await the AI response, and send a reply to the correct Slack conversation.
7. Log useful information and handle failures without exposing credentials.

Steps 3 onward are still planned. No live Slack connection or model request was made during this documentation review.

### Recommended next implementation steps

First fix the module format, prompt import, start script, and dependency mismatch. Then implement a minimal Slack handler that returns a fixed reply and start the connection. That verifies Slack configuration before adding model calls. Next connect the handler to the model, add error handling and input filtering, and decide whether Express routes or persistent storage are needed.

Use `node --check index.js` to check syntax after changing the module configuration. A successful syntax check does not verify dependencies, credentials, Slack permissions, missing methods, or end-to-end behavior. Those need separate checks as their implementations are added.

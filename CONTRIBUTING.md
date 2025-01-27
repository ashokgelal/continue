# Contributing to Continue

## Table of Contents

- [❤️ Ways to Contribute](#️-ways-to-contribute)
  - [👋 Continue Contribution Ideas](#-continue-contribution-ideas)
  - [🐛 Report Bugs](#-report-bugs)
  - [✨ Suggest Enhancements](#-suggest-enhancements)
  - [📖 Updating / Improving Documentation](#-updating--improving-documentation)
  - [🧑‍💻 Contributing Code](#-contributing-code)
    - [Environment Setup](#environment-setup)
    - [Writing Slash Commands](#writing-slash-commands)
    - [Writing Context Providers](#writing-context-providers)
    - [Adding an LLM Provider](#adding-an-llm-provider)
    - [Adding Models](#adding-models)
- [📐 Continue Architecture](#-continue-architecture)
  - [Continue VS Code Extension](#continue-vs-code-extension)
  - [Continue JetBrains Extension](#continue-jetbrains-extension)

# ❤️ Ways to Contribute

## 👋 Continue Contribution Ideas

[This GitHub project board](https://github.com/orgs/continuedev/projects/2) is a list of ideas for how you can contribute to Continue. These aren't the only ways, but are a great starting point if you are new to the project.

## 🐛 Report Bugs

If you find a bug, please [create an issue](https://github.com/continuedev/continue/issues) to report it! A great bug report includes:

- A description of the bug
- Steps to reproduce
- What you expected to happen
- What actually happened
- Screenshots or videos

## ✨ Suggest Enhancements

Continue is quickly adding features, and we'd love to hear which are the most important to you. The best ways to suggest an enhancement are

- Create an issue

  - First, check whether a similar proposal has already been made
  - If not, [create an issue](https://github.com/continuedev/continue/issues)
  - Please describe the enhancement in as much detail as you can, and why it would be useful

- Join the [Continue Discord](https://discord.gg/NWtdYexhMs) and tell us about your idea in the `#feedback` channel

## 📖 Updating / Improving Documentation

Continue is continuously improving, but a feature isn't complete until it is reflected in the documentation! If you see something out-of-date or missing, you can help by clicking "Edit this page" at the bottom of any page on [continue.dev/docs](https://continue.dev/docs).

## 🧑‍💻 Contributing Code

> Please make PRs to the `preview` branch. We use this to first test changes in a pre-release version of the extension.

### Environment Setup

VS Code is assumed for development as Continue is primarily a VS Code tool at the moment. Most of the setup and running is automated through VS Code tasks and launch configurations.

<!-- Pre-requisite: you will need `cargo` the rust package manager installed ([get it on rust-lang.org](https://www.rust-lang.org/tools/install)). -->
Pre-requisite: you have Node.js version 20.11.0 (LTS) or higher installed.  [get it on https://nodejs.org](https://nodejs.org/en/download/
Alternatively, if you are using NVM (Node Version Manager), you can set the correct version of Node.js for this project by running the following command in the root of the project:

```bash
nvm use


1. Clone and open in VS Code the Continue repo `https://github.com/continuedev/continue`

2. Open VS Code command pallet (`cmd+shift+p`) and select `Tasks: Run Task` and then select `install-all-dependencies`

3. Start debugging:

   1. Switch to Run and Debug view
   2. Select `Extension (VS Code)` from drop down
   3. Hit play button
   4. This will start the extension in debug mode and open a new VS Code window with it installed
      1. I call the VS Code window with the extension the _Host VS Code_
      2. The window you started debugging from is referred to as the _Main VS Code_

4. Try using breakpoints:
   > Note: Breakpoints for the code inside of the `gui` folder are not currently supported while debugging the entire extension, but can be used with the "Vite" launch configuration and Google Chrome.
   1. _In Main VS Code_: Search for `function addHighlightedCodeToContext` and set a breakpoint at the top of the function. This is the method invoked whenever code in the editor is selected to be used as context.
   2. _In Host VS Code_: Select part of the `example.ts` file and use the keyboard shortcut cmd/ctrl+m to select the code and notice that your breakpoint should be hit.

### Formatting

Continue uses [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) to format JavaScript/TypeScript. Please install these extensions in VS Code and enable "Format on Save" in your settings.

### Writing Slash Commands

The slash command interface, defined in [core/index.d.ts](./core/index.d.ts), requires you to define a `name` (the text that will be typed to invoke the command), a `description` (the text that will be shown in the slash command menu), and a `run` function that will be called when the command is invoked. The `run` function is an async generator that yields the content to be displayed in the chat. The `run` function is passed a `ContinueSDK` object that can be used to interact with the IDE, call the LLM, and see the chat history, among a few other utilities.

```ts
export interface SlashCommand {
  name: string;
  description: string;
  params?: { [key: string]: any };
  run: (sdk: ContinueSDK) => AsyncGenerator<string | undefined>;
}
```

There are many example of slash commands in [core/commands/slash](./core/commands/slash) that we recommend borrowing from. Once you've created your new `SlashCommand` in this folder, also be sure to complete the following:

- Add your command to the array in [core/commands/slash/index.ts](./core/commands/slash/index.ts)
- Add your command to the list in [`config_schema.json`](./extensions/vscode/config_schema.json). This makes sure that Intellisense shows users what commands are available for your provider when they are editing `config.json`. If there are any parameters that your command accepts, you should also follow existing examples in adding them to the JSON Schema.

### Writing Context Providers

A `ContextProvider` is a Continue plugin that lets type '@' to quickly select documents as context for the language model. The `IContextProvider` interface is defined in [`core/index.d.ts`](./core/index.d.ts), but all built-in context providers extend [`BaseContextProvider`](./core/context/index.ts).

Before defining your context provider, determine which "type" you want to create. The `"query"` type will show a small text input when selected, giving the user the chance to enter something like a Google search query for the [`GoogleContextProvider`](./core/context/providers/GoogleContextProvider.ts). The `"submenu"` type will open up a submenu of items that can be searched through and selected. Examples are the [`GitHubIssuesContextProvider`](./core/context/providers/GitHubIssuesContextProvider.ts) and the [`DocsContextProvider`](./core/context/providers/DocsContextProvider.ts). The `"normal"` type will just immediately add the context item. Examples include the [`DiffContextProvider`](./core/context/providers/DiffContextProvider.ts) and the [`OpenFilesContextProvider`](./core/context/providers/OpenFilesContextProvider.ts).

After you've written your context provider, make sure to complete the following:

- Add it to the array of context providers in [core/context/providers/index.ts](./core/context/providers/index.ts)
- Add it to the `ContextProviderName` type in [core/index.d.ts](./core/index.d.ts)
- Add it to the list in [`config_schema.json`](./extensions/vscode/config_schema.json). If there are any parameters that your context provider accepts, you should also follow existing examples in adding them to the JSON Schema.

### Adding an LLM Provider

Continue has support for more than a dozen different LLM "providers", making it easy to use models running on OpenAI, Ollama, Together, LM Studio, Msty, and more. You can find all of the existing providers [here](https://github.com/continuedev/continue/tree/main/core/llm/llms), and if you see one missing, you can add it with the following steps:

1. Create a new file in the `core/llm/llms` directory. The name of the file should be the name of the provider, and it should export a class that extends `BaseLLM`. This class should contain the following minimal implementation. We recommend viewing pre-existing providers for more details. The [LlamaCpp Provider](./core/llm/llms/LlamaCpp.ts) is a good simple example.

- `providerName` - the identifier for your provider
- At least one of `_streamComplete` or `_streamChat` - This is the function that makes the request to the API and returns the streamed response. You only need to implement one because Continue can automatically convert between "chat" and "raw completion".

2. Add your provider to the `LLMs` array in [core/llm/llms/index.ts](./core/llm/llms/index.ts).
3. If your provider supports images, add it to the `PROVIDER_SUPPORTS_IMAGES` array in [core/llm/index.ts](./core/llm/index.ts).
4. Add the necessary JSON Schema types to [`config_schema.json`](./extensions/vscode/config_schema.json). This makes sure that Intellisense shows users what options are available for your provider when they are editing `config.json`.
5. Add a documentation page for your provider in [`docs/docs/reference/Model Providers`](./docs/docs/reference/Model%20Providers). This should show an example of configuring your provider in `config.json` and explain what options are available.

### Adding Models

While any model that works with a supported provider can be used with Continue, we keep a list of recommended models that can be automatically configured from the UI or `config.json`. The following files should be updated when adding a model:

- [config_schema.json](./extensions/vscode/config_schema.json) - This is the JSON Schema definition that is used to validate `config.json`. You'll notice a number of rules defined in "definitions.ModelDescription.allOf". Here is where you write rules that can specify something like "for the provider 'anthropic', only models 'claude-2' and 'claude-instant-1' are allowed. Look through all of these rules and make sure that your model is included for providers that support it.
- [modelData.ts](./gui/src/util/modelData.ts) - This file defines that information that is shown in the model selection UI in the side bar. To add a new model:
  1. create a `ModelPackage` object, following the lead of the many examples near the top of the file
  2. add the `ModelPackage` to the `MODEL_INFO` array if you would like it to be displayed in the "Models" tab
  3. if you would like it to be displayed as an option under any of the providers, go to the `PROVIDER_INFO` object and add it to the `packages` array for each provider that you want it to be displayed under. If it is an OS model that should be valid for most providers offering OS models, you might just be able to add it to the `osModels` array as shorthand.
- [index.d.ts](./core/index.d.ts) - This file defines the TypeScript types used throughout Continue. You'll find a `ModelName` type. Be sure to add the name of your model to this.
- LLM Providers: Since many providers use their own custom strings to identify models, you'll have to add the translation from Continue's model name (the one you added to `index.d.ts`) and the model string for each of these providers: [Ollama](./core/llm/llms/Ollama.ts), [Together](./core/llm/llms/Together.ts), and [Replicate](./core/llm/llms/Replicate.ts). You can find their full model lists here: [Ollama](https://ollama.ai/library), [Together](https://docs.together.ai/docs/inference-models), [Replicate](https://replicate.com/collections/streaming-language-models).
- [Prompt Templates](./core/llm/index.ts) - In this file you'll find the `autodetectTemplateType` function. Make sure that for the model name you just added, this function returns the correct template type. This is assuming that the chat template for that model is already built in Continue. If not, you will have to add the template type and corresponding edit and chat templates.

## 📐 Continue Architecture

Continue consists of 2 parts that are split so that it can be extended to work in other IDEs as easily as possible:

1. **Continue GUI** - The Continue GUI is a React application that gives the user control over Continue. It displays the current chat history, allows the user to ask questions, invoke slash commands, and use context providers. The GUI also handles most state and holds as much of the logic as possible so that it can be reused between IDEs.

2. **Continue Extension** - The Continue Extension is a plugin for the IDE which implements the [IDE Interface](./core/index.d.ts#L229). This allows the GUI to request information from or actions to be taken within the IDE. This same interface is used regardless of IDE. The first Continue extensions we have built are for VS Code and JetBrains, but we plan to build clients for other IDEs in the future. The IDE Client must 1. implement IDE Interface, as is done [here](./extensions/vscode/src/ideProtocol.ts) for VS Code and 2. display the Continue GUI in a sidebar, like [here](./extensions/vscode/src/debugPanel.ts).

### Continue VS Code Extension

The starting point for the VS Code extension is [activate.ts](./extensions/vscode/src/activation/activate.ts). The `activateExtension` function here will register all commands and load the Continue GUI in the sidebar of the IDE as a webview.

### Continue JetBrains Extension

The JetBrains extension is currently in alpha testing. Please reach out on [Discord](https://discord.gg/vapESyrFmJ) if you are interested in contributing to its development.

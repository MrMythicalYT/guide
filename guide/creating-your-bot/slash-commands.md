# Creating slash commands

<DiscordMessages>
	<DiscordMessage profile="bot">
		<template #interactions>
			<DiscordInteraction profile="user" :command="true">ping</DiscordInteraction>
		</template>
		Pong!
	</DiscordMessage>
</DiscordMessages>

Discord allows developers to register [slash commands](https://discord.com/developers/docs/interactions/application-commands), which provide users a first-class way of interacting directly with your application. 

Slash commands provide a huge number of benefits over manual message parsing, including:

- Integration with the Discord client interface.
- Automatic command detection and parsing of the associated options/arguments.
- Typed argument inputs for command options, e.g. "String", "User", or "Role".
- Validated or dynamic choices for command options.
- In-channel private responses (ephemeral messages).
- Pop-up form-style inputs for capturing additional information.

...and many more!

## Before you continue

Assuming you've followed the guide so far, your project directory should look something like this:

```:no-line-numbers
discord-bot/
├── node_modules
├── config.json
├── index.js
├── package-lock.json
└── package.json
```

To go from this starter code to fully functional slash commands, there are three key pieces of code that need to be written. They are:

1. The individual command files, containing their defintions and functionality.
2. The command handler, which dynamically reads the files and executes the commands.
3. The command deployment script, to register your slash commands with Discord so they appear in the interface.

These steps can be done in any order, but all are required before the commands are fully functional. This section of the guide will use the order listed above. So let's get started!

## Indivdual command files

Create a new folder named `commands`, which is where you'll store all of your command files. You'll be using the <DocsLink section="builders" path="class/SlashCommandBuilder"/> class to construct the command definitions.

At a minimum, the definition of a slash command must have a name and a description. Slash command names must be between 1-32 characters and contain no capital letters, spaces, or symbols other than `-` and `_`. Using the builder, a simple `ping` command definition would look like this:

```js
new SlashCommandBuilder()
	.setName('ping')
	.setDescription('Replies with Pong!');
```

A slash command also requires a function to run when the command is used, to respond to the interaction. Using an interaction response method confirms to Discord that your bot successfully received the interaction, and has responded to the user. Discord enforces this to ensure that all slash commands provide a good user experience (UX). Failing to respond will cause Discord to show that the command failed, even if your bot is performing other actions as a result.

The simplest way to acknowledge and respond to an interaction is the `interaction.reply()` method. Other methods of replying are covered on the [Response methods](/slash-commands/response-methods.md) page later in this section.

<!-- eslint-skip -->

```js
async execute(interaction) {
	await interaction.reply('Pong'!)
}
```

Put these two together by creating a `commands/ping.js` file for your first command. Inside this file, you're going to define and export two items.
- The `data` property, which will provide the command definition shown above for registering to Discord.
- The `execute` method, which will contain the functionality to run from our event handler when the command is used.

These are placed inside `module.exports` so they can be read by other files; namely the command loader and command deployment scripts mentioned earlier.

:::: code-group
::: code-group-item commands/ping.js
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('ping')
		.setDescription('Replies with Pong!'),
	async execute(interaction) {
		await interaction.reply('Pong!');
	},
};
```
:::
::::

::: tip
[`module.exports`](https://nodejs.org/api/modules.html#modules_module_exports) is how you export data in Node.js so that you can [`require()`](https://nodejs.org/api/modules.html#modules_require_id) it in other files.

If you need to access your client instance from inside a command file, you can access it via `interaction.client`. If you need to access external files, packages, etc., you should `require()` them at the top of the file.
:::

That's it for your basic ping command. Below are examples of two more commands we're going to build upon throughout the guide, so create two more files for these before you continue reading.

:::: code-group
::: code-group-item commands/user.js
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('user')
		.setDescription('Provides information about the user.'),
	async execute(interaction) {
		// interaction.user is the object representing the User who ran the command
		// interaction.member is the GuildMember object, which represents the user in the specific guild
		await interaction.reply(`This command was run by ${interaction.user.username}, who joined on ${interaction.member.joinedAt}.`);
	},
};
```
:::
::: code-group-item commands/server.js
```js
const { SlashCommandBuilder } = require('discord.js');

module.exports = {
	data: new SlashCommandBuilder()
		.setName('server')
		.setDescription('Provides information about the server.'),
	async execute(interaction) {
		// interaction.guild is the object representing the Guild in which the command was run
		await interaction.reply(`This server is ${interaction.guild.name} and has ${interaction.guild.memberCount} members.`);
	},
};
```
:::
::::

#### Next steps

You can implement additional commands by creating additional files in the `commands` folder, but these three are the ones we're going to use for the examples as we go on. For now let's move on to the code you'll need for command handling, to load the files and respond to incoming interactions.

#### Resulting code

<ResultingCode path="creating-your-bot/slash-commands" />
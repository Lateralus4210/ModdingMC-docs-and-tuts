Creating Commands

This page is written for version:

1.21.4


Page Authors
Atakku
dicedpixels
haykam821
i509VCB
Juuxel
modmuss50
mschae23
natanfudge
Pyrofab
SolidBlock-cn
Technici4n
Treeways
xpple
Creating commands can allow a mod developer to add functionality that can be used through a command. This tutorial will teach you how to register commands and the general command structure of Brigadier.

INFO

Brigadier is a command parser and dispatcher written by Mojang for Minecraft. It is a tree-based command library where you build a tree of commands and arguments.

Brigadier is open-source: https://github.com/Mojang/brigadier

The Command Interface
com.mojang.brigadier.Command is a functional interface, which runs some specific code, and throws a CommandSyntaxException in certain cases. It has a generic type S, which defines the type of the command source. The command source provides some context in which a command was run. In Minecraft, the command source is typically a ServerCommandSource which can represent a server, a command block, a remote connection (RCON), a player or an entity.

The single method in Command, run(CommandContext<S>) takes a CommandContext<S> as the sole parameter and returns an integer. The command context holds your command source of S and allows you to obtain arguments, look at the parsed command nodes and see the input used in this command.

Like other functional interfaces, it is usually used as a lambda or a method reference:


Command<ServerCommandSource> command = context -> {
    return 0;
};
1
2
3
The integer can be considered the result of the command. Typically values less than or equal to zero mean a command has failed and will do nothing. Positive values mean the command was successful and did something. Brigadier provides a constant to indicate success; Command#SINGLE_SUCCESS.

What Can the ServerCommandSource Do?
A ServerCommandSource provides some additional implementation-specific context when a command is run. This includes the ability to get the entity that executed the command, the world the command was run in or the server the command was run on.

You can access the command source from a command context by calling getSource() on the CommandContext instance.


Command<ServerCommandSource> command = context -> {
    ServerCommandSource source = context.getSource();
    return 0;
};
1
2
3
4
Registering a Basic Command
Commands are registered within the CommandRegistrationCallback provided by the Fabric API.

INFO

For information on registering callbacks, please see the Events guide.

The event should be registered in your mod's initializer.

The callback has three parameters:

CommandDispatcher<ServerCommandSource> dispatcher - Used to register, parse and execute commands. S is the type of command source the command dispatcher supports.
CommandRegistryAccess registryAccess - Provides an abstraction to registries that may be passed to certain command argument methods
CommandManager.RegistrationEnvironment environment - Identifies the type of server the commands are being registered on.
In the mod's initializer, we just register a simple command:


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("test_command").executes(context -> {
        context.getSource().sendFeedback(() -> Text.literal("Called /test_command."), false);
        return 1;
    }));
});
1
2
3
4
5
6
In the sendFeedback() method, the first parameter is the text to be sent, which is a Supplier<Text> to avoid instantiating Text objects when not needed.

The second parameter determines whether to broadcast the feedback to other operators. Generally, if the command is to query something without actually affecting the world, such as query the current time or some player's score, it should be false. If the command does something, such as changing the time or modifying someone's score, it should be true.

If the command fails, instead of calling sendFeedback(), you may directly throw any exception and the server or client will handle it appropriately.

CommandSyntaxException is generally thrown to indicate syntax errors in commands or arguments. You can also implement your own exception.

To execute this command, you must type /test_command, which is case-sensitive.

INFO

From this point onwards, we will be extracting the logic written within the lambda passed into .execute() builders into individual methods. We can then pass a method reference to .execute(). This is done for clarity.

Registration Environment
If desired, you can also make sure a command is only registered under some specific circumstances, for example, only in the dedicated environment:


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    if (environment.dedicated) {
        dispatcher.register(CommandManager.literal("dedicated_command")
                .executes(FabricDocsReferenceCommands::executeDedicatedCommand));
    }
});
1
2
3
4
5
6

private static int executeDedicatedCommand(CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /dedicated_command."), false);
    return 1;
}
1
2
3
4
Command Requirements
Let's say you have a command that you only want operators to be able to execute. This is where the requires() method comes into play. The requires() method has one argument of a Predicate<S> which will supply a ServerCommandSource to test with and determine if the CommandSource can execute the command.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("required_command")
            .requires(source -> source.hasPermissionLevel(1))
            .executes(FabricDocsReferenceCommands::executeRequiredCommand));
});
1
2
3
4
5

private static int executeRequiredCommand(CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /required_command."), false);
    return 1;
}
1
2
3
4
This command will only execute if the source of the command is a level 2 operator at a minimum, including command blocks. Otherwise, the command is not registered.

This has the side effect of not showing this command in tab completion to anyone who is not a level 2 operator. This is also why you cannot tab-complete most commands when you do not enable cheats.

Sub Commands
To add a sub command, you register the first literal node of the command normally. To have a sub command, you have to append the next literal node to the existing node.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_one")
            .then(CommandManager.literal("sub_command_one").executes(FabricDocsReferenceCommands::executeSubCommandOne)));
});
1
2
3
4

private static int executeSubCommandOne(CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /command sub_command_one."), false);
    return 1;
}
1
2
3
4
Similar to arguments, sub command nodes can also be set optional. In the following case, both /command_two and /command_two sub_command_two will be valid.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_two")
            .executes(FabricDocsReferenceCommands::executeCommandTwo)
            .then(CommandManager.literal("sub_command_two").executes(FabricDocsReferenceCommands::executeSubCommandTwo)));
});
1
2
3
4
5

private static int executeCommandTwo(CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /command_two."), false);
    return 1;
}

private static int executeSubCommandTwo(CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /sub_command_two."), false);
    return 1;
}
1
2
3
4
5
6
7
8
9
Client Commands
Fabric API has a ClientCommandManager in net.fabricmc.fabric.api.client.command.v2 package that can be used to register client-side commands. The code should exist only in client-side code.


ClientCommandRegistrationCallback.EVENT.register((dispatcher, registryAccess) -> {
    dispatcher.register(ClientCommandManager.literal("clienttater").executes(context -> {
        context.getSource().sendFeedback(Text.literal("Called /clienttater with no arguments."));
        return 1;
    }));
});
1
2
3
4
5
6
Command Redirects
Command redirects - also known as aliases - are a way to redirect the functionality of one command to another. This is useful for when you want to change the name of a command, but still want to support the old name.

WARNING

Brigadier will only redirect command nodes with arguments. If you want to redirect a command node without arguments, provide an .executes() builder with a reference to the same logic as outlined in the example.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    var redirectedBy = dispatcher.register(CommandManager.literal("redirected_by").executes(FabricDocsReferenceCommands::executeRedirectedBy));
    dispatcher.register(CommandManager.literal("to_redirect").executes(FabricDocsReferenceCommands::executeRedirectedBy).redirect(redirectedBy));
});
1
2
3
4

private static int executeRedirectedBy(CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /redirected_by."), false);
    return 1;
}
1
2
3
4
FAQ
Why Does My Code Not Compile?
Catch or throw a CommandSyntaxException - CommandSyntaxException is not a RuntimeException. If you throw it, that should be in methods that throw CommandSyntaxException in method signatures, or it should be caught. Brigadier will handle the checked exceptions and forward the proper error message in the game for you.

Issues with generics - You may have an issue with generics once in a while. If you are registering server commands (which are most of the case), make sure you are using CommandManager.literal or CommandManager.argument instead of LiteralArgumentBuilder.literal or RequiredArgumentBuilder.argument.

Check sendFeedback() method - You may have forgotten to provide a boolean as the second argument. Also remember that, since Minecraft 1.20, the first parameter is Supplier<Text> instead of Text.

A Command should return an integer - When registering commands, the executes() method accepts a Command object, which is usually a lambda. The lambda should return an integer, instead of other types.

Can I Register Commands at Runtime?
WARNING

You can do this, but it is not recommended. You would get the CommandManager from the server and add anything commands you wish to its CommandDispatcher.

After that, you need to send the command tree to every player again using CommandManager.sendCommandTree(ServerPlayerEntity).

This is required because the client locally caches the command tree it receives during login (or when operator packets are sent) for local completions-rich error messages.

Can I Unregister Commands at Runtime?
WARNING

You can also do this, however, it is much less stable than registering commands at runtime and could cause unwanted side effects.

To keep things simple, you need to use reflection on Brigadier and remove nodes. After this, you need to send the command tree to every player again using sendCommandTree(ServerPlayerEntity).

If you don't send the updated command tree, the client may think a command still exists, even though the server will fail execution.

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Command Arguments

This page is written for version:

1.21.4


Arguments are used in most of the commands. Sometimes they can be optional, which means if you do not provide that argument, the command will also run. One node may have multiple argument types, but be aware that there is a possibility of ambiguity, which should be avoided.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_with_arg")
            .then(CommandManager.argument("value", IntegerArgumentType.integer())
                    .executes(FabricDocsReferenceCommands::executeCommandWithArg)));
});
1
2
3
4
5

private static int executeCommandWithArg(CommandContext<ServerCommandSource> context) {
    int value = IntegerArgumentType.getInteger(context, "value");
    context.getSource().sendFeedback(() -> Text.literal("Called /command_with_arg with value = %s".formatted(value)), false);
    return 1;
}
1
2
3
4
5
In this case, after the command text /command_with_arg, you should type an integer. For example, if you run /command_with_arg 3, you will get the feedback message:

Called /command_with_arg with value = 3

If you type /command_with_arg without arguments, the command cannot be correctly parsed.

Then we add an optional second argument:


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_with_two_args")
            .then(CommandManager.argument("value_one", IntegerArgumentType.integer())
                    .executes(FabricDocsReferenceCommands::executeWithOneArg)
                    .then(CommandManager.argument("value_two", IntegerArgumentType.integer())
                            .executes(FabricDocsReferenceCommands::executeWithTwoArgs))));
});
1
2
3
4
5
6
7

private static int executeWithOneArg(CommandContext<ServerCommandSource> context) {
    int value1 = IntegerArgumentType.getInteger(context, "value_one");
    context.getSource().sendFeedback(() -> Text.literal("Called /command_with_two_args with value one = %s".formatted(value1)), false);
    return 1;
}

private static int executeWithTwoArgs(CommandContext<ServerCommandSource> context) {
    int value1 = IntegerArgumentType.getInteger(context, "value_one");
    int value2 = IntegerArgumentType.getInteger(context, "value_two");
    context.getSource().sendFeedback(() -> Text.literal("Called /argtater2 with value one = %s and value two = %s".formatted(value1, value2)),
            false);
    return 1;
}
1
2
3
4
5
6
7
8
9
10
11
12
13
Now you can type one or two integers. If you give one integer, a feedback text with a single value is printed. If you provide two integers, a feedback text with two values will be printed.

You may find it unnecessary to specify similar executions twice. Therefore, we can create a method that will be used in both executions.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_with_common_exec")
            .then(CommandManager.argument("value_one", IntegerArgumentType.integer())
                    .executes(context -> executeCommon(IntegerArgumentType.getInteger(context, "value_one"), 0, context))
                    .then(CommandManager.argument("value_two", IntegerArgumentType.integer())
                            .executes(context -> executeCommon(
                                    IntegerArgumentType.getInteger(context, "value_one"),
                                    IntegerArgumentType.getInteger(context, "value_two"),
                                    context)))));
});
1
2
3
4
5
6
7
8
9
10

private static int executeCommon(int value1, int value2, CommandContext<ServerCommandSource> context) {
    context.getSource().sendFeedback(() -> Text.literal("Called /command_with_common_exec with value 1 = %s and value 2 = %s".formatted(value1, value2)), false);
    return 1;
}
1
2
3
4
Custom Argument Types
If vanilla does not have the argument type you need, you can create your own. To do this, you need to create a class that inherits the ArgumentType<T> interface where T is the type of the argument.

You will need to implement the parse method, which will parse the input string into the desired type.

For example, you can create an argument type that parses a BlockPos from a string with the following format: {x, y, z}


public class BlockPosArgumentType implements ArgumentType<BlockPos> {
    /**
     * Parse the BlockPos from the reader in the {x, y, z} format.
     */
    @Override
    public BlockPos parse(StringReader reader) throws CommandSyntaxException {
        try {
            // This requires the argument to be surrounded by quotation marks.
            // eg: "{1, 2, 3}"
            String string = reader.readString();

            // Remove the { and } from the string using regex.
            string = string.replace("{", "").replace("}", "");

            // Split the string into the x, y, and z values.
            String[] split = string.split(",");

            // Parse the x, y, and z values from the split string.
            int x = Integer.parseInt(split[0].trim());
            int y = Integer.parseInt(split[1].trim());
            int z = Integer.parseInt(split[2].trim());

            // Return the BlockPos.
            return new BlockPos(x, y, z);
        } catch (Exception e) {
            // Throw an exception if anything fails inside the try block.
            throw CommandSyntaxException.BUILT_IN_EXCEPTIONS.dispatcherParseException().create("Invalid BlockPos format. Expected {x, y, z}");
        }
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
Registering Custom Argument Types
WARNING

You need to register the custom argument type on both the server and the client or else the command will not work!

You can register your custom argument type in the onInitialize method of your mod's initializer using the ArgumentTypeRegistry class:


ArgumentTypeRegistry.registerArgumentType(
        Identifier.of("fabric-docs", "block_pos"),
        BlockPosArgumentType.class,
        ConstantArgumentSerializer.of(BlockPosArgumentType::new)
);
1
2
3
4
5
Using Custom Argument Types
We can use our custom argument type in a command - by passing an instance of it into the .argument method on the command builder.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_with_custom_arg").then(
            CommandManager.argument("block_pos", new BlockPosArgumentType())
                    .executes(FabricDocsReferenceCommands::executeCustomArgCommand)
    ));
});
1
2
3
4
5
6

private static int executeCustomArgCommand(CommandContext<ServerCommandSource> context) {
    BlockPos arg = context.getArgument("block_pos", BlockPos.class);
    context.getSource().sendFeedback(() -> Text.literal("Called /command_with_custom_arg with block pos = %s".formatted(arg)), false);
    return 1;
}
1
2
3
4
5
Running the command, we can test whether or not the argument type works:

Invalid argument

Valid argument

Command result

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Command Suggestions

This page is written for version:

1.21.4


Page Authors
IMB11
Minecraft has a powerful command suggestion system that's used in many places, such as the /give command. This system allows you to suggest values for command arguments to the user, which they can then select from - it's a great way to make your commands more user-friendly and ergonomic.

Suggestion Providers
A SuggestionProvider is used to make a list of suggestions that will be sent to the client. A suggestion provider is a functional interface that takes a CommandContext and a SuggestionBuilder and returns some Suggestions. The SuggestionProvider returns a CompletableFuture as the suggestions may not be available immediately.

Using Suggestion Providers
To use a suggestion provider, you need to call the suggests method on the argument builder. This method takes a SuggestionProvider and returns the modified argument builder with the suggestion provider attached.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_with_suggestions").then(
            CommandManager.argument("entity", RegistryEntryReferenceArgumentType.registryEntry(registryAccess, RegistryKeys.ENTITY_TYPE))
                    .suggests(SuggestionProviders.SUMMONABLE_ENTITIES)
                    .executes(FabricDocsReferenceCommands::executeCommandWithSuggestions)
    ));
});
1
2
3
4
5
6
7

private static int executeCommandWithSuggestions(CommandContext<ServerCommandSource> context) throws CommandSyntaxException {
    var entityType = RegistryEntryReferenceArgumentType.getSummonableEntityType(context, "entity");
    context.getSource().sendFeedback(() -> Text.literal("Called /command_with_suggestions with entity = %s".formatted(entityType.value().getUntranslatedName())), false);
    return 1;
}
1
2
3
4
5
Built-in Suggestion Providers
There are a few built-in suggestion providers that you can use:

Suggestion Provider	Description
SuggestionProviders.SUMMONABLE_ENTITIES	Suggests all entities that can be summoned.
SuggestionProviders.AVAILABLE_SOUNDS	Suggests all sounds that can be played.
LootCommand.SUGGESTION_PROVIDER	Suggests all loot tables that are available.
SuggestionProviders.ALL_BIOMES	Suggests all biomes that are available.
Creating a Custom Suggestion Provider
If a built-in provider doesn't satisfy your needs, you can create your own suggestion provider. To do this, you need to create a class that implements the SuggestionProvider interface and override the getSuggestions method.

For this example, we'll make a suggestion provider that suggests all the player usernames on the server.


public class PlayerSuggestionProvider implements SuggestionProvider<ServerCommandSource> {
    @Override
    public CompletableFuture<Suggestions> getSuggestions(CommandContext<ServerCommandSource> context, SuggestionsBuilder builder) throws CommandSyntaxException {
        ServerCommandSource source = context.getSource();

        // Thankfully, the ServerCommandSource has a method to get a list of player names.
        Collection<String> playerNames = source.getPlayerNames();

        // Add all player names to the builder.
        for (String playerName : playerNames) {
            builder.suggest(playerName);
        }

        // Lock the suggestions after we've modified them.
        return builder.buildFuture();
    }
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
To use this suggestion provider, you would simply pass an instance of it into the .suggests method on the argument builder.


CommandRegistrationCallback.EVENT.register((dispatcher, registryAccess, environment) -> {
    dispatcher.register(CommandManager.literal("command_with_custom_suggestions").then(
            CommandManager.argument("player_name", StringArgumentType.string())
                    .suggests(new PlayerSuggestionProvider())
                    .executes(FabricDocsReferenceCommands::executeCommandWithCustomSuggestions)
    ));
});
1
2
3
4
5
6
7

private static int executeCommandWithCustomSuggestions(CommandContext<ServerCommandSource> context) {
    String name = StringArgumentType.getString(context, "player_name");
    context.getSource().sendFeedback(() -> Text.literal("Called /command_with_custom_suggestions with value = %s".formatted(name)), false);
    return 1;
}
1
2
3
4
5
Obviously, suggestion providers can be more complex, since they can also read the command context to provide suggestions based on the command's state - such as the arguments that have already been provided.

This could be in the form of reading the player's inventory and suggesting items, or entities that are nearby the player.


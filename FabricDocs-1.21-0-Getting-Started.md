# Start Here doc

>Taken and summarized from https://wiki.fabricmc.net/start

```
Prerequisites

It's strongly recommended to learn Java before diving into the modding world (especially if you don't have prior programming experience). 
Minecraft, in its codebase, and mods, using Fabric API, use more advanced Java concepts like lambdas, generics and polymorphism. 
If you don't know all of these concepts, you may have some difficulties modding. 
Here are some online resources that will help learning Java.

Basic Java Tutorials: https://www.w3schools.com/java/default.asp

Introduction to Programming using Java by David J. Eck: http://math.hws.edu/javanotes/
```

# Tutorial Start

Let's go over some of the terms that you will encounter when modding with Fabric.

```Mod``` A modification to the game, adding new features or changing existing ones.
Mod Loader A tool that loads mods into the game, such as Fabric Loader.

```Mixin``` A tool for modifying the game's code at runtime - see Mixin Introduction for more information.

```Gradle``` A build automation tool used to build and compile mods, used by Fabric to build mods.

```Mappings``` A set of mappings that map obfuscated code to human-readable code.

```Obfuscation``` The process of making code difficult to understand, used by Mojang to protect Minecraft's code.

```Remapping``` The process of mapping obfuscated code to human-readable code.

# What Is Fabric?
Fabric is a lightweight modding toolchain for Minecraft: Java Edition.
It is designed to be a simple and easy-to-use modding platform. Fabric is a community-driven project, and it is open-source, meaning that anyone can contribute to the project.
You should be aware of the four main components of Fabric:

```Fabric Loader``` A flexible platform-independent mod loader designed for Minecraft and other games and applications.

```Fabric Loom``` A tool that helps developers easily develop and debug mods (using Gradle, a build automation system that manages dependencies and compiles code).

```Fabric API``` A set of APIs and tools for mod developers to use when creating mods.

```Yarn``` A set of open Minecraft mappings, free for everyone to use under the Creative Commons Zero license.

# Why Is Fabric Necessary to Mod Minecraft?
Modding is the process of modifying a game in order to change its behavior or add new features - in the case of Minecraft, this can be anything from adding new items, blocks, or entities, to changing the game's mechanics or adding new game modes.

Minecraft: Java Edition is obfuscated by Mojang, making modification alone difficult. However, with the help of modding tools like Fabric, modding becomes much easier. There are several mapping systems that can assist in this process.

Loom remaps the obfuscated code to a human-readable format using these mappings, making it easier for modders to understand and modify the game's code. Yarn is a popular and excellent mappings choice for this, but other options exist as well. Each mapping project may have its own strengths or focus.

Loom allows you to easily develop and compile mods against remapped code, and Fabric Loader allows you to load these mods into the game.

# What Does Fabric API Provide, and Why Is It Needed?
Fabric API is a set of APIs and tools for mod developers to use when creating mods.

Fabric API provides a wide set of APIs that build on top of Minecraft's existing functionality - for example, providing new hooks and events for modders to use, or providing new utilities and tools to make modding easier - such as transitive access wideners and the ability to access internal registries, such as the compostable items registry.

While Fabric API offers powerful features, some tasks, like basic block registration, can be accomplished without it using the vanilla APIs.

You can use the Fabric Template Mod Generator (https://fabricmc.net/develop/template/) to generate a new project for your mod - you should fill in the required fields, such as the mod name, package name, and the Minecraft version that you want to develop for.

The package name should be lowercase, separated by dots, and unique to avoid conflicts with other programmers' packages. It is typically formatted as a reversed internet domain, such as com.example.mod-id.

# Project Structure
This page will go over the structure of a Fabric mod project, and the purpose of each file and folder in the project.

```fabric.mod.json```
The main file that describes your mod to Fabric Loader. It contains information such as the mod's ID, version, and dependencies.

The most important fields in the fabric.mod.json file are:

```id``` The mod's ID, which should be unique.

```name``` The mod's name.

```environment``` The environment that your mod runs in, such as client, server, or * for both.

```entrypoints``` The entrypoints that your mod provides, such as main or client.

```depends``` The mods that your mod depends on.

```mixins``` The mixins that your mod provides.

# Entrypoints
As mentioned before, the ```fabric.mod.json``` file contains a field called entrypoints - this field is used to specify the entrypoints that your mod provides. The template mod generator creates both a main and client entrypoint by default.

The main entrypoint is used for common code, and it's contained in a class which implements ```ModInitializer```. 
The client entrypoint is used for client-specific code, and its class implements ```ClientModInitializer```.
These entrypoints are called respectively when the game starts.

---

### ```src/main/resources```
The ```src/main/resources``` folder is used to store the resources that your mod uses, such as textures, models, and sounds.

It's also the location of ```fabric.mod.json``` and any mixin configuration files that your mod uses.

Assets are stored in a structure that mirrors the structure of resource packs - for example, a texture for a block would be stored in ```assets/mod-id/textures/block/block.png```.

### ```src/client/resources```
The src/client/resources folder is used to store client-specific resources, such as textures, models, and sounds that are only used on the client side.

### ```src/main/java```
The ```src/main/java``` folder is used to store the Java source code for your mod - it exists on both the client and server environments.

### ```src/client/java```
The ```src/client/java``` folder is used to store client-specific Java source code, such as rendering code or client-side logic - such as block color providers.

# Launching the Game
Fabric Loom provides a variety of launch profiles to help you start and debug your mods in a live game environment. This guide will cover the various launch profiles and how to use them to debug and playtest your mods.

### Launch Profiles
If you're using IntelliJ IDEA, you can find the launch profiles in the top-right corner of the window. Click the dropdown menu to see the available launch profiles.

There should be a client and server profile, with the option to either run it normally or in debug mode:

# Gradle Tasks
If you're using the command line, you can use the following Gradle commands to start the game:

```./gradlew runClient``` Start the game in client mode.

```./gradlew runServer``` Start the game in server mode.

The only problem with this approach is that you can't easily debug your code. If you want to debug your code, you'll need to use the launch profiles in IntelliJ IDEA or via your IDE's Gradle integration.

# Hotswapping Classes
When you run the game in debug mode, you can hotswap your classes without restarting the game. This is useful for quickly testing changes to your code. You're still quite limited though:

- You can't add or remove methods

- You can't change method parameters

- You can't add or remove fields

However, by using the JetBrains Runtime, you are able to circumvent most of the limitations, and even add or remove classes and methods. This should allow most changes to take effect without restarting the game.

Don't forget to add the following to the VM Arguments option in your Minecraft run configuration:

```java
-XX:+AllowEnhancedClassRedefinition
```

# Hotswapping Mixins
If you're using Mixins, you can hotswap your Mixin classes without restarting the game. This is useful for quickly testing changes to your Mixins. You will need to install the Mixin Java agent for this to work though.

1. Locate the Mixin Library Jar
In IntelliJ IDEA, you can find the mixin library jar in the "External Libraries" section of the "Project" section. You will need to copy the jar's "Absolute Path" for the next step.

2. Add the -javaagent VM Argument
In your "Minecraft Client" and or "Minecraft Server" run configuration, add the following to the VM Arguments option:

```
-javaagent:"path to mixin library jar here"
```

Now, you should be able to modify the contents of your mixin methods during debugging and have the changes take effect without restarting the game.
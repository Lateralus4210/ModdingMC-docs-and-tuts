Data Generation Setup

This page is written for version:

1.21.4


Page Authors
ArkoSammy12
Earthcomputer
haykam821
Jab125
jmanc3
matthewperiut
mcrafterzz
modmuss50
Shnupbups
skycatminepokie
SolidBlock-cn
What Is Data Generation?
Data generation (or datagen) is an API for programmatically generating recipes, advancements, tags, item models, language files, loot tables, and basically anything JSON-based.

Enabling Data Generation
At Project Creation
The easiest way to enable datagen is at project creation. Check the "Enable Data Generation" box when using the template generator.

The checked "Data Generation" box on the template generator

TIP

If datagen is enabled, you should have a "Data Generation" run configuration and a runDatagen Gradle task.

Manually
First, we need to enable datagen in the build.gradle file.


fabricApi {
    configureDataGeneration() {
        client = true
    }
}
1
2
3
4
5
Next, we need an entrypoint class. This is where our datagen starts. Place this somewhere in the client package - this example places it at src/client/java/com/example/docs/datagen/FabricDocsReferenceDataGenerator.java.


public class FabricDocsReferenceDataGenerator implements DataGeneratorEntrypoint {
    @Override
    public void onInitializeDataGenerator(FabricDataGenerator fabricDataGenerator) {
    }

}
1
2
3
4
5
6
Finally, we need to tell Fabric about the entrypoint in our fabric.mod.json:


{
  // ...
  "entrypoints": {
    // ...
    "client": [
      // ...
    ],
    "fabric-datagen": [ 
      "com.example.docs.datagen.FabricDocsReferenceDataGenerator"
    ] 
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
WARNING

Don't forget to add a comma (,) after the previous entrypoint block!

Close and reopen IntelliJ to create a run configuration for datagen.

Creating a Pack
Inside your datagen entrypoint's onInitializeDataGenerator method, we need to create a Pack. Later, you'll add providers, which put generated data into this Pack.


FabricDataGenerator.Pack pack = fabricDataGenerator.createPack();
1
Running Data Generation
To run datagen, use the run configuration in your IDE, or run ./gradlew runDatagen in the console. The generated files will be created in src/main/generated.

Next Steps
Now that datagen is set up, we need to add providers. These are what generate the data to add to your Pack. The following pages outline how to do this.

Advancements
Loot Tables
Recipes
Tags
Translations
Edit this page on GitHub


Tag Generation

This page is written for version:

1.21.4


Page Authors
IMB11
mcrafterzz
skycatminepokie
Spinoscythe
PREREQUISITES

Make sure you've completed the datagen setup process first.

Setup
First, create your own class that extends FabricTagProvider<T>, where T is the type of thing you'd like to provide a tag for. This is your provider. Here we'll show how to create Item tags, but the same principle applies for other things. Let your IDE fill in the required code, then replace the registryKey constructor parameter with the RegistryKey for your type:


public class FabricDocsReferenceItemTagProvider extends FabricTagProvider<Item> {
    public FabricDocsReferenceItemTagProvider(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
        super(output, RegistryKeys.ITEM, registriesFuture);
    }

    @Override
    protected void configure(RegistryWrapper.WrapperLookup wrapperLookup) {
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
TIP

You will need a different provider for each type of tag (eg. one FabricTagProvider<EntityType<?>> and one FabricTagProvider<Item>).

To finish setup, add this provider to your DataGeneratorEntrypoint within the onInitializeDataGenerator method.


pack.addProvider(FabricDocsReferenceItemTagProvider::new);
1
Creating a Tag
Now that you've created a provider, let's add a tag to it. First, create a TagKey<T>:


public static final TagKey<Item> SMELLY_ITEMS = TagKey.of(RegistryKeys.ITEM, Identifier.of(FabricDocsReference.MOD_ID, "smelly_items"));
1
Next, call getOrCreateTagBuilder inside your provider's configure method. From there, you can add individual items, add other tags, or make this tag replace pre-existing tags.

If you want to add a tag, use addOptionalTag, as the tag's contents may not be loaded during datagen. If you are certain the tag is loaded, call addTag.

To forcefully add a tag and ignore the broken format, use forceAddTag.


getOrCreateTagBuilder(SMELLY_ITEMS)
        .add(Items.SLIME_BALL)
        .add(Items.ROTTEN_FLESH)
        .addOptionalTag(ItemTags.DIRT)
        .add(Identifier.ofVanilla("oak_planks"))
        .forceAddTag(ItemTags.BANNERS)
        .setReplace(true);

Translation Generation

This page is written for version:

1.21.4


Page Authors
IMB11
jmanc3
MattiDragon
mcrafterzzsjk1949
skycatminepokie
Spinoscythe
PREREQUISITES

Make sure you've completed the datagen setup process first.

Setup
First, we'll make our provider. Remember, providers are what actually generate data for us. Create a class that extends FabricLanguageProvider and fill out the base methods:


public class FabricDocsReferenceEnglishLangProvider extends FabricLanguageProvider {
    protected FabricDocsReferenceEnglishLangProvider(FabricDataOutput dataOutput, CompletableFuture<RegistryWrapper.WrapperLookup> registryLookup) {
        // Specifying en_us is optional, as it's the default language code
        super(dataOutput, "en_us", registryLookup);
    }

    @Override
    public void generateTranslations(RegistryWrapper.WrapperLookup wrapperLookup, TranslationBuilder translationBuilder) {
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
TIP

You will need a different provider for each language you want to generate (eg. one ExampleEnglishLangProvider and one ExamplePirateLangProvider).

To finish setup, add this provider to your DataGeneratorEntrypoint within the onInitializeDataGenerator method.


pack.addProvider(FabricDocsReferenceEnglishLangProvider::new);
1
Creating Translations
Along with creating raw translations, translations from Identifiers, and copying them from an already existing file (by passing a Path), there are helper methods for translating items, blocks, tags, stats, entities, status effects, item groups, entity attributes, and enchantments. Simply call add on the translationBuilder with what you want to translate and what it should translate to:


translationBuilder.add("text.fabric_docs_reference.greeting", "Hello there!");
1
Using Translations
Generated translations take the place of a lot of translations added in other tutorials, but you can also use them anywhere you use a Text object. In our example, if we wanted to allow resource packs to translate our greeting, we use Text.translatable instead of Text.of:


ChatHud chatHud = MinecraftClient.getInstance().inGameHud.getChatHud();
chatHud.addMessage(Text.literal("Hello there!")); 
chatHud.addMessage(Text.translatable("text.fabric_docs_reference.greeting")); 

Advancement Generation

This page is written for version:

1.21.4


Page Authors
jmanc3
MattiDragon
mcrafterzz
skycatminepokie
Spinoscythe
PREREQUISITES

Make sure you've completed the datagen setup process first.

Setup
First, we need to make our provider. Create a class that extends FabricAdvancementProvider and fill out the base methods:


public class FabricDocsReferenceAdvancementProvider extends FabricAdvancementProvider {
    protected FabricDocsReferenceAdvancementProvider(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registryLookup) {
        super(output, registryLookup);
    }

    @Override
    public void generateAdvancement(RegistryWrapper.WrapperLookup wrapperLookup, Consumer<AdvancementEntry> consumer) {
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
To finish setup, add this provider to your DataGeneratorEntrypoint within the onInitializeDataGenerator method.


pack.addProvider(FabricDocsReferenceAdvancementProvider::new);
1
Advancement Structure
An advancement is made up a few different components. Along with the requirements, called "criterion," it may have:

An AdvancementDisplay that tells the game how to show the advancement to players,
AdvancementRequirements, which are lists of lists of criteria, requiring at least one criterion from each sub-list to be completed,
AdvancementRewards, which the player receives for completing the advancement.
A CriterionMerger, which tells the advancement how to handle multiple criterion, and
A parent Advancement, which organizes the hierarchy you see on the "Advancements" screen.
Simple Advancements
Here's a simple advancement for getting a dirt block:


AdvancementEntry getDirt = Advancement.Builder.create()
        .display(
                Items.DIRT, // The display icon
                Text.literal("Your First Dirt Block"), // The title
                Text.literal("Now make a house from it"), // The description
                Identifier.ofVanilla("textures/gui/advancements/backgrounds/adventure.png"), // Background image for the tab in the advancements page, if this is a root advancement (has no parent)
                AdvancementFrame.TASK, // TASK, CHALLENGE, or GOAL
                true, // Show the toast when completing it
                true, // Announce it to chat
                false // Hide it in the advancement tab until it's achieved
        )
        // "got_dirt" is the name referenced by other advancements when they want to have "requirements."
        .criterion("got_dirt", InventoryChangedCriterion.Conditions.items(Items.DIRT))
        // Give the advancement an id
        .build(consumer, FabricDocsReference.MOD_ID + ":get_dirt");
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
WARNING

When building your advancement entries, remember that the function accepts the Identifier of the advancement in String format!

One More Example
Just to get the hang of it, let's add one more advancement. We'll practice adding rewards, using multiple criterion, and assigning parents:


final RegistryWrapper.Impl<Item> itemLookup = wrapperLookup.getOrThrow(RegistryKeys.ITEM);
AdvancementEntry appleAndBeef = Advancement.Builder.create()
        .parent(getDirt)
        .display(
                Items.APPLE,
                Text.literal("Apple and Beef"),
                Text.literal("Ate an apple and beef"),
                null, // Children don't need a background, the root advancement takes care of that
                AdvancementFrame.CHALLENGE,
                true,
                true,
                false
        )
        .criterion("ate_apple", ConsumeItemCriterion.Conditions.item(itemLookup, Items.APPLE))
        .criterion("ate_cooked_beef", ConsumeItemCriterion.Conditions.item(itemLookup, Items.COOKED_BEEF))
        .build(consumer, FabricDocsReference.MOD_ID + ":apple_and_beef");
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
Custom Criteria
WARNING

While datagen can be on the client side, Criterions and Predicates are in the main source set (both sides), since the server needs to trigger and evaluate them.

Definitions
A criterion (plural: criteria) is something a player can do (or that can happen to a player) that may be counted towards an advancement. The game comes with many criteria, which can be found in the net.minecraft.advancement.criterion package. Generally, you'll only need a new criterion if you implement a custom mechanic into the game.

Conditions are evaluated by criteria. A criterion is only counted if all the relevant conditions are met. Conditions are usually expressed with a predicate.

A predicate is something that takes a value and returns a boolean. For example, a Predicate<Item> might return true if the item is a diamond, while a Predicate<LivingEntity> might return true if the entity is not hostile to villagers.

Creating Custom Criteria
First, we'll need a new mechanic to implement. Let's tell the player what tool they used every time they break a block.


public class FabricDocsReferenceDatagenAdvancement implements ModInitializer {
    @Override
    public void onInitialize() {
        HashMap<Item, Integer> tools = new HashMap<>();

        PlayerBlockBreakEvents.AFTER.register(((world, player, blockPos, blockState, blockEntity) -> {
            if (player instanceof ServerPlayerEntity serverPlayer) { // Only triggers on the server side
                Item item = player.getMainHandStack().getItem();

                Integer usedCount = tools.getOrDefault(item, 0);
                usedCount++;
                tools.put(item, usedCount);

                serverPlayer.sendMessage(Text.of("You've used \"" + item + "\" as a tool " + usedCount + " times!"));
            }
        }));
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
Note that this code is really bad. The HashMap is not stored anywhere persistent, so it will be reset every time the game is restarted. It's just to show off Criterions. Start the game and try it out!

Next, let's create our custom criterion, UseToolCriterion. It's going to need its own Conditions class to go with it, so we'll make them both at once:


public class UseToolCriterion extends AbstractCriterion<UseToolCriterion.Conditions> {

    @Override
    public Codec<Conditions> getConditionsCodec() {
        return Conditions.CODEC;
    }

    public record Conditions(Optional<LootContextPredicate> playerPredicate) implements AbstractCriterion.Conditions {
        public static Codec<UseToolCriterion.Conditions> CODEC = LootContextPredicate.CODEC.optionalFieldOf("player")
                .xmap(Conditions::new, Conditions::player).codec();

        @Override
        public Optional<LootContextPredicate> player() {
            return playerPredicate;
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
Whew, that's a lot! Let's break it down.

UseToolCriterion is an AbstractCriterion, which Conditions can apply to.
Conditions has a playerPredicate field. All Conditions should have a player predicate (technically a LootContextPredicate).
Conditions also has a CODEC. This Codec is simply the codec for its one field, playerPredicate, with extra instructions to convert between them (xmap).
INFO

To learn more about codecs, see the Codecs page.

We're going to need a way to check if the conditions are met. Let's add a helper method to Conditions:


public boolean requirementsMet() {
    return true; // AbstractCriterion#trigger helpfully checks the playerPredicate for us.
}
1
2
3
Now that we've got a criterion and its conditions, we need a way to trigger it. Add a trigger method to UseToolCriterion:


public void trigger(ServerPlayerEntity player) {
    trigger(player, Conditions::requirementsMet);
}
1
2
3
Almost there! Next, we need an instance of our criterion to work with. Let's put it in a new class, called ModCriteria.


public class ModCriteria {
    public static final UseToolCriterion USE_TOOL = Criteria.register(FabricDocsReference.MOD_ID + ":use_tool", new UseToolCriterion());
}
1
2
3
To make sure that our criteria are initialized at the right time, add a blank init method:


// :::datagen-advancements:mod-criteria
public static final UseToolCriterion USE_TOOL = Criteria.register(FabricDocsReference.MOD_ID + ":use_tool", new UseToolCriterion());
// :::datagen-advancements:mod-criteria
// :::datagen-advancements:new-mod-criteria
public static final ParameterizedUseToolCriterion PARAMETERIZED_USE_TOOL = Criteria.register(FabricDocsReference.MOD_ID + ":parameterized_use_tool", new ParameterizedUseToolCriterion());

// :::datagen-advancements:mod-criteria
1
2
3
4
5
6
7
And call it in your mod initializer:


ModCriteria.init();
1
Finally, we need to trigger our criteria. Add this to where we sent a message to the player in the main mod class.


ModCriteria.USE_TOOL.trigger(serverPlayer);
1
Your shiny new criterion is now ready to use! Let's add it to our provider:


AdvancementEntry breakBlockWithTool = Advancement.Builder.create()
        .parent(getDirt)
        .display(
                Items.DIAMOND_SHOVEL,
                Text.literal("Not a Shovel"),
                Text.literal("That's not a shovel (probably)"),
                null,
                AdvancementFrame.GOAL,
                true,
                true,
                false
        )
        .criterion("break_block_with_tool", ModCriteria.USE_TOOL.create(new UseToolCriterion.Conditions(Optional.empty())))
        .build(consumer, FabricDocsReference.MOD_ID + ":break_block_with_tool");
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
Run the datagen task again, and you've got your new advancement to play with!

Conditions with Parameters
This is all well and good, but what if we want to only grant an advancement once we've done something 5 times? And why not another one at 10 times? For this, we need to give our condition a parameter. You can stay with UseToolCriterion, or you can follow along with a new ParameterizedUseToolCriterion. In practice, you should only have the parameterized one, but we'll keep both for this tutorial.

Let's work bottom-up. We'll need to check if the requirements are met, so let's edit our Conditions#requirementsMet method:


public boolean requirementsMet(int totalTimes) {
    return totalTimes > requiredTimes; // AbstractCriterion#trigger helpfully checks the playerPredicate for us.
}
1
2
3
requiredTimes doesn't exist, so make it a parameter of Conditions:


    public record Conditions(Optional<LootContextPredicate> playerPredicate, int requiredTimes) implements AbstractCriterion.Conditions {
        @Override
        public Optional<LootContextPredicate> player() {
            return playerPredicate;
        }

        // :::datagen-advancements:new-requirements-met
        public boolean requirementsMet(int totalTimes) {
            return totalTimes > requiredTimes; // AbstractCriterion#trigger helpfully checks the playerPredicate for us.
        }

        // :::datagen-advancements:new-requirements-met
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
Now our codec is erroring. Let's write a new codec for the new changes:


        public static Codec<ParameterizedUseToolCriterion.Conditions> CODEC = RecordCodecBuilder.create(instance -> instance.group(
                LootContextPredicate.CODEC.optionalFieldOf("player").forGetter(Conditions::player),
                Codec.INT.fieldOf("requiredTimes").forGetter(Conditions::requiredTimes)
        ).apply(instance, Conditions::new));
        // :::datagen-advancements:new-parameter
        @Override
        public Optional<LootContextPredicate> player() {
            return playerPredicate;
        }

        // :::datagen-advancements:new-requirements-met
        public boolean requirementsMet(int totalTimes) {
            return totalTimes > requiredTimes; // AbstractCriterion#trigger helpfully checks the playerPredicate for us.
        }

        // :::datagen-advancements:new-requirements-met
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
Moving on, we now need to fix our trigger method:


public void trigger(ServerPlayerEntity player, int totalTimes) {
    trigger(player, conditions -> conditions.requirementsMet(totalTimes));
}
1
2
3
If you've made a new criterion, we need to add it to ModCriteria


public static final ParameterizedUseToolCriterion PARAMETERIZED_USE_TOOL = Criteria.register(FabricDocsReference.MOD_ID + ":parameterized_use_tool", new ParameterizedUseToolCriterion());

// :::datagen-advancements:mod-criteria
// :::datagen-advancements:mod-criteria-init
public static void init() {
}
1
2
3
4
5
6
And call it in our main class, right where the old one is:


ModCriteria.PARAMETERIZED_USE_TOOL.trigger(serverPlayer, usedCount);
1
Add the advancement to your provider:


AdvancementEntry breakBlockWithToolFiveTimes = Advancement.Builder.create()
        .parent(breakBlockWithTool)
        .display(
                Items.GOLDEN_SHOVEL,
                Text.literal("Not a Shovel Still"),
                Text.literal("That's still not a shovel (probably)"),
                null,
                AdvancementFrame.GOAL,
                true,
                true,
                false
        )
        .criterion("break_block_with_tool_five_times", ModCriteria.PARAMETERIZED_USE_TOOL.create(new ParameterizedUseToolCriterion.Conditions(Optional.empty(), 5)))
        .build(consumer, FabricDocsReference.MOD_ID + ":break_block_with_tool_five_times");
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
Run datagen again, and you're finally done!

Recipe Generation

This page is written for version:

1.21.4


Page Authors
jmanc3mcrafterzz
skycatminepokie
Spinoscythe
PREREQUISITES

Make sure you've completed the datagen setup process first.

Setup
First, we'll need our provider. Make a class that extends FabricRecipeProvider. All our recipe generation will happen inside the generate method of our provider.


import java.util.List;
import java.util.concurrent.CompletableFuture;

import net.minecraft.data.recipe.RecipeExporter;
import net.minecraft.data.recipe.RecipeGenerator;
import net.minecraft.item.Item;
import net.minecraft.item.Items;
import net.minecraft.recipe.Ingredient;
import net.minecraft.recipe.book.RecipeCategory;
import net.minecraft.registry.RegistryKeys;
import net.minecraft.registry.RegistryWrapper;
import net.minecraft.registry.tag.ItemTags;

import net.fabricmc.fabric.api.datagen.v1.FabricDataOutput;
import net.fabricmc.fabric.api.datagen.v1.provider.FabricRecipeProvider;

public class FabricDocsReferenceRecipeProvider extends FabricRecipeProvider {
    public FabricDocsReferenceRecipeProvider(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
        super(output, registriesFuture);
    }

    @Override
    protected RecipeGenerator getRecipeGenerator(RegistryWrapper.WrapperLookup registryLookup, RecipeExporter exporter) {
        return new RecipeGenerator(registryLookup, exporter) {
            @Override
            public void generate() {
                RegistryWrapper.Impl<Item> itemLookup = registries.getOrThrow(RegistryKeys.ITEM);
            }
        };
    }

    @Override
    public String getName() {
        return "FabricDocsReferenceRecipeProvider";
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
31
32
33
34
35
36
To finish setup, add this provider to your DataGeneratorEntrypoint within the onInitializeDataGenerator method.


pack.addProvider(FabricDocsReferenceRecipeProvider::new);
1
Shapeless Recipes
Shapeless recipes are fairly straightforward. Just add them to the generate method in your provider:



createShapeless(RecipeCategory.BUILDING_BLOCKS, Items.DIRT) // You can also specify an int to produce more than one
        .input(Items.COARSE_DIRT) // You can also specify an int to require more than one, or a tag to accept multiple things
        // Create an advancement that gives you the recipe
        .criterion(hasItem(Items.COARSE_DIRT), conditionsFromItem(Items.COARSE_DIRT))
        .offerTo(exporter);
1
2
3
4
5
6
Shaped Recipes
For a shaped recipe, you define the shape using a String, then define what each char in the String represents.


createShaped(RecipeCategory.MISC, Items.CRAFTING_TABLE, 4)
        .pattern("ll")
        .pattern("ll")
        .input('l', ItemTags.LOGS) // 'l' means "any log"
        .group("multi_bench") // Put it in a group called "multi_bench" - groups are shown in one slot in the recipe book
        .criterion(hasItem(Items.CRAFTING_TABLE), conditionsFromItem(Items.CRAFTING_TABLE))
        .offerTo(exporter);
createShaped(RecipeCategory.MISC, Items.LOOM, 4)
        .pattern("ww")
        .pattern("ll")
        .input('w', ItemTags.WOOL) // 'w' means "any wool"
        .input('l', ItemTags.LOGS)
        .group("multi_bench")
        .criterion(hasItem(Items.LOOM), conditionsFromItem(Items.LOOM))
        .offerTo(exporter);
createDoorRecipe(Items.OAK_DOOR, Ingredient.ofItems(Items.OAK_BUTTON)) // Using a helper method!
        .criterion(hasItem(Items.OAK_BUTTON), conditionsFromItem(Items.OAK_BUTTON))
        .offerTo(exporter);
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
TIP

There's a lot of helper methods for creating common recipes. Check out what RecipeProvider has to offer! Use Alt + 7 in IntelliJ to open the structure of a class, including a method list.

Other Recipes
Other recipes work similarly, but require a few extra parameters. For example, smelting recipes need to know how much experience to award.


offerSmelting(
        List.of(Items.BREAD, Items.COOKIE, Items.HAY_BLOCK), // Inputs
        RecipeCategory.FOOD, // Category
        Items.WHEAT, // Output
        0.1f, // Experience
        300, // Cooking time
        "food_to_wheat" // group
);
1
2
3
4
5
6
7
8
Custom Recipe Types

Loot Table Generation

This page is written for version:

1.21.4


Page Authors
Alphagamer47
jmanc3
JustinHuPrime
matthewperiut
mcrafterzz
skycatminepokie
Spinoscythe
PREREQUISITES

Make sure you've completed the datagen setup process first.

You will need different providers (classes) for blocks, chests, and entities. Remember to add them all to your pack in your DataGeneratorEntrypoint within the onInitializeDataGenerator method.


pack.addProvider(FabricDocsReferenceBlockLootTableProvider::new);
pack.addProvider(FabricDocsReferenceChestLootTableProvider::new);
1
2
Loot Tables Explained
Loot tables define what you get from breaking a block (not including contents, like in chests), killing an entity, or opening a newly-generated container. Each loot table has pools from which items are selected. Loot tables also have functions, which modify the resulting loot in some way.

Loot pools have entries, conditions, functions, rolls, and bonus rolls. Entries are groups, sequences, or possibilities of items, or just items. Conditions are things that are tested for in the world, such as enchantments on a tool or a random chance. The minimum number of entries chosen by a pool are called rolls, and anything over that is called a bonus roll.

Blocks
In order for blocks to drop items - including itself - we need to make a loot table. Create a class that extends FabricBlockLootTableProvider:


public class FabricDocsReferenceBlockLootTableProvider extends FabricBlockLootTableProvider {
    protected FabricDocsReferenceBlockLootTableProvider(FabricDataOutput dataOutput, CompletableFuture<RegistryWrapper.WrapperLookup> registryLookup) {
        super(dataOutput, registryLookup);
    }

    @Override
    public void generate() {
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
Make sure to add this provider to your pack!

There's a lot of helper methods available to help you build your loot tables. We won't go over all of them, so make sure to check them out in your IDE.

Let's add a few drops in the generate method:


// Make condensed dirt drop its block item.
// Also adds the condition that it survives the explosion that broke it, if applicable,
addDrop(ModBlocks.CONDENSED_DIRT);
// Make prismarine lamps drop themselves with silk touch only
addDropWithSilkTouch(ModBlocks.PRISMARINE_LAMP);
// Make condensed oak logs drop between 7 and 9 oak logs
addDrop(ModBlocks.CONDENSED_OAK_LOG, LootTable.builder().pool(addSurvivesExplosionCondition(Items.OAK_LOG, LootPool.builder()
        .rolls(new UniformLootNumberProvider(new ConstantLootNumberProvider(7), new ConstantLootNumberProvider(9)))
        .with(ItemEntry.builder(Items.OAK_LOG))))
);
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
Chests
Chest loot is a little bit tricker than block loot. Create a class that extends SimpleFabricLootTableProvider similar to the example below and add it to your pack.


public class FabricDocsReferenceChestLootTableProvider extends SimpleFabricLootTableProvider {
    public FabricDocsReferenceChestLootTableProvider(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registryLookup) {
        super(output, registryLookup, LootContextTypes.CHEST);
    }

    @Override
    public void accept(BiConsumer<RegistryKey<LootTable>, LootTable.Builder> lootTableBiConsumer) {
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
We'll need a RegistryKey<LootTable> for our loot table. Let's put that in a new class called ModLootTables. Make sure this is in your main source set if you're using split sources.


public class ModLootTables {
    public static RegistryKey<LootTable> TEST_CHEST_LOOT = RegistryKey.of(RegistryKeys.LOOT_TABLE, Identifier.of(FabricDocsReference.MOD_ID, "chests/test_loot"));
}
1
2
3
Then, we can generate a loot table inside the generate method of your provider.


lootTableBiConsumer.accept(ModLootTables.TEST_CHEST_LOOT, LootTable.builder()
        .pool(LootPool.builder() // One pool
                .rolls(ConstantLootNumberProvider.create(2.0f)) // That has two rolls
                .with(ItemEntry.builder(Items.DIAMOND) // With an entry that has diamond(s)
                        .apply(SetCountLootFunction.builder(ConstantLootNumberProvider.create(1.0f)))) // One diamond
                .with(ItemEntry.builder(Items.DIAMOND_SWORD) // With an entry that has a plain diamond sword
                )
        ));

Block Model Generation

This page is written for version:

1.21.4


Page Authors
Fellteros
IMB11
its-miroma
natri0
PREREQUISITES

Make sure you've completed the datagen setup process first.

Setup
First, we will need to create our ModelProvider. Create a class that extends FabricModelProvider. Implement both abstract methods: generateBlockStateModels and generateItemModels. Lastly, create a constructor matching super.


public class FabricDocsReferenceModelProvider extends FabricModelProvider {
    public FabricDocsReferenceModelProvider(FabricDataOutput output) {
        super(output);
    }

    @Override
    public void generateBlockStateModels(BlockStateModelGenerator blockStateModelGenerator) {
    }


    @Override
    public void generateItemModels(ItemModelGenerator itemModelGenerator) {
    }

    @Override
    public String getName() {
        return "FabricDocsReference Model Provider";
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
Register this class in your DataGeneratorEntrypoint within the onInitializeDataGenerator method.

Blockstates and Block Models

@Override
public void generateBlockStateModels(BlockStateModelGenerator blockStateModelGenerator) {
}
1
2
3
For block models, we will primarily be focusing on the generateBlockStateModels method. Notice the parameter BlockStateModelGenerator blockStateModelGenerator - this object will be responsible for generating all the JSON files. Here are some handy examples you can use to generate your desired models:

Simple Cube All

blockStateModelGenerator.registerSimpleCubeAll(ModBlocks.STEEL_BLOCK);
1
This is the most commonly used function. It generates a JSON model file for a normal cube_all block model. One texture is used for all six sides, in this case we use steel_block.


{
  "parent": "minecraft:block/cube_all",
  "textures": {
    "all": "fabric-docs-reference:block/steel_block"
  }
}
1
2
3
4
5
6
It also generates a blockstate JSON file. Since we have no blockstate properties (e.g. Axis, Facing, ...), one variant is sufficient, and is used every time the block is placed.


{
  "variants": {
    "": {
      "model": "fabric-docs-reference:block/steel_block"
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

Download Steel Block
Singletons
The registerSingleton method provides JSON model files based on the TexturedModel you pass in and a single blockstate variant.


blockStateModelGenerator.registerSingleton(ModBlocks.PIPE_BLOCK, TexturedModel.END_FOR_TOP_CUBE_COLUMN);
1
This method will generate models for a normal cube, that uses the texture file pipe_block for the sides and the texture file pipe_block_top for the top and bottom sides.


{
  "parent": "minecraft:block/cube_column",
  "textures": {
    "end": "fabric-docs-reference:block/pipe_block_top",
    "side": "fabric-docs-reference:block/pipe_block"
  }
}
1
2
3
4
5
6
7
TIP

If you're stuck choosing which TextureModel you should use, open the TexturedModel class and look at the TextureMaps!


Download Pipe Block
Block Texture Pool

blockStateModelGenerator.registerCubeAllModelTexturePool(ModBlocks.RUBY_BLOCK)
        .stairs(ModBlocks.RUBY_STAIRS)
        .slab(ModBlocks.RUBY_SLAB)
        .fence(ModBlocks.RUBY_FENCE);
1
2
3
4
Another useful method is registerCubeAllModelTexturePool: define the textures by passing in the "base block", and then append the "children", which will have the same textures. In this case, we passed in the RUBY_BLOCK, so the stairs, slab and fence will use the RUBY_BLOCK texture.

WARNING

It will also generate a simple cube all JSON model for the "base block" to ensure that it has a block model.

Be aware of this, if you're changing block model of this particular block, as it will result in en error.

You can also append a BlockFamily, which will generate models for all of its "children".


public static final BlockFamily RUBY_FAMILY =
        new BlockFamily.Builder(ModBlocks.RUBY_BLOCK)
        .stairs(ModBlocks.RUBY_STAIRS)
        .slab(ModBlocks.RUBY_SLAB)
        .fence(ModBlocks.RUBY_FENCE)
        .build();
1
2
3
4
5
6

blockStateModelGenerator.registerCubeAllModelTexturePool(ModBlocks.RUBY_BLOCK).family(ModBlocks.RUBY_FAMILY);
1

Download Ruby Block
Doors and Trapdoors

blockStateModelGenerator.registerDoor(ModBlocks.RUBY_DOOR);
blockStateModelGenerator.registerTrapdoor(ModBlocks.RUBY_TRAPDOOR);
// blockStateModelGenerator.registerOrientableTrapdoor(ModBlocks.RUBY_TRAPDOOR);
1
2
3
Doors and trapdoors are a little different. Here, you have to make three new textures - two for the door, and one for the trapdoor.

The door:
It has two parts - the upper half and the lower half. Each needs its own texture: in this case ruby_door_top for the upper half and ruby_door_bottom for the lower.
The registerDoor() method will create models for all orientations of the door, both open and closed.
You also need an item texture! Put it in assets/mod_id/textures/item/ folder.
The trapdoor:
Here, you need only one texture, in this case named ruby_trapdoor. It will be used for all sides.
Since TrapdoorBlock has a FACING property, you can use the commented out method to generate model files with rotated textures = the trapdoor will be "orientable". Otherwise, it will look the same no matter the direction it's facing.

Download Ruby Door and Trapdoor
Custom Block Models
In this section, we'll create the models for a Vertical Oak Log Slab, with Oak Log textures.

Points 2. - 6. are declared in an inner static helper class called CustomBlockStateModelGenerator.

Custom Block Class
Create a VerticalSlab block with a FACING property and a SINGLE boolean property, like in the Block States tutorial. SINGLE will indicate if there are both slabs. Then you should override getOutlineShape and getCollisionShape, so that the outline is rendered correctly, and the block has the correct collision shape.


public static final VoxelShape NORTH_SHAPE = Block.createCuboidShape(0.0, 0.0, 0.0, 16.0, 16.0, 8.0);
public static final VoxelShape SOUTH_SHAPE = Block.createCuboidShape(0.0, 0.0, 8.0, 16.0, 16.0, 16.0);
public static final VoxelShape WEST_SHAPE = Block.createCuboidShape(0.0, 0.0, 0.0, 8.0, 16.0, 16.0);
public static final VoxelShape EAST_SHAPE = Block.createCuboidShape(8.0, 0.0, 0.0, 16.0, 16.0, 16.0);
1
2
3
4

@Override
protected VoxelShape getSidesShape(BlockState state, BlockView world, BlockPos pos) {
    boolean type = state.get(SINGLE);
    Direction direction = state.get(FACING);
    VoxelShape voxelShape;

    if (type) {
        switch (direction) {
            case WEST -> voxelShape = WEST_SHAPE.asCuboid();
            case EAST -> voxelShape = EAST_SHAPE.asCuboid();
            case SOUTH -> voxelShape = SOUTH_SHAPE.asCuboid();
            case NORTH -> voxelShape = NORTH_SHAPE.asCuboid();
            default -> throw new MatchException(null, null);
        }

        return voxelShape;
    } else {
        return VoxelShapes.fullCube();
    }
}

@Override
protected VoxelShape getOutlineShape(BlockState state, BlockView world, BlockPos pos, ShapeContext context) {
    return this.getSidesShape(state, world, pos);
}

@Override
protected VoxelShape getCollisionShape(BlockState state, BlockView world, BlockPos pos, ShapeContext context) {
    return this.getSidesShape(state, world, pos);
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
Also override the canReplace() method, otherwise you couldn't make the slab a full block.


@Override
protected boolean canReplace(BlockState state, ItemPlacementContext context) {
    Direction direction = state.get(FACING);

    if (context.getStack().isOf(this.asItem()) && state.get(SINGLE)) {
        if (context.canReplaceExisting()) {
            return context.getSide().getOpposite() == direction;
        }
    }

    return false;
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
And you're done! You can now test the block out and place it in game.

Parent Block Model
Now, let's create a parent block model. It will determine the size, position in hand or other slots and the x and y coordinates of the texture. It's recommended to use an editor such as Blockbench for this, as making it manually is a really tedious process. It should look something like this:


{
  "parent": "minecraft:block/block",
  "textures": {
    "particle": "#side"
  },
  "display": {
    "gui": {
      "rotation": [
        30,
        -135,
        0
      ],
      "translation": [
        -1.5,
        0.75,
        0
      ],
      "scale": [
        0.625,
        0.625,
        0.625
      ]
    },
    "firstperson_righthand": {
      "rotation": [
        0,
        -45,
        0
      ],
      "translation": [
        0,
        2,
        0
      ],
      "scale": [
        0.375,
        0.375,
        0.375
      ]
    },
    "firstperson_lefthand": {
      "rotation": [
        0,
        315,
        0
      ],
      "translation": [
        0,
        2,
        0
      ],
      "scale": [
        0.375,
        0.375,
        0.375
      ]
    },
    "thirdperson_righthand": {
      "rotation": [
        75,
        -45,
        0
      ],
      "translation": [
        0,
        0,
        2
      ],
      "scale": [
        0.375,
        0.375,
        0.375
      ]
    },
    "thirdperson_lefthand": {
      "rotation": [
        75,
        315,
        0
      ],
      "translation": [
        0,
        0,
        2
      ],
      "scale": [
        0.375,
        0.375,
        0.375
      ]
    }
  },
  "elements": [
    {
      "from": [
        0,
        0,
        0
      ],
      "to": [
        16,
        16,
        8
      ],
      "faces": {
        "down": {
          "uv": [
            0,
            8,
            16,
            16
          ],
          "texture": "#bottom",
          "cullface": "down",
          "tintindex": 0
        },
        "up": {
          "uv": [
            0,
            0,
            16,
            8
          ],
          "texture": "#top",
          "cullface": "up",
          "tintindex": 0
        },
        "north": {
          "uv": [
            0,
            0,
            16,
            16
          ],
          "texture": "#side",
          "cullface": "north",
          "tintindex": 0
        },
        "south": {
          "uv": [
            0,
            0,
            16,
            16
          ],
          "texture": "#side",
          "tintindex": 0
        },
        "west": {
          "uv": [
            0,
            0,
            8,
            16
          ],
          "texture": "#side",
          "cullface": "west",
          "tintindex": 0
        },
        "east": {
          "uv": [
            8,
            0,
            16,
            16
          ],
          "texture": "#side",
          "cullface": "east",
          "tintindex": 0
        }
      }
    }
  ]
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
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
See how blockstates are formatted for more information. Notice the #bottom, #top, #side keywords. They act as variables that can be set by models that have this one as a parent:


{
  "parent": "minecraft:block/cube_bottom_top",
  "textures": {
    "bottom": "minecraft:block/sandstone_bottom",
    "side": "minecraft:block/sandstone",
    "top": "minecraft:block/sandstone_top"
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
The bottom value will replace the #bottom placeholder and so on. Put it in the resources/assets/mod_id/models/block/ folder.

Custom Model
Another thing we will need is an instance of the Model class. It will represent the actual parent block model inside our mod.


public static final Model VERTICAL_SLAB = block("vertical_slab", TextureKey.BOTTOM, TextureKey.TOP, TextureKey.SIDE);

//helper method for creating Models
private static Model block(String parent, TextureKey... requiredTextureKeys) {
    return new Model(Optional.of(Identifier.of(FabricDocsReference.MOD_ID, "block/" + parent)), Optional.empty(), requiredTextureKeys);
}

//helper method for creating Models with variants
private static Model block(String parent, String variant, TextureKey... requiredTextureKeys) {
    return new Model(Optional.of(Identifier.of(FabricDocsReference.MOD_ID, "block/" + parent)), Optional.of(variant), requiredTextureKeys);
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
The block() method creates a new Model, pointing to the vertical_slab.json file inside the resources/assets/mod_id/models/block/ folder. The TextureKeys represent the "placeholders" (#bottom, #top, ...) as an Object.

Using Texture Map
What does TextureMap do? It actually provides the Identifiers that point to the textures. It technically behaves like a normal map - you associate a TextureKey (Key) with an Identifier (Value).

You can either use the vanilla ones, like TextureMap.all()(which associates all TextureKeys with the same Identifier), or create a new one, by creating a new instance and then using .put() to associate keys with values.

TIP

TextureMap.all() associates all TextureKeys with the same Identifier, no matter how many of them there are!

Since we want to use the Oak Log textures, but have the BOTTOM, TOP and SIDE TextureKeys, we need to create a new one.


public static TextureMap blockAndTopForEnds(Block block) {
    return new TextureMap()
            .put(TextureKey.TOP, ModelIds.getBlockSubModelId(block, "_top"))
            .put(TextureKey.BOTTOM, ModelIds.getBlockSubModelId(block, "_top"))
            .put(TextureKey.SIDE, ModelIds.getBlockModelId(block));
}
1
2
3
4
5
6
The bottom and top faces will use oak_log_top.png, the sides will use oak_log.png.

WARNING

All TextureKeys in the TextureMap have to match all TextureKeys in your parent block model!

Custom BlockStateSupplier Method
The BlockStateSupplier contains all blockstate variants, their rotation, and other options like uvlock.


private static BlockStateSupplier createVerticalSlabBlockStates(Block vertSlabBlock, Identifier vertSlabId, Identifier fullBlockId) {
    VariantSetting<Boolean> uvlock = VariantSettings.UVLOCK;
    VariantSetting<VariantSettings.Rotation> yRot = VariantSettings.Y;
    return VariantsBlockStateSupplier.create(vertSlabBlock).coordinate(BlockStateVariantMap.create(VerticalSlabBlock.FACING, VerticalSlabBlock.SINGLE)
            .register(Direction.NORTH, true, BlockStateVariant.create().put(VariantSettings.MODEL, vertSlabId).put(uvlock, true))
            .register(Direction.EAST, true, BlockStateVariant.create().put(VariantSettings.MODEL, vertSlabId).put(uvlock, true).put(yRot, VariantSettings.Rotation.R90))
            .register(Direction.SOUTH, true, BlockStateVariant.create().put(VariantSettings.MODEL, vertSlabId).put(uvlock, true).put(yRot, VariantSettings.Rotation.R180))
            .register(Direction.WEST, true, BlockStateVariant.create().put(VariantSettings.MODEL, vertSlabId).put(uvlock, true).put(yRot, VariantSettings.Rotation.R270))
            .register(Direction.NORTH, false, BlockStateVariant.create().put(VariantSettings.MODEL, fullBlockId).put(uvlock, true))
            .register(Direction.EAST, false, BlockStateVariant.create().put(VariantSettings.MODEL, fullBlockId).put(uvlock, true))
            .register(Direction.SOUTH, false, BlockStateVariant.create().put(VariantSettings.MODEL, fullBlockId).put(uvlock, true))
            .register(Direction.WEST, false, BlockStateVariant.create().put(VariantSettings.MODEL, fullBlockId).put(uvlock, true)));
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
First, we create a new VariantsBlockStateSupplier using VariantsBlockStateSupplier.create(). Then we create a new BlockStateVariantMap that contains parameters for all variants of the block, in this case FACING and SINGLE, and pass it into the VariantsBlockStateSupplier. Specify which model and which transformations (uvlock, rotation) is used when using .register(). For example:

On the first line, the block is facing north, and is single => we use the model with no rotation.
On the fourth line, the block is facing west, and is single => we rotate the model on the Y axis by 270°.
On the sixth line, the block is facing east, but isn't single => it looks like a normal oak log => we don't have to rotate it.
Custom Datagen Method
The last step - creating an actual method you can call and that will generate the JSONs. But what are the parameters for?

BlockStateModelGenerator generator, the same one that got passed into generateBlockStateModels.
Block vertSlabBlock is the block to which we will generate the JSONs.
Block fullBlock - is the model used when the SINGLE property is false = the slab block looks like a full block.
TextureMap textures defines the actual textures the model uses. See the Using Texture Map chapter.

public static void registerVerticalSlab(BlockStateModelGenerator generator, Block vertSlabBlock, Block fullBlock, TextureMap textures) {
    Identifier slabModel = VERTICAL_SLAB.upload(vertSlabBlock, textures, generator.modelCollector);
    Identifier fullBlockModel = ModelIds.getBlockModelId(fullBlock);
    generator.blockStateCollector.accept(createVerticalSlabBlockStates(vertSlabBlock, slabModel, fullBlockModel));
    generator.registerParentedItemModel(vertSlabBlock, slabModel);
}
1
2
3
4
5
6
First, we get the Identifier of the single slab model with VERTICAL_SLAB.upload(). Then we get the Identifier of the full block model with ModelIds.getBlockModelId(), and pass those two models into createVerticalSlabBlockStates. The BlockStateSupplier gets passed into the blockStateCollector, so that the JSON files are actually generated. Also, we create a model for the vertical slab item with BlockStateModelGenerator.registerParentedItemModel().

And that is all! Now all that's left to do is to call our method in our ModelProvider:


CustomBlockStateModelGenerator.registerVerticalSlab(
        blockStateModelGenerator,
        ModBlocks.VERTICAL_OAK_LOG_SLAB,
        Blocks.OAK_LOG,
        CustomBlockStateModelGenerator.blockAndTopForEnds(Blocks.OAK_LOG)
);
1
2
3
4
5
6
Sources and Links
You can view the example tests in Fabric API and this documentation's Reference Mod for more information.

You can also find more examples of using custom datagen methods by browsing mods' open-source code, for example Vanilla+ Blocks and Vanilla+ Verticals by Fellteros.


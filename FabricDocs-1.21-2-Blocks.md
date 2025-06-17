Creating Your First Block

This page is written for version:

1.21.4


Page Authors
Earthcomputer
IMB11
its-miroma
xEobardThawne
Blocks are the building blocks of Minecraft (no pun intended) - just like everything else in Minecraft, they're stored in registries.

Preparing Your Blocks Class
If you've completed the Creating Your First Item page, this process will feel extremely familiar - you will need to create a method that registers your block, and its block item.

You should put this method in a class called ModBlocks (or whatever you want to name it).

Mojang does something extremely similar like this with vanilla blocks; you can refer to the Blocks class to see how they do it.


public class ModBlocks {
    private static Block register(String name, Function<AbstractBlock.Settings, Block> blockFactory, AbstractBlock.Settings settings, boolean shouldRegisterItem) {
        // Create a registry key for the block
        RegistryKey<Block> blockKey = keyOfBlock(name);
        // Create the block instance
        Block block = blockFactory.apply(settings.registryKey(blockKey));

        // Sometimes, you may not want to register an item for the block.
        // Eg: if it's a technical block like `minecraft:moving_piston` or `minecraft:end_gateway`
        if (shouldRegisterItem) {
            // Items need to be registered with a different type of registry key, but the ID
            // can be the same.
            RegistryKey<Item> itemKey = keyOfItem(name);

            BlockItem blockItem = new BlockItem(block, new Item.Settings().registryKey(itemKey));
            Registry.register(Registries.ITEM, itemKey, blockItem);
        }

        return Registry.register(Registries.BLOCK, blockKey, block);
    }

    private static RegistryKey<Block> keyOfBlock(String name) {
        return RegistryKey.of(RegistryKeys.BLOCK, Identifier.of(FabricDocsReference.MOD_ID, name));
    }

    private static RegistryKey<Item> keyOfItem(String name) {
        return RegistryKey.of(RegistryKeys.ITEM, Identifier.of(FabricDocsReference.MOD_ID, name));
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
Just like with items, you need to ensure that the class is loaded so that all static fields containing your block instances are initialized.

You can do this by creating a dummy initialize method, which can be called in your mod's initializer to trigger the static initialization.

INFO

If you are unaware of what static initialization is, it is the process of initializing static fields in a class. This is done when the class is loaded by the JVM, and is done before any instances of the class are created.


public class ModBlocks {
    // ...

    public static void initialize() {}
}
1
2
3
4
5

public class FabricDocsReferenceBlocks implements ModInitializer {
    @Override
    public void onInitialize() {
        ModBlocks.initialize();
    }
}
1
2
3
4
5
6
Creating And Registering Your Block
Similarly to items, blocks take a AbstractBlock.Settings class in their constructor, which specifies properties about the block, such as its sound effects and mining level.

We will not cover all the options here: you can view the class yourself to see the various options, which should be self-explanatory.

For example purposes, we will be creating a simple block that has the properties of dirt, but is a different material.

We create our block settings in a similar way to how we created item settings in the item tutorial.
We tell the register method to create a Block instance from the block settings by calling the Block constructor.
TIP

You can also use AbstractBlock.Settings.copy(AbstractBlock block) to copy the settings of an existing block, in this case, we could have used Blocks.DIRT to copy the settings of dirt, but for example purposes we'll use the builder.


public static final Block CONDENSED_DIRT = register(
        "condensed_dirt",
        Block::new,
        AbstractBlock.Settings.create().sounds(BlockSoundGroup.GRASS),
        true
);
1
2
3
4
5
6
To automatically create the block item, we can pass true to the shouldRegisterItem parameter of the register method we created in the previous step.

Adding Your Block's Item to an Item Group
Since the BlockItem is automatically created and registered, to add it to an item group, you must use the Block.asItem() method to get the BlockItem instance.

For this example, we'll use a custom item group created in the Custom Item Groups page.


ItemGroupEvents.modifyEntriesEvent(ModItems.CUSTOM_ITEM_GROUP_KEY).register((itemGroup) -> {
    itemGroup.add(ModBlocks.CONDENSED_DIRT.asItem());
});
1
2
3
You should place this within the initialize() function of your class.

You should now notice that your block is in the creative inventory, and can be placed in the world!

Block in world without suitable model or texture

There are a few issues though - the block item is not named, and the block has no texture, block model or item model.

Adding Block Translations
To add a translation, you must create a translation key in your translation file - assets/mod-id/lang/en_us.json.

Minecraft will use this translation in the creative inventory and other places where the block name is displayed, such as command feedback.


{
  "block.mod_id.condensed_dirt": "Condensed Dirt"
}
1
2
3
You can either restart the game or build your mod and press F3+T to apply changes - and you should see that the block has a name in the creative inventory and other places such as the statistics screen.

Models and Textures
All block textures can be found in the assets/mod-id/textures/block folder - an example texture for the "Condensed Dirt" block is free to use.


Download Texture
To make the texture show up in-game, you must create a block model which can be found in the assets/mod-id/models/block/condensed_dirt.json file for the "Condensed Dirt" block. For this block, we're going to use the block/cube_all model type.


{
  "parent": "minecraft:block/cube_all",
  "textures": {
    "all": "fabric-docs-reference:block/condensed_dirt"
  }
}
1
2
3
4
5
6
For the block to show in your inventory, you will need to create an Item Model Description that points to your block model. For this example, the item model description for the "Condensed Dirt" block can be found at assets/mod-id/items/condensed_dirt.json.


{
  "model": {
    "type": "minecraft:model",
    "model": "fabric-docs-reference:block/condensed_dirt"
  }
}
1
2
3
4
5
6
TIP

You only need to create an item model description if you've registered a BlockItem along with your block!

When you load into the game, you may notice that the texture is still missing. This is because you need to add a blockstate definition.

Creating the Block State Definition
The blockstate definition is used to instruct the game on which model to render based on the current state of the block.

For the example block, which doesn't have a complex blockstate, only one entry is needed in the definition.

This file should be located in the assets/mod-id/blockstates folder, and its name should match the block ID used when registering your block in the ModBlocks class. For instance, if the block ID is condensed_dirt, the file should be named condensed_dirt.json.


{
  "variants": {
    "": {
      "model": "fabric-docs-reference:block/condensed_dirt"
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
TIP

Blockstates are incredibly complex, which is why they will be covered next in their own separate page.

Restarting the game, or reloading via F3+T to apply changes - you should be able to see the block texture in the inventory and physically in the world:

Block in world with suitable texture and model

Adding Block Drops
When breaking the block in survival, you may see that the block does not drop - you might want this functionality, however to make your block drop as an item on break you must implement its loot table - the loot table file should be placed in the data/mod-id/loot_table/blocks/ folder.

INFO

For a greater understanding of loot tables, you can refer to the Minecraft Wiki - Loot Tables page.


{
  "type": "minecraft:block",
  "pools": [
    {
      "rolls": 1,
      "entries": [
        {
          "type": "minecraft:item",
          "name": "fabric-docs-reference:condensed_dirt"
        }
      ],
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ]
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
This loot table provides a single item drop of the block item when the block is broken, and when it is blown up by an explosion.

Recommending a Harvesting Tool
You may also want your block to be harvestable only by a specific tool - for example, you may want to make your block faster to harvest with a shovel.

All the tool tags should be placed in the data/minecraft/tags/block/mineable/ folder - where the name of the file depends on the type of tool used, one of the following:

hoe.json
axe.json
pickaxe.json
shovel.json
The contents of the file are quite simple - it is a list of items that should be added to the tag.

This example adds the "Condensed Dirt" block to the shovel tag.


{
  "replace": false,
  "values": ["fabric-docs-reference:condensed_dirt"]
}
1
2
3
4
If you wish for a tool to be required to mine the block, you'll want to append .requiresTool() to your block settings, as well as add the appropriate mining level tag.

Mining Levels
Similarly, the mining level tag can be found in the data/minecraft/tags/block/ folder, and respects the following format:

needs_stone_tool.json - A minimum level of stone tools
needs_iron_tool.json - A minimum level of iron tools
needs_diamond_tool.json - A minimum level of diamond tools.
The file has the same format as the harvesting tool file - a list of items to be added to the tag.

Extra Notes
If you're adding multiple blocks to your mod, you may want to consider using Data Generation to automate the process of creating block and item models, blockstate definitions, and loot tables.

Block States

This page is written for version:

1.21.4


Page Authors
IMB11
A block state is a piece of data attached to a singular block in the Minecraft world containing information on the block in the form of properties - some examples of properties vanilla stores in block states:

Rotation: Mostly used for logs and other natural blocks.
Activated: Heavily used in redstone devices, and blocks such as the furnace or smoker.
Age: Used in crops, plants, saplings, kelp etc.
You can probably see why they are useful - they avoid the need to store NBT data in a block entity - reducing the world size, and preventing TPS issues!

Blockstate definitions are found in the assets/mod-id/blockstates folder.

Example: Pillar Block
Minecraft has some custom classes already that allow you quickly create certain types of blocks - this example goes through the creation of a block with the axis property by creating a "Condensed Oak Log" block.

The vanilla PillarBlock class allows the block to be placed in the X, Y or Z axis.


public static final Block CONDENSED_OAK_LOG = register(
        "condensed_oak_log",
        PillarBlock::new,
        AbstractBlock.Settings.create().sounds(BlockSoundGroup.WOOD),
        true
);
1
2
3
4
5
6
Pillar blocks have two textures, top and side - they use the block/cube_column model.

As always, with all block textures, the texture files can be found in assets/mod-id/textures/block


Download Textures
Since the pillar block has two positions, horizontal and vertical, we'll need to make two separate model files:

condensed_oak_log_horizontal.json which extends the block/cube_column_horizontal model.
condensed_oak_log.json which extends the block/cube_column model.
An example of the condensed_oak_log_horizontal.json file:


{
  "parent": "minecraft:block/cube_column_horizontal",
  "textures": {
    "end": "fabric-docs-reference:block/condensed_oak_log_top",
    "side": "fabric-docs-reference:block/condensed_oak_log"
  }
}
1
2
3
4
5
6
7
INFO

Remember, blockstate files can be found in the assets/mod-id/blockstates folder, the name of the blockstate file should match the block ID used when registering your block in the ModBlocks class. For instance, if the block ID is condensed_oak_log, the file should be named condensed_oak_log.json.

For a more in-depth look at all the modifiers available in the blockstate files, check out the Minecraft Wiki - Models (Block States) page.

Next, we need to create a blockstate file, which is where the magic happens. Pillar blocks have three axes, so we'll use specific models for the following situations:

axis=x - When the block is placed along the X axis, we will rotate the model to face the positive X direction.
axis=y - When the block is placed along the Y axis, we will use the normal vertical model.
axis=z - When the block is placed along the Z axis, we will rotate the model to face the positive X direction.

{
  "variants": {
    "axis=x": {
      "model": "fabric-docs-reference:block/condensed_oak_log_horizontal",
      "x": 90,
      "y": 90
    },
    "axis=y": {
      "model": "fabric-docs-reference:block/condensed_oak_log"
    },
    "axis=z": {
      "model": "fabric-docs-reference:block/condensed_oak_log_horizontal",
      "x": 90
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
As always, you'll need to create a translation for your block, and an item model which parents either of the two models.

Example of Pillar block in-game

Custom Block States
Custom block states are great if your block has unique properties - sometimes you may find that your block can re-use vanilla properties.

This example will create a unique boolean property called activated - when a player right-clicks on the block, the block will go from activated=false to activated=true - changing its texture accordingly.

Creating The Property
Firstly, you'll need to create the property itself - since this is a boolean, we'll use the BooleanProperty.of method.


public class PrismarineLampBlock extends Block {
    public static final BooleanProperty ACTIVATED = BooleanProperty.of("activated");

}
1
2
3
4
Next, we have to append the property to the blockstate manager in the appendProperties method. You'll need to override the method to access the builder:


@Override
protected void appendProperties(StateManager.Builder<Block, BlockState> builder) {
    builder.add(ACTIVATED);
}
1
2
3
4
You'll also have to set a default state for the activated property in the constructor of your custom block.


public PrismarineLampBlock(Settings settings) {
    super(settings);

    // Set the default state of the block to be deactivated.
    setDefaultState(getDefaultState().with(ACTIVATED, false));
}
1
2
3
4
5
6
Using The Property
This example flips the boolean activated property when the player interacts with the block. We can override the onUse method for this:


@Override
protected ActionResult onUse(BlockState state, World world, BlockPos pos, PlayerEntity player, BlockHitResult hit) {
    if (!player.getAbilities().allowModifyWorld) {
        // Skip if the player isn't allowed to modify the world.
        return ActionResult.PASS;
    } else {
        // Get the current value of the "activated" property
        boolean activated = state.get(ACTIVATED);

        // Flip the value of activated and save the new blockstate.
        world.setBlockState(pos, state.with(ACTIVATED, !activated));

        // Play a click sound to emphasise the interaction.
        world.playSound(player, pos, SoundEvents.BLOCK_COMPARATOR_CLICK, SoundCategory.BLOCKS, 1.0F, 1.0F);

        return ActionResult.SUCCESS;
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
Visualizing The Property
Before creating the blockstate file, you will need to provide textures for both the activated and deactivated states of the block, as well as the block model.


Download Textures
Use your knowledge of block models to create two models for the block: one for the activated state and one for the deactivated state. Once you've done that, you can begin creating the blockstate file.

Since you created a new property, you will need to update the blockstate file for the block to account for that property.

If you have multiple properties on a block, you'll need to account for all possible combinations. For example, activated and axis would lead to 6 combinations (two possible values for activated and three possible values for axis).

Since this block only has two possible variants, as it only has one property (activated), the blockstate JSON will look something like this:


{
  "multipart": [
    {
      "apply": {
        "model": "fabric-docs-reference:block/prismarine_lamp_on"
      },
      "when": {
        "activated": "true"
      }
    },
    {
      "apply": {
        "model": "fabric-docs-reference:block/prismarine_lamp"
      },
      "when": {
        "activated": "false"
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
TIP

Don't forget to add an Item Model Description for the block so that it will show in the inventory!

Since the example block is a lamp, we also need to make it emit light when the activated property is true. This can be done through the block settings passed to the constructor when registering the block.

You can use the luminance method to set the light level emitted by the block, we can create a static method in the PrismarineLampBlock class to return the light level based on the activated property, and pass it as a method reference to the luminance method:


public static int getLuminance(BlockState currentBlockState) {
    // Get the value of the "activated" property.
    boolean activated = currentBlockState.get(PrismarineLampBlock.ACTIVATED);

    // Return a light level if activated = true
    return activated ? 15 : 0;
}
1
2
3
4
5
6
7

public static final Block PRISMARINE_LAMP = register(
        "prismarine_lamp",
        PrismarineLampBlock::new,
        AbstractBlock.Settings.create()
                .sounds(BlockSoundGroup.LANTERN)
                .luminance(PrismarineLampBlock::getLuminance),
        true
);
1
2
3
4
5
6
7
8
Once you've completed everything, the final result should look something like the following:

Block Entities

This page is written for version:

1.21.4


Page Authors
natri0
Block entities are a way to store additional data for a block, that is not part of the block state: inventory contents, custom name and so on. Minecraft uses block entities for blocks like chests, furnaces, and command blocks.

As an example, we will create a block that counts how many times it has been right-clicked.

Creating the Block Entity
To make Minecraft recognize and load the new block entities, we need to create a block entity type. This is done by extending the BlockEntity class and registering it in a new ModBlockEntities class.


public class CounterBlockEntity extends BlockEntity {
    public CounterBlockEntity(BlockPos pos, BlockState state) {
        super(ModBlockEntities.COUNTER_BLOCK_ENTITY, pos, state);
    }

}
1
2
3
4
5
6
Registering a BlockEntity yields a BlockEntityType like the COUNTER_BLOCK_ENTITY we've used above:


public static final BlockEntityType<CounterBlockEntity> COUNTER_BLOCK_ENTITY =
        register("counter", CounterBlockEntity::new, ModBlocks.COUNTER_BLOCK);

private static <T extends BlockEntity> BlockEntityType<T> register(
        String name,
        FabricBlockEntityTypeBuilder.Factory<? extends T> entityFactory,
        Block... blocks
) {
    Identifier id = Identifier.of(FabricDocsReference.MOD_ID, name);
    return Registry.register(Registries.BLOCK_ENTITY_TYPE, id, FabricBlockEntityTypeBuilder.<T>create(entityFactory, blocks).build());
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
TIP

Note how the constructor of the CounterBlockEntity takes two parameters, but the BlockEntity constructor takes three: the BlockEntityType, the BlockPos, and the BlockState. If we didn't hard-code the BlockEntityType, the ModBlockEntities class wouldn't compile! This is because the BlockEntityFactory, which is a functional interface, describes a function that only takes two parameters, just like our constructor.

Creating the Block
Next, to actually use the block entity, we need a block that implements BlockEntityProvider. Let's create one and call it CounterBlock.

TIP

There's two ways to approach this:

create a block that extends BlockWithEntity and implement the createBlockEntity method
create a block that implements BlockEntityProvider by itself and override the createBlockEntity method
We'll use the first approach in this example, since BlockWithEntity also provides some nice utilities.


public class CounterBlock extends BlockWithEntity {
    public CounterBlock(Settings settings) {
        super(settings);
    }

    @Override
    protected MapCodec<? extends BlockWithEntity> getCodec() {
        return createCodec(CounterBlock::new);
    }

    @Nullable
    @Override
    public BlockEntity createBlockEntity(BlockPos pos, BlockState state) {
        return new CounterBlockEntity(pos, state);
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
Using BlockWithEntity as the parent class means we also need to implement the createCodec method, which is rather easy.

Unlike blocks, which are singletons, a new block entity is created for every instance of the block. This is done with the createBlockEntity method, which takes the position and BlockState, and returns a BlockEntity, or null if there shouldn't be one.

Don't forget to register the block in the ModBlocks class, just like in the Creating Your First Block guide:


public static final Block COUNTER_BLOCK = register(
        "counter_block",
        CounterBlock::new,
        AbstractBlock.Settings.create(),
        true
);
1
2
3
4
5
6
Using the Block Entity
Now that we have a block entity, we can use it to store the number of times the block has been right-clicked. We'll do this by adding a clicks field to the CounterBlockEntity class:


private int clicks = 0;
public int getClicks() {
    return clicks;
}

public void incrementClicks() {
    clicks++;
    markDirty();
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
The markDirty method, used in incrementClicks, tells the game that this entity's data has been updated; this will be useful when we add the methods to serialize the counter and load it back from the save file.

Next, we need to increment this field every time the block is right-clicked. This is done by overriding the onUse method in the CounterBlock class:


@Override
protected ActionResult onUse(BlockState state, World world, BlockPos pos, PlayerEntity player, BlockHitResult hit) {
    if (!(world.getBlockEntity(pos) instanceof CounterBlockEntity counterBlockEntity)) {
        return super.onUse(state, world, pos, player, hit);
    }

    counterBlockEntity.incrementClicks();
    player.sendMessage(Text.literal("You've clicked the block for the " + counterBlockEntity.getClicks() + "th time."), true);

    return ActionResult.SUCCESS;
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
Since the BlockEntity is not passed into the method, we use world.getBlockEntity(pos), and if the BlockEntity is not valid, return from the method.

"You've clicked the block for the 6th time" message on screen after right-clicking

Saving and Loading Data
Now that we have a functional block, we should make it so that the counter doesn't reset between game restarts. This is done by serializing it into NBT when the game saves, and deserializing when it's loading.

Serialization is done with the writeNbt method:


@Override
protected void writeNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup registryLookup) {
    nbt.putInt("clicks", clicks);

    super.writeNbt(nbt, registryLookup);
}
1
2
3
4
5
6
Here, we add the fields that should be saved into the passed NbtCompound: in the case of the counter block, that's the clicks field.

Reading is similar, but instead of saving to the NbtCompound you get the values you saved previously, and save them in the BlockEntity's fields:


@Override
protected void readNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup registryLookup) {
    super.readNbt(nbt, registryLookup);

    clicks = nbt.getInt("clicks");
}
1
2
3
4
5
6
Now, if we save and reload the game, the counter block should continue from where it left off when saved.

While writeNbt and readNbt handle saving and loading to and from disk, there is still an issue:

The server knows the correct clicks value.
The client does not receive the correct value when loading a chunk.
To fix this, we override toInitialChunkDataNbt:


@Override
public NbtCompound toInitialChunkDataNbt(RegistryWrapper.WrapperLookup registryLookup) {
    return createNbt(registryLookup);
}
1
2
3
4
Now, when a player logs in or moves into a chunk where the block exists, they will see the correct counter value right away.

Tickers
The BlockEntityProvider interface also defines a method called getTicker, which can be used to run code every tick for each instance of the block. We can implement that by creating a static method that will be used as the BlockEntityTicker:

The getTicker method should also check if the passed BlockEntityType is the same as the one we're using, and if it is, return the function that will be called every tick. Thankfully, there is a utility function that does the check in BlockWithEntity:


@Nullable
@Override
public <T extends BlockEntity> BlockEntityTicker<T> getTicker(World world, BlockState state, BlockEntityType<T> type) {
    return validateTicker(type, ModBlockEntities.COUNTER_BLOCK_ENTITY, CounterBlockEntity::tick);
}
1
2
3
4
5
CounterBlockEntity::tick is a reference to the static method tick we should create in the CounterBlockEntity class. Structuring it like this is not required, but it's a good practice to keep the code clean and organized.

Let's say we want to make it so that the counter can only be incremented once every 10 ticks (2 times a second). We can do this by adding a ticksSinceLast field to the CounterBlockEntity class, and increasing it every tick:


public static void tick(World world, BlockPos blockPos, BlockState blockState, CounterBlockEntity entity) {
    entity.ticksSinceLast++;
}
1
2
3
Don't forget to serialize and deserialize this field!

Now we can use ticksSinceLast to check if the counter can be increased in incrementClicks:


if (ticksSinceLast < 10) return;
ticksSinceLast = 0;
1
2
TIP

If the block entity does not seem to tick, try checking the registration code! It should pass the blocks that are valid for this entity into the BlockEntityType.Builder, or else it will give a warning in the console:


[13:27:55] [Server thread/WARN] (Minecraft) Block entity fabric-docs-reference:counter @ BlockPos{x=-29, y=125, z=18} state Block{fabric-docs-reference:counter_block} invalid for ticking:
1
Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Block Entity Renderers

This page is written for version:

1.21.4


Page Authors
natri0
Sometimes, using Minecraft's model format is not enough. If you need to add dynamic rendering to your block's visuals, you will need to use a BlockEntityRenderer.

For example, let's make the Counter Block from the Block Entities article show the number of clicks on its top side.

Creating a BlockEntityRenderer
First, we need to create a BlockEntityRenderer for our CounterBlockEntity.

When creating a BlockEntityRenderer for the CounterBlockEntity, it's important to place the class in the appropriate source set, such as src/client/, if your project uses split source sets for client and server. Accessing rendering-related classes directly in the src/main/ source set is not safe because those classes might be loaded on a server.


public class CounterBlockEntityRenderer implements BlockEntityRenderer<CounterBlockEntity> {
    public CounterBlockEntityRenderer(BlockEntityRendererFactory.Context context) {
    }

    @Override
    public void render(CounterBlockEntity entity, float tickDelta, MatrixStack matrices, VertexConsumerProvider vertexConsumers, int light, int overlay) {
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
The new class has a constructor with BlockEntityRendererFactory.Context as a parameter. The Context has a few useful rendering utilities, like the ItemRenderer or TextRenderer. Also, by including a constructor like this, it becomes possible to use the constructor as the BlockEntityRendererFactory functional interface itself:


public class FabricDocsBlockEntityRenderer implements ClientModInitializer {
    @Override
    public void onInitializeClient() {
        BlockEntityRendererFactories.register(ModBlockEntities.COUNTER_BLOCK_ENTITY, CounterBlockEntityRenderer::new);
    }
}
1
2
3
4
5
6
You should register block entity renderers in your ClientModInitializer class.

BlockEntityRendererFactories is a registry that maps each BlockEntityType with custom rendering code to its respective BlockEntityRenderer.

Drawing on Blocks
Now that we have a renderer, we can draw. The render method is called every frame, and it's where the rendering magic happens.

Moving Around
First, we need to offset and rotate the text so that it's on the block's top side.

INFO

As the name suggests, the MatrixStack is a stack, meaning that you can push and pop transformations. A good rule-of-thumb is to push a new one at the beginning of the render method and pop it at the end, so that the rendering of one block doesn't affect others.

More information about the MatrixStack can be found in the Basic Rendering Concepts article.

To make the translations and rotations needed easier to understand, let's visualize them. In this picture, the green block is where the text would be drawn, by default in the furthest bottom-left point of the block:

Default rendering position

So first we need to move the text halfway across the block on the X and Z axes, and then move it up to the top of the block on the Y axis:

Green block in the topmost center point

This is done with a single translate call:


matrices.translate(0.5, 1, 0.5);
1
That's the translation done, rotation and scale remain.

By default, the text is drawn on the XY plane, so we need to rotate it 90 degrees around the X axis to make it face upwards on the XZ plane:

Green block in the topmost center point, facing upwards

The MatrixStack does not have a rotate function, instead we need to use multiply and RotationAxis.POSITIVE_X:


matrices.multiply(RotationAxis.POSITIVE_X.rotationDegrees(90));
1
Now the text is in the correct position, but it's too large. The BlockEntityRenderer maps the whole block to a [-0.5, 0.5] cube, while the TextRenderer uses Y coordinates of [0, 9]. As such, we need to scale it down by a factor of 18:


matrices.scale(1/18f, 1/18f, 1/18f);
1
Now, the whole transformation looks like this:


matrices.push();
matrices.translate(0.5, 1, 0.5);
matrices.multiply(RotationAxis.POSITIVE_X.rotationDegrees(90));
matrices.scale(1/18f, 1/18f, 1/18f);
1
2
3
4
Drawing Text
As mentioned earlier, the Context passed into the constructor of our renderer has a TextRenderer that we can use to draw text. For this example we'll save it in a field.

The TextRenderer has methods to measure text (getWidth), which is useful for centering, and to draw it (draw).


String text = entity.getClicks() + "";
float width = textRenderer.getWidth(text);

// draw the text. params:
// text, x, y, color, shadow, matrix, vertexConsumers, layerType, backgroundColor, light
textRenderer.draw(
        text,
        -width/2, -4f,
        0xffffff,
        false,
        matrices.peek().getPositionMatrix(),
        vertexConsumers,
        TextRenderer.TextLayerType.SEE_THROUGH,
        0,
        light
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
11
12
13
14
15
16
The draw method takes a lot of parameters, but the most important ones are:

the Text (or String) to draw;
its x and y coordinates;
the RGB color value;
the Matrix4f describing how it should be transformed (to get one from a MatrixStack, we can use .peek().getPositionMatrix() to get the Matrix4f for the topmost entry).
And after all this work, here's the result:

Counter Block with a number on top
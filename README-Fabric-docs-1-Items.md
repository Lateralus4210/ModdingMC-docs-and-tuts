# Creating Your First Item
This page will demonstrate some key concepts relating to items, and how you can register, texture, model and name them.

If you aren't aware, everything in Minecraft is stored in registries, and items are no exception to that.

### Preparing Your Items Class
To simplify the registering of items, you can create a method that accepts a string identifier, some item settings and a factory (a type of function) to create the Item instance.
This method will create an item with the provided identifier and register it with the game's item registry.
You can put this method in a class called ModItems (or whatever you want to name the class). 
Mojang does this with their items as well! Check out the Items class for inspiration.

```json
src/main/java
└── com
    └── yourname
        └── yourmodid
            └── ModItems.java
```
```java
public class ModItems {
    public static Item register(String name, Function<Item.Settings, Item> itemFactory, Item.Settings settings) {
        // Create the item key.
        RegistryKey<Item> itemKey = RegistryKey.of(RegistryKeys.ITEM, Identifier.of(FabricDocsReference.MOD_ID, name));

        // Create the item instance.
        Item item = itemFactory.apply(settings.registryKey(itemKey));

        // Register the item.
        Registry.register(Registries.ITEM, itemKey, item);

        return item;
    }
}
```

Notice the usage of a Function interface for the factory, which will later allow us to specify how we want our item to be created from the item settings using ```Item::new```.


### Registering an Item

You can now register an item using the method now. 
The register method takes in an instance of the ```Item.Settings``` class as a parameter. 
This class allows you to configure the item's properties through various builder methods. 

If you want to change your item's stack size, you can use the maxCount method in the ```Item.Settings``` class.
This will not work if you've marked the item as damageable, as the stack size is always 1 for damageable items to prevent duplication exploits.

```java
public static final Item SUSPICIOUS_SUBSTANCE = register("suspicious_substance", Item::new, new Item.Settings());
```

```Item::new``` tells the register function to create an Item instance from an ```Item.Settings``` by calling the Item constructor ```(new Item(...))```, which takes an ```Item.Settings``` as a parameter.
However, if you now try to run the modified client, you can see that our item doesn't exist in-game yet! This is because you didn't statically initialize the class.
To do this, you can add a public static initialize method to your class and call it from your mod's initializer class. Currently, this method doesn't need anything inside it.
```java
public static void initialize() {
}
```
Calling a method on a class statically initializes it if it hasn't been previously loaded - this means that all static fields are evaluated. This is what this dummy initialize method is for.
```java
public class FabricDocsReferenceItems implements ModInitializer {
	@Override
	public void onInitialize() {
		ModItems.initialize();
	}
}
```


### Adding the Item to an Item Group

If you want to add the item to a custom ItemGroup, check out the Custom Item Groups page for more information.
For example purposes, we will add this item to the ingredients ItemGroup, you will need to use Fabric API's item group events - specifically ```ItemGroupEvents.modifyEntriesEvent```.
This can be done in the initialize method of your items class.

```java
// Get the event for modifying entries in the ingredients group.
// And register an event handler that adds our suspicious item to the ingredients group.
ItemGroupEvents.modifyEntriesEvent(ItemGroups.INGREDIENTS).register((itemGroup) -> itemGroup.add(ModItems.SUSPICIOUS_SUBSTANCE));
```

Loading into the game, you can see that our item has been registered, and is in the Ingredients item group. However, it's missing the following: Item Model, Texture, and Translation (name)

### Naming The Item
The item currently doesn't have a translation, so you will need to add one. The translation key has already been provided by Minecraft: ```item.mod_id.suspicious_substance```.

Create a new JSON file at: 
```json
src/
└── main/
    └── resources/
        └── assets/
            └── mod-id/
                └── lang/
                    └── en_us.json
```
 and put in the translation key, and its value:


```java
{
  "item.mod_id.suspicious_substance": "Suspicious Substance"
}
```

You can either restart the game or build your mod and press F3+T to apply changes.

### Adding a Texture and Model
To give your item a texture and model, simply create a 16x16 texture image for your item and save it in 
```json
assets/
└── mod-id/
    └── textures/
        └── item
``` 
Name the texture file the same as the item's identifier, but with a .png extension. When restarting/reloading the game,  you should see that the item still has no texture - you will need to add a model that uses this texture. You're going to create a simple item/generated model, which takes in an input texture and nothing else. Create the model JSON in 
```json
assets/
└── mod-id/
    └── models/
        └── item
            └── suspicious_substance.json
```
```json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "fabric-docs-reference:item/suspicious_substance"
  }
}
```

This is the parent model that this model will inherit from. In this case, it's the item/generated model.
```Textures``` is where you define the textures for the model. The ```layer0``` key is the texture that the model will use. Most items will use the item/generated model as their parent, as it's a simple model that just displays the texture. There are alternatives, such as item/handheld which is used for items that are "held" in the player's hand, such as tools.

### Creating the Item Model Description
Minecraft doesn't automatically know where your items' model files can be found, so we need to provide an item model description. Create the item description as ```suspicious_substance.json``` in 
```json
assets
└── mod-id
    └── items
```

```json
{
  "model": {
    "type": "minecraft:model",
    "model": "fabric-docs-reference:item/suspicious_substance"
  }
}
```
```Type``` is the type of our model. For most items, this should be ```minecraft:model```.

```Model``` is the model's identifier. It should have this form: ```mod-id:item/item_name```. Fabric API provides various registries that can be used to add additional properties to your item, like making it compostable or a fuel. For example, if you want to make your item compostable, you can use the ```CompostableItemRegistry```:


```java
// Add the suspicious substance to the composting registry with a 30% chance of increasing the composter's level.
CompostingChanceRegistry.INSTANCE.add(ModItems.SUSPICIOUS_SUBSTANCE, 0.3f);
```

Alternatively, if you want to make your item a fuel, you can use the ```FuelRegistryEvents.BUILD``` event:


```java
// add the suspicious substance to the registry of fuels, with a burn time of 30 seconds.
// Remember, Minecraft deals with logical based-time using ticks.
// 20 ticks = 1 second.
FuelRegistryEvents.BUILD.register((builder, context) -> {builder.add{ModItems.SUSPICIOUS_SUBSTANCE, 30 * 20}});
```

### Adding a Basic Crafting Recipe
If you want to add a crafting recipe for your item, you will need to place a recipe JSON file in the ```data/mod-id/recipe folder```.

For more information on the recipe format, check out these resources:

Generate a recipe .json > https://crafting.thedestruc7i0n.ca/

Recipe wiki, including the JSON format > https://minecraft.wiki/w/Recipe#JSON_Format

### Custom Tooltips
If you want your item to have a custom tooltip, you will need to create a class that extends Item and overrides the ```appendTooltip``` method.

This example uses the LightningStick class created in the Custom Item Interactions page.

```java
@Override
public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
    tooltip.add(Text.translatable("itemTooltip.fabric-docs-reference.lightning_stick").formatted(Formatting.GOLD));
}
```

Each call to ```add()``` will add one line to the tooltip.

# Food Items
Food is a core aspect of survival Minecraft, so when creating edible items you have to consider the food's usage with other edible items. Unless you're making a mod with overpowered items, you should consider:

- How much hunger your edible item adds or removes.

- What potion effect(s) does it grant?

- Is it early-game or endgame accessible?

### Adding the Food Component

To add a food component to an item, we can pass it to the ```Item.Settings``` instance:

```java
new Item.Settings().food(new FoodComponent.Builder().build())
```

Right now, this just makes the item edible and nothing more. The ```FoodComponent.Builder``` class has some methods that allow you to modify what happens when a player eats your item:

```nutrition```	sets the amount of hunger points your item will replenish.
```saturationModifier```	sets the amount of saturation points your item will add.
```alwaysEdible```	allows your item to be eaten regardless of hunger level.

When you've modified the builder to your liking, you can call the ```build()``` method to get the ```FoodComponent```.

If you want to add status effects to the player when they eat your food, you will need to use the ```ConsumableComponent``` alongside the ```FoodComponent``` class as seen in the following example:

```java
public static final ConsumableComponent POISON_FOOD_CONSUMABLE_COMPONENT = ConsumableComponents.food()
        // The duration is in ticks, 20 ticks = 1 second
        .consumeEffect(new ApplyEffectsConsumeEffect(new StatusEffectInstance(StatusEffects.POISON, 6 * 20, 1), 1.0f))
        .build();
public static final FoodComponent POISON_FOOD_COMPONENT = new FoodComponent.Builder()
        .alwaysEdible()
        .build();
```

Similar to the example in the Creating Your First Item page, I'll be using the above component:

```java
public static final Item POISONOUS_APPLE = register(
        "poisonous_apple",
        Item::new,
        new Item.Settings().food(POISON_FOOD_COMPONENT, POISON_FOOD_CONSUMABLE_COMPONENT)
);
```

This makes the item:

Always edible; it can be eaten regardless of hunger level.
Also, it always gives Poison II for 6 seconds when eaten.

# Tools and Weapons
Tools are essential for survival and progression, allowing players to gather resources, construct buildings, and defend themselves.

### Creating a Tool Material
You can create a tool material by instantiating a new ```ToolMaterial``` object and storing it in a field that can be used later to create the tool items that use the material.

```java
public static final ToolMaterial GUIDITE_TOOL_MATERIAL = new ToolMaterial(
        BlockTags.INCORRECT_FOR_WOODEN_TOOL,
        455,
        5.0F,
        1.5F,
        22,
        GuiditeArmorMaterial.REPAIRS_GUIDITE_ARMOR
);
```

The ```ToolMaterial``` constructor accepts the following parameters, in this specific order:

```incorrectBlocksForDrops```	If a block is in the incorrectBlocksForDrops tag, it means that when you use a tool made from this ToolMaterial on that block, the block will not drop any items.

```durability```	The durability of all tools that are of this ToolMaterial.

```speed```	The mining speed of the tools that are of this ToolMaterial.

```attackDamageBonus```	The additional attack damage of the tools that are of this ToolMaterial will have.

```enchantmentValue```	The "Enchantability" of tools which are of this ToolMaterial.

```repairItems```	Any items within this tag can be used to repair tools of this ToolMaterial in an anvil.

If you're struggling to determine balanced values for any of the numerical parameters, you should consider looking at the vanilla tool material constants, such as ```ToolMaterial.STONE``` or ```ToolMaterial.DIAMOND```.

### Creating Tool Items
Using the same utility function as in the Creating Your First Item guide, you can create your tool items:

```java
public static final Item GUIDITE_SWORD = register(
        "guidite_sword",
        settings -> new SwordItem(GUIDITE_TOOL_MATERIAL, 1f, 1f, settings),
        new Item.Settings()
);
```

The two float values (1f, 1f) refer to the attack damage of the tool and the attack speed of the tool respectively.

Remember to add them to an item group if you want to access them from the creative inventory!

```java
ItemGroupEvents.modifyEntriesEvent(ItemGroups.TOOLS)
        .register((itemGroup) -> itemGroup.add(ModItems.GUIDITE_SWORD));
```

You will also have to add a texture, item translation and item model. However, for the item model, you'll want to use the item/handheld model as your parent instead of the usual item/generated.

For this example, I will be using the following model and texture for the "Guidite Sword" item:

```json
{
  "parent": "minecraft:item/handheld",
  "textures": {
    "layer0": "fabric-docs-reference:item/guidite_sword"
  }
}
```

# Custom Armor

### Creating an Armor Materials Class
Technically, you don't need a dedicated class for your armor material, but it's good practice anyways with the number of static fields you will need.

For this example, we'll create an ```GuiditeArmorMaterial``` class to store our static fields.

```Base Durability```
This constant will be used in the ```Item.Settings#maxDamage(int damageValue)``` method when creating our armor items. It is also required as a parameter in the ArmorMaterial constructor when we create our ArmorMaterial object later.

```java
public static final int BASE_DURABILITY = 15;
```

If you're struggling to determine a balanced base durability, you can refer to the vanilla armor material instances found in the ArmorMaterials interface.

Equipment Asset Registry Key
Even though we don't have to register our ArmorMaterial to any registries, it's generally good practice to store any registry keys as constants, as the game will use this to find the relevant textures for our armor.

```java
public static final RegistryKey<EquipmentAsset> GUIDITE_ARMOR_MATERIAL_KEY = RegistryKey.of(EquipmentAssetKeys.REGISTRY_KEY, Identifier.of(FabricDocsReference.MOD_ID, "guidite"));
```

We will pass this to the ArmorMaterial constructor later.

ArmorMaterial Instance
To create our material, we need to create a new instance of the ArmorMaterial record, the base durability and material registry key constants will be used here.

```java
public static final ArmorMaterial INSTANCE = new ArmorMaterial(
        BASE_DURABILITY,
        Map.of(
                EquipmentType.HELMET, 3,
                EquipmentType.CHESTPLATE, 8,
                EquipmentType.LEGGINGS, 6,
                EquipmentType.BOOTS, 3
        ),
        5,
        SoundEvents.ITEM_ARMOR_EQUIP_IRON,
        0.0F,
        0.0F,
        REPAIRS_GUIDITE_ARMOR,
        GUIDITE_ARMOR_MATERIAL_KEY
);
```

The ArmorMaterial constructor accepts the following parameters, in this specific order

durability	The base durability of all armor pieces, this is used when calculating the total durability of each individual armor piece that use this material. This should be the base durability constant you created earlier.
defense	A map of EquipmentType (an enum representing each armor slot) to an integer value, which indicates the defense value of the material when used in the corresponding armor slot.
enchantmentValue	The "enchantability" of armor items which use this material.
equipSound	A registry entry of a sound event that is played when you equip a piece of armor which uses this material. For more information on sounds, check out the Custom Sounds page.
toughness	A float value which represents the "toughness" attribute of the armor material - essentially how well the armor will absorb damage.
knockbackResistance	A float value which represents the amount of knockback resistance the armor material grants the wearer.
repairIngredient	An item tag that represents all items which can be used to repair the armor items of this material in an anvil.
assetId	An EquipmentAsset registry key, this should be the equipment asset registry key constant you created earlier.
If you're struggling to determine values for any of the parameters, you can consult the vanilla ArmorMaterial instances which can be found in the ArmorMaterials interface.

Creating the Armor Items
Now that you've registered the material, you can create the armor items in your ModItems class:

Obviously, an armor set doesn't need every type to be satisfied, you can have a set with just boots, or leggings etc. - the vanilla turtle shell helmet is a good example of an armor set with missing slots.

Unlike ToolMaterial, ArmorMaterial does not store any information about the durability of items. For this reason the base durability needs to be manually added to the armor items' Item.Settings when registering them.

This is achieved by passing the BASE_DURABILITY constant we created previously into the maxDamage method in the Item.Settings class.

```java
public static final Item GUIDITE_HELMET = register(
        "guidite_helmet",
        settings -> new ArmorItem(GuiditeArmorMaterial.INSTANCE, EquipmentType.HELMET, settings),
        new Item.Settings().maxDamage(EquipmentType.HELMET.getMaxDamage(GuiditeArmorMaterial.BASE_DURABILITY))
);
public static final Item GUIDITE_CHESTPLATE = register("guidite_chestplate",
        settings -> new ArmorItem(GuiditeArmorMaterial.INSTANCE, EquipmentType.CHESTPLATE, settings),
        new Item.Settings().maxDamage(EquipmentType.CHESTPLATE.getMaxDamage(GuiditeArmorMaterial.BASE_DURABILITY))
);

public static final Item GUIDITE_LEGGINGS = register(
        "guidite_leggings",
        settings -> new ArmorItem(GuiditeArmorMaterial.INSTANCE, EquipmentType.LEGGINGS, settings),
        new Item.Settings().maxDamage(EquipmentType.LEGGINGS.getMaxDamage(GuiditeArmorMaterial.BASE_DURABILITY))
);

public static final Item GUIDITE_BOOTS = register(
        "guidite_boots",
        settings -> new ArmorItem(GuiditeArmorMaterial.INSTANCE, EquipmentType.BOOTS, settings),
        new Item.Settings().maxDamage(EquipmentType.BOOTS.getMaxDamage(GuiditeArmorMaterial.BASE_DURABILITY))
);
```

You will also need to add the items to an item group if you want them to be accessible from the creative inventory.

As with all items, you should create translation keys for them as well.

Textures and Models
You will need to create a set of textures for the items, and a set of textures for the actual armour when it's worn by a "humanoid" entity (players, zombies, skeletons, etc).

Item Textures and Model
These textures are no different to other items - you must create the textures, and create a generic generated item model - which was covered in the Creating Your First Item guide.

INFO

You will need model JSON files for all the items, not just the helmet, it's the same principle as other item models.

```json
{
  "parent": "minecraft:item/generated",
  "textures": {
    "layer0": "fabric-docs-reference:item/guidite_helmet"
  }
}
```

Armor Textures
When an entity wears your armor, nothing will be shown. This is because you're missing textures and the equipment model definitions.

There are two layers for the armor texture, both must be present.

Previously, we created a RegistryKey<EquipmentAsset> constant called GUIDITE_ARMOR_MATERIAL_KEY which we passed into our ArmorMaterial constructor. It's recommended to name the texture similarly, so in our case, guidite.png

assets/mod-id/textures/entity/equipment/humanoid/guidite.png - Contains upper body and boot textures.
assets/mod-id/textures/entity/equipment/humanoid_leggings/guidite.png - Contains legging textures.

TIP

If you're updating to 1.21.4 from an older version of the game, the humanoid folder is where your layer0.png armor texture goes, and the humanoid_leggings folder is where your layer1.png armor texture goes.

Next, you'll need to create an associated equipment model definition. These go in the /assets/mod-id/equipment/ folder.

The RegistryKey<EquipmentAsset> constant we created earlier will determine the name of the JSON file. In this case, it'll be guidite.json.

Since we only plan to add "humanoid" (helmet, chestplate, leggings, boots etc.) armor pieces, our equipment model definition will look like this:

```json
{
  "layers": {
    "humanoid": [
      {
        "texture": "fabric-docs-reference:guidite"
      }
    ],
    "humanoid_leggings": [
      {
        "texture": "fabric-docs-reference:guidite"
      }
    ]
  }
}
```

With the textures and equipment model definition present, you should be able to see your armor on entities that wear it:

Custom Item Interactions
Basic items can only go so far - eventually you will need an item that interacts with the world when it is used.

There are some key classes you must understand before taking a look at the vanilla item events.

TypedActionResult
For items, the most common TypedActionResult you'll see is for ItemStacks - this class tells the game what to replace the item stack (or not to replace) after the event has occured.

If nothing has occured in the event, you should use the TypedActionResult#pass(stack) method where stack is the current item stack.

You can get the current item stack by getting the stack in the player's hand. Usually events that require a TypedActionResult pass the hand to the event method.

```java
TypedActionResult.pass(user.getStackInHand(hand))
```

If you pass the current stack - nothing will change, regardless of if you declare the event as failed, passed/ignored or successful.

If you want to delete the current stack, you should pass an empty one. The same can be said about decrementing, you fetch the current stack and decrement it by the amount you want:

```java
ItemStack heldStack = user.getStackInHand(hand);
heldStack.decrement(1);
TypedActionResult.success(heldStack);
```

ActionResult
Similarly, an ActionResult tells the game the status of the event, whether it was passed/ignored, failed or successful.

Overridable Events
Luckily, the Item class has many methods that can be overriden to add extra functionality to your items.

INFO

A great example of these events being used can be found in the Playing SoundEvents page, which uses the useOnBlock event to play a sound when the player right clicks a block.

postHit	Ran when the player hits an entity.
postMine	Ran when the player mines a block.
inventoryTick	Ran every tick whilst the item is in an inventory.
onCraft	Ran when the item is crafted.
useOnBlock	Ran when the player right clicks a block with the item.
use	Ran when the player right clicks the item.
The use() Event
Let's say you want to make an item that summons a lightning bolt in front of the player - you would need to create a custom class.

```java
public class LightningStick extends Item {
    public LightningStick(Settings settings) {
        super(settings);
    }

}
```

The use event is probably the most useful out of them all - you can use this event to spawn our lightning bolt, you should spawn it 10 blocks in front of the players facing direction.

```java
@Override
public ActionResult use(World world, PlayerEntity user, Hand hand) {
    // Ensure we don't spawn the lightning only on the client.
    // This is to prevent desync.
    if (world.isClient) {
        return ActionResult.PASS;
    }

    BlockPos frontOfPlayer = user.getBlockPos().offset(user.getHorizontalFacing(), 10);

    // Spawn the lightning bolt.
    LightningEntity lightningBolt = new LightningEntity(EntityType.LIGHTNING_BOLT, world);
    lightningBolt.setPosition(frontOfPlayer.toCenterPos());
    world.spawnEntity(lightningBolt);

    return ActionResult.SUCCESS;
}
```

As usual, you should register your item, add a model and texture.

As you can see, the lightning bolt should spawn 10 blocks in front of you - the player.

Custom Enchantment Effects
Starting from version 1.21, custom enchantments in Minecraft use a "data-driven" approach. This makes it easier to add simple enchantments, like increasing attack damage, but more challenging to create complex ones. The process involves breaking down enchantments into effect components.

An effect component contains the code that defines the special effects of an enchantment. Minecraft supports various default effects, such as item damage, knockback, and experience.

TIP

Be sure to check if the default Minecraft effects satisfy your needs by visiting the Minecraft Wiki's Enchantment Effect Components page. This guide assumes you understand how to configure "simple" data-driven enchantments and focuses on creating custom enchantment effects that aren't supported by default.

Custom Enchantment Effects
Start by creating an enchantment folder, and within it, create an effect folder. Within that, we'll create the LightningEnchantmentEffect record.

Next, we can create a constructor and override the EnchantmentEntityEffect interface methods. We'll also create a CODEC variable to encode and decode our effect; you can read more about Codecs here.

The bulk of our code will go into the apply() event, which is called when the criteria for your enchantment to work is met. We'll later configure this Effect to be called when an entity is hit, but for now, let's write simple code to strike the target with lightning.

```java
public record LightningEnchantmentEffect(EnchantmentLevelBasedValue amount) implements EnchantmentEntityEffect {
    public static final MapCodec<LightningEnchantmentEffect> CODEC = RecordCodecBuilder.mapCodec(instance ->
            instance.group(
                    EnchantmentLevelBasedValue.CODEC.fieldOf("amount").forGetter(LightningEnchantmentEffect::amount)
            ).apply(instance, LightningEnchantmentEffect::new)
    );

    @Override
    public void apply(ServerWorld world, int level, EnchantmentEffectContext context, Entity target, Vec3d pos) {
        if (target instanceof LivingEntity victim) {
            if (context.owner() != null && context.owner() instanceof PlayerEntity player) {
                float numStrikes = this.amount.getValue(level);

                for (float i = 0; i < numStrikes; i++) {
                    BlockPos position = victim.getBlockPos();
                    EntityType.LIGHTNING_BOLT.spawn(world, position, SpawnReason.TRIGGERED);
                }
            }
        }
    }

    @Override
    public MapCodec<? extends EnchantmentEntityEffect> getCodec() {
        return CODEC;
    }
}
```

Here, the amount variable indicates a value scaled to the level of the enchantment. We can use this to modify how effective the enchantment is based on level. In the code above, we are using the level of the enchantment to determine how many lightning strikes are spawned.

Registering the Enchantment Effect
Like every other component of your mod, we'll have to add this EnchantmentEffect to Minecraft's registry. To do so, add a class ModEnchantmentEffects (or whatever you want to name it) and a helper method to register the enchantment. Be sure to call the registerModEnchantmentEffects() in your main class, which contains the onInitialize() method.

```java
public class ModEnchantmentEffects {
    public static final RegistryKey<Enchantment> THUNDERING = of("thundering");
    public static MapCodec<LightningEnchantmentEffect> LIGHTNING_EFFECT = register("lightning_effect", LightningEnchantmentEffect.CODEC);

    private static RegistryKey<Enchantment> of(String path) {
        Identifier id = Identifier.of(FabricDocsReference.MOD_ID, path);
        return RegistryKey.of(RegistryKeys.ENCHANTMENT, id);
    }

    private static <T extends EnchantmentEntityEffect> MapCodec<T> register(String id, MapCodec<T> codec) {
        return Registry.register(Registries.ENCHANTMENT_ENTITY_EFFECT_TYPE, Identifier.of(FabricDocsReference.MOD_ID, id), codec);
    }

    public static void registerModEnchantmentEffects() {
        FabricDocsReference.LOGGER.info("Registering EnchantmentEffects for" + FabricDocsReference.MOD_ID);
    }
}
```

Creating the Enchantment
Now we have an enchantment effect! The final step is to create an enchantment that applies our custom effect. While this can be done by creating a JSON file similar to those in datapacks, this guide will show you how to generate the JSON dynamically using Fabric's data generation tools. To begin, create an EnchantmentGenerator class.

Within this class, we'll first register a new enchantment, and then use the configure() method to create our JSON programmatically.

```java
public class EnchantmentGenerator extends FabricDynamicRegistryProvider {
    public EnchantmentGenerator(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
        super(output, registriesFuture);
        System.out.println("REGISTERING ENCHANTS");
    }

    @Override
    protected void configure(RegistryWrapper.WrapperLookup registries, Entries entries) {
        // Our new enchantment, "Thundering."
        register(entries, ModEnchantmentEffects.THUNDERING, Enchantment.builder(
                Enchantment.definition(
                    registries.getOrThrow(RegistryKeys.ITEM).getOrThrow(ItemTags.WEAPON_ENCHANTABLE),
                    // this is the "weight" or probability of our enchantment showing up in the table
                    10,
                    // the maximum level of the enchantment
                    3,
                    // base cost for level 1 of the enchantment, and min levels required for something higher
                    Enchantment.leveledCost(1, 10),
                    // same fields as above but for max cost
                    Enchantment.leveledCost(1, 15),
                    // anvil cost
                    5,
                    // valid slots
                    AttributeModifierSlot.HAND
                )
            )
                    .addEffect(
                        // enchantment occurs POST_ATTACK
                        EnchantmentEffectComponentTypes.POST_ATTACK,
                        EnchantmentEffectTarget.ATTACKER,
                        EnchantmentEffectTarget.VICTIM,
                        new LightningEnchantmentEffect(EnchantmentLevelBasedValue.linear(0.4f, 0.2f)) // scale the enchantment linearly.
                    )
        );
    }

    private void register(Entries entries, RegistryKey<Enchantment> key, Enchantment.Builder builder, ResourceCondition... resourceConditions) {
        entries.add(key, builder.build(key.getValue()), resourceConditions);
    }

    @Override
    public String getName() {
        return "ReferenceDocEnchantmentGenerator";
    }
}
```

Before proceeding, you should ensure your project is configured for data generation; if you are unsure, view the respective docs page.

Lastly, we must tell our mod to add our EnchantmentGenerator to the list of data generation tasks. To do so, simply add the EnchantmentGenerator to this inside of the onInitializeDataGenerator method.

```java
pack.addProvider(EnchantmentGenerator::new);
```

Now, when you run your mod's data generation task, enchantment JSONs will be generated inside the generated folder. An example can be seen below:

```json
{
  "anvil_cost": 5,
  "description": {
    "translate": "enchantment.fabric-docs-reference.thundering"
  },
  "effects": {
    "minecraft:post_attack": [
      {
        "affected": "victim",
        "effect": {
          "type": "fabric-docs-reference:lightning_effect",
          "amount": {
            "type": "minecraft:linear",
            "base": 0.4,
            "per_level_above_first": 0.2
          }
        },
        "enchanted": "attacker"
      }
    ]
  },
  "max_cost": {
    "base": 1,
    "per_level_above_first": 15
  },
  "max_level": 3,
  "min_cost": {
    "base": 1,
    "per_level_above_first": 10
  },
  "slots": [
    "hand"
  ],
  "supported_items": "#minecraft:enchantable/weapon",
  "weight": 10
}
```

You should also add translations to your en_us.json file to give your enchantment a readable name:

```json
"enchantment.FabricDocsReference.thundering": "Thundering",
```

You should now have a working custom enchantment effect! Test it by enchanting a weapon with the enchantment and hitting a mob. 

Custom Data Components
As your items grow more complex, you may find yourself needing to store custom data associated with each item. The game allows you to store persistent data within an ItemStack, and as of 1.20.5 the way we do that is by using Data Components.

Data Components replace NBT data from previous versions with structured data types which can be applied to an ItemStack to store persistent data about that stack. Data components are namespaced, meaning we can implement our own data components to store custom data about an ItemStack and access it later. A full list of the vanilla data components can be found on this Minecraft wiki page.

Along with registering custom components, this page covers the general usage of the components API, which also applies to vanilla components. You can see and access the definitions of all vanilla components in the DataComponentTypes class.

Registering a Component
As with anything else in your mod you will need to register your custom component using a ComponentType. This component type takes a generic argument containing the type of your component's value. We will be focusing on this in more detail further down when covering basic and advanced components.

Choose a sensible class to place this in. For this example we're going to make a new package called component and a class to contain all of our component types called ModComponents. Make sure you call ModComponents.initialize() in your mod's initializer.
```java
public class ModComponents {
    protected static void initialize() {
        FabricDocsReference.LOGGER.info("Registering {} components", FabricDocsReference.MOD_ID);
        // Technically this method can stay empty, but some developers like to notify
        // the console, that certain parts of the mod have been successfully initialized
    }
}
```
This is the basic template to register a component type:

```java
public static final ComponentType<?> MY_COMPONENT_TYPE = Registry.register(
    Registries.DATA_COMPONENT_TYPE,
    Identifier.of(FabricDocsReference.MOD_ID, "my_component"),
    ComponentType.<?>builder().codec(null).build()
);
```

There are a few things here worth noting. On the first and fourth lines, you can see a ?. This will be replaced with the type of your component's value. We'll fill this in soon.

Secondly, you must provide an Identifier containing the intended ID of your component. This is namespaced with your mod's ID.

Lastly, we have a ComponentType.Builder that creates the actual ComponentType instance that's being registered. This contains another crucial detail we will need to discuss: your component's Codec. This is currently null but we will also fill it in soon.

Basic Data Components
Basic data components (like minecraft:damage) consist of a single data value, such as an int, float, boolean or String.

As an example, let's create an Integer value that will track how many times the player has right-clicked while holding our item. Let's update our component registration to the following:

```java
public static final ComponentType<Integer> CLICK_COUNT_COMPONENT = Registry.register(
        Registries.DATA_COMPONENT_TYPE,
        Identifier.of(FabricDocsReference.MOD_ID, "click_count"),
        ComponentType.<Integer>builder().codec(Codec.INT).build()
);
```

You can see that we're now passing <Integer> as our generic type, indicating that this component will be stored as a single int value. For our codec, we are using the provided Codec.INT codec. We can get away with using basic codecs for simple components like this, but more complex scenarios might require a custom codec (this will be covered briefly later on).

If you start the game, you should be able to enter a command like this:

/give @p minecraft:diamond[fabric-docs-reference:click_count=5]

When you run the command, you should receive the item containing the component. However, we are not currently using our component to do anything useful. Let's start by reading the value of the component in a way we can see.

Reading Component Value
Let's add a new item which will increase the counter each time it is right clicked. You should read the Custom Item Interactions page which will cover the techniques we will use in this guide.

```java
public class CounterItem extends Item {
    public CounterItem(Settings settings) {
        super(settings);
    }

}
```

Remember as usual to register the item in your ModItems class.

```java
public static final Item COUNTER = register(new CounterItem(
    new Item.Settings()
), "counter");
```

We're going to add some tooltip code to display the current value of the click count when we hover over our item in the inventory. We can use the get() method on our ItemStack to get our component value like so:
```java
int clickCount = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
```


This will return the current component value as the type we defined when we registered our component. We can then use this value to add a tooltip entry. Add this line to the appendTooltip method in the CounterItem class:

```java
public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
    int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
    tooltip.add(Text.translatable("item.fabric-docs-reference.counter.info", count).formatted(Formatting.GOLD));
}
```

Don't forget to update your lang file (/assets/mod-id/lang/en_us.json) and add these two lines:

```java
{
  "item.fabric-docs-reference.counter": "Counter",
  "item.fabric-docs-reference.counter.info": "Used %1$s times"
}
```

Start up the game and run this command to give yourself a new Counter item with a count of 5.

/give @p fabric-docs-reference:counter[fabric-docs-reference:click_count=5]

When you hover over this item in your inventory, you should see the count displayed in the tooltip!

However, if you give yourself a new Counter item without the custom component, the game will crash when you hover over the item in your inventory. You should see an error like this in the crash report:

java.lang.NullPointerException: Cannot invoke "java.lang.Integer.intValue()" because the return value of "net.minecraft.item.ItemStack.get(net.minecraft.component.ComponentType)" is null
        at com.example.docs.item.custom.CounterItem.appendTooltip(LightningStick.java:45)
        at net.minecraft.item.ItemStack.getTooltip(ItemStack.java:767)

As expected, since the ItemStack doesn't currently contain an instance of our custom component, calling stack.get() with our component type will return null.

There are three solutions we can use to address this problem.

Setting a Default Component Value
When you register your item and pass a Item.Settings object to your item constructor, you can also provide a list of default components that are applied to all new items. If we go back to our ModItems class, where we register the CounterItem, we can add a default value for our custom component. Add this so that new items display a count of 0.

```java
public static final Item COUNTER = register(
        "counter",
        CounterItem::new,
        new Item.Settings()
                // Initialize the click count component with a default value of 0
                .component(ModComponents.CLICK_COUNT_COMPONENT, 0)
);
```

When a new item is created, it will automatically apply our custom component with the given value.

WARNING

Using commands, it is possible to remove a default component from an ItemStack. You should refer to the next two sections to properly handle a scenario where the component is not present on your item.

Reading with a Default Value
In addition, when reading the component value, we can use the getOrDefault() method on our ItemStack object to return a specified default value if the component is not present on the stack. This will safeguard against any errors resulting from a missing component. We can adjust our tooltip code like so:

int clickCount = stack.getOrDefault(ModComponents.CLICK_COUNT_COMPONENT, 0);

As you can see, this method takes two arguments: our component type like before, and a default value to return if the component is not present.

Checking if a Component Exists
You can also check for the existence of a specific component on an ItemStack using the contains() method. This takes the component type as an argument and returns true or false depending on whether the stack contains that component.


boolean exists = stack.contains(ModComponents.CLICK_COUNT_COMPONENT);
1
Fixing the Error
We're going to go with the third option. So along with adding a default component value, we'll also check if the component is present on the stack and only show the tooltip if it is.


public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
    if (stack.contains(ModComponents.CLICK_COUNT_COMPONENT)) {
        int count = stack.get(ModComponents.CLICK_COUNT_COMPONENT);
        tooltip.add(Text.translatable("item.fabric-docs-reference.counter.info", count).formatted(Formatting.GOLD));
    }
}
1
2
3
4
5
6
Start the game again and hover over the item without the component, you should see that it displays "Used 0 times" and no longer crashes the game.

Tooltip showing "Used 0 times"

Try giving yourself a Counter with our custom component removed. You can use this command to do so:


/give @p fabric-docs-reference:counter[!fabric-docs-reference:click_count]
1
When hovering over this item, the tooltip should be missing.

Counter item with no tooltip

Updating Component Value
Now let's try updating our component value. We're going to increase the click count each time we use our Counter item. To change the value of a component on an ItemStack we use the set() method like so:


stack.set(ModComponents.CLICK_COUNT_COMPONENT, newValue);
1
This takes our component type and the value we want to set it to. In this case it will be our new click count. This method also returns the old value of the component (if one is present) which may be useful in some situations. For example:


int oldValue = stack.set(ModComponents.CLICK_COUNT_COMPONENT, newValue);
1
Let's set up a new use() method to read the old click count, increase it by one, and then set the updated click count.


public ActionResult use(World world, PlayerEntity user, Hand hand) {
    ItemStack stack = user.getStackInHand(hand);

    // Don't do anything on the client
    if (world.isClient()) {
        return ActionResult.SUCCESS;
    }

    // Read the current count and increase it by one
    int count = stack.getOrDefault(ModComponents.CLICK_COUNT_COMPONENT, 0);
    stack.set(ModComponents.CLICK_COUNT_COMPONENT, ++count);

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
12
13
14
Now try starting the game and right-clicking with the Counter item in your hand. If you open up your inventory and look at the item again you should see that the usage number has gone up by the amount of times you've clicked it.

Tooltip showing "Used 8 times"

Removing Component Value
You can also remove a component from your ItemStack if it is no longer needed. This is done by using the remove() method, which takes in your component type.


stack.remove(ModComponents.CLICK_COUNT_COMPONENT);
1
This method also returns the value of the component before being removed, so you can also use it as follows:


int oldCount = stack.remove(ModComponents.CLICK_COUNT_COMPONENT);
1
Advanced Data Components
You may need to store multiple attributes in a single component. As a vanilla example, the minecraft:food component stores several values related to food, such as nutrition, saturation, eat_seconds and more. In this guide we'll refer to them as "composite" components.

For composite components, you must create a record class to store the data. This is the type we'll register in our component type and what we'll read and write when interacting with an ItemStack. Start by making a new record class in the component package we made earlier.


public record MyCustomComponent() {
}
1
2
Notice that there's a set of brackets after the class name. This is where we define the list of properties we want our component to have. Let's add a float and a boolean called temperature and burnt respectively.


public record MyCustomComponent(float temperature, boolean burnt) {
}
1
2
Since we are defining a custom data structure, there won't be a pre-existing Codec for our use case like with the basic component. This means we're going to have to construct our own codec. Let's define one in our record class using a RecordCodecBuilder which we can reference once we register the component. For more details on using a RecordCodecBuilder you can refer to this section of the Codecs page.


public static final Codec<MyCustomComponent> CODEC = RecordCodecBuilder.create(builder -> {
    return builder.group(
        Codec.FLOAT.fieldOf("temperature").forGetter(MyCustomComponent::temperature),
        Codec.BOOL.optionalFieldOf("burnt", false).forGetter(MyCustomComponent::burnt)
    ).apply(builder, MyCustomComponent::new);
});
1
2
3
4
5
6
You can see that we are defining a list of custom fields based on the primitive Codec types. However, we are also telling it what our fields are called using fieldOf(), and then using forGetter() to tell the game which attribute of our record to populate.

You can also define optional fields by using optionalFieldOf() and passing a default value as the second argument. Any fields not marked optional will be required when setting the component using /give so make sure you mark any optional arguments as such when creating your codec.

Finally, we call apply() and pass our record's constructor. For more details on how to construct codecs and more advanced use cases, be sure to read the Codecs page.

Registering a composite component is similar to before. We just pass our record class as the generic type, and our custom Codec to the codec() method.


public static final ComponentType<MyCustomComponent> MY_CUSTOM_COMPONENT = Registry.register(
        Registries.DATA_COMPONENT_TYPE,
        Identifier.of(FabricDocsReference.MOD_ID, "custom"),
        ComponentType.<MyCustomComponent>builder().codec(MyCustomComponent.CODEC).build()
);
1
2
3
4
5
Now start the game. Using the /give command, try applying the component. Composite component values are passed as an object enclosed with {}. If you put blank curly brackets, you'll see an error telling you that the required key temperature is missing.

Give command showing missing key "temperature"

Add a temperature value to the object using the syntax temperature:8.2. You can also optionally pass a value for burnt using the same syntax but either true or false. You should now see that the command is valid, and can give you an item containing the component.

Valid give command showing both properties

Getting, Setting and Removing Advanced Components
Using the component in code is the same as before. Using stack.get() will return an instance of your record class, which you can then use to read the values. Since records are read-only, you will need to create a new instance of your record to update the values.


// read values of component
MyCustomComponent comp = stack.get(ModComponents.MY_CUSTOM_COMPONENT);
float temp = comp.temperature();
boolean burnt = comp.burnt();

// set new component values
stack.set(ModComponents.MY_CUSTOM_COMPONENT, new MyCustomComponent(8.4f, true));

// check for component
if (stack.contains(ModComponents.MY_CUSTOM_COMPONENT)) {
    // do something
}

// remove component
stack.remove(ModComponents.MY_CUSTOM_COMPONENT);
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
You can also set a default value for a composite component by passing a component object to your Item.Settings. For example:


public static final Item COUNTER = register(new CounterItem(
    new Item.Settings().component(ModComponents.MY_CUSTOM_COMPONENT, new MyCustomComponent(0.0f, false))
), "counter");
1
2
3
Now you can store custom data on an ItemStack. Use responsibly!

Potions
Potions are consumables that grants an entity an effect. A player can brew potions using a Brewing Stand or obtain them as items through various other game mechanics.

Custom Potions
Just like items and blocks, potions need to be registered.

Creating the Potion
Let's start by declaring a field to store your Potion instance. We will be directly using a ModInitializer-implementing class to hold this.


public class FabricDocsReferencePotions implements ModInitializer {
    public static final Potion TATER_POTION =
            Registry.register(
                    Registries.POTION,
                    Identifier.of(FabricDocsReference.MOD_ID, "tater"),
                    new Potion("tater",
                            new StatusEffectInstance(
                                    FabricDocsReferenceEffects.TATER,
                                    3600,
                                    0)));
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
We pass an instance of StatusEffectInstance, which takes 3 parameters:

RegistryEntry<StatusEffect> type - An effect. We use our custom effect here. Alternatively you can access vanilla effects through vanilla's StatusEffects class.
int duration - Duration of the effect in game ticks.
int amplifier - An amplifier for the effect. For example, Haste II would have an amplifier of 1.
INFO

To create your own potion effect, please see the Effects guide.

Registering the Potion
In our initializer, we will use the FabricBrewingRecipeRegistryBuilder.BUILD event to register our potion using the BrewingRecipeRegistry.registerPotionRecipe method.


@Override
public void onInitialize() {
    FabricBrewingRecipeRegistryBuilder.BUILD.register(builder -> {
        builder.registerPotionRecipe(
                // Input potion.
                Potions.WATER,
                // Ingredient
                Items.POTATO,
                // Output potion.
                Registries.POTION.getEntry(TATER_POTION)
        );
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
11
12
registerPotionRecipe takes 3 parameters:

RegistryEntry<Potion> input - The starting potion's registry entry. Usually this can be a Water Bottle or an Awkward Potion.
Item item - The item which is the main ingredient of the potion.
RegistryEntry<Potion> output - The resultant potion's registry entry.
Once registered, you can brew a Tater potion using a potato.


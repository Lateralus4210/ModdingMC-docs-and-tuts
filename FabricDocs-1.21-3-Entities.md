Status Effects

This page is written for version:

1.21.4


Page Authors
dicedpixels
Friendly-Banana
Manchick0
SattesKrokodil
siglongtao0lu
TheFireBlast
YanisBft
Status effects, also known as effects, are a condition that can affect an entity. They can be positive, negative or neutral in nature. The base game applies these effects in various ways such as food, potions etc.

The /effect command can be used to apply effects on an entity.

Custom Status Effects
In this tutorial we'll add a new custom effect called Tater which gives you one experience point every game tick.

Extend StatusEffect
Let's create a custom effect class by extending StatusEffect, which is the base class for all effects.


public class TaterEffect extends StatusEffect {
    protected TaterEffect() {
        // category: StatusEffectCategory - describes if the effect is helpful (BENEFICIAL), harmful (HARMFUL) or useless (NEUTRAL)
        // color: int - Color is the color assigned to the effect (in RGB)
        super(StatusEffectCategory.BENEFICIAL, 0xe9b8b3);
    }

    // Called every tick to check if the effect can be applied or not
    @Override
    public boolean canApplyUpdateEffect(int duration, int amplifier) {
        // In our case, we just make it return true so that it applies the effect every tick
        return true;
    }

    // Called when the effect is applied.
    @Override
    public boolean applyUpdateEffect(ServerWorld world, LivingEntity entity, int amplifier) {
        if (entity instanceof PlayerEntity) {
            ((PlayerEntity) entity).addExperience(1 << amplifier); // Higher amplifier gives you experience faster
        }

        return super.applyUpdateEffect(world, entity, amplifier);
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
Registering Your Custom Effect
Similar to block and item registration, we use Registry.register to register our custom effect into the STATUS_EFFECT registry. This can be done in our initializer.


public class FabricDocsReferenceEffects implements ModInitializer {
    public static final RegistryEntry<StatusEffect> TATER =
            Registry.registerReference(Registries.STATUS_EFFECT, Identifier.of(FabricDocsReference.MOD_ID, "tater"), new TaterEffect());

    @Override
    public void onInitialize() {
        // ...
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
Texture
The status effect icon is a 18x18 PNG which will appear in the player's inventory screen. Place your custom icon in:


resources/assets/fabric-docs-reference/textures/mob_effect/tater.png

Download Example Texture
Translations
Like any other translation, you can add an entry with ID format "effect.mod-id.effect-identifier": "Value" to the language file.


{
  "effect.fabric-docs-reference.tater": "Tater"
}
1
2
3
Applying The Effect
It's worth taking a look at how you'd typically apply an effect to an entity.

TIP

For a quick test, it might be a better idea to use the previously mentioned /effect command:


effect give @p fabric-docs-reference:tater
1
To apply an effect internally, you'd want to use the LivingEntity#addStatusEffect method, which takes in a StatusEffectInstance, and returns a boolean, specifying whether the effect was successfully applied.


var instance = new StatusEffectInstance(FabricDocsReferenceEffects.TATER, 5 * 20, 0, false, true, true);
entity.addStatusEffect(instance);
1
2
Argument	Type	Description
effect	RegistryEntry<StatusEffect>	A registry entry that represents the effect.
duration	int	The duration of the effect in ticks; not seconds
amplifier	int	The amplifier to the level of the effect. It doesn't correspond to the level of the effect, but is rather added on top. Hence, amplifier of 4 => level of 5
ambient	boolean	This is a tricky one. It basically specifies that the effect was added by the environment (e.g. a Beacon) and doesn't have a direct cause. If true, the icon of the effect in the HUD will appear with an aqua overlay.
particles	boolean	Whether to show particles.
icon	boolean	Whether to display an icon of the effect in the HUD. The effect will be displayed in the inventory regardless of this flag.
INFO

To create a potion that uses this effect, please see the Potions guide.

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Damage Types

This page is written for version:

1.21.4


Page Authors
dicedpixels
hiisuuii
MattiDragon
Damage types define types of damage that entities can take. Since Minecraft 1.19.4, the creation of new damage types has become data-driven, meaning they are created using JSON files.

Creating a Damage Type
Let's create a custom damage type called Tater. We start by creating a JSON file for your custom damage. This file will be placed in your mod's data directory, in a subdirectory named damage_type.


resources/data/fabric-docs-reference/damage_type/tater.json
It has the following structure:


{
  "exhaustion": 0.1,
  "message_id": "tater",
  "scaling": "when_caused_by_living_non_player"
}
1
2
3
4
5
This custom damage type causes 0.1 increase in hunger exhaustion each time a player takes damage, when the damage is caused by a living, non-player source (e.g., a block). Additionally, the amount of damage dealt will scale with the world's difficulty

INFO

Refer to the Minecraft Wiki for all the possible keys and values.

Accessing Damage Types Through Code
When we need to access our custom damage type through code, we will use it's RegistryKey to build an instance of DamageSource.

The RegistryKey can be obtained as follows:


public static final RegistryKey<DamageType> TATER_DAMAGE = RegistryKey.of(RegistryKeys.DAMAGE_TYPE, Identifier.of(FabricDocsReference.MOD_ID, "tater"));
1
Using Damage Types
To demonstrate the use of custom damage types, we will use a custom block called Tater Block. Let's make is so that when a living entity steps on a Tater Block, it deals Tater damage.

You can override onSteppedOn to inflict this damage.

We start by creating a DamageSource of our custom damage type.


DamageSource damageSource = new DamageSource(
        world.getRegistryManager()
                .getOrThrow(RegistryKeys.DAMAGE_TYPE)
                .getEntry(FabricDocsReferenceDamageTypes.TATER_DAMAGE.getValue()).get()
);
1
2
3
4
5
Then, we call entity.damage() with our DamageSource and an amount.


entity.damage(serverWorld, damageSource, 5.0f);
1
The complete block implementation:


public class TaterBlock extends Block {
    public TaterBlock(Settings settings) {
        super(settings);
    }

    @Override
    public void onSteppedOn(World world, BlockPos pos, BlockState state, Entity entity) {
        if (entity instanceof LivingEntity && world instanceof ServerWorld serverWorld) {
            DamageSource damageSource = new DamageSource(
                    world.getRegistryManager()
                            .getOrThrow(RegistryKeys.DAMAGE_TYPE)
                            .getEntry(FabricDocsReferenceDamageTypes.TATER_DAMAGE.getValue()).get()
            );
            entity.damage(serverWorld, damageSource, 5.0f);
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
Now whenever a living entity steps on our custom block, it'll take 5 damage (2.5 hearts) using our custom damage type.

Custom Death Message
You can define a death message for the damage type in the format of death.attack.message_id in our mod's en_us.json file.


{
  // ...
  "death.attack.tater": "%1$s died from Tater damage!",
  // ...
}
1
2
3
4
5
Upon death from our damage type, you'll see the following death message:

Effect in player inventory

Damage Type Tags
Some damage types can bypass armor, bypass status effects, and such. Tags are used to control these kinds of properties of damage types.

You can find existing damage type tags in data/minecraft/tags/damage_type.

INFO

Refer to the Minecraft Wiki for a comprehensive list of damage type tags.

Let's add our Tater damage type to the bypasses_armor damage type tag.

To add our damage type to one of these tags, we create a JSON file under the minecraft namespace.


data/minecraft/tags/damage_type/bypasses_armor.json
With the following content:


{
  "values": [
    "fabric-docs-reference:tater"
  ]
}
1
2
3
4
5
Ensure your tag does not replace the existing tag by setting the replace key to false.

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM
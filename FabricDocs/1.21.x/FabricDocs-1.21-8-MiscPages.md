Codecs

This page is written for version:

1.21.4


Page Authors
enjarai
Syst3ms
Codec is a system for easily serializing Java objects, and is included in Mojang's DataFixerUpper (DFU) library, which is included with Minecraft. In a modding context they can be used as an alternative to GSON and Jankson when reading and writing custom json files, though they're starting to become more and more relevant, as Mojang is rewriting a lot of old code to use Codecs.

Codecs are used in conjunction with another API from DFU, DynamicOps. A codec defines the structure of an object, while dynamic ops are used to define a format to be serialized to and from, such as json or NBT. This means any codec can be used with any dynamic ops, and vice versa, allowing for great flexibility.

Using Codecs
Serializing and Deserializing
The basic usage of a codec is to serialize and deserialize objects to and from a specific format.

Since a few vanilla classes already have codecs defined, we can use those as an example. Mojang has also provided us with two dynamic ops classes by default, JsonOps and NbtOps, which tend to cover most use cases.

Now, let's say we want to serialize a BlockPos to json and back. We can do this using the codec statically stored at BlockPos.CODEC with the Codec#encodeStart and Codec#parse methods, respectively.


BlockPos pos = new BlockPos(1, 2, 3);

// Serialize the BlockPos to a JsonElement
DataResult<JsonElement> result = BlockPos.CODEC.encodeStart(JsonOps.INSTANCE, pos);
1
2
3
4
When using a codec, values are returned in the form of a DataResult. This is a wrapper that can represent either a success or a failure. We can use this in several ways: If we just want our serialized value, DataResult#result will simply return an Optional containing our value, while DataResult#resultOrPartial also lets us supply a function to handle any errors that may have occurred. The latter is particularly useful for custom datapack resources, where we'd want to log errors without causing issues elsewhere.

So let's grab our serialized value and turn it back into a BlockPos:


// When actually writing a mod, you'll want to properly handle empty Optionals of course
JsonElement json = result.resultOrPartial(LOGGER::error).orElseThrow();

// Here we have our json value, which should correspond to `[1, 2, 3]`,
// as that's the format used by the BlockPos codec.
LOGGER.info("Serialized BlockPos: {}", json);

// Now we'll deserialize the JsonElement back into a BlockPos
DataResult<BlockPos> result = BlockPos.CODEC.parse(JsonOps.INSTANCE, json);

// Again, we'll just grab our value from the result
BlockPos pos = result.resultOrPartial(LOGGER::error).orElseThrow();

// And we can see that we've successfully serialized and deserialized our BlockPos!
LOGGER.info("Deserialized BlockPos: {}", pos);
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
Built-in Codecs
As mentioned earlier, Mojang has already defined codecs for several vanilla and standard Java classes, including but not limited to BlockPos, BlockState, ItemStack, Identifier, Text, and regex Patterns. Codecs for Mojang's own classes are usually found as static fields named CODEC on the class itself, while most others are kept in the Codecs class. It should also be noted that all vanilla registries contain a getCodec() method, for example, you can use Registries.BLOCK.getCodec() to get a Codec<Block> which serializes to the block id and back.

The Codec API itself also contains some codecs for primitive types, such as Codec.INT and Codec.STRING. These are available as statics on the Codec class, and are usually used as the base for more complex codecs, as explained below.

Building Codecs
Now that we've seen how to use codecs, let's take a look at how we can build our own. Suppose we have the following class, and we want to deserialize instances of it from json files:


public class CoolBeansClass {

    private final int beansAmount;
    private final Item beanType;
    private final List<BlockPos> beanPositions;

    public CoolBeansClass(int beansAmount, Item beanType, List<BlockPos> beanPositions) {...}

    public int getBeansAmount() { return this.beansAmount; }
    public Item getBeanType() { return this.beanType; }
    public List<BlockPos> getBeanPositions() { return this.beanPositions; }
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
The corresponding json file might look something like this:


{
  "beans_amount": 5,
  "bean_type": "beanmod:mythical_beans",
  "bean_positions": [
    [1, 2, 3],
    [4, 5, 6]
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
We can make a codec for this class by putting together multiple smaller codecs into a larger one. In this case, we'll need one for each field:

a Codec<Integer>
a Codec<Item>
a Codec<List<BlockPos>>
We can get the first one from the aforementioned primitive codecs in the Codec class, specifically Codec.INT. While the second one can be obtained from the Registries.ITEM registry, which has a getCodec() method that returns a Codec<Item>. We don't have a default codec for List<BlockPos>, but we can make one from BlockPos.CODEC.

Lists
Codec#listOf can be used to create a list version of any codec:


Codec<List<BlockPos>> listCodec = BlockPos.CODEC.listOf();
1
It should be noted that codecs created in this way will always deserialize to an ImmutableList. If you need a mutable list instead, you can make use of xmap to convert to one during deserialization.

Merging Codecs for Record-Like Classes
Now that we have separate codecs for each field, we can combine them into one codec for our class using a RecordCodecBuilder. This assumes that our class has a constructor containing every field we want to serialize, and that every field has a corresponding getter method. This makes it perfect to use in conjunction with records, but it can also be used with regular classes.

Let's take a look at how to create a codec for our CoolBeansClass:


public static final Codec<CoolBeansClass> CODEC = RecordCodecBuilder.create(instance -> instance.group(
    Codec.INT.fieldOf("beans_amount").forGetter(CoolBeansClass::getBeansAmount),
    Registries.ITEM.getCodec().fieldOf("bean_type").forGetter(CoolBeansClass::getBeanType),
    BlockPos.CODEC.listOf().fieldOf("bean_positions").forGetter(CoolBeansClass::getBeanPositions)
    // Up to 16 fields can be declared here
).apply(instance, CoolBeansClass::new));
1
2
3
4
5
6
Each line in the group specifies a codec, a field name, and a getter method. The Codec#fieldOf call is used to convert the codec into a map codec, and the forGetter call specifies the getter method used to retrieve the value of the field from an instance of the class. Meanwhile, the apply call specifies the constructor used to create new instances. Note that the order of the fields in the group should be the same as the order of the arguments in the constructor.

You can also use Codec#optionalFieldOf in this context to make a field optional, as explained in the Optional Fields section.

MapCodec, Not to Be Confused With Codec<Map>
Calling Codec#fieldOf will convert a Codec<T> into a MapCodec<T>, which is a variant, but not direct implementation of Codec<T>. MapCodecs, as their name suggests are guaranteed to serialize into a key to value map, or its equivalent in the DynamicOps used. Some functions may require one over a regular codec.

This particular way of creating a MapCodec essentially boxes the value of the source codec inside a map, with the given field name as the key. For example, a Codec<BlockPos> when serialized into json would look like this:


[1, 2, 3]
1
But when converted into a MapCodec<BlockPos> using BlockPos.CODEC.fieldOf("pos"), it would look like this:


{
  "pos": [1, 2, 3]
}
1
2
3
While the most common use for map codecs is to be merged with other map codecs to construct a codec for a full class worth of fields, as explained in the Merging Codecs for Record-like Classes section above, they can also be turned back into regular codecs using MapCodec#codec, which will retain the same behavior of boxing their input value.

Optional Fields
Codec#optionalFieldOf can be used to create an optional map codec. This will, when the specified field is not present in the container during deserialization, either be deserialized as an empty Optional or a specified default value.


// Without a default value
MapCodec<Optional<BlockPos>> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos");

// With a default value
MapCodec<BlockPos> optionalCodec = BlockPos.CODEC.optionalFieldOf("pos", BlockPos.ORIGIN);
1
2
3
4
5
Do note that optional fields will silently ignore any errors that may occur during deserialization. This means that if the field is present, but the value is invalid, the field will always be deserialized as the default value.

Since 1.20.2, Minecraft itself (not DFU!) does however provide Codecs#createStrictOptionalFieldCodec, which fails to deserialize at all if the field value is invalid.

Constants, Constraints, and Composition
Unit
Codec.unit can be used to create a codec that always deserializes to a constant value, regardless of the input. When serializing, it will do nothing.


Codec<Integer> theMeaningOfCodec = Codec.unit(42);
1
Numeric Ranges
Codec.intRange and its pals, Codec.floatRange and Codec.doubleRange can be used to create a codec that only accepts number values within a specified inclusive range. This applies to both serialization and deserialization.


// Can't be more than 2
Codec<Integer> amountOfFriendsYouHave = Codec.intRange(0, 2);
1
2
Pair
Codec.pair merges two codecs, Codec<A> and Codec<B>, into a Codec<Pair<A, B>>. Keep in mind it only works properly with codecs that serialize to a specific field, such as converted MapCodecs or record codecs. The resulting codec will serialize to a map combining the fields of both codecs used.

For example, running this code:


// Create two separate boxed codecs
Codec<Integer> firstCodec = Codec.INT.fieldOf("i_am_number").codec();
Codec<Boolean> secondCodec = Codec.BOOL.fieldOf("this_statement_is_false").codec();

// Merge them into a pair codec
Codec<Pair<Integer, Boolean>> pairCodec = Codec.pair(firstCodec, secondCodec);

// Use it to serialize data
DataResult<JsonElement> result = pairCodec.encodeStart(JsonOps.INSTANCE, Pair.of(23, true));
1
2
3
4
5
6
7
8
9
Will output this json:


{
  "i_am_number": 23,
  "this_statement_is_false": true
}
1
2
3
4
Either
Codec.either combines two codecs, Codec<A> and Codec<B>, into a Codec<Either<A, B>>. The resulting codec will, during deserialization, attempt to use the first codec, and only if that fails, attempt to use the second one. If the second one also fails, the error of the second codec will be returned.

Maps
For processing maps with arbitrary keys, such as HashMaps, Codec.unboundedMap can be used. This returns a Codec<Map<K, V>> for a given Codec<K> and Codec<V>. The resulting codec will serialize to a json object or whatever equivalent is available for the current dynamic ops.

Due to limitations of json and nbt, the key codec used must serialize to a string. This includes codecs for types that aren't strings themselves, but do serialize to them, such as Identifier.CODEC. See the example below:


// Create a codec for a map of identifiers to integers
Codec<Map<Identifier, Integer>> mapCodec = Codec.unboundedMap(Identifier.CODEC, Codec.INT);

// Use it to serialize data
DataResult<JsonElement> result = mapCodec.encodeStart(JsonOps.INSTANCE, Map.of(
    new Identifier("example", "number"), 23,
    new Identifier("example", "the_cooler_number"), 42
));
1
2
3
4
5
6
7
8
This will output this json:


{
  "example:number": 23,
  "example:the_cooler_number": 42
}
1
2
3
4
As you can see, this works because Identifier.CODEC serializes directly to a string value. A similar effect can be achieved for simple objects that don't serialize to strings by using xmap & friends to convert them.

Mutually Convertible Types
xmap
Say we have two classes that can be converted to each other, but don't have a parent-child relationship. For example, a vanilla BlockPos and Vec3d. If we have a codec for one, we can use Codec#xmap to create a codec for the other by specifying a conversion function for each direction.

BlockPos already has a codec, but let's pretend it doesn't. We can create one for it by basing it on the codec for Vec3d like this:


Codec<BlockPos> blockPosCodec = Vec3d.CODEC.xmap(
    // Convert Vec3d to BlockPos
    vec -> new BlockPos(vec.x, vec.y, vec.z),
    // Convert BlockPos to Vec3d
    pos -> new Vec3d(pos.getX(), pos.getY(), pos.getZ())
);

// When converting an existing class (`X` for example)
// to your own class (`Y`) this way, it may be nice to
// add `toX` and static `fromX` methods to `Y` and use
// method references in your `xmap` call.
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
flatComapMap, comapFlatMap, and flatXMap
Codec#flatComapMap, Codec#comapFlatMap and flatXMap are similar to xmap, but they allow one or both of the conversion functions to return a DataResult. This is useful in practice because a specific object instance may not always be valid for conversion.

Take for example vanilla Identifiers. While all identifiers can be turned into strings, not all strings are valid identifiers, so using xmap would mean throwing ugly exceptions when the conversion fails. Because of this, its built-in codec is actually a comapFlatMap on Codec.STRING, nicely illustrating how to use it:


public class Identifier {
    public static final Codec<Identifier> CODEC = Codec.STRING.comapFlatMap(
        Identifier::validate, Identifier::toString
    );

    // ...

    public static DataResult<Identifier> validate(String id) {
        try {
            return DataResult.success(new Identifier(id));
        } catch (InvalidIdentifierException e) {
            return DataResult.error("Not a valid resource location: " + id + " " + e.getMessage());
        }
    }

    // ...
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
While these methods are really helpful, their names are a bit confusing, so here's a table to help you remember which one to use:

Method	A -> B always valid?	B -> A always valid?
Codec<A>#xmap	Yes	Yes
Codec<A>#comapFlatMap	No	Yes
Codec<A>#flatComapMap	Yes	No
Codec<A>#flatXMap	No	No
Registry Dispatch
Codec#dispatch let us define a registry of codecs and dispatch to a specific one based on the value of a field in the serialized data. This is very useful when deserializing objects that have different fields depending on their type, but still represent the same thing.

For example, say we have an abstract Bean interface with two implementing classes: StringyBean and CountingBean. To serialize these with a registry dispatch, we'll need a few things:

Separate codecs for every type of bean.
A BeanType<T extends Bean> class or record that represents the type of bean, and can return the codec for it.
A function on Bean to retrieve its BeanType<?>.
A map or registry to map Identifiers to BeanType<?>s.
A Codec<BeanType<?>> based on this registry. If you use a net.minecraft.registry.Registry, one can be easily made using Registry#getCodec.
With all of this, we can create a registry dispatch codec for beans:


// The abstract type we want to create a codec for
public interface Bean {
    // Now we can create a codec for bean types based on the previously created registry.
    Codec<Bean> BEAN_CODEC = BeanType.REGISTRY.getCodec()
            // And based on that, here's our registry dispatch codec for beans!
            // The first argument is the field name for the bean type.
            // When left out, it will default to "type".
            .dispatch("type", Bean::getType, BeanType::codec);

    BeanType<?> getType();
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

// A record to keep information relating to a specific
// subclass of Bean, in this case only holding a Codec.
public record BeanType<T extends Bean>(MapCodec<T> codec) {
    // Create a registry to map identifiers to bean types
    public static final Registry<BeanType<?>> REGISTRY = new SimpleRegistry<>(
            RegistryKey.ofRegistry(Identifier.of("example", "bean_types")), Lifecycle.stable());
}
1
2
3
4
5
6
7

// An implementing class of Bean, with its own codec.
public class StringyBean implements Bean {
    public static final MapCodec<StringyBean> CODEC = RecordCodecBuilder.mapCodec(instance -> instance.group(
            Codec.STRING.fieldOf("stringy_string").forGetter(StringyBean::getStringyString)
    ).apply(instance, StringyBean::new));

    private String stringyString;

    // It is important to be able to retrieve the
    // BeanType of a Bean from it's instance.
    @Override
    public BeanType<?> getType() {
        return BeanTypes.STRINGY_BEAN;
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

// Another implementation
public class CountingBean implements Bean {
    public static final MapCodec<CountingBean> CODEC = RecordCodecBuilder.mapCodec(instance -> instance.group(
            Codec.INT.fieldOf("counting_number").forGetter(CountingBean::getCountingNumber)
    ).apply(instance, CountingBean::new));

    private int countingNumber;

    @Override
    public BeanType<?> getType() {
        return BeanTypes.COUNTING_BEAN;
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

// An empty class to hold static references to all BeanTypes
public class BeanTypes {
    // Make sure to register the bean types and leave them accessible to
    // the getType method in their respective subclasses.
    public static final BeanType<StringyBean> STRINGY_BEAN = register("stringy_bean", new BeanType<>(StringyBean.CODEC));
    public static final BeanType<CountingBean> COUNTING_BEAN = register("counting_bean", new BeanType<>(CountingBean.CODEC));

    public static <T extends Bean> BeanType<T> register(String id, BeanType<T> beanType) {
        return Registry.register(BeanType.REGISTRY, Identifier.of("example", id), beanType);
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

// Now we can create a codec for bean types
// based on the previously created registry
Codec<BeanType<?>> beanTypeCodec = BeanType.REGISTRY.getCodec();

// And based on that, here's our registry dispatch codec for beans!
// The first argument is the field name for the bean type.
// When left out, it will default to "type".
Codec<Bean> beanCodec = beanTypeCodec.dispatch("type", Bean::getType, BeanType::codec);
1
2
3
4
5
6
7
8
Our new codec will serialize beans to json like this, grabbing only fields that are relevant to their specific type:


{
  "type": "example:stringy_bean",
  "stringy_string": "This bean is stringy!"
}
1
2
3
4

{
  "type": "example:counting_bean",
  "counting_number": 42
}
1
2
3
4
Recursive Codecs
Sometimes it is useful to have a codec that uses itself to decode specific fields, for example when dealing with certain recursive data structures. In vanilla code, this is used for Text objects, which may store other Texts as children. Such a codec can be constructed using Codec#recursive.

For example, let's try to serialize a singly-linked list. This way of representing lists consists of a bunch of nodes that hold both a value and a reference to the next node in the list. The list is then represented by its first node, and traversing the list is done by following the next node until none remain. Here is a simple implementation of nodes that store integers.


public record ListNode(int value, ListNode next) {}
1
We can't construct a codec for this by ordinary means, because what codec would we use for the next field? We would need a Codec<ListNode>, which is what we are in the middle of constructing! Codec#recursive lets us achieve that using a magic-looking lambda:


Codec<ListNode> codec = Codec.recursive(
  "ListNode", // a name for the codec
  selfCodec -> {
    // Here, `selfCodec` represents the `Codec<ListNode>`, as if it was already constructed
    // This lambda should return the codec we wanted to use from the start,
    // that refers to itself through `selfCodec`
    return RecordCodecBuilder.create(instance ->
      instance.group(
        Codec.INT.fieldOf("value").forGetter(ListNode::value),
         // the `next` field will be handled recursively with the self-codec
        Codecs.createStrictOptionalFieldCodec(selfCodec, "next", null).forGetter(ListNode::next)
      ).apply(instance, ListNode::new)
    );
  }
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
A serialized ListNode may then look like this:


{
  "value": 2,
  "next": {
    "value": 3,
    "next": {
      "value": 5
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
References
A much more comprehensive documentation of Codecs and related APIs can be found at the Unofficial DFU JavaDoc.
The general structure of this guide was heavily inspired by the Forge Community Wiki's page on Codecs, a more Forge-specific take on the same topic.

Events

This page is written for version:

1.21.4


Page Authors
Daomephsta
dicedpixels
Draylar
JamiesWhiteShirt
Juuxel
liach
mkpoli
natanfudge
PhoenixVX
SolidBlock-cn
stormyfabric
YanisBft
Fabric API provides a system that allows mods to react to actions or occurrences, also defined as events that occur in the game.

Events are hooks that satisfy common use cases and/or provide enhanced compatibility and performance between mods that hook into the same areas of the code. The use of events often substitutes the use of mixins.

Fabric API provides events for important areas in the Minecraft codebase that multiple modders may be interested in hooking into.

Events are represented by instances of net.fabricmc.fabric.api.event.Event which stores and calls callbacks. Often there is a single event instance for a callback, which is stored in a static field EVENT of the callback interface, but there are other patterns as well. For example, ClientTickEvents groups several related events together.

Callbacks
Callbacks are a piece of code that is passed as an argument to an event. When the event is triggered by the game, the passed piece of code will be executed.

Callback Interfaces
Each event has a corresponding callback interface, conventionally named <EventName>Callback. Callbacks are registered by calling register() method on an event instance, with an instance of the callback interface as the argument.

All event callback interfaces provided by Fabric API can be found in the net.fabricmc.fabric.api.event package.

Listening to Events
This example registers an AttackBlockCallback to damage the player when they hit blocks that don't drop an item when hand-mined.


AttackBlockCallback.EVENT.register((player, world, hand, pos, direction) -> {
    BlockState state = world.getBlockState(pos);

    // Manual spectator check is necessary because AttackBlockCallbacks fire before the spectator check
    if (!player.isSpectator() && player.getMainHandStack().isEmpty() && state.isToolRequired() && world instanceof ServerWorld serverWorld) {
        player.damage(serverWorld, world.getDamageSources().generic(), 1.0F);
    }

    return ActionResult.PASS;
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
Adding Items to Existing Loot Tables
Sometimes you may want to add items to loot tables. For example, adding your drops to a vanilla block or entity.

The simplest solution, replacing the loot table file, can break other mods. What if they want to change them as well? We'll take a look at how you can add items to loot tables without overriding the table.

We'll be adding eggs to the coal ore loot table.

Listening to Loot Table Loading
Fabric API has an event that is fired when loot tables are loaded, LootTableEvents.MODIFY. You can register a callback for it in your mod's initializer. Let's also check that the current loot table is the coal ore loot table.


// :::2
LootTableEvents.MODIFY.register((key, tableBuilder, source, registries) -> {
    // Let's only modify built-in loot tables and leave data pack loot tables untouched by checking the source.
1
2
3
Adding Items to the Loot Table
In loot tables, items are stored in loot pool entries, and entries are stored in loot pools. To add an item, we'll need to add a pool with an item entry to the loot table.

We can make a pool with LootPool#builder, and add it to the loot table.

Our pool doesn't have any items either, so we'll make an item entry using ItemEntry#builder and add it to the pool.


LootTableEvents.MODIFY.register((key, tableBuilder, source, registries) -> {
    // Let's only modify built-in loot tables and leave data pack loot tables untouched by checking the source.
    // We also check that the loot table ID is equal to the ID we want.
    if (source.isBuiltin() && COAL_ORE_LOOT_TABLE_ID.equals(key)) {
        // We make the pool and add an item
        LootPool.Builder poolBuilder = LootPool.builder().with(ItemEntry.builder(Items.EGG));
        tableBuilder.pool(poolBuilder);
    }
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
Custom Events
Some areas of the game do not have hooks provided by the Fabric API, so you can either use a mixin or create your own event.

We'll look at creating an event that is triggered when sheep are sheared. The process of creating an event is:

Creating the event callback interface
Triggering the event from a mixin
Creating a test implementation
Creating the Event Callback Interface
The callback interface describes what must be implemented by event listeners that will listen to your event. The callback interface also describes how the event will be called from our mixin. It is conventional to place an Event object as a field in the callback interface, which will identify our actual event.

For our Event implementation, we will choose to use an array-backed event. The array will contain all event listeners that are listening to the event.

Our implementation will call the event listeners in order until one of them does not return ActionResult.PASS. This means that a listener can say "cancel this", "approve this" or "don't care, leave it to the next event listener" using its return value.

Using ActionResult as a return value is a conventional way to make event handlers cooperate in this fashion.

You'll need to create an interface that has an Event instance and method for response implementation. A basic setup for our sheep shear callback is:


public interface SheepShearCallback {
    Event<SheepShearCallback> EVENT = EventFactory.createArrayBacked(SheepShearCallback.class,
            (listeners) -> (player, sheep) -> {
                for (SheepShearCallback listener : listeners) {
                    ActionResult result = listener.interact(player, sheep);

                    if (result != ActionResult.PASS) {
                        return result;
                    }
                }

                return ActionResult.PASS;
            });

    ActionResult interact(PlayerEntity player, SheepEntity sheep);
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
Let's look at this more in-depth. When the invoker is called, we iterate over all listeners:


(listeners) -> (player, sheep) -> {
    for (SheepShearCallback listener : listeners) {
1
2
We then call our method (in this case, interact) on the listener to get its response:


ActionResult interact(PlayerEntity player, SheepEntity sheep);
1
If the listener says we have to cancel (ActionResult.FAIL) or fully finish (ActionResult.SUCCESS), the callback returns the result and finishes the loop. ActionResult.PASS moves on to the next listener, and in most cases should result in success if there are no more listeners registered:


    if (result != ActionResult.PASS) {
        return result;
    }
}

return ActionResult.PASS;
1
2
3
4
5
6
We can add Javadoc comments to the top of callback classes to document what each ActionResult does. In our case, it might be:


/**
 * Callback for shearing a sheep.
 * Called before the sheep is sheared, items are dropped, and items are damaged.
 * Upon return:
 * - SUCCESS cancels further processing and continues with normal shearing behavior.
 * - PASS falls back to further processing and defaults to SUCCESS if no other listeners are available
 * - FAIL cancels further processing and does not shear the sheep.
 */
1
2
3
4
5
6
7
8
Triggering the Event From a Mixin
We now have the basic event skeleton, but we need to trigger it. Because we want to have the event called when a player attempts to shear a sheep, we call the event invoker in SheepEntity#interactMob when sheared() is called (i.e. sheep can be sheared, and the player is holding shears):


@Mixin(SheepEntity.class)
public class SheepEntityMixin {
    @Inject(at = @At(value = "INVOKE", target = "Lnet/minecraft/entity/passive/SheepEntity;sheared(Lnet/minecraft/server/world/ServerWorld;Lnet/minecraft/sound/SoundCategory;Lnet/minecraft/item/ItemStack;)V"), method = "interactMob", cancellable = true)
    private void onShear(final PlayerEntity player, final Hand hand, final CallbackInfoReturnable<ActionResult> info) {
        ActionResult result = SheepShearCallback.EVENT.invoker().interact(player, (SheepEntity) (Object) this);

        if (result == ActionResult.FAIL) {
            info.setReturnValue(result);
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
Creating a Test Implementation
Now we need to test our event. You can register a listener in your initialization method (or another area, if you prefer) and add custom logic there. Here's an example that drops a diamond instead of wool at the sheep's feet:


SheepShearCallback.EVENT.register((player, sheep) -> {
    sheep.setSheared(true);

    // Create diamond item entity at sheep's position.
    ItemStack stack = new ItemStack(Items.DIAMOND);
    ItemEntity itemEntity = new ItemEntity(player.getWorld(), sheep.getX(), sheep.getY(), sheep.getZ(), stack);
    player.getWorld().spawnEntity(itemEntity);

    return ActionResult.FAIL;
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
If you enter into your game and shear a sheep, a diamond should drop instead of wool.

Networking

This page is written for version:

1.21.4


Page Authors
Daomephsta
dicedpixels
Earthcomputer
FlooferLand
FxMorin
i509VCB
modmuss50
natanfudge
NetUserGet
NShak
parzivail
skycatminepokie
SolidBlock-cn
Voleil
Wxffel
YTG123-Mods
zulrang
Networking in Minecraft is used so the client and server can communicate with each other. Networking is a broad topic, so this page is split up into a few categories.

Why Is Networking Important?
Packets are the core concept of networking in Minecraft. Packets are made up of arbitrary data that can be sent either from server to client or from client to server. Check out the diagram below, which provides a visual representation of the networking architecture in Fabric:

Sided Architecture

Notice how packets are the bridge between the server and the client; that's because almost everything you do in the game involves networking in some way, whether you know it or not. For example, when you send a chat message, a packet is sent to the server with the content. The server then sends another packet to all the other clients with your message.

One important thing to keep in mind is there is always a server running, even in singleplayer and LAN. Packets are still used to communicate between the client and server even when no one else is playing with you. When talking about sides in networking, the terms "logical client" and "logical server" are used. The integrated singleplayer/LAN server and the dedicated server are both logical servers, but only the dedicated server can be considered a physical server.

When state is not synced between the client and server, you can run into issues where the server or other clients don't agree with what another client is doing. This is often known as a "desync". When writing your own mod you may need to send a packet of data to keep the state of the server and all clients in sync.

An Introduction to Networking
Defining a Payload
INFO

A payload is the data that is sent within a packet.

This can be done by creating a Java Record with a BlockPos parameter that implements CustomPayload.


public record SummonLightningS2CPayload(BlockPos pos) implements CustomPayload {
    public static final Identifier SUMMON_LIGHTNING_PAYLOAD_ID = Identifier.of(FabricDocsReference.MOD_ID, "summon_lightning");
    public static final CustomPayload.Id<SummonLightningS2CPayload> ID = new CustomPayload.Id<>(SUMMON_LIGHTNING_PAYLOAD_ID);
    public static final PacketCodec<RegistryByteBuf, SummonLightningS2CPayload> CODEC = PacketCodec.tuple(BlockPos.PACKET_CODEC, SummonLightningS2CPayload::pos, SummonLightningS2CPayload::new);

    @Override
    public Id<? extends CustomPayload> getId() {
        return ID;
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
At the same time, we've defined:

An Identifier used to identify our packet's payload. For this example our identifier will be fabric-docs-reference:summon_lightning.

public static final Identifier SUMMON_LIGHTNING_PAYLOAD_ID = Identifier.of(FabricDocsReference.MOD_ID, "summon_lightning");
1
A public static instance of CustomPayload.Id to uniquely identify this custom payload. We will be referencing this ID in both our common and client code.

public static final CustomPayload.Id<SummonLightningS2CPayload> ID = new CustomPayload.Id<>(SUMMON_LIGHTNING_PAYLOAD_ID);
1
A public static instance of a PacketCodec so that the game knows how to serialize/deserialize the contents of the packet.

public static final PacketCodec<RegistryByteBuf, SummonLightningS2CPayload> CODEC = PacketCodec.tuple(BlockPos.PACKET_CODEC, SummonLightningS2CPayload::pos, SummonLightningS2CPayload::new);
1
We have also overridden getId to return our payload ID.


@Override
public Id<? extends CustomPayload> getId() {
    return ID;
}
1
2
3
4
Registering a Payload
Before we send a packet with our custom payload, we need to register it.

INFO

S2C and C2S are two common suffixes that mean Server-to-Client and Client-to-Server respectively.

This can be done in our common initializer by using PayloadTypeRegistry.playS2C().register which takes in a CustomPayload.Id and a PacketCodec.


PayloadTypeRegistry.playS2C().register(SummonLightningS2CPayload.ID, SummonLightningS2CPayload.CODEC);
1
A similar method exists to register client-to-server payloads: PayloadTypeRegistry.playC2S().register.

Sending a Packet to the Client
To send a packet with our custom payload, we can use ServerPlayNetworking.send which takes in a ServerPlayerEntity and a CustomPayload.

Let's start by creating our Lightning Tater item. You can override use to trigger an action when the item is used. In this case, let's send packets to the players in the server world.


public class LightningTaterItem extends Item {
    public LightningTaterItem(Settings settings) {
        super(settings);
    }

    @Override
    public ActionResult use(World world, PlayerEntity user, Hand hand) {
        if (world.isClient()) {
            return ActionResult.PASS;
        }

        SummonLightningS2CPayload payload = new SummonLightningS2CPayload(user.getBlockPos());

        for (ServerPlayerEntity player : PlayerLookup.world((ServerWorld) world)) {
            ServerPlayNetworking.send(player, payload);
        }

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
19
20
Let's examine the code above.

We only send packets when the action is initiated on the server, by returning early with a isClient check:


if (world.isClient()) {
    return ActionResult.PASS;
}
1
2
3
We create an instance of the payload with the user's position:


SummonLightningS2CPayload payload = new SummonLightningS2CPayload(user.getBlockPos());
1
Finally, we get the players in the server world through PlayerLookup and send a packet to each player.


for (ServerPlayerEntity player : PlayerLookup.world((ServerWorld) world)) {
    ServerPlayNetworking.send(player, payload);
}
1
2
3
INFO

Fabric API provides PlayerLookup, a collection of helper functions that will look up players in a server.

A term frequently used to describe the functionality of these methods is "tracking". It means that an entity or a chunk on the server is known to a player's client (within their view distance) and the entity or block entity should notify tracking clients of changes.

Tracking is an important concept for efficient networking, so that only the necessary players are notified of changes by sending packets.

Receiving a Packet on the Client
To receive a packet sent from a server on the client, you need to specify how you will handle the incoming packet.

This can be done in the client initializer, by calling ClientPlayNetworking.registerGlobalReceiver and passing a CustomPayload.Id and a PlayPayloadHandler, which is a Functional Interface.

In this case, we'll define the action to trigger within the implementation of PlayPayloadHandler implementation (as a lambda expression).


ClientPlayNetworking.registerGlobalReceiver(SummonLightningS2CPayload.ID, (payload, context) -> {
    ClientWorld world = context.client().world;

    if (world == null) {
        return;
    }

    BlockPos lightningPos = payload.pos();
    LightningEntity entity = EntityType.LIGHTNING_BOLT.create(world, SpawnReason.TRIGGERED);

    if (entity != null) {
        entity.setPosition(lightningPos.getX(), lightningPos.getY(), lightningPos.getZ());
        world.addEntity(entity);
    }
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
13
14
15
Let's examine the code above.

We can access the data from our payload by calling the Record's getter methods. In this case payload.pos(). Which then can be used to get the x, y and z positions.


BlockPos lightningPos = payload.pos();
1
Finally, we create a LightningEntity and add it to the world.


LightningEntity entity = EntityType.LIGHTNING_BOLT.create(world, SpawnReason.TRIGGERED);

if (entity != null) {
    entity.setPosition(lightningPos.getX(), lightningPos.getY(), lightningPos.getZ());
    world.addEntity(entity);
}
1
2
3
4
5
6
Now, if you add this mod to a server and when a player uses our Lightning Tater item, every player will see lightning striking at the user's position.

Sending a Packet to the Server
Just like sending a packet to the client, we start by creating a custom payload. This time, when a player uses a Poisonous Potato on a living entity, we request the server to apply the Glowing effect to it.


public record GiveGlowingEffectC2SPayload(int entityId) implements CustomPayload {
    public static final Identifier GIVE_GLOWING_EFFECT_PAYLOAD_ID = Identifier.of(FabricDocsReference.MOD_ID, "give_glowing_effect");
    public static final CustomPayload.Id<GiveGlowingEffectC2SPayload> ID = new CustomPayload.Id<>(GIVE_GLOWING_EFFECT_PAYLOAD_ID);
    public static final PacketCodec<RegistryByteBuf, GiveGlowingEffectC2SPayload> CODEC = PacketCodec.tuple(PacketCodecs.INTEGER, GiveGlowingEffectC2SPayload::entityId, GiveGlowingEffectC2SPayload::new);

    @Override
    public Id<? extends CustomPayload> getId() {
        return ID;
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
We pass in the appropriate codec along with a method reference to get the value from the Record to build this codec.

Then we register our payload in our common initializer. However, this time as Client-to-Server payload by using PayloadTypeRegistry.playC2S().register.


PayloadTypeRegistry.playC2S().register(GiveGlowingEffectC2SPayload.ID, GiveGlowingEffectC2SPayload.CODEC);
1
To send a packet, let's add an action when the player uses a Poisonous Potato. We'll be using the UseEntityCallback event to keep things concise.

We register the event in our client initializer, and we use isClient() to ensure that the action is only triggered on the logical client.


UseEntityCallback.EVENT.register((player, world, hand, entity, hitResult) -> {
    if (!world.isClient()) {
        return ActionResult.PASS;
    }

    ItemStack usedItemStack = player.getStackInHand(hand);

    if (entity instanceof LivingEntity && usedItemStack.isOf(Items.POISONOUS_POTATO) && hand == Hand.MAIN_HAND) {
        GiveGlowingEffectC2SPayload payload = new GiveGlowingEffectC2SPayload(hitResult.getEntity().getId());
        ClientPlayNetworking.send(payload);

        return ActionResult.SUCCESS;
    }

    return ActionResult.PASS;
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
13
14
15
16
We create an instance of our GiveGlowingEffectC2SPayload with the necessary arguments. In this case, the network ID of the targeted entity.


GiveGlowingEffectC2SPayload payload = new GiveGlowingEffectC2SPayload(hitResult.getEntity().getId());
1
Finally, we send a packet to the server by calling ClientPlayNetworking.send with the instance of our GiveGlowingEffectC2SPayload.


ClientPlayNetworking.send(payload);
1
Receiving a Packet on the Server
This can be done in the common initializer, by calling ServerPlayNetworking.registerGlobalReceiver and passing a CustomPayload.Id and a PlayPayloadHandler.


ServerPlayNetworking.registerGlobalReceiver(GiveGlowingEffectC2SPayload.ID, (payload, context) -> {
    Entity entity = context.player().getWorld().getEntityById(payload.entityId());

    if (entity instanceof LivingEntity livingEntity && livingEntity.isInRange(context.player(), 5)) {
        livingEntity.addStatusEffect(new StatusEffectInstance(StatusEffects.GLOWING, 100));
    }
});
1
2
3
4
5
6
7
INFO

It is important that you validate the content of the packet on the server side.

In this case, we validate if the entity exists based on its network ID.


Entity entity = context.player().getWorld().getEntityById(payload.entityId());
1
Additionally, the targeted entity has to be a living entity, and we restrict the range of the target entity from the player to 5.


if (entity instanceof LivingEntity livingEntity && livingEntity.isInRange(context.player(), 5)) {
1
Now when any player tries to use a Poisonous Potato on a living entity, the glowing effect will be applied to it.

Text and Translations

This page is written for version:

1.21.4


Page Authors
IMB11
LordEnder-Kitty
Whenever Minecraft displays text in-game, it's probably defined using a Text object. This custom type is used instead of a String to allow for more advanced formatting, including colors, boldness, obfuscation, and click events. They also allow easy access to the translation system, making it simple to translate any UI element into different languages.

If you've worked with datapacks or functions before, you may see parallels with the json text format used for displayNames, books, and signs among other things. As you can probably guess, this is just a json representation of a Text object, and can be converted to and from using Text.Serializer.

When making a mod, it is generally preferred to construct your Text objects directly in code, making use of translations whenever possible.

Text Literals
The simplest way to create a Text object is to make a literal. This is just a string that will be displayed as-is, by default without any formatting.

These are created using the Text.of or Text.literal methods, which both act slightly differently. Text.of accepts nulls as input, and will return a Text instance. In contrast, Text.literal should not be given a null input, but returns a MutableText, this being a subclass of Text that can be easily styled and concatenated. More about this later.


Text literal = Text.of("Hello, world!");
MutableText mutable = Text.literal("Hello, world!");
// Keep in mind that a MutableText can be used as a Text, making this valid:
Text mutableAsText = mutable;
1
2
3
4
Translatable Text
When you want to provide multiple translations for the same string of text, you can use the Text.translatable method to reference a translation key in any language file. If the key doesn't exist, the translation key is converted to a literal.


Text translatable = Text.translatable("my_mod.text.hello");

// Similarly to literals, translatable text can be easily made mutable.
MutableText mutable = Text.translatable("my_mod.text.bye");
1
2
3
4
The language file, en_us.json, looks like the following:


{
  "my_mod.text.hello": "Hello!",
  "my_mod.text.bye": "Goodbye :("
}
1
2
3
4
If you wish to be able to use variables in the translation, similar to how death messages allow you to use the involved players and items in the translation, you may add said variables as parameters. You may add however many parameters you like.


Text translatable = Text.translatable("my_mod.text.hello", player.getDisplayName());
1
You may reference these variables in the translation like so:


{
  "my_mod.text.hello": "%1$s said hello!"
}
1
2
3
In the game, %1$s will be replaced with the name of the player you referenced in the code. Using player.getDisplayName() will make it so that additional information about the entity will appear in a tooltip when hovering over the name in the chat message as opposed to using player.getName(), which will still get the name; however, it will not show the extra details. Similar can be done with itemStacks, using stack.toHoverableText().

As for what %1$s even means, all you really need to know is that the number corresponds to which variable you are trying to use. Let's say you have three variables that you are using.


Text translatable = Text.translatable("my_mod.text.whack.item", victim.getDisplayName(), attacker.getDisplayName(), itemStack.toHoverableText());
1
If you want to reference what, in our case, is the attacker, you would use %2$s because it's the second variable that we passed in. Likewise, %3$s refers to the itemStack. A translation with this many additional parameters might look like this:


{
  "my_mod.text.whack.item": "%1$s was whacked by %2$s using %3$s"
}
1
2
3
Serializing Text
As mentioned before, you can serialize text to JSON using the text codec. For more information on codecs, see the Codec page.


Gson gson = new Gson();
MutableText mutable = Text.translatable("my_mod.text.bye");
String json = gson.toJson(TextCodecs.CODEC.encodeStart(JsonOps.INSTANCE, mutable).getOrThrow());
1
2
3
This produces JSON that can be used datapacks, commands and other places that accept the JSON format of text instead of literal or translatable text.

Deserializing Text
Furthermore, to deserialize a JSON text object into an actual Text class, again, use the codec.


String jsonString = "...";
Text deserialized = TextCodecs.CODEC
        .decode(JsonOps.INSTANCE, gson.fromJson(jsonString, JsonElement.class))
        .getOrThrow()
        .getFirst();
1
2
3
4
5
Formatting Text
You may be familiar with Minecraft's formatting standards:

You can apply these formatting styles using the Formatting enum on the MutableText class:


MutableText result = Text.literal("Hello World!")
  .formatted(Formatting.AQUA, Formatting.BOLD, Formatting.UNDERLINE);
1
2
Color	Name	Chat Code	MOTD Code	Hex Code
Black
black	§0	\u00A70	#000000
Dark Blue
dark_blue	§1	\u00A71	#0000AA
Dark Green
dark_green	§2	\u00A72	#00AA00
Dark Aqua
dark_aqua	§3	\u00A73	#00AAAA
Dark Red
dark_red	§4	\u00A74	#AA0000
Dark Purple
dark_purple	§5	\u00A75	#AA00AA
Gold
gold	§6	\u00A76	#FFAA00
Gray
gray	§7	\u00A77	#AAAAAA
Dark Gray
dark_gray	§8	\u00A78	#555555
Blue
blue	§9	\u00A79	#5555FF
Green
green	§a	\u00A7a	#55FF55
Aqua
aqua	§b	\u00A7b	#55FFFF
Red
red	§c	\u00A7c	#FF5555
Light Purple
light_purple	§d	\u00A7d	#FF55FF
Yellow
yellow	§e	\u00A7e	#FFFF55
White
white	§f	\u00A7f	#FFFFFF
Reset	§r		
Bold	§l		
Strikethrough	§m		
Underline	§n		
Italic	§o		
Obfuscated	§k		
Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

IDE Tips and Tricks

This page is written for version:

1.21.4


Page Authors
AnAwesomGuy
JR1811
This page gives useful bits of information, to speed up and ease the workflow of developers. Incorporate them into yours, to your liking. It may take some time to learn and get used to the shortcuts and other options. You can use this page as a reference for that.

WARNING

Key binds in the text refer to the default keymap of IntelliJ IDEA, if not stated otherwise. Refer to the File > Settings > Keymap Settings or search for the functionality elsewhere if you are using a different keyboard layout.

Traversing Projects
Manually
IntelliJ has many different ways of traversing projects. If you have generated sources using the ./gradlew genSources commands in the terminal or used the Tasks > fabric > genSources Gradle tasks in the Gradle Window, you can manually go through the source files of Minecraft in the Project Window's External Libraries.

Gradle tasks

The Minecraft source code can be found if you look for net.minecraft in the Project Window's External Libraries. If your project uses split sources from the online Template mod generator, there will be two sources, as indicated by the name (client/common). Additionally other sources of projects, libraries and dependencies, which are imported via the build.gradle file will also be available. This method is often used when browsing for assets, tags and other files.

External Library

Split sources

Search
Pressing Shift twice opens up a Search window. In there you can search for your project's files and classes. When Activating the checkbox include non-project items or by pressing Shift two times again, the search will look not only in your own project, but also in other's, such as the External Libraries.

You can also use the shortcuts ⌘/CTRL+N to search classes and ⌘/CTRL+Shift+N to search all files.

Search window

Recent Window
Another useful tool in IntelliJ is the Recent window. You can open it with the Shortcut ⌘/CTRL+E. In there you can jump to the files, which you have already visited and open tool windows, such as the Structure or Bookmarks window.

Recent window

Traversing Code
Jump to Definition / Usage
If you need to check out either the definition or the usage of variables, methods, classes, and other things, you can press ⌘/CTRL+Left Click / B or use Middle Mouse Button (pressing mouse wheel) on their name. This way you can avoid long scrolling sessions or a manual search for a definition which is located in another file.

You can also use ⌘/CTRL+⌥/Shift+Left Click / B to view all the implementations of a class or interface.

Bookmarks
You can bookmark lines of code, files or even opened Editor tabs. Especially when researching source codes, it can help out to mark spots which you want to find again quickly in the future.

Either right click a file in the Project window, an editor tab or the line number in a file. Creating Mnemonic Bookmarks enables you to quickly switch back to those bookmarks using their hotkeys, ⌘/CTRL and the digit, which you have chosen for it.

set Bookmark

It is possible to create multiple Bookmark lists at the same time if you need to separate or order them, in the Bookmarks window. Breakpoints will also be displayed there.

Bookmark window

Analyzing Classes
Structure of a Class
By opening the Structure window (⌘/Alt+7) you will get an overview of your currently active class. You can see which Classes and Enums are located in that file, which methods have been implemented and which fields and variables are declared.

Sometimes it can be helpful, to activate the Inherited option at the top in the View options as well, when looking for potential methods to override.

Structure window

Type Hierarchy of a Class
By placing the cursor on a class name and pressing ⌘/CTRL+H you can open a new Type Hierarchy window, which shows all parent and child classes.

Type Hierarchy window

Code Utility
Code Completion
Code completion should be activated by default. You will automatically get the recommendations while writing your code. If you closed it by accident or just moved your cursor to a new place, you can use ⌘/CTRL+Space to open them up again.

For example, when using lambda expressions, you can write them quickly using this method.

Lambda with many parameters

Code Generation
The Generate menu can be quickly accessed with Alt+Insert (⌘ Command+N on Mac) or by going to Code at the top and selecting Generate. In a Java file, you will be able to generate constructors, getters, setters, and override or implement methods, and much more. You can also generate accessors and invokers if you have the Minecraft Development plugin installed.

In addition, you can quickly override methods with ⌘/CTRL+O and implement methods with ⌘/CTRL+I.

Code generation menu in a Java file

In a Java test file, you will be given options to generate related testing methods, as follows:

Code generation menu in a Java test file

Displaying Parameters
Displaying parameters should be activated by default. You will automatically get the types and names of the parameters while writing your code. If you closed them by accident or just moved your cursor to a new place, you can use ⌘/CTRL+P to open them up again.

Methods and classes can have multiple implementations with different parameters, also known as Overloading. This way you can decide on which implementation you want to use, while writing the method call.

Displaying method parameters

Refactoring
Refactoring is the process of restructuring code without changing its runtime functionality. Renaming and deleting parts of the code safely is a part of that, but things like extracting parts of the code into separate methods and introducing new variables for repeated code statements are also called "refactoring".

Many IDEs have an extensive tool kit to aid in this process. In IntelliJ simply right click files or parts of the code to get access to the available refactoring tools.

Refactoring

It is especially useful to get used to the Rename refactoring tool's key bind, Shift+F6, since you will likely rename many things in the future. Using this feature, every code occurrence of the renamed code will be renamed and will stay functionally the same.

You are also able to reformat code according to your code style. To do this, select the code you want to reformat (if nothing is selected, the entire file will be reformatted) and press ⌘/CTRL+⌥/ALT+L. To change how IntelliJ formats code, see the settings at File > Settings > Editor > Code Style > Java.

Context Actions
Context actions allow for specific sections of code to be refactored based on the context. To use it, simply move your cursor to the area you want to refactor, and press ⌥/ALT+Enter or click the lightbulb to the left. There will be a popup showing context actions that can be used for the code selected.

Example of context actions

Example of context actions

Search and Replace File Content
Sometimes simpler tools are needed to edit code occurrences.

Keybind	Function
⌘/CTRL+F	Find in current file
⌘/CTRL+R	Replace in current file
⌘/CTRL+Shift+F	Find in a bigger scope (can set specific file type mask)
⌘/CTRL+Shift+R	Replace in a bigger scope (can set specific file type mask)
If used, all of those tools allow for a more specific pattern matching through Regex.

Regex replace

Other Useful Keybinds
Selecting some text and using ⌘/CTRL+Shift+↑ Up / ↓ Down can move the selected text up or down.

In IntelliJ, the keybind for Redo may not be the usual ⌘/CTRL+Y (Delete Line). Instead, it may be ⌘/CTRL+Shift+Z. You can change this in the Keymap.

For even more keyboard shortcuts, you can see IntelliJ's documentation.

Comments
Good code should be easily readable and self-documenting. Picking expressive names for variables, classes and methods can help a lot, but sometimes comments are necessary to leave notes or temporarily disable code for testing.

To comment out code faster, you can select some text and use the ⌘/CTRL+/ (line comment) and ⌘/CTRL+⌥/Shift+/ (block comment) keybinds.

Now you can highlight the necessary code (or just have your cursor on it) and use the shortcuts, to comment the section out.


// private static final int PROTECTION_BOOTS = 2;
private static final int PROTECTION_LEGGINGS = 5;
// private static final int PROTECTION_CHESTPLATE = 6;
private static final int PROTECTION_HELMET = 1;
1
2
3
4

/*
ModItems.initialize();
ModSounds.initializeSounds();
ModParticles.initialize();
*/

private static int secondsToTicks(float seconds) {
    return (int) (seconds * 20 /*+ 69*/);
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
Code Folding
In IntelliJ, next to the line numbers, you may see small arrow icons. They can be used to temporarily collapse methods, if-statements, classes and many other things if you are not actively working on them. To create a custom block which can be collapsed, use the region and endregion comments.


// region collapse block name
    ModBlocks.initialize();
    ModBlockEntities.registerBlockEntityTypes();
    ModItems.initialize();
    ModSounds.initializeSounds();
    ModParticles.initialize();
// endregion
1
2
3
4
5
6
7
Region collapse

WARNING

If you notice that you are using too many of them, consider refactoring your code to make it more readable!

Disabling the Formatter
Comments can also disable the formatter during code refactoring mentioned above by surrounding a piece of code like so:


//formatter:off (disable formatter)
    public static void disgustingMethod() { /* ew this code sucks */ }
//formatter:on (re-enable the formatter)
1
2
3
Suppressing Inspections
//noinspection comments can be used to suppress inspections and warnings. They are functionally identical to the @SuppressWarnings annotation, but without the limitation of being an annotation and can be used on statements.


// below is bad code and IntelliJ knows that

@SuppressWarnings("rawtypes") // annotations can be used here
List list = new ArrayList();

//noinspection unchecked (annotations cannot be here so we use the comment)
this.processList((List<String>)list);

//noinspection rawtypes,unchecked,WriteOnlyObject (you can even suppress multiple!)
new ArrayList().add("bananas");
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
WARNING

If you notice that you are suppressing too many warnings, consider rewriting your code to not produce so many warnings!

TODO and FIXME Notes
When working on code, it can come in handy to leave notes, on what still needs to be taken care of. Sometimes you may also spot a potential issue in the code, but you don't want to stop focusing on the current problem. In this case, use the TODO or FIXME comments.

TODO and FIXME comments

IntelliJ will keep track of them in the TODO window and may notify you, if you commit code, which uses those type of comments.

TODO and FIXME comments

Commit with TODO

Javadocs
A great way of documenting your code is the usage of JavaDoc. JavaDocs not only provide useful information for implementation of methods and classes, but are also deeply integrated into IntelliJ.

When hovering over method or class names, which have JavaDoc comments added to them, they will display this information in their information window.

JavaDoc

To get started, simply write /** above the method or class definition and press enter. IntelliJ will automatically generate lines for the return value and the parameters but you can change them however you want. There are many custom functionalities available and you can also use HTML if needed.

Minecraft's ScreenHandler class has some examples. To toggle the render view, use the pen button near the line numbers.

JavaDoc editing

Optimizing IntelliJ Further
There are many more shortcuts and handy little tricks, which would go above the scope of this page. Jetbrains has many good talks, videos and documentation pages about how to further customize your workspace.

PostFix Completion
Use PostFix Completion to alter code after writing it quickly. Often used examples contain .not, .if, .var, .null, .nn, .for, .fori, .return and .new. Besides the existing ones, you can also create your own in IntelliJ's Settings.

Live Templates
Use Live Templates to generate your custom boilerplate code faster.

More Tips and Tricks
Anton Arhipov from Jetbrains also had an in depth talk about Regex Matching, Code Completion, Debugging and many other topics in IntelliJ.

For even more information, check out Jetbrains' Tips & Tricks site and IntelliJ's documentation. Most of their posts are also applicable to Fabric's ecosystem.

Automated Testing

This page is written for version:

1.21.4


Page Authors
kevinthegreat1
This page explains how to write code to automatically test parts of your mod. There are two ways to automatically test your mod: unit tests with Fabric Loader JUnit or game tests with the Gametest framework from Minecraft.

Unit tests should be used to test components of your code, such as methods and helper classes, while game tests spin up an actual Minecraft client and server to run your tests, which makes it suitable for testing features and gameplay.

Unit Testing
Since Minecraft modding relies on runtime byte-code modification tools such as Mixin, simply adding and using JUnit normally would not work. That's why Fabric provides Fabric Loader JUnit, a JUnit plugin that enables unit testing in Minecraft.

Setting up Fabric Loader JUnit
First, we need to add Fabric Loader JUnit to the development environment. Add the following to your dependencies block in your build.gradle:


testImplementation "net.fabricmc:fabric-loader-junit:${project.loader_version}"
1
Then, we need to tell Gradle to use Fabric Loader JUnit for testing. You can do so by adding the following code to your build.gradle:


test {
    useJUnitPlatform()
}
1
2
3
Writing Tests
Once you reload Gradle, you're now ready to write tests.

These tests are written just like regular JUnit tests, with a bit of additional setup if you want to access any registry-dependent class, such as ItemStack. If you're comfortable with JUnit, you can skip to Setting Up Registries.

Setting Up Your First Test Class
Tests are written in the src/test/java directory.

One naming convention is to mirror the package structure of the class you are testing. For example, to test src/main/java/com/example/docs/codec/BeanType.java, you'd create a class at src/test/java/com/example/docs/codec/BeanTypeTest.java. Notice how we added Test to the end of the class name. This also allows you to easily access package-private methods and fields.

Another naming convention is to have a test package, such as src/test/java/com/example/docs/test/codec/BeanTypeTest.java. This prevents some problems that may arise with using the same package if you use Java modules.

After creating the test class, use ⌘/CTRLN to bring up the Generate menu. Select Test and start typing your method name, usually starting with test. Press ENTER when you're done. For more tips and tricks on using the IDE, see IDE Tips and Tricks.

Generating a test method

You can, of course, write the method signature by hand, and any instance method with no parameters and a void return type will be identified as a test method. You should end up with the following:

A blank test method with test indicators

Notice the green arrow indicators in the gutter: you can easily run a test by clicking them. Alternately, your tests will run automatically on every build, including CI builds such as GitHub Actions. If you're using GitHub Actions, don't forget to read Setting Up GitHub Actions.

Now, it's time to write your actual test code. You can assert conditions using org.junit.jupiter.api.Assertions. Check out the following test:


public class BeanTypeTest {
    private static final Gson GSON = new GsonBuilder().create();

    @BeforeAll
    static void beforeAll() {
        BeanTypes.register();
    }

    @Test
    void testBeanCodec() {
        StringyBean expectedBean = new StringyBean("This bean is stringy!");
        Bean actualBean = Bean.BEAN_CODEC.parse(JsonOps.INSTANCE, GSON.fromJson("{\"type\":\"example:stringy_bean\",\"stringy_string\":\"This bean is stringy!\"}", JsonObject.class)).getOrThrow();

        Assertions.assertInstanceOf(StringyBean.class, actualBean);
        Assertions.assertEquals(expectedBean.getType(), actualBean.getType());
        Assertions.assertEquals(expectedBean.getStringyString(), ((StringyBean) actualBean).getStringyString());
    }

    @Test
    void testDiamondItemStack() {
        // I know this isn't related to beans, but I need an example :)
        ItemStack diamondStack = new ItemStack(Items.DIAMOND, 65);

        Assertions.assertTrue(diamondStack.isOf(Items.DIAMOND));
        Assertions.assertEquals(65, diamondStack.getCount());
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
For an explanation of what this code actually does, see Codecs.

Setting Up Registries
Great, the first test worked! But wait, the second test failed? In the logs, we get one of the following errors.


java.lang.ExceptionInInitializerError
    at net.minecraft.item.ItemStack.<clinit>(ItemStack.java:94)
    at com.example.docs.codec.BeanTypeTest.testBeanCodec(BeanTypeTest.java:20)
    at java.base/java.lang.reflect.Method.invoke(Method.java:580)
    at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
    at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
Caused by: java.lang.IllegalArgumentException: Not bootstrapped (called from registry ResourceKey[minecraft:root / minecraft:game_event])
    at net.minecraft.Bootstrap.createNotBootstrappedException(Bootstrap.java:118)
    at net.minecraft.Bootstrap.ensureBootstrapped(Bootstrap.java:111)
    at net.minecraft.registry.Registries.create(Registries.java:238)
    at net.minecraft.registry.Registries.create(Registries.java:229)
    at net.minecraft.registry.Registries.<clinit>(Registries.java:139)
    ... 5 more

Not bootstrapped (called from registry ResourceKey[minecraft:root / minecraft:game_event])
java.lang.IllegalArgumentException: Not bootstrapped (called from registry ResourceKey[minecraft:root / minecraft:game_event])
    at net.minecraft.Bootstrap.createNotBootstrappedException(Bootstrap.java:118)
    at net.minecraft.Bootstrap.ensureBootstrapped(Bootstrap.java:111)
    at net.minecraft.registry.Registries.create(Registries.java:238)
    at net.minecraft.registry.Registries.create(Registries.java:229)
    at net.minecraft.registry.Registries.<clinit>(Registries.java:139)
    at net.minecraft.item.ItemStack.<clinit>(ItemStack.java:94)
    at com.example.docs.codec.BeanTypeTest.testBeanCodec(BeanTypeTest.java:20)
    at java.base/java.lang.reflect.Method.invoke(Method.java:580)
    at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
    at java.base/java.util.ArrayList.forEach(ArrayList.java:1596)
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
This is because we're trying to access the registry or a class that depends on the registry (or, in rare cases, depends on other Minecraft classes such as SharedConstants), but Minecraft has not been initialized. We just need to initialize it a little bit to have registries working. Simply add the following code to the beginning of your beforeAll method.


SharedConstants.createGameVersion();
Bootstrap.initialize();
1
2
Setting Up GitHub Actions
INFO

This section assumes that you are using the standard GitHub Action workflow included with the example mod and with the mod template.

Your tests will now run on every build, including those by CI providers such as GitHub Actions. But what if a build fails? We need to upload the logs as an artifact so we can view the test reports.

Add this to your .github/workflows/build.yml file, below the ./gradlew build step.


- name: Store reports
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: reports
    path: |
      **/build/reports/
      **/build/test-results/
1
2
3
4
5
6
7
8
Game Tests
Minecraft provides the game test framework for testing server-side features. Fabric additionally provides client game tests for testing client-side features, similar to an end-to-end test.

Setting up Game Tests with Fabric Loom
Both server and client game tests can be set up manually or with Fabric Loom. This guide will use Loom.

To add game tests to your mod, add the following to your build.gradle:


fabricApi {
    configureTests {
        createSourceSet = true
        modId = "fabric-docs-reference-test-${project.name}"
        eula = true
    }
}
1
2
3
4
5
6
7
To see all available options, see the Loom documentation on tests.

Setting up Game Test Directory
INFO

You only need this section if you enabled createSourceSet, which is recommended. You can, of course, do your own gradle magic, but you'll be on your own.

If you enabled createSourceSet like the example above, your gametest will be in a separate source set with a separate fabric.mod.json. The module name defaults to gametest. Create a new fabric.mod.json in src/gametest/resources/ as shown:


{
  "schemaVersion": 1,
  "id": "fabric-docs-reference-test",
  "version": "1.0.0",
  "name": "Fabric docs reference",
  "icon": "assets/fabric-docs-reference/icon.png",
  "environment": "*",
  "entrypoints": {
    "fabric-gametest": [
      "com.example.docs.FabricDocsGameTest"
    ],
    "fabric-client-gametest": [
      "com.example.docs.FabricDocsClientGameTest"
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
13
14
15
16
Note that this fabric.mod.json expects a server game test at src/gametest/java/com/example/docs/FabricDocsGameTest, and a client game test at src/gametest/java/com/example/docs/FabricDocsClientGameTest.

Writing Game Tests
You can now create server and client game tests in the src/gametest/java directory. Here is a basic example for each:


Server

Client

package com.example.docs;

import net.minecraft.block.Blocks;
import net.minecraft.test.GameTest;
import net.minecraft.test.TestContext;

import net.fabricmc.fabric.api.gametest.v1.FabricGameTest;

public class FabricDocsGameTest implements FabricGameTest {
    @GameTest(templateName = EMPTY_STRUCTURE)
    public void test(TestContext context) {
        context.expectBlock(Blocks.AIR, 0, 0, 0);
        context.complete();
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
See the respective Javadocs in Fabric API for more info.

Loom

This page is written for version:

1.21.4


Page Authors
Atakku
caoimhebyrne
Daomephsta
JamiesWhiteShirt
Juuxel
kb-1000
modmuss50
SolidBlock-cn
Fabric Loom, or just Loom for short, is a Gradle plugin for development of mods in the Fabric ecosystem.

Loom provides utilities to install Minecraft and mods in a development environment so that you can link against them with respect to Minecraft obfuscation and its differences between distributions and versions. It also provides run configurations for use with Fabric Loader, Mixin compile processing and utilities for Fabric Loader's jar-in-jar system.

Loom supports all versions of Minecraft, even those not officially supported by Fabric API, because it is version-independent.

This page is a reference of all options and features of Loom. If you are just getting started, please see the Getting Started page.

Depending on Subprojects
While setting up a multi-project build that depends on another Loom project, you should use the namedElements configuration when depending on the other project. By default, a project's "outputs" are remapped to intermediary names. The namedElements configuration contains the project outputs that have not been remapped.


dependencies {
 implementation project(path: ":name", configuration: "namedElements")
}
1
2
3
If you are using split source sets in a multi-project build, you will also need to add a dependency for the other project's client source set.


dependencies {
 clientImplementation project(":name").sourceSets.client.output
}
1
2
3
Split Client & Common Code
For years, a common source of server crashes had been mods accidentally calling client-only code when installed on a server. Newer Loom and Loader versions provide an option to require all client code to be moved into its own source set. This is to stop the problem at compile time, but the build will still result in a single jar file which works on either side.

The following snippet from a build.gradle file shows how you can enable this for your mod. As your mod will now be split across two source sets, you will need to use the new DSL to define your mod's source sets. This enables Fabric Loader to group your mod's classpath together. This is also useful for some other complex multi-project setups.

Minecraft 1.18 (1.19 recommended), Loader 0.14 and Loom 1.0 or later are required to split the client and common code.


loom {
 splitEnvironmentSourceSets()

 mods {
   modid {
     sourceSet sourceSets.main
     sourceSet sourceSets.client
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
Resolving Issues
Loom and/or Gradle can sometimes fail due to corrupted cache files. Running ./gradlew build --refresh-dependencies will force Gradle and Loom to re-download and recreate all files. This may take a few minutes, but is usually enough to resolve cache-related issues.

Development Environment Setup
Loom is designed to work out-of-the-box, by simply setting up a workspace in your IDE. It does quite a few things behind the scenes to create a development environment with Minecraft:

Downloads the client and server jar from official channels for the configured version of Minecraft
Merges the client and server jar to produce a merged jar with @Environment and @EnvironmentInterface annotations
Downloads the configured mappings
Remaps the merged jar with intermediary mappings to produce an intermediary jar
Remaps the merged jar with Yarn mappings to produce a mapped jar
Optional: Decompiles the mapped jar to produce a mapped sources jar and linemap, and applies the linemap to the mapped jar
Adds Minecraft dependencies
Downloads Minecraft assets
Processes and includes mod-augmented dependencies
Caches
${GRADLE_HOME}/caches/fabric-loom: The user cache, a cache shared by all Loom projects for a user. Used to cache Minecraft assets, jars, merged jars, intermediary jars and mapped jars
.gradle/loom-cache: The root project persistent cache, a cache shared by a project and its subprojects. Used to cache remapped mods, as well as generated included mod jars
**/build/loom-cache: The (sub)projects' build cache
Dependency Configurations
minecraft: Defines the version of Minecraft to be used in the development environment
mappings: Defines the mappings to be used in the development environment
modImplementation, modApi and modRuntime: Augmented variants of implementation, api and runtime for mod dependencies. Will be remapped to match the mappings in the development environment and has any nested jars removed
include: Declares a dependency that should be included as a jar-in-jar in the remapJar output. This dependency configuration is not transitive. For non-mod dependencies, Loom will generate a mod jar with a fabric.mod.json using the mod ID for the name, and the same version
Default Configuration
Applies the following plugins: java, eclipse
Adds the following Maven repositories: Fabric, Mojang and Maven Central
Configures the eclipse task to be finalized by the genEclipseRuns task
If an .idea folder exists in the root project, downloads assets (if not up-to-date) and installs run configurations in .idea/runConfigurations
Adds net.fabricmc:fabric-mixin-compile-extensions and its dependencies with the annotationProcessor dependency configuration
Configures all non-test JavaCompile tasks with configurations for the Mixin annotation processor
Configures the remapJar task to output a JAR with the same name as the jar task output, then adds a "dev" classifier to the jar task
Configures the remapSourcesJar task to process the sourcesJar task output if the task exists
Adds the remapJar task and the remapSourcesJar task as dependencies of the build task
Configures the remapJar task and the remapSourcesJar task to add their outputs as archives artifacts when executed
For each MavenPublication (from the maven-publish plugin), manually appends dependencies to the POM for mod-augmented dependency configurations, provided the dependency configuration has a Maven scope
All run configurations have the run directory ${projectDir}/run and the VM argument -Dfabric.development=true. The main classes for run configurations are usually defined by a fabric-installer.json file in the root of Fabric Loader's JAR file when it is included as a mod dependency, but the file can be defined by any mod dependency. If no such file is found, the main classes default to net.fabricmc.loader.launch.knot.KnotClient and net.fabricmc.loader.launch.knot.KnotServer.

Developer Guides

This page is written for version:

1.21.4


Written by the community, these guides cover a wide range of topics, from setting up your development environment to more advanced areas like rendering and networking.

Check out the sidebar for a list of all available guides. If you're searching for something specific, the search bar at the top of the page is your best friend.

Remember: a fully-working mod with all the code for this documentation is available in the /reference folder on GitHub.

If you want to contribute to the Fabric Documentation, you can find the source code on GitHub, and the relevant contribution guidelines.
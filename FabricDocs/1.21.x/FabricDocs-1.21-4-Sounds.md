Playing Sounds

This page is written for version:

1.21.4


Page Authors
JR1811
Minecraft has a big selection of sounds which you can choose from. Check out the SoundEvents class to view all the vanilla sound event instances that Mojang has provided.

Using Sounds in Your Mod
Make sure to execute the playSound() method on the logical server side when using sounds!

In this example, the useOnEntity() and useOnBlock() methods for a custom interactive item are used to play a "placing copper block" and a pillager sound.


@Override
public ActionResult useOnEntity(ItemStack stack, PlayerEntity user, LivingEntity entity, Hand hand) {
    // As stated above, don't use the playSound() method on the client side
    // ... it won't work!
    if (!entity.getWorld().isClient()) {
        // Play the sound as if it was coming from the entity.
        entity.playSound(SoundEvents.ENTITY_PILLAGER_AMBIENT, 2f, 0.7f);
    }

    return super.useOnEntity(stack, user, entity, hand);
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
The playSound() method is used with the LivingEntity object. Only the SoundEvent, the volume and the pitch need to be specified. You can also use the playSound() method from the world instance to have a higher level of control.


@Override
public ActionResult useOnBlock(ItemUsageContext context) {
    if (!context.getWorld().isClient()) {
        // Play the sound and specify location, category and who made the sound.
        // No entity made the sound, so we specify null.
        context.getWorld().playSound(null, context.getBlockPos(),
                SoundEvents.BLOCK_COPPER_PLACE, SoundCategory.PLAYERS,
                1f, 1f);
    }

    return super.useOnBlock(context);
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
SoundEvent and SoundCategory
The SoundEvent defines which sound will be played. You can also register your own SoundEvents to include your own sound.

Minecraft has several audio sliders in the in-game settings. The SoundCategory enum is used to determine which slider will adjust your sound's volume.

Volume and Pitch
The volume parameter can be a bit misleading. In the range of 0.0f - 1.0f the actual volume of the sound can be changed. If the number gets bigger than that, the volume of 1.0f will be used and only the distance, in which your sound can be heard, gets adjusted. The block distance can be roughly calculated by volume * 16.

The pitch parameter increases or decreases the pitch value and also changes the duration of the sound. In the range of (0.5f - 1.0f) the pitch and the speed gets decreased, while bigger numbers will increase the pitch and the speed. Numbers below 0.5f will stay at the pitch value of 0.5f.

Creating Custom Sounds

This page is written for version:

1.21.4


Page Authors
JR1811
Preparing the Audio File
Your audio files need to be formatted in a specific way. OGG Vorbis is an open container format for multimedia data, such as audio, and is used in case of Minecraft's sound files. To avoid problems with how Minecraft handles distancing, your audio needs to have only a single channel (Mono).

Many modern DAWs (Digital Audio Workstation) software can import and export using this file format. In the following example the free and open-source software "Audacity" will be used to bring the audio file into the right format, however any other DAW should suffice as well.

Unprepared audio file in Audacity

In this example, a sound of a whistle is imported into Audacity. It currently is saved as a .wav file and has two audio channels (Stereo). Edit the sound to your liking and make sure to delete one of the channels using the drop-down element at the top of the "track head".

Splitting Stereo track

Deleting one of the channels

When exporting or rendering the audio file, make sure to choose the OGG file format. Some DAWs, like REAPER, might support multiple OGG audio layer formats. In this case OGG Vorbis should work just fine.

Exporting as OGG file

Also keep in mind that audio files can increase the file size of your mod drastically. If needed, compress the audio when editing and exporting the file to keep the file size of your finished product to a minimum.

Loading the Audio File
Add the new resources/assets/mod-id/sounds directory for the sounds in your mod, and put the exported audio file metal_whistle.ogg in there.

Continue with creating the resources/assets/mod-id/sounds.json file if it doesn't exist yet and add your sound to the sound entries.


{
  "metal_whistle": {
    "subtitle": "sound.fabric-docs-reference.metal_whistle",
    "sounds": [
      "fabric-docs-reference:metal_whistle"
    ]
  },
  "engine": {
    "subtitle": "sound.fabric-docs-reference.engine",
    "sounds": [
      "fabric-docs-reference:engine"
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
The subtitle entry provides more context for the player. The subtitle name is used in the language files in the resources/assets/mod-id/lang directory and will be displayed if the in-game subtitle setting is turned on and this custom sound is being played.

Registering the Custom Sound
To add the custom sound to the mod, register a SoundEvent in your mod's initializer.


Registry.register(Registries.SOUND_EVENT, Identifier.of(MOD_ID, "metal_whistle"),
        SoundEvent.of(Identifier.of(MOD_ID, "metal_whistle")));
1
2
Cleaning up the Mess
Depending on how many Registry entries there are, this can get messy quickly. To avoid that, we can make use of a new helper class.

Add two new methods to the newly created helper class. One, which registers all the sounds and one which is used to initialize this class in the first place. After that you can comfortably add new custom SoundEvent static class variables as needed.


public class CustomSounds {
    private CustomSounds() {
        // private empty constructor to avoid accidental instantiation
    }

    // ITEM_METAL_WHISTLE is the name of the custom sound event
    // and is called in the mod to use the custom sound
    public static final SoundEvent ITEM_METAL_WHISTLE = registerSound("metal_whistle");
    public static final SoundEvent ENGINE_LOOP = registerSound("engine");

    // actual registration of all the custom SoundEvents
    private static SoundEvent registerSound(String id) {
        Identifier identifier = Identifier.of(FabricDocsReferenceSounds.MOD_ID, id);
        return Registry.register(Registries.SOUND_EVENT, identifier, SoundEvent.of(identifier));
    }

    // This static method starts class initialization, which then initializes
    // the static class variables (e.g. ITEM_METAL_WHISTLE).
    public static void initialize() {
        FabricDocsReferenceSounds.LOGGER.info("Registering " + FabricDocsReferenceSounds.MOD_ID + " Sounds");
        // Technically this method can stay empty, but some developers like to notify
        // the console, that certain parts of the mod have been successfully initialized
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
This way, the mod's initializer only needs to implement one line to register all custom SoundEvents.


public class FabricDocsReferenceSounds implements ModInitializer {
    public static final String MOD_ID = FabricDocsReference.MOD_ID;
    public static final Logger LOGGER = FabricDocsReference.LOGGER;

    @Override
    public void onInitialize() {
        // This is the basic registering. Use a new class for registering sounds
        // instead, to keep the ModInitializer implementing class clean!
        Registry.register(Registries.SOUND_EVENT, Identifier.of(MOD_ID, "metal_whistle_simple"),
                SoundEvent.of(Identifier.of(MOD_ID, "metal_whistle_simple")));

        // ... the cleaner approach.
        CustomSounds.initialize(); 
    }

    public static Identifier identifierOf(String path) {
        return Identifier.of(FabricDocsReference.MOD_ID, path);
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
Using the Custom SoundEvent
Use the helper class to access the custom SoundEvent. Check out the Playing SoundEvents page to learn how to play sounds.

Dynamic and Interactive Sounds

This page is written for version:

1.21.4


Page Authors
JR1811
PREREQUISITES

This page builds on top of the Playing Sounds and the Creating Custom Sounds pages!

Problems with SoundEvents
As we have learned in the Using Sounds page, it is preferable to use SoundEvents on a logical server side, even if it is a bit counterintuitive. After all, a client needs to handle the sound, which is transmitted to your headphones, right?

This way of thinking is correct. Technically the client side needs to handle the audio. However, for simple SoundEvent playing, the server side prepared a big step in between which might not be obvious at first glance. Which clients should be able to hear that sound?

Using the sound on the logical server side will solve the issue of broadcasting SoundEvents. To put simple, every client (ClientPlayerEntity) in tracking range, gets sent a network packet to play this specific sound. The sound event is basically broadcasted from the logical server side, to every participating client, without you having to think about it at all. The sound is played once, with the specified volume and pitch values.

But what if this is not enough? What if the sound needs to loop, change volume and pitch dynamically while playing and all that based on values which come from things like Entities or BlockEntities?

The simple way of using the SoundEvent on the logical server side is not enough for this use case.

Preparing the Audio File
We are going to create a new looping audio for another SoundEvent. If you can find an audio file which is looping seamlessly already, you can just follow the steps from Creating Custom Sounds. If the sound is not looping perfectly yet, we will have to prepare it for that.

Again, most modern DAWs (Digital Audio Workstation Software) should be capable of this, but I like to use Reaper if the audio editing is a bit more involved.

Set Up
Our starting sound will be coming from an engine.

Let's load the file into our DAW of choice.

Reaper loaded with the audio file

We can hear and see, that the engine gets started in the beginning and stopped at the end which is not great for looping sounds. Let's cut those out and adjust the time selection handles to match the new length. Also enable the Toggle Repeat mode to let the audio loop, while we adjust it.

Trimmed audio file

Removing Disruptive Audio Elements
If we listen closely, there is a beeping noise in the background, which could've come from the machine. I think, that this wouldn't sound great in-game, so lets try to remove it.

It is a constant sound which keeps its frequency over the length of the audio. So a simple EQ filter should be enough to filter it out.

Reaper comes with an EQ filter equipped already, called "ReaEQ". This might be located somewhere else and named differently in other DAWs but using EQ is standard in most DAWs nowadays.

If you are sure that your DAW doesn't have an EQ filter available, check for free VST alternatives online which you may be able to install in your DAW of choice.

In Reaper use the Effects Window to add the "ReaEQ" audio effect, or any other EQ.

Adding an EQ filter

If we play the audio now, while keeping the EQ filter window open, the EQ filter will show the incoming audio in its display. We can see many bumps there.

Identifying the problem

If you are not a trained audio engineer, this part is mostly about experimenting and "trial and error". There is a pretty harsh bump between node 2 and 3. Let's move the nodes so, that we lower the frequency only for that part.

Lowered the bad frequency

Also, other effects can be achieved with a simple EQ filter. For example, cutting high and/or low frequencies can give the impression of radio transmitted sounds.

You can also layer more audio files, change the pitch, add some reverb or use more elaborate sound effects like "bit-crusher". Sound design can be fun, especially if you find good sounding variations of your audio by accident. Experimenting is key and maybe your sound might end up even better than before.

We will only continue with the EQ filter, which we used to cut out the problematic frequency.

Comparison
Let's compare the original file with the cleaned up version.

You can hear a distinct humming and beeping sound from maybe an electrical element of the engine, in the original sound.

With an EQ filter we were able to remove it almost completely. It is definitely more pleasant to listen to.

Making It Loop
If we let the sound play to the end and let it start from the beginning again, we can clearly hear the transition happening. The goal is to get rid of this by applying a smooth transition.

Start by cutting a piece from the end, which is as big as you want the transition to be, and place it on the beginning of a new audio track. In Reaper, you can split the audio by simply moving the cursor to the position of the cut and pressing S.

Cut the end and move it to a new track

You may have to copy the EQ audio effect of the first audio track over to the second one too.

Now let the end piece of the new track fade out and let the start of the first audio track fade in.

Looping with fading audio tracks

Exporting
Export the audio with the two audio tracks, but with only one audio channel (Mono) and create a new SoundEvent for that .ogg file in your mod. If you are not sure how to do that, take a look at the Creating Custom Sounds page.

This is now the finished looping engine audio for the SoundEvent called ENGINE_LOOP.

Using a SoundInstance
To play sounds on the client side, a SoundInstance is needed. They still make use of SoundEvent though.

If you only want to play something like a click on a UI element, there is already the existing PositionedSoundInstance class.

Keep in mind that this will only be played on the specific client, which executed this part of the code.


MinecraftClient client = MinecraftClient.getInstance();
client.getSoundManager().play(PositionedSoundInstance.master(SoundEvents.UI_BUTTON_CLICK, 1.0F));
1
2
WARNING

Please note that in the AbstractSoundInstance class, which SoundInstances inherit from, has the @Environment(EnvType.CLIENT) annotation.

This means that this class (and all its subclasses) will only be available to the client side.

If you try using that in a logical server side context, you may not notice the issue in Single player at first, but a server in a Multiplayer environment will crash, since it won't be able to find that part of the code at all.

If you struggle with those issues, it is recommended to create your mod from the Online template generator with the Split client and common sources option turned on.

A SoundInstance can be more powerful than just playing sounds once.

Check out the AbstractSoundInstance class and what kind of values it can keep track of. Besides the usual volume and pitch variables, it also holds XYZ coordinates and if it should repeat itself after finishing the SoundEvent.

Then taking a look at its subclass, MovingSoundInstance we get the TickableSoundInstance interface introduced too, which adds ticking functionality to the SoundInstance.

So to make use of those utilities, simply create a new class for your custom SoundInstance and extend from MovingSoundInstance.


public class CustomSoundInstance extends MovingSoundInstance {
    private final LivingEntity entity;

    public CustomSoundInstance(LivingEntity entity, SoundEvent soundEvent, SoundCategory soundCategory) {
        super(soundEvent, soundCategory, SoundInstance.createRandom());
        // In this constructor we also add the sound source (LivingEntity) of
        // the SoundInstance and store it in the current object
        this.entity = entity;
        // set up default values when the sound is about to start
        this.volume = 1.0f;
        this.pitch = 1.0f;
        this.repeat = true;
        this.setPositionToEntity();
    }

    @Override
    public void tick() {
        // stop sound instantly if sound source does not exist anymore
        if (this.entity == null || this.entity.isRemoved() || this.entity.isDead()) {
            this.setDone();
            return;
        }

        // move sound position over to the new position for every tick
        this.setPositionToEntity();
    }

    @Override
    public boolean shouldAlwaysPlay() {
        // override to true, so that the SoundInstance can start
        // or add your own condition to the SoundInstance, if necessary
        return true;
    }

    // small utility method to move the sound instance position
    // to the sound source's position
    private void setPositionToEntity() {
        this.x = this.entity.getX();
        this.y = this.entity.getY();
        this.z = this.entity.getZ();
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
37
38
39
40
41
42
Using your custom Entity or BlockEntity instead of that basic LivingEntity instance can give you even more control e.g. in the tick() method based on accessor methods, but you don't necessarily need a reference to a sound source like that. Instead, you could also access a BlockPos from somewhere else or even set it by hand once in the constructor only.

Just keep in mind that all the referenced objects in the SoundInstance are the versions from the client side. In specific situations, a logical server side entity's properties can differ from its client side counterpart. If you notice that your values don't line up, make sure that your values are synchronized either with entity's TrackedData, BlockEntity S2C packets or complete custom S2C network packets.

After you have finished creating your custom SoundInstance, it's ready to be used anywhere as long as it's been executed on the client side using the sound manager. In the same way, you can also stop the custom SoundInstance manually, if necessary.


CustomSoundInstance instance = new CustomSoundInstance(client.player, CustomSounds.ENGINE_LOOP, SoundCategory.NEUTRAL);

// play the sound instance
client.getSoundManager().play(instance);

// stop the sound instance
client.getSoundManager().stop(instance);
1
2
3
4
5
6
7
The sound loop will be played now only for the client, which ran that SoundInstance. In this case, the sound will follow the ClientPlayerEntity itself.

This concludes the explanation of creating and using a simple custom SoundInstance.

Advanced Sound Instances
WARNING

The following content covers an advanced topic.

At this point you should have a solid grasp on Java, object-oriented programming, generics and callback systems.

Having knowledge on Entities, BlockEntities and custom networking will also help a lot in understanding the use case and the applications of advanced sounds.

To show an example of how more elaborate SoundInstance systems can be created, we will add extra functionality, abstractions and utilities to make working with such sounds in a bigger scope, easier, more dynamic and flexible.

Theory
Let's think about what our goal with the SoundInstance is.

The sound should loop as long as the linked custom EngineBlockEntity is running
The SoundInstance should move around, following its custom EngineBlockEntity's position (The BlockEntity won't move, so this might be more useful on Entities)
We need smooth transitions. Turning it on or off should almost never be instant.
Change volume and pitch based on external factors (e.g. from the source of the sound)
To summarize, we need to keep track of an instance of a custom BlockEntity, adjust volume and pitch values while the SoundInstance is running based on values from that custom BlockEntity and implement "Transition States".

If you plan on making several different SoundInstances, which behave in different ways, I would recommend creating a new abstract AbstractDynamicSoundInstance class, which implements the default behavior and let the actual custom SoundInstance classes extend from it.

If you just plan on using a single one, you can skip the abstract super class, and instead implement that functionality in your custom SoundInstance class directly.

In addition it will be a good idea to have a centralized place, where the SoundInstance's are being tracked, played and stopped. This means that it needs to handle incoming signals, e.g. from custom S2C Network Packets, list all currently running instances and handle special cases, for example which sounds are allowed to play at the same time and which sounds could potentially disable other sounds upon activation. For that a new DynamicSoundManager class can be created, to easily interact with this sound system.

Overall our sound system might look like this, when we are done.

Structure of the custom Sound System

INFO

All of those enums, interfaces and classes will be newly created. Adjust the system and utilities to your specific use case as you see fit. This is only an example of how you can approach such topics.

DynamicSoundSource Interface
If you choose to create a new, more modular, custom AbstractDynamicSoundInstance class as a super class, you may want to reference not only a single type of Entity but different ones, or even a BlockEntity too.

In that case, making use of abstraction is the key. Instead of referencing e.g. a custom BlockEntity directly, only keeping track of an interface, which provides the data, solves that problem.

Going forward we will make use of a custom interface called DynamicSoundSource. It is implemented in all classes which want to make use of that dynamic sound functionality, like your custom BlockEntity, Entities or even, using Mixins, on already existing classes, like ZombieEntity. It basically represents only the necessary data of the sound source.


public interface DynamicSoundSource {
    // gets access to how many ticks have passed for e.g. a BlockEntity instance
    int getTick();

    // gets access to where currently this instance is placed in the world
    Vec3d getPosition();

    // holds a normalized (range of 0-1) value, showing how much stress this instance is currently experiencing
    // It is more or less just an arbitrary value, which will cause the sound to change its pitch while playing.
    float getNormalizedStress();
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
After creating this interface, make sure to implement it in the necessary classes too.

INFO

This is a utility, which may be used on both the client and the logical server side.

So this interface should be stored in the common packages instead of the client only packages, if you make use of the "split sources" option.

TransitionState Enum
As mentioned earlier, you could stop running SoundInstances with the client's SoundManager, but this will cause the SoundInstance to go silent instantly. Our goal is, when a stopping signal comes in, to not stop the sound but to execute an ending phase of its "Transition State". Only after the ending phase is finished the custom SoundInstance should be stopped.

A TransitionState is a newly created enum, which contains three values. They will be used to keep track on what phase the sound should be in.

STARTING Phase: sound starts silent, but slowly increases in volume
RUNNING Phase: sound is running normally
ENDING Phase: sound starts at the original volume and slowly decreases until it is silent
Technically a simple enum with the phases can be enough.


public enum TransitionState {
    STARTING, RUNNING, ENDING
}
1
2
3
But when those values are sent over the network you might want to define an Identifier for them or even add other custom values.


public enum TransitionState {
    STARTING("starting_phase"),
    RUNNING("idle_phase"),
    ENDING("ending_phase");

    private final Identifier identifier;

    TransitionState(String name) {
        this.identifier = Identifier.of(FabricDocsReference.MOD_ID, name);
    }

    public Identifier getIdentifier() {
        return identifier;
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
INFO

Again, if you make use of "split sources" you need to think about where you will be using this enum. Technically, only the custom SoundInstances which are only available on the client side, will use those enum values.

But if this enum is used anywhere else, e.g. in custom network packets, you may have to put this enum also into the common packages instead of the client only packages.

SoundInstanceCallback Interface
This interface is used as a callback. For now, we only need a onFinished method, but you can add your own methods if you need to send other signals from the SoundInstance object too.


public interface SoundInstanceCallback {
    // deliver the custom SoundInstance, from which this signal originates,
    // using the method parameters
    <T extends AbstractDynamicSoundInstance> void onFinished(T soundInstance);
}
1
2
3
4
5
Implement this interface on any class, which should be able to handle the incoming signals, for example the AbstractDynamicSoundInstance, which we will create soon and create the functionality in the custom SoundInstance itself.

AbstractDynamicSoundInstance Class
Let's finally get started on the core of the dynamic SoundInstances system. The AbstractDynamicSoundInstance is a newly created abstract class. It implements the default defining features and utilities of our custom SoundInstances, which will inherit from it.

We can take the CustomSoundInstance from earlier and improve on that. Instead of the LivingEntity we will now reference our DynamicSoundSource. In addition, we will define more properties.

TransitionState to keep track of the current phase
tick durations of how long the start and the end phases should last
minimum and maximum values for volume and pitch
boolean value to notify if this instance has been finished and can get cleaned up
tick holders to keep track of the current sound's progress.
a callback which sends a signal back to the DynamicSoundManager for the final clean up, when the SoundInstance is actually finished

public abstract class AbstractDynamicSoundInstance extends MovingSoundInstance {
    protected final DynamicSoundSource soundSource;                 // Entities, BlockEntities, ...
    protected TransitionState transitionState;                      // current TransitionState of the SoundInstance

    protected final int startTransitionTicks, endTransitionTicks;   // duration of starting and ending phases

    // possible volume range when adjusting sound values
    protected final float maxVolume;                                // only max value since the minimum is always 0
    // possible pitch range when adjusting sound values
    protected final float minPitch, maxPitch;

    protected int currentTick = 0, transitionTick = 0;              // current tick values for the instance

    protected final SoundInstanceCallback callback;                 // callback for soundInstance states

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
Then set up the default starting values for the custom SoundInstance in the constructor of the abstract class.


// ...

// set up default settings of the SoundInstance in this constructor
protected AbstractDynamicSoundInstance(DynamicSoundSource soundSource, SoundEvent soundEvent, SoundCategory soundCategory,
                                    int startTransitionTicks, int endTransitionTicks, float maxVolume, float minPitch, float maxPitch,
                                    SoundInstanceCallback callback) {
    super(soundEvent, soundCategory, SoundInstance.createRandom());

    // store important references to other objects
    this.soundSource = soundSource;
    this.callback = callback;

    // store the limits for the SoundInstance
    this.maxVolume = maxVolume;
    this.minPitch = minPitch;
    this.maxPitch = maxPitch;
    this.startTransitionTicks = startTransitionTicks;    // starting phase duration
    this.endTransitionTicks = endTransitionTicks;        // ending phase duration

    // set start values
    this.volume = 0.0f;
    this.pitch = minPitch;
    this.repeat = true;
    this.transitionState = TransitionState.STARTING;
    this.setPositionToEntity();
}

// ...
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
After the constructor is finished, you need to allow the SoundInstance to be able to play.


@Override
public boolean shouldAlwaysPlay() {
    // override to true, so that the SoundInstance can start
    // or add your own condition to the SoundInstance, if necessary
    return true;
}
1
2
3
4
5
6
Now comes the important part for this dynamic SoundInstance. Based on the current tick of the instance, it can apply different values and behaviors.


@Override
public void tick() {
    // handle states where sound might be actually stopped instantly
    if (this.soundSource == null) {
        this.callback.onFinished(this);
    }

    // basic tick behaviour
    this.currentTick++;
    this.setPositionToEntity();

    // SoundInstance phase switching
    switch (this.transitionState) {
        case STARTING -> {
            this.transitionTick++;

            // go into next phase if starting phase finished its duration
            if (this.transitionTick > this.startTransitionTicks) {
                this.transitionTick = 0;	// reset tick for future ending phase
                this.transitionState = TransitionState.RUNNING;
            }
        }
        case ENDING -> {
            this.transitionTick++;

            // set SoundInstance as finished if ending phase finished its duration
            if (this.transitionTick > this.endTransitionTicks) {
                this.callback.onFinished(this);
            }
        }
    }

    // apply volume and pitch modulation here,
    // if you use a normal SoundInstance class
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
As you can see, we haven't applied the volume and pitch modulation here yet. We only apply the shared behavior. So in this AbstractDynamicSoundInstance class we only provide the basic structure and the tools for the subclasses, which can decide themselves, which kind of sound modulation they actually want to apply.

So let's take a look at some examples of such sound modulation methods.


// increase or decrease volume and pitch based on the current phase of the sound
protected void modulateSoundForTransition() {
    float normalizedTick = switch (transitionState) {
        case STARTING -> (float) this.transitionTick / this.startTransitionTicks;
        case ENDING -> 1.0f - ((float) this.transitionTick / this.endTransitionTicks);
        default -> 1.0f;
    };

    this.volume = MathHelper.lerp(normalizedTick, 0.0f, this.maxVolume);
}

// increase or decrease pitch based on the sound source's stress value
protected void modulateSoundForStress() {
    this.pitch = MathHelper.lerp(this.soundSource.getNormalizedStress(), this.minPitch, this.maxPitch);
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
As you can see, normalized values in combination with linear interpolation (lerp) help out to shape the values to the preferred audio limits. Keep in mind that if you add multiple methods, which change the same value, you will need to observe and adjust how they work together with each other.

Now we just need to add the remaining utility methods, and we are done with the AbstractDynamicSoundInstance class.


// moves the sound instance position to the sound source's position
protected void setPositionToEntity() {
    this.x = soundSource.getPosition().getX();
    this.y = soundSource.getPosition().getY();
    this.z = soundSource.getPosition().getZ();
}

// Sets the SoundInstance into its ending phase.
// This is especially useful for external access to this SoundInstance
public void end() {
    this.transitionState = TransitionState.ENDING;
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
Example SoundInstance Implementation
If we take a look at the actual custom SoundInstance class, which extends from the newly created AbstractDynamicSoundInstance, we only need to think about what conditions would bring the sound to a stop and what sound modulation we want to apply.


public class EngineSoundInstance extends AbstractDynamicSoundInstance {
    // Here we just use the default constructor parameters.
    // If you want to specifically set values here already,
    // you can clean up the constructor parameters a bit
    public EngineSoundInstance(DynamicSoundSource soundSource, SoundEvent soundEvent, SoundCategory soundCategory,
                            int startTransitionTicks, int endTransitionTicks, float maxVolume, float minPitch, float maxPitch,
                            SoundInstanceCallback callback) {
        super(soundSource, soundEvent, soundCategory, startTransitionTicks, endTransitionTicks, maxVolume, minPitch, maxPitch, callback);
    }

    @Override
    public void tick() {
        // check conditions which set this sound automatically into the ending phase
        if (soundSource instanceof EngineBlockEntity blockEntity && blockEntity.isRemoved()) {
            this.end();
        }

        // apply the default tick behaviour from the parent class
        super.tick();

        // modulate volume and pitch of the SoundInstance
        this.modulateSoundForTransition();
        this.modulateSoundForStress();
    }

    // you can also add sound modulation methods here,
    // which should be only accessible to this
    // specific SoundInstance
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
DynamicSoundManager Class
We discussed earlier how to play and stop a SoundInstance. To clean up, centralize and manage those interactions, you can create your own SoundInstance handler, which builds on top of that.

This new DynamicSoundManager class will manage the custom SoundInstances so it will also only be available to the client side. On top of that, a client should only ever allow one instance of this class to exist. Multiple sound managers for a single client wouldn't make much sense and complicate the interactions even more. So, let's use a "Singleton Design Pattern".


public class DynamicSoundManager implements SoundInstanceCallback {
    // An instance of the client to use Minecraft's default SoundManager
    private static final MinecraftClient client = MinecraftClient.getInstance();
    // static field to store the current instance for the Singleton Design Pattern
    private static DynamicSoundManager instance;
    // The list which keeps track of all currently playing dynamic SoundInstances
    private final List<AbstractDynamicSoundInstance> activeSounds = new ArrayList<>();

    private DynamicSoundManager() {
        // private constructor to make sure that the normal
        // instantiation of that object is not used externally
    }

    // when accessing this class for the first time a new instance
    // is created and stored. If this is called again only the already
    // existing instance will be returned, instead of creating a new instance
    public static DynamicSoundManager getInstance() {
        if (instance == null) {
            instance = new DynamicSoundManager();
        }

        return instance;
    }


    // This is where the callback signal of a finished custom SoundInstance will arrive.
    // For now, we can just stop and remove the sound from the list, but you can add
    // your own functionality too
    @Override
    public <T extends AbstractDynamicSoundInstance> void onFinished(T soundInstance) {
        this.stop(soundInstance);
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
After getting the basic structure right, you can add the methods, which are needed to interact with the sound system.

playing sounds
stopping sounds
checking if a sound is currently playing

// Plays a sound instance, if it doesn't already exist in the list
public <T extends AbstractDynamicSoundInstance> void play(T soundInstance) {
    if (this.activeSounds.contains(soundInstance)) return;

    client.getSoundManager().play(soundInstance);
    this.activeSounds.add(soundInstance);
}

// Stops a sound immediately. in most cases it is preferred to use
// the sound's ending phase, which will clean it up after completion
public <T extends AbstractDynamicSoundInstance> void stop(T soundInstance) {
    client.getSoundManager().stop(soundInstance);
    this.activeSounds.remove(soundInstance);
}

// Finds a SoundInstance from a SoundEvent, if it exists and is currently playing
public Optional<AbstractDynamicSoundInstance> getPlayingSoundInstance(SoundEvent soundEvent) {
    for (var activeSound : this.activeSounds) {
        // SoundInstances use their SoundEvent's id by default
        if (activeSound.getId().equals(soundEvent.id())) {
            return Optional.of(activeSound);
        }
    }

    return Optional.empty();
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
Instead of only having a list of all currently playing SoundInstances you could also keep track of which sound sources are playing which sounds. For example, an engine having two engine sounds at the same time would make no sense, while multiple engines playing their respective engine sounds is a valid edge case. For the sake of simplicity we just created a List<AbstractDynamicSoundInstance> but in many cases a HashMap of DynamicSoundSource and a AbstractDynamicSoundInstance might be a better choice.

Using the Advanced Sound System
To use this sound system, simply make use of either the DynamicSoundManager methods or the SoundInstance methods. Using onStartedTrackingBy and onStoppedTrackingBy from entities or just custom S2C networking, you can now start and stop your custom dynamic SoundInstances.


private static void handleS2CEngineSoundPacket(EngineSoundInstancePacket packet, ClientPlayNetworking.Context context) {
    ClientWorld world = context.client().world;
    if (world == null) return;

    DynamicSoundManager soundManager = DynamicSoundManager.getInstance();

    if (world.getBlockEntity(packet.blockEntityPos()) instanceof EngineBlockEntity engineBlockEntity) {
        if (packet.shouldStart()) {
            soundManager.play(new EngineSoundInstance(engineBlockEntity,
                    CustomSounds.ENGINE_LOOP, SoundCategory.BLOCKS,
                    60, 30, 1.2f, 0.8f, 1.4f,
                    soundManager)
            );

            return;
        }
    }

    if (!packet.shouldStart()) {
        soundManager.getPlayingSoundInstance(CustomSounds.ENGINE_LOOP).ifPresent(AbstractDynamicSoundInstance::end);
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
The final product can adjust its volume based on the sound phase to smoothen out the transitions and change the pitch based on a stress value, which is coming from the sound source.

You could add another value to your sound source, which keeps track of an "overheat" value and, in addition, let a hissing SoundInstance slowly fade in if the value is above 0 or add a new interface to your custom dynamic SoundInstances which assigns a priority value to the sound types, which helps out choosing which sound to play, if they collide with each other.

With the current system, you can easily handle multiple SoundInstances at once and design the audio to your needs.

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM
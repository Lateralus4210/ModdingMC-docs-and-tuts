Basic Rendering Concepts

This page is written for version:

1.21.4


Page Authors
0x3C50
IMB11
WARNING

Although Minecraft is built using OpenGL, as of version 1.17+ you cannot use legacy OpenGL methods to render your own things. Instead, you must use the new BufferBuilder system, which formats rendering data and uploads it to OpenGL to draw.

To summarize, you have to use Minecraft's rendering system, or build your own that utilizes GL.glDrawElements().

This page will cover the basics of rendering using the new system, going over key terminology and concepts.

Although much of rendering in Minecraft is abstracted through the various DrawContext methods, and you'll likely not need to touch anything mentioned here, it's still important to understand the basics of how rendering works.

The Tessellator
The Tessellator is the main class used to render things in Minecraft. It is a singleton, meaning that there is only one instance of it in the game. You can get the instance using Tessellator.getInstance().

The BufferBuilder
The BufferBuilder is the class used to format and upload rendering data to OpenGL. It is used to create a buffer, which is then uploaded to OpenGL to draw.

The Tessellator is used to create a BufferBuilder, which is used to format and upload rendering data to OpenGL.

Initializing the BufferBuilder
Before you can write anything to the BufferBuilder, you must initialize it. This is done using Tessellator#begin(...) method, which takes in a VertexFormat and a draw mode and returns a BufferBuilder.

Vertex Formats
The VertexFormat defines the elements that we include in our data buffer and outlines how these elements should be transmitted to OpenGL.

The following VertexFormat elements are available:

Element	Format
BLIT_SCREEN	{ position (3 floats: x, y and z), uv (2 floats), color (4 ubytes) }
POSITION_COLOR_TEXTURE_LIGHT_NORMAL	{ position, color, texture uv, texture light (2 shorts), texture normal (3 sbytes) }
POSITION_COLOR_TEXTURE_OVERLAY_LIGHT_NORMAL	{ position, color, texture uv, overlay (2 shorts), texture light, normal (3 sbytes) }
POSITION_TEXTURE_COLOR_LIGHT	{ position, texture uv, color, texture light }
POSITION	{ position }
POSITION_COLOR	{ position, color }
LINES	{ position, color, normal }
POSITION_COLOR_LIGHT	{ position, color, light }
POSITION_TEXTURE	{ position, uv }
POSITION_COLOR_TEXTURE	{ position, color, uv }
POSITION_TEXTURE_COLOR	{ position, uv, color }
POSITION_COLOR_TEXTURE_LIGHT	{ position, color, uv, light }
POSITION_TEXTURE_LIGHT_COLOR	{ position, uv, light, color }
POSITION_TEXTURE_COLOR_NORMAL	{ position, uv, color, normal }
Draw Modes
The draw mode defines how the data is drawn. The following draw modes are available:

Draw Mode	Description
DrawMode.LINES	Each element is made up of 2 vertices and is represented as a single line.
DrawMode.LINE_STRIP	The first element requires 2 vertices. Additional elements are drawn with just 1 new vertex, creating a continuous line.
DrawMode.DEBUG_LINES	Similar to DrawMode.LINES, but the line is always exactly one pixel wide on the screen.
DrawMode.DEBUG_LINE_STRIP	Same as DrawMode.LINE_STRIP, but lines are always one pixel wide.
DrawMode.TRIANGLES	Each element is made up of 3 vertices, forming a triangle.
DrawMode.TRIANGLE_STRIP	Starts with 3 vertices for the first triangle. Each additional vertex forms a new triangle with the last two vertices.
DrawMode.TRIANGLE_FAN	Starts with 3 vertices for the first triangle. Each additional vertex forms a new triangle with the first vertex and the last vertex.
DrawMode.QUADS	Each element is made up of 4 vertices, forming a quadrilateral.
Writing to the BufferBuilder
Once the BufferBuilder is initialized, you can write data to it.

The BufferBuilder allows us to construct our buffer, vertex by vertex. To add a vertex, we use the buffer.vertex(matrix, float, float, float) method. The matrix parameter is the transformation matrix, which we'll discuss in more detail later. The three float parameters represent the (x, y, z) coordinates of the vertex position.

This method returns a vertex builder, which we can use to specify additional information for the vertex. It's crucial to follow the order of our defined VertexFormat when adding this information. If we don't, OpenGL might not interpret our data correctly. After we've finished building a vertex, just continue adding more vertices and data to the buffer until you're done.

It's also worth understanding the concept of culling. Culling is the process of removing faces of a 3D shape that aren't visible from the viewer's perspective. If the vertices for a face are specified in the wrong order, the face might not render correctly due to culling.

What Is a Transformation Matrix?
A transformation matrix is a 4x4 matrix that is used to transform a vector. In Minecraft, the transformation matrix is just transforming the coordinates we give into the vertex call. The transformations can scale our model, move it around and rotate it.

It's sometimes referred to as a position matrix, or a model matrix.

It's usually obtained via the MatrixStack class, which can be obtained via the DrawContext object:


drawContext.getMatrices().peek().getPositionMatrix();
1
Rendering a Triangle Strip
It's easier to explain how to write to the BufferBuilder using a practical example. Let's say we want to render something using the DrawMode.TRIANGLE_STRIP draw mode and the POSITION_COLOR vertex format.

We're going to draw vertices at the following points on the HUD (in order):


(20, 20)
(5, 40)
(35, 40)
(20, 60)
1
2
3
4
This should give us a lovely diamond - since we're using the TRIANGLE_STRIP draw mode, the renderer will perform the following steps:

Four steps that show the placement of the vertices on the screen to form two triangles

Since we're drawing on the HUD in this example, we'll use the HudRenderCallback event:


HudRenderCallback.EVENT.register((drawContext, tickDeltaManager) -> {
    // Get the transformation matrix from the matrix stack, alongside the tessellator instance and a new buffer builder.
    Matrix4f transformationMatrix = drawContext.getMatrices().peek().getPositionMatrix();
    Tessellator tessellator = Tessellator.getInstance();

    // Begin a triangle strip buffer using the POSITION_COLOR vertex format.
    BufferBuilder buffer = tessellator.begin(VertexFormat.DrawMode.TRIANGLE_STRIP, VertexFormats.POSITION_COLOR);

    // Write our vertices, Z doesn't really matter since it's on the HUD.
    buffer.vertex(transformationMatrix, 20, 20, 5).color(0xFF414141);
    buffer.vertex(transformationMatrix, 5, 40, 5).color(0xFF000000);
    buffer.vertex(transformationMatrix, 35, 40, 5).color(0xFF000000);
    buffer.vertex(transformationMatrix, 20, 60, 5).color(0xFF414141);

    // Make sure the correct shader for your chosen vertex format is set!
    // You can find all the shaders in the ShaderProgramKeys class.
    RenderSystem.setShader(ShaderProgramKeys.POSITION_COLOR);
    RenderSystem.setShaderColor(1.0F, 1.0F, 1.0F, 1.0F);

    // Draw the buffer onto the screen.
    BufferRenderer.drawWithGlobalProgram(buffer.end());
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
17
18
19
20
21
22
This results in the following being drawn on the HUD:

Final Result

TIP

Try mess around with the colors and positions of the vertices to see what happens! You can also try using different draw modes and vertex formats.

The MatrixStack
After learning how to write to the BufferBuilder, you might be wondering how to transform your model - or even animate it. This is where the MatrixStack class comes in.

The MatrixStack class has the following methods:

push() - Pushes a new matrix onto the stack.
pop() - Pops the top matrix off the stack.
peek() - Returns the top matrix on the stack.
translate(x, y, z) - Translates the top matrix on the stack.
scale(x, y, z) - Scales the top matrix on the stack.
You can also multiply the top matrix on the stack using quaternions, which we will cover in the next section.

Taking from our example above, we can make our diamond scale up and down by using the MatrixStack and the tickDelta - which is the "progress" between the last game tick and the next game tick. We'll clarify this later in the Rendering in the HUD page.

WARNING

You must first push the matrix stack and then pop it after you're done with it. If you don't, you'll end up with a broken matrix stack, which will cause rendering issues.

Make sure to push the matrix stack before you get a transformation matrix!


MatrixStack matrices = drawContext.getMatrices();

// Store the total tick delta in a field, so we can use it later.
totalTickDelta += tickDeltaManager.getTickDelta(true);

// Push a new matrix onto the stack.
matrices.push();
// Scale the matrix by 0.5 to make the triangle smaller and larger over time.
float scaleAmount = MathHelper.sin(totalTickDelta / 10F) / 2F + 1.5F;

// Apply the scaling amount to the matrix.
// We don't need to scale the Z axis since it's on the HUD and 2D.
matrices.scale(scaleAmount, scaleAmount, 1F);

// ... write to the buffer.

// Pop our matrix from the stack.
matrices.pop();
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
A video showing the diamond scaling up and down

Quaternions (Rotating Things)
Quaternions are a way of representing rotations in 3D space. They are used to rotate the top matrix on the MatrixStack via the multiply(Quaternion, x, y, z) method.

It's highly unlikely you'll need to ever use a Quaternion class directly, since Minecraft provides various pre-built Quaternion instances in it's RotationAxis utility class.

Let's say we want to rotate our diamond around the z-axis. We can do this by using the MatrixStack and the multiply(Quaternion, x, y, z) method.


// Lerp between 0 and 360 degrees over time.
float rotationAmount = (float) (totalTickDelta / 50F % 360);
matrices.multiply(RotationAxis.POSITIVE_Z.rotation(rotationAmount));
// Shift entire diamond so that it rotates in its center.
matrices.translate(-20f, -40f, 0f);
1
2
3
4
5
The result of this is the following:

A video showing the diamond rotating around the z-axis

Using the Drawing Context

This page is written for version:

1.21.4


Page Authors
IMB11
This page assumes you've taken a look at the Basic Rendering Concepts page.

The DrawContext class is the main class used for rendering in the game. It is used for rendering shapes, text and textures, and as previously seen, used to manipulate MatrixStacks and use BufferBuilders.

Drawing Shapes
The DrawContext class can be used to easily draw square-based shapes. If you want to draw triangles, or any non-square based shape, you will need to use a BufferBuilder.

Drawing Rectangles
You can use the DrawContext.fill(...) method to draw a filled rectangle.


int rectangleX = 10;
int rectangleY = 10;
int rectangleWidth = 100;
int rectangleHeight = 50;
// x1, y1, x2, y2, color
context.fill(rectangleX, rectangleY, rectangleX + rectangleWidth, rectangleY + rectangleHeight, 0xFF0000FF);
1
2
3
4
5
6
A rectangle

Drawing Outlines/Borders
Let's say we want to outline the rectangle we just drew. We can use the DrawContext.drawBorder(...) method to draw an outline.


// x, y, width, height, color
context.drawBorder(rectangleX, rectangleY, rectangleWidth, rectangleHeight, 0xFFFF0000);
1
2
Rectangle with border

Drawing Individual Lines
We can use the DrawContext.drawHorizontalLine(...) and DrawContext.drawVerticalLine(...) methods to draw lines.


// Let's split the rectangle in half using a green line.
// x, y1, y2, color
context.drawVerticalLine(rectangleX + rectangleWidth / 2, rectangleY, rectangleY + rectangleHeight, 0xFF00FF00);
1
2
3
Lines

The Scissor Manager
The DrawContext class has a built-in scissor manager. This allows you to easily clip your rendering to a specific area. This is useful for rendering things like tooltips, or other elements that should not be rendered outside of a specific area.

Using the Scissor Manager
TIP

Scissor regions can be nested! But make sure that you disable the scissor manager the same amount of times as you enabled it.

To enable the scissor manager, simply use the DrawContext.enableScissor(...) method. Likewise, to disable the scissor manager, use the DrawContext.disableScissor() method.


// Let's create a scissor region that covers a middle bar section of the screen.
int scissorRegionX = 200;
int scissorRegionY = 20;
int scissorRegionWidth = 100;

// The height of the scissor region is the height of the screen minus the height of the top and bottom bars.
int scissorRegionHeight = this.height - 40;

// x1, y1, x2, y2
context.enableScissor(scissorRegionX, scissorRegionY, scissorRegionX + scissorRegionWidth, scissorRegionY + scissorRegionHeight);

// Let's fill the entire screen with a color gradient, it should only be visible in the scissor region.
// x1, y1, x2, y2, color1, color2
context.fillGradient(0, 0, this.width, this.height, 0xFFFF0000, 0xFF0000FF);

// Disable the scissor region.
context.disableScissor();
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
Scissor region in action

As you can see, even though we tell the game to render the gradient across the entire screen, it only renders within the scissor region.

Drawing Textures
There is no one "correct" way to draw textures onto a screen, as the drawTexture(...) method has many different overloads. This section will go over the most common use cases.

Drawing an Entire Texture
Generally, it's recommended that you use the overload that specifies the textureWidth and textureHeight parameters. This is because the DrawContext class will assume these values if you don't provide them, which can sometimes be wrong.

You will also need to specify the render layer which your texture is drawn onto. For basic textures, this will usually always be RenderLayer::getGuiTextured.


Identifier texture = Identifier.of("minecraft", "textures/block/deepslate.png");
// renderLayer, texture, x, y, u, v, width, height, textureWidth, textureHeight
context.drawTexture(RenderLayer::getGuiTextured, texture, 90, 90, 0, 0, 16, 16, 16, 16);
1
2
3
Drawing whole texture example

Drawing a Portion of a Texture
This is where u and v come in. These parameters specify the top-left corner of the texture to draw, and the regionWidth and regionHeight parameters specify the size of the portion of the texture to draw.

Let's take this texture as an example.

Recipe Book Texture

If we want to only draw a region that contains the magnifying glass, we can use the following u, v, regionWidth and regionHeight values:


Identifier texture2 = Identifier.of("fabric-docs-reference", "textures/gui/test-uv-drawing.png");
int u = 10, v = 13, regionWidth = 14, regionHeight = 14;
// renderLayer, texture, x, y, width, height, u, v, regionWidth, regionHeight, textureWidth, textureHeight
context.drawTexture(RenderLayer::getGuiTextured, texture2, 90, 190, 14, 14, u, v, regionWidth, regionHeight, 256, 256);
1
2
3
4
Region Texture

Drawing Text
The DrawContext class has various self-explanatory text rendering methods - for the sake of brevity, they will not be covered here.

Let's say we want to draw "Hello World" onto the screen. We can use the DrawContext.drawText(...) method to do this.


// TextRenderer, text (string, or Text object), x, y, color, shadow
context.drawText(client.textRenderer, "Hello, world!", 10, 200, 0xFFFFFFFF, false);
1
2
Drawing text

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Rendering in the HUD

This page is written for version:

1.21.4


Page Authors
IMB11
kevinthegreat1
We already briefly touched on rendering things to the HUD in the Basic Rendering Concepts page and Using The Drawing Context, so on this page we'll stick to the Hud API and the RenderTickCounter parameter.

HudRenderCallback
WARNING

Previously, Fabric provided HudRenderCallback to render to the HUD. Due to changes to HUD rendering, this event became extremely limited and is deprecated since Fabric API 0.116. Usage is strongly discouraged.

HudLayerRegistrationCallback
Fabric provides the Hud API to render and layer elements on the HUD.

To start, we need to register a listener to HudLayerRegistrationCallback which registers your layers. Each layer is an IdentifiedLayer, which is a vanilla LayeredDrawer.Layer with an Identifier attached. A LayeredDrawer.Layer instance is usually a lambda that takes a DrawContext and a RenderTickCounter instance as parameters. See HudLayerRegistrationCallback and related Javadocs for more details on how to use the API.

The draw context can be used to access the various rendering utilities provided by the game, and access the raw matrix stack. You should check out the Draw Context page to learn more about the draw context.

Render Tick Counter
The RenderTickCounter class allows you to retrieve the current tickDelta value. tickDelta is the "progress" between the last game tick and the next game tick.

For example, if we assume a 200 FPS scenario, the game runs a new tick roughly every 10 frames. Each frame, tickDelta represents how far we are between the last tick and the next. Over 11 frames, you might see:

Frame	tickDelta
1	1: New tick
2	1/10 = 0.1
3	2/10 = 0.2
4	3/10 = 0.3
5	4/10 = 0.4
6	5/10 = 0.5
7	6/10 = 0.6
8	7/10 = 0.7
9	8/10 = 0.8
10	9/10 = 0.9
11	1: New tick
In practice, you should only use tickDelta when your animations depend on Minecraft's ticks. For time-based animations, use Util.getMeasuringTimeMs(), which measures real-world time.

You can retrieve tickDelta by calling renderTickCounter.getTickDelta(false), where the boolean parameter is ignoreFreeze, which essentially just allows you to ignore whenever players use the /tick freeze command.

In this example, we'll use Util.getMeasuringTimeMs() to linearly interpolate the color of a square that is being rendered to the HUD.


public class HudRenderingEntrypoint implements ClientModInitializer {
    private static final Identifier EXAMPLE_LAYER = Identifier.of(FabricDocsReference.MOD_ID, "hud-example-layer");

    @Override
    public void onInitializeClient() {
        // Attach our rendering code to before the chat hud layer. Our layer will render right before the chat. The API will take care of z spacing and automatically add 200 after every layer.
        HudLayerRegistrationCallback.EVENT.register(layeredDrawer -> layeredDrawer.attachLayerBefore(IdentifiedLayer.CHAT, EXAMPLE_LAYER, HudRenderingEntrypoint::render));
    }

    private static void render(DrawContext context, RenderTickCounter tickCounter) {
        int color = 0xFFFF0000; // Red
        int targetColor = 0xFF00FF00; // Green

        // You can use the Util.getMeasuringTimeMs() function to get the current time in milliseconds.
        // Divide by 1000 to get seconds.
        double currentTime = Util.getMeasuringTimeMs() / 1000.0;

        // "lerp" simply means "linear interpolation", which is a fancy way of saying "blend".
        float lerpedAmount = MathHelper.abs(MathHelper.sin((float) currentTime));
        int lerpedColor = ColorHelper.lerp(lerpedAmount, color, targetColor);

        // Draw a square with the lerped color.
        // x1, x2, y1, y2, z, color
        context.fill(0, 0, 10, 10, 0, lerpedColor);
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
Lerping a color over time

Why don't you try use tickDelta and see what happens to the animation when you run the /tick freeze command? You should see the animation freeze in place as tickDelta becomes constant (assuming you have passed false as the parameter to RenderTickCounter#getTickDelta)

Custom Screens

This page is written for version:

1.21.4


Page Authors
IMB11
INFO

This page refers to normal screens, not handled ones - these screens are the ones that are opened by the player on the client, not the ones that are handled by the server.

Screens are essentially the GUIs that the player interacts with, such as the title screen, pause screen etc.

You can create your own screens to display custom content, a custom settings menu, or more.

Creating a Screen
To create a screen, you need to extend the Screen class and override the init method - you may optionally override the render method as well - but make sure to call it's super method or it wont render the background, widgets etc.

You should take note that:

Widgets are not being created in the constructor because the screen is not yet initialized at that point - and certain variables, such as width and height, are not yet available or not yet accurate.
The init method is called when the screen is being initialized, and it is the best place to create widgets.
You add widgets using the addDrawableChild method, which accepts any drawable widget.
The render method is called every frame - you can access the draw context, and the mouse position from this method.
As an example, we can create a simple screen that has a button and a label above it.


public class CustomScreen extends Screen {
    public CustomScreen(Text title) {
        super(title);
    }

    @Override
    protected void init() {
        ButtonWidget buttonWidget = ButtonWidget.builder(Text.of("Hello World"), (btn) -> {
            // When the button is clicked, we can display a toast to the screen.
            this.client.getToastManager().add(
                    SystemToast.create(this.client, SystemToast.Type.NARRATOR_TOGGLE, Text.of("Hello World!"), Text.of("This is a toast."))
            );
        }).dimensions(40, 40, 120, 20).build();
        // x, y, width, height
        // It's recommended to use the fixed height of 20 to prevent rendering issues with the button
        // textures.

        // Register the button widget.
        this.addDrawableChild(buttonWidget);

    }

    @Override
    public void render(DrawContext context, int mouseX, int mouseY, float delta) {
        super.render(context, mouseX, mouseY, delta);

        // Minecraft doesn't have a "label" widget, so we'll have to draw our own text.
        // We'll subtract the font height from the Y position to make the text appear above the button.
        // Subtracting an extra 10 pixels will give the text some padding.
        // textRenderer, text, x, y, color, hasShadow
        context.drawText(this.textRenderer, "Special Button", 40, 40 - this.textRenderer.fontHeight - 10, 0xFFFFFFFF, true);
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
Custom Screen 1

Opening the Screen
You can open the screen using the MinecraftClient's setScreen method - you can do this from many places, such as a key binding, a command, or a client packet handler.


MinecraftClient.getInstance().setScreen(
  new CustomScreen(Text.empty())
);
1
2
3
Closing the Screen
If you want to close the screen, simply set the screen to null:


MinecraftClient.getInstance().setScreen(null);
1
If you want to be fancy, and return to the previous screen, you can pass the current screen into the CustomScreen constructor and store it in a field, then use it to return to the previous screen when the close method is called.


public Screen parent;
public CustomScreen(Text title, Screen parent) {
    super(title);
    this.parent = parent;
}

@Override
public void close() {
    this.client.setScreen(this.parent);
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
Now, when opening the custom screen, you can pass the current screen as the second argument - so when you call CustomScreen#close - it will return to the previous screen.


Screen currentScreen = MinecraftClient.getInstance().currentScreen;
MinecraftClient.getInstance().setScreen(
  new CustomScreen(Text.empty(), currentScreen)
);
1
2
3
4
Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Custom Widgets

This page is written for version:

1.21.4


Page Authors
IMB11
Widgets are essentially containerized rendering components that can be added to a screen, and can be interacted with by the player through various events such as mouse clicks, key presses, and more.

Creating a Widget
There are multiple ways to create a widget class, such as extending ClickableWidget. This class provides a lot of useful utilities, such as managing width, height, position, and handling events - it implements the Drawable, Element, Narratable, and Selectable interfaces:

Drawable - for rendering - Required to register the widget to the screen via the addDrawableChild method.
Element - for events - Required if you want to handle events such as mouse clicks, key presses, and more.
Narratable - for accessibility - Required to make your widget accessible to screen readers and other accessibility tools.
Selectable - for selection - Required if you want to make your widget selectable using the Tab key - this also aids in accessibility.

public class CustomWidget extends ClickableWidget {
    public CustomWidget(int x, int y, int width, int height) {
        super(x, y, width, height, Text.empty());
    }

    @Override
    protected void renderWidget(DrawContext context, int mouseX, int mouseY, float delta) {
        // We'll just draw a simple rectangle for now.
        // x1, y1, x2, y2, startColor, endColor
        int startColor = 0xFF00FF00; // Green
        int endColor = 0xFF0000FF; // Blue

        context.fillGradient(getX(), getY(), getX() + this.width, getY() + this.height, startColor, endColor);
    }

    @Override
    protected void appendClickableNarrations(NarrationMessageBuilder builder) {
        // For brevity, we'll just skip this for now - if you want to add narration to your widget, you can do so here.
        return;
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
Adding the Widget to the Screen
Like all widgets, you need to add it to the screen using the addDrawableChild method, which is provided by the Screen class. Make sure you do this in the init method.


// Add a custom widget to the screen.
// x, y, width, height
CustomWidget customWidget = new CustomWidget(40, 80, 120, 20);
this.addDrawableChild(customWidget);
1
2
3
4
Custom widget on screen

Widget Events
You can handle events such as mouse clicks, key presses, by overriding the onMouseClicked, onMouseReleased, onKeyPressed, and other methods.

For example, you can make the widget change color when it's hovered over by using the isHovered() method provided by the ClickableWidget class:


// This is in the "renderWidget" method, so we can check if the mouse is hovering over the widget.
if (isHovered()) {
    startColor = 0xFFFF0000; // Red
    endColor = 0xFF00FFFF; // Cyan
}
1
2
3
4
5
Hover Event Example

Edit this page on GitHub
Last updated: 5/31/25, 6:10 PM

Creating Custom Particles

This page is written for version:

1.21.4


Page Authors
Superkat32
Particles are a powerful tool. They can add ambience to a beautiful scene, or add tension to an edge of your seat boss battle. Let's add one!

Register a Custom Particle
We'll be adding a new sparkle particle which will mimic the movement of an end rod particle.

We first need to register a ParticleType in your mod's initializer class using your mod id.


// This DefaultParticleType gets called when you want to use your particle in code.
public static final SimpleParticleType SPARKLE_PARTICLE = FabricParticleTypes.simple();

    // Register our custom particle type in the mod initializer.
    Registry.register(Registries.PARTICLE_TYPE, Identifier.of(MOD_ID, "sparkle_particle"), SPARKLE_PARTICLE);
1
2
3
4
5
The "sparkle_particle" in lowercase letters is the JSON path for the particle's texture. You will be creating a new JSON file with that exact name later.

Client-Side Registration
After you have registered the particle in the mod's initializer, you will also need to register the particle in the client-side initializer.


// For this example, we will use the end rod particle behaviour.
ParticleFactoryRegistry.getInstance().register(FabricDocsReference.SPARKLE_PARTICLE, EndRodParticle.Factory::new);
1
2
In this example, we are registering our particle on the client-side. We are then giving the particle some movement using the end rod particle's factory. This means our particle will move exactly like an end rod particle.

TIP

You can see all the particle factories by looking at all the implementations of the ParticleFactory interface. This is helpful if you want to use another particle's behaviour for your own particle.

IntelliJ's hotkey: Ctrl+Alt+B
Visual Studio Code's hotkey: Ctrl+F12
Creating a JSON File and Adding Textures
You will need to create 2 folders in your resources/assets/mod-id/ folder.

Folder Path	Explanation
/textures/particle	The particle folder will contain all the textures for all of your particles.
/particles	The particles folder will contain all of the json files for all of your particles.
For this example, we will have only one texture in textures/particle called "sparkle_particle_texture.png".

Next, create a new JSON file in particles with the same name as the JSON path that you used when registering your ParticleType. For this example, we will need to create sparkle_particle.json. This file is important because it lets Minecraft know which textures our particle should use.


{
  "textures": ["fabric-docs-reference:sparkle_particle_texture"]
}
1
2
3
TIP

You can add more textures to the textures array to create a particle animation. The particle will cycle through the textures in the array, starting with the first texture.

Testing the New Particle
Once you have completed the JSON file and saved your work, you are good to load up Minecraft and test everything out!

You can see if everything has worked by typing the following command:


/particle fabric-docs-reference:sparkle_particle ~ ~1 ~
1
Showcase of the particle

INFO

The particle will spawn inside the player with this command. You will likely need to walk backwards to actually see it.

Alternatively, you can also use a command block to summon the particle with the exact same command.
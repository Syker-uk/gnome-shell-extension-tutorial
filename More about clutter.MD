
# Knowing clutter
I think that this should go into a clutter specific documentation.
## Introducing clutter
Clutter is a powerfull C programming API that allows you to create simple but visually appealing and engaging user interfaces. It offers a variety of objects (actors) which can be placed on a canvas (stage) and manipulated by the application or the user. It is therefore a "retained mode" graphics API. Unlike traditional 2D canvas APIs, Clutter allows these actors to move partly in the Z dimension. It is used by gnome-shell to implement the entire GUI.

There's three technologies that allow us to easily use clutter with javascript:

Clutter use GObjectIntrospection, a project that allows us to use JavaScript (as well as other languages), instead of using c. It works by transforming the c program API in general files(xml or custom files) allowing us to easily write a binding in another program language that can interpret it.

Next, we need a program that gets the general file result from GObjectIntrospection and transforms it into a specific programming language, like javascript. This program is specific for each programming language, and for javascript it is called Gjs.

Also, gnome-shell has his own clutter-based toolkit, called St, that gives you some useful actors (components in the GUI). St is also the way for gnome-shell to interact with CSS theming; so the form and color of every button, dialog etc. of gnome shell is defined in /usr/share/gnome-shell/theme, and the only class that manages this css is St.

So we have: Clutter in c -> GObjectIntrospection(result in a general file) ->

-> Gjs (interprets general file from GObjectIntrospection to javascript)

-> pyclutter (interprets general file from GObjectIntrospection to python)

-> clutter-perl (interprets general file from GObjectIntrospection to perl)

-> etc.

Important characteristic things in clutter are:

Stage: contains some actors (text, rectangles, etc). It's the window of the application. Also is an actor.
    - Actor: GUI object.
          - show() : you have to show each actor to be visible to the user.
          - hide() : hide the actor.

## Some examples with raw clutter
Let's have a look at some example with clutter alone. No gnome shell here.

### Initiate clutter and stage

//Import the clutter class form the gi repository (the object introspection repository)
const Clutter = imports.gi.Clutter;
// Initialize clutter
Clutter.init(null);
/*
 * Create a stage. This function returns a new default stage, with its own
 * window. ClutterStage is derived from the ClutterActor object so many of that
 * object's functions are useful for the stage. For instance, call
 * Clutter.Stage.get_default().show() to make the stage visible.
 */
let stage = new Clutter.Stage();

// We connect the destroy event to quit from the mainloop when we close the
// window.
stage.connect("destroy", Clutter.main_quit);
// Put some title
stage.title = "Test";
// Set a color to the stage to show that it is working
stage.set_background_color(new Clutter.Color({
    red : 150,
    blue : 0,
    green : 0,
    alpha : 255
}));
// As we say, the stage is also an actor, so we show it to make visible
stage.show();
// Start a main loop so that the stage can animate its contents and respond to
// user interaction.
Clutter.main();

### Draw something
Let's try to draw a rectangle in the stage:

//Import the clutter class from the gi repository (the object introspection repository).
const Clutter = imports.gi.Clutter;
// Initialize clutter
Clutter.init(null);
/*
 * Create a stage. This function returns a new default stage, with its own
 * window. ClutterStage is derived from the ClutterActor object so many of that
 * object's functions are useful for the stage. For instance, call
 * Clutter.Stage.get_default().show() to make the stage visible.
 */
let stage = new Clutter.Stage();

// We connect the destroy event to quit from the main loop when we close the
// window.
stage.connect("destroy", Clutter.main_quit);
// Set the title
stage.title = "Test";
// Set a color to the stage to show that it is working
stage.set_background_color(new Clutter.Color({
    red : 10,
    blue : 10,
    green : 10,
    alpha : 255
}));

// Create a new actor
let actorRectangle = new Clutter.Actor();
// Make it like a rectangle
actorRectangle.set_size(100, 100);
actorRectangle.set_position(100, 100);
/*
 * Colors are made in RGBA http://en.wikipedia.org/wiki/RGBA_color_space
 * Basically, Red, Green, Blue, Alpha (transparency). Each is a value between 0 and 255
 */
actorRectangle.set_background_color(new Clutter.Color({
    red : 100,
    blue : 100,
    green : 100,
    alpha : 255
}));

stage.add_actor(actorRectangle);
// As we say, the stage is also an actor, so we call show() to make it visible
stage.show();
// Start a main loop so that the stage can animate its contents and respond to
// user interaction.
Clutter.main();

### Add some signals to make it interactive
Now, we do something funny. We will add a hover event to the rectangle actor, and each time we hover the actor, the actor will change the position randomly.

//Import the clutter class from the gi repository(the object introspection repository)
const Clutter = imports.gi.Clutter;

// Initialize clutter
Clutter.init(null);
/*
 * Create a stage. This function returns a new default stage, with its own
 * window. ClutterStage is derived from the ClutterActor object so many of that
 * object's functions are useful for the stage. For instance, call
 * Clutter.Stage.get_default().show() to make the stage visible.
 */
let stage = new Clutter.Stage();

// We connect the destroy event to quit from the main loop when we close the
// window.
stage.connect("destroy", Clutter.main_quit);
// Put some tittle
stage.title = "Test";
// Set a color to the stage to show that it is working
stage.set_background_color(new Clutter.Color({
    red : 10,
    blue : 10,
    green : 10,
    alpha : 255
}));

// Create a new actor
let
actorRectangle = new Clutter.Actor();
// Put his x, y size, position and background color
actorRectangle.set_size(100, 100);
actorRectangle.set_position(100, 100);
actorRectangle.set_background_color(new Clutter.Color({
    red : 100,
    blue : 100,
    green : 100,
    alpha : 255
}));

// Sets actor as reactive. Reactive actors will receive events.
actorRectangle.set_reactive(true);
/*
 * Connect the actor to a event. When you hover it, function in the second
 * parameter will be called. So we are passing a reference to a function.
 */
actorRectangle.connect('enter-event', changeRectanglePosition);

stage.add_actor(actorRectangle);
// As we say, the stage is also an actor, so we show it to make visible
stage.show();
// Start a main loop so that the stage can animate its contents and respond to
// user interaction.
Clutter.main();

function changeRectanglePosition()
{
    // We get the size of the stage (our window)
    let [sizex, sizey] = stage.get_size();
    /*
     * Math.random returns a float between 0 and 1, so we multiply by the size of
     * the stage and we acomplish a number between 0-sizeStage
     */
    let newx = Math.floor(Math.random() * sizex);
    let newy = Math.floor(Math.random() * sizey);
    /*
     * We can access that because it is a global variable. Also, remember that
     * with "let" the scope is the block and with "var" the scope is the whole
     * environment.
     */
    actorRectangle.set_position(newx, newy);
}

### Animations
Now, we add an animation to the actor. This animation only will start if we hover over the actor. So we have to put the animation inside the function we call when we hover over the actor.

//Import the clutter class from the gi repository(the object introspection repository)
const
Clutter = imports.gi.Clutter;

// Initialize clutter
Clutter.init(null);
/*
 * Create a stage. This function returns a new default stage, with its own
 * window. ClutterStage is derived from the ClutterActor object, so many of these
 * object's functions are useful for the stage. For instance, call
 * Clutter.Stage.get_default().show() to make the stage visible.
 */
let stage = new Clutter.Stage();

// We connect the destroy event to quit from the mainloop when we close the
// window.
stage.connect("destroy", Clutter.main_quit);
// Set a title
stage.title = "Test";
// Give the stage a color to show that it is working
stage.set_background_color(new Clutter.Color({
    red : 10,
    blue : 10,
    green : 10,
    alpha : 255
}));

// Create a new actor
let actorRectangle = new Clutter.Actor();
// Set its x, y, size, position, and background color
actorRectangle.set_size(100, 100);
actorRectangle.set_position(100, 100);
actorRectangle.set_background_color(new Clutter.Color({
    red : 100,
    blue : 100,
    green : 100,
    alpha : 255
}));

// Sets actor as reactive. Reactive actors will receive events.
actorRectangle.set_reactive(true);
/*
 * Connect the actor to a event. When you hover over it, the function in the
 * second parameter will be called. So we are passing a reference to a function.
 */
actorRectangle.connect('enter-event', changeRectanglePosition);
// Add the rectangle to the stage
stage.add_actor(actorRectangle);

// As we say, the stage is also an actor, so we show it to make visible
stage.show();
// Start a main loop so that the stage can animate its contents and respond to
// user interaction.
Clutter.main();

function changeRectanglePosition()
{
    // We get the size of the stage(our window)
    let [sizex, sizey] = stage.get_size();
    /*
     * Math.random return a float between 0 and 1, so we multiply by the size of
     * the stage and we acomplish a number between 0-sizeStage
     */
    let newx = Math.floor(Math.random() * sizex);
    let newy = Math.floor(Math.random() * sizey);
    /*
     * We can access that because is a global variable. Also, remember that
     * with "let" the scope is the block and with "var" the scope is the whole
     * environment.
     */
    actorRectangle.set_position(newx, newy);

    /*
     * WARNING: The next code is deprecated since clutter 1.6, but we don't have
     * an easy way to animate actors in raw clutter in stable version of
     * fedora 17, so we use deprecated code. The easy way in new code is:
     * actorRectangle.animate(Clutter.AnimationMode.EASE_OUT_ELASTIC, 500,
     * "scale-x", 0.5, "scale-y", 0.5, NULL); New code to animate actors:
     * http://docs.clutter-project.org/docs/clutter/stable/clutter-Implicit-Animations.html
     */

    /*
     * To animate an actor we need three things timeline: think about it like
     * play, pause and stop and bar time of your videos alpha: how is made the
     * transition, because we can made a transition bounding, quickly at first
     * and later slowly, etc. These are all the transitions:
     * http://docs.clutter-project.org/docs/clutter/stable/clutter-Implicit-Animations.html#ClutterAnimationMode
     * behaviour: what property of the actor we want to animate, scale, opacity,
     * etc. each property has his own class, behaviourScale, behaviourOpacity,
     * etc. all properties of behaviourScale:
     * http://docs.clutter-project.org/docs/clutter/stable/ClutterBehaviourScale.html
     */
    let timeline = new Clutter.Timeline({
        'duration' : 500
    });
    let alpha = new Clutter.Alpha({
        'timeline' : timeline,
        'mode' : Clutter.AnimationMode.EASE_OUT_ELASTIC
    });
    let behaviourScale = new Clutter.BehaviourScale({
        'alpha' : alpha,
        'x_scale_start' : 1.0,
        'y_scale_start' : 1.0,
        'x_scale_end' : 0.5,
        'y_scale_end' : 0.5
    });
    behaviourScale.apply(actorRectangle);
    timeline.start();
}

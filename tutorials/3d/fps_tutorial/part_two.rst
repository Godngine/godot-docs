.. _doc_fps_tutorial_part_two:

Part 2
======

Part Overview
-------------

In this part we will be giving our player weapons to play with.

.. image:: img/PartTwoFinished.png

By the end of this part, you will have a player that can fire a pistol,
rifle, and attack using a knife. The player will also now have animations with transitions,
and the weapons can interact with objects in the environment.

.. note:: You are assumed to have finished :ref:`doc_fps_tutorial_part_one` before moving on to this part of the tutorial.

Let's get started!

Making a system to handle animations
------------------------------------

First we need a way to handle changing animations. Open up ``Player.tscn`` and select the :ref:`AnimationPlayer <class_AnimationPlayer>`
Node (``Player``->``Rotation_helper``->``Model``->``AnimationPlayer``).

Create a new script called ``AnimationPlayer_Manager.gd`` and attach that to the :ref:`AnimationPlayer <class_AnimationPlayer>`.

Add the following code to ``AnimationPlayer_Manager.gd``:

::

    # Structure -> Animation name :[Connecting Animation states]
    var states = {
    "Idle_unarmed":["Knife_equip", "Pistol_equip", "Rifle_equip", "Idle_unarmed"],

    "Pistol_equip":["Pistol_idle"],
    "Pistol_fire":["Pistol_idle"],
    "Pistol_idle":["Pistol_fire", "Pistol_reload", "Pistol_unequip", "Pistol_idle"],
    "Pistol_reload":["Pistol_idle"],
    "Pistol_unequip":["Idle_unarmed"],

    "Rifle_equip":["Rifle_idle"],
    "Rifle_fire":["Rifle_idle"],
    "Rifle_idle":["Rifle_fire", "Rifle_reload", "Rifle_unequip", "Rifle_idle"],
    "Rifle_reload":["Rifle_idle"],
    "Rifle_unequip":["Idle_unarmed"],

    "Knife_equip":["Knife_idle"],
    "Knife_fire":["Knife_idle"],
    "Knife_idle":["Knife_fire", "Knife_unequip", "Knife_idle"],
    "Knife_unequip":["Idle_unarmed"],
    }

    var animation_speeds = {
    "Idle_unarmed":1,

    "Pistol_equip":1.4,
    "Pistol_fire":1.8,
    "Pistol_idle":1,
    "Pistol_reload":1,
    "Pistol_unequip":1.4,

    "Rifle_equip":2,
    "Rifle_fire":6,
    "Rifle_idle":1,
    "Rifle_reload":1.45,
    "Rifle_unequip":2,

    "Knife_equip":1,
    "Knife_fire":1.35,
    "Knife_idle":1,
    "Knife_unequip":1,
    }

    var current_state = null
    var callback_function = null

    func _ready():
        set_animation("Idle_unarmed")
        connect("animation_finished", self, "animation_ended")



    func set_animation(animation_name):
        if animation_name == current_state:
            print ("AnimationPlayer_Manager.gd -- WARNING: animation is already ", animation_name)
            return true

        if has_animation(animation_name) == true:
            if current_state != null:
                var possible_animations = states[current_state]
                if animation_name in possible_animations:
                    current_state = animation_name
                    play(animation_name, -1, animation_speeds[animation_name])
                    return true
                else:
                    print ("AnimationPlayer_Manager.gd -- WARNING: Cannot change to ", animation_name, " from ", current_state)
                    return false
            else:
                current_state = animation_name
                play(animation_name, -1, animation_speeds[animation_name])
                return true
        return false


    func animation_ended(anim_name):
        # UNARMED transitions
        if current_state == "Idle_unarmed":
            pass
        # KNIFE transitions
        elif current_state == "Knife_equip":
            set_animation("Knife_idle")
        elif current_state == "Knife_idle":
            pass
        elif current_state == "Knife_fire":
            set_animation("Knife_idle")
        elif current_state == "Knife_unequip":
            set_animation("Idle_unarmed")
        # PISTOL transitions
        elif current_state == "Pistol_equip":
            set_animation("Pistol_idle")
        elif current_state == "Pistol_idle":
            pass
        elif current_state == "Pistol_fire":
            set_animation("Pistol_idle")
        elif current_state == "Pistol_unequip":
            set_animation("Idle_unarmed")
        elif current_state == "Pistol_reload":
            set_animation("Pistol_idle")
        # RIFLE transitions
        elif current_state == "Rifle_equip":
            set_animation("Rifle_idle")
        elif current_state == "Rifle_idle":
            pass;
        elif current_state == "Rifle_fire":
            set_animation("Rifle_idle")
        elif current_state == "Rifle_unequip":
            set_animation("Idle_unarmed")
        elif current_state == "Rifle_reload":
            set_animation("Rifle_idle")

    func animation_callback():
        if callback_function == null:
            print ("AnimationPlayer_Manager.gd -- WARNING: No callback function for the animation to call!")
        else:
            callback_function.call_func()

Lets go over what this script is doing:

_________

Lets start with this script's global variables:

- ``states``: A dictionary for holding our animation states. (Further explanation below)
- ``animation_speeds``: A dictionary for holding all of the speeds we want to play our animations at.
- ``current_state``: A variable for holding the name of the animation state we are currently in.
- ``callback_function``: A variable for holding the callback function. (Further explanation below)

If you are familiar with state machines, then you may have noticed that ``states`` is structured
like a basic state machine. Here is roughly how ``states`` is set up:

``states`` is a dictionary with the key being the name of the current state, and the value being
an array holding all of the states we can transition to. For example, if we are in currently in
state ``Idle_unarmed``, we can only transition to ``Knife_equip``, ``Pistol_equip``, ``Rifle_equip``, and
``Idle_unarmed``.

If we try to transition to a state that is not included in our possible transitions states,
then we get a warning message and the animation does not change. We will also automatically
transition from some states into others, as will be explained further below in ``animation_ended``

.. note:: For the sake of keeping this tutorial simple we are not using a 'proper'
          state machine. If you are interested to know more about state machines,
          see the following articles:

          - (Python example) https://dev.to/karn/building-a-simple-state-machine-in-python
          - (C# example) https://www.codeproject.com/Articles/489136/UnderstandingplusandplusImplementingplusStateplusP
          - (Wiki article) https://en.wikipedia.org/wiki/Finite-state_machine

          In a future part of this tutorial series we may revise this script to include a proper state machine.

``animation_speeds`` is how fast each animation will play. Some of the animations are a little slow
and in an effort to make everything smooth, we need to play them at faster speeds than some
of the others.

-- note:: Notice that all of the firing animations are faster than their normal speed. Remember this for later!

``current_state`` will hold the name of the animation state we are currently in.

Finally, ``callback_function`` will be a :ref:`FuncRef <class_FuncRef>` passed in by our player for spawning bullets
at the proper frame of animation. A :ref:`FuncRef <class_FuncRef>` allows us to pass in a function as an argument,
effectively allowing us to call a function from another script, which is how we will use it later.

_________

Now lets look at ``_ready``. First we are setting our animation to ``Idle_unarmed``, using the ``set_animation`` function,
so we for sure start in that animation. Next we connect the ``animation_finished`` signal to this script and assign
it to call ``animation_ended``.

_________

Lets look at ``set_animation`` next.

``set_animation`` sets the animation to the that of the passed in
animation state *if* we can transition to it. In other words, if the animation state we are currently in
has the passed in animation state name in ``states``, then we will change to that animation.

First we check if the passed in animation is the same as the animation state we are currently in.
If it is, then we write a warning to the console and return ``true``.

Next we see if :ref:`AnimationPlayer <class_AnimationPlayer>` has the passed in animation using ``has_animation``. If it does not, we return ``false``.

Then we check if ``current_state`` is set or not. If ``current_state`` is *not* currently set, we
set ``current_state`` to the passed in animation and tell :ref:`AnimationPlayer <class_AnimationPlayer>` to start playing the animation with
a blend time of ``-1`` and at the speed set in ``animation_speeds`` and then we return ``true``.

If we have a state in ``current_state``, then we get all of the possible states we can transition to.
If the animation name is in the array of possible transitions, then we set ``current_state`` to the passed
in animation, tell :ref:`AnimationPlayer <class_AnimationPlayer>` to play the animation with a blend time of ``-1`` at the speed set in ``animation_speeds``
and then we return ``true``.

_________

Now lets look at ``animation_ended``.

``animation_ended`` is the function that will be called by :ref:`AnimationPlayer <class_AnimationPlayer>` when it's done playing a animation.


For certain animation states, we may need to transition into another state when its finished. To handle this, we
check for every possible animation state. If we need to, we transition into another state.

.. warning:: If you are using your own animated models, make sure that none of the animations are set
             to loop. Looping animations do not send the ``animation_finished`` signal when they reach
             the end of the animation and are about to loop.

.. note:: the transitions in ``animation_ended`` ideally would be part of the data in ``states``, but in
          an effort to make the tutorial easier to understand, we'll just hard code each state transition
          in ``animation_ended``.

_________

Finally we have ``animation_callback``. This function will be called by a function track in our animations.
If we have a :ref:`FuncRef <class_FuncRef>` assigned to ``callback_function``, then we call that passed in function. If we do not
have a :ref:`FuncRef <class_FuncRef>` assigned to ``callback_function``, we print out a warning to the console.

.. tip:: Try running ``Testing_Area.tscn`` just to make sure there is no runtime issues. If the game runs but nothing
         seems to have changed, then everything is working correctly.

Getting the animations ready
----------------------------

Now that we have a working animation manager, we need to call it from our player script.
Before that though, we need to set some animation callback tracks in our firing animations.

Open up ``Player.tscn`` if you don't have it open and navigate to the :ref:`AnimationPlayer <class_AnimationPlayer>` node
(``Player``->``Rotation_helper``->``Model``->``AnimationPlayer``).

We need to attach a function track to three of our animations: The firing animation for the pistol, rifle, and knife.
Let's start with the pistol. Click the animation drop down list and select "Pistol_fire".

Now scroll down to the very bottom of the list of animation tracks. The final item in the list should read
``Armature/Skeleton:Left_UpperPointer``. Now at the bottom of the list, click the plus icon on the bottom
bar of animation window, right plus right next to the loop button and the up arrow.

.. image:: img/AnimationPlayerAddTrack.png

This will bring up a window with three choices. We're wanting to add a function callback track, so click the
option that reads "Add Call Func Track". This will open a window showing the entire node tree. Navigate to the
:ref:`AnimationPlayer <class_AnimationPlayer>` node, select it, and press OK.

.. image:: img/AnimationPlayerCallFuncTrack.png

Now at the bottom of list of animation tracks you will have a green track that reads "AnimationPlayer".
Now we need to add the point where we want to call our callback function. Scrub the timeline until you
reach the point where the muzzle just starts to flash.

.. note:: The timeline is the window where all of the points in our animation are stored. Each of the little
          points represents a point of animation data.

          Scrubbing the timeline means moving ourselves through the animation. So when we say "scrub the timeline
          until you reach a point", what we mean is move through the animation window until you reach the a point
          on the timeline.

          Also, the muzzle of a gun is the end point where the bullet comes out. The muzzle flash is the flash of
          light that escapes the muzzle when a bullet is fired. The muzzle is also sometimes referred to as the
          barrel of the gun.

.. tip:: For finer control when scrubbing the timeline, press ``control`` and scroll forwards with the mouse wheel to zoom in.
         Scrolling backwards will zoom out.

         You can also change how the timeline scrubbing snaps by changing the value in ``Step (s)`` to a lower/higher value.

Once you get to a point you like, press the little green plus symbol on the far right side of the
``AnimationPlayer`` track. This will place a little green point at the position you are currently
at in the animation on your ``AnimationPlayer`` track.

.. image:: img/AnimationPlayerAddPoint.png

Now we have one more step before we are done with the pistol. Select the "enable editing of individual keys"
button on the far right corner of the animation window. It looks like a pencil with a little point beside it.

.. image:: img/AnimationPlayerEditPoints.png

Once you've click that, a new window will open on the right side. Now click the green point on the ``AnimationPlayer``
track. This will bring up the information associated with that point in the timeline. In the empty name field, enter
"animation_callback" and press ``enter``.

Now when we are playing this animation the callback function will be triggered at that specific point of the animation.

.. warning:: Be sure to press the "enable editing of individual keys" button again to turn off the ability to edit individual keys
              so you cannot change one of the transform tracks by accident!

_________

Let's repeat the process for the rifle and knife firing animations!

.. note:: Because the process is exactly the same as the pistol, the process is going to explained in a little less depth.
          Follow the steps in the above if you get lost! It is exactly the same, just on a different animation.

Go to the "Rifle_fire" animation from the animation drop down. Add the function callback track once you reach the bottom of the
animation track list by clicking the little plus icon at the bottom of the screen. Find the point where the muzzle just starts
to flash and click the little green plus symbol to add a function callback point at that position on the track.

Next, click the "enable editing of individual keys" button, the button with a plus at the bottom right side of the animation window.
Select the newly created function callback point, put "animation_callback" into the name field and press ``enter``.
Click the "enable editing of individual keys" button again to turn off individual key editing.
so we cannot change one of the transform tracks by accident.

Now we just need to apply the callback function track to the knife animation. Select the "Knife_fire" animation and scroll to the bottom of the
animation tracks. Click the plus symbol at the bottom of the animation window and add a function callback track.
Next find a point around the first third of the animation to place the animation callback function point at.

.. note:: We will not actually be firing the knife, and the animation really is a stabbing animation rather than a firing one.
         For this tutorial we are just reusing the gun firing logic for our knife, so the animation has been named in a style that
         is consistent with the other animations.

From there click the little green plus to add a function callback point at the current position. Then click the "enable editing of individual keys"
button, the button with a plus at the bottom right side of the animation window.
Select the newly created function callback point, put "animation_callback" into the name field and press ``enter``.
Click the "enable editing of individual keys" button again to turn off individual key editing.
so we cannot change one of the transform tracks by accident.

.. tip:: Be sure to save your work!

With that done, we are almost ready to start adding the ability to fire to our player script! We just need to setup one last scene:
The scene for our bullet object.

Creating the bullet scene
-------------------------

There are several ways to handle a gun's bullets in video games. In this tutorial series,
we will be exploring two of the more common ways: Objects, and raycasts.

_________

One of the two ways is using a bullet object. This will be a object that travels through the world and handles
its own collision code. This method we create/spawn a bullet object in the direction our gun is facing, and then
it sends itself forward.

There are several advantages to this method. The first being we do not have to store the bullets in our player. We can simply create the bullet
and then move on, and the bullet itself with handle checking for collisions, sending the proper signal(s) to the object it collides with, and destroying itself.

Another advantage is we can have more complex bullet movement. If we want to make the bullet fall ever so slightly as time goes on, we can make the bullet
controlling script slowly push the bullet towards the ground. Using a object also makes the bullet take time to reach its target, it doesn't just instantly
hit whatever its pointed at. This feels more realistic because nothing in real life really moves instantly from one point to another.

One of the huge disadvantages performance. While having each bullet calculate their own paths and handle their own collision allows for a lot of flexibility,
it comes at the cost of performance. With this method we are calculating every bullet's movement every step, and while this may not be a problem for a few dozen
bullets, it can become a huge problem when you potentially have several hundred bullets.

Despite the performance hit, many first person shooters include some form of object bullets. Rocket launchers are a prime example because in many
first person shooters, Rockets do not just instantly explode at their target position. You can also find bullets as object many times with grenades
because they generally bounce around the world before exploding.

.. note:: While I cannot say for sure this is the case, these games *probably* use bullet objects in some form or another:
          (These are entirely from my observations. **They may be entirely wrong**. I have never worked on **any** of the following games)

          - Halo (Rocket launchers, fragment grenades, sniper rifles, brute shot, and more)
          - Destiny (Rocket launchers, grenades, fusion rifles, sniper rifles, super moves, and more)
          - Call of Duty (Rocket launchers, grenades, ballistic knifes, crossbows, and more)
          - Battlefield (Rocket launchers, grenades, claymores, mortars, and more)

Another disadvantage with bullet objects is networking. Bullet objects have to sync the positions (at least) with however many clients are connected
to the server.

While we are not implementing any form of networking (as that would be it's own entire tutorial series), it is a consideration
to keep in mind when creating your first person shooter, especially if you plan on adding some form of networking in the future.

_________

The other way of handling bullet collisions we will be looking at, is raycasting.

This method is extremely common in guns that have fast moving bullets that rarely change trajectory change over time.

Instead of creating a bullet object and sending it through space, we instead send a ray starting from the barrel/muzzle of the gun forwards.
We set the raycast's origin to the starting position of the bullet, and based on the length we can adjust how far the bullet 'travels' through space.

.. note:: While I cannot say for sure this is the case, these games *probably* use raycasts in some form or another:
          (These are entirely from my observations. **They may be entirely wrong**. I have never worked on **any** of the following games)

          - Halo (Assault rifles, DMRs, battle rifles, covenant carbine, spartan laser, and more)
          - Destiny (Auto rifles, pulse rifles, scout rifles, hand cannons, machine guns, and more)
          - Call of Duty (Assault rifles, light machine guns, sub machine guns, pistols, and more)
          - Battlefield (Assault rifles, SMGs, carbines, pistols, and more)

One huge advantage for this method is it's really light on performance.
Sending a couple hundred rays through space is *way* easier for the computer to calculate than sending a couple hundred
bullet objects.

Another advantage is we can instantly know if we've hit something or not exactly when we call for it. For networking this is important because we do not need
to sync the bullet movements over the Internet, we just need to send whether or not the raycast hit.

Raycasting does have some disadvantages though. One major disadvantage is we cannot easily cast a ray in anything but a linear line.
This means we can only fire in a straight line for however long our ray length is. You can create the illusion of bullet movement by casting
multiple rays at different positions, but not only is this hard to implement in code, it is also is heavier on performance.

Another disadvantage is we cannot see the bullet. With bullet objects we can actually see the bullet travel through space if we attach a mesh
to it, but because raycasts happen instantly, we do not really have a decent way of showing the bullets. You could draw a line from the origin of the
raycast to the point where the raycast collided, and that is one popular way of showing raycasts. Another way is simply not drawing the raycast
at all, because theoretically the bullets move so fast our eyes could not see it anyway.

_________

Lets get the bullet object setup. This is what our pistol will create when the "Pistol_fire" animation callback function is called.

Open up ``Bullet_Scene.tscn``. The scene contains :ref:`Spatial <class_Spatial>` node called bullet, with a :ref:`MeshInstance <class_MeshInstance>`
and an :ref:`Area <class_Area>` with a :ref:`CollisionShape <class_CollisionShape>` childed to it.

Create a new script called ``Bullet_script.gd`` and attach it to the ``Bullet`` :ref:`Spatial <class_Spatial>`.

We are going to move the entire bullet object at the root (``Bullet``). We will be using the :ref:`Area <class_Area>` to check whether or not we've collided with something

.. note:: Why are we using a :ref:`Area <class_Area>` and not a :ref:`RigidBody <class_RigidBody>`? The mean reason we're not using a :ref:`RigidBody <class_RigidBody>`
          is because we do not want the bullet to interact with other :ref:`RigidBody <class_RigidBody>` nodes.
          By using an :ref:`Area <class_Area>` we are assuring that none of the other :ref:`RigidBody <class_RigidBody>` nodes, including other bullets, will be effected.

          Another reason is simply because it is easier to detect collisions with a :ref:`Area <class_Area>`!

Here's the script that will control our bullet:

::

    extends Spatial

    const BULLET_SPEED = 80
    const BULLET_DAMAGE = 15

    const KILL_TIMER = 4
    var timer = 0

    var hit_something = false

    func _ready():
        get_node("Area").connect("body_entered", self, "collided")
        set_physics_process(true)


    func _physics_process(delta):
        var forward_dir = global_transform.basis.z.normalized()
        global_translate(forward_dir * BULLET_SPEED * delta)

        timer += delta;
        if timer >= KILL_TIMER:
            queue_free()


    func collided(body):
        if hit_something == false:
            if body.has_method("bullet_hit"):
                body.bullet_hit(BULLET_DAMAGE, self.global_transform.origin)

        hit_something = true
        queue_free()


Lets go through the script:

_________

First we define a few global variables:

- ``BULLET_SPEED``: The speed the bullet travels at.
- ``BULLET_DAMAGE``: The damage the bullet will cause to whatever it collides with.
- ``KILL_TIMER``: How long the bullet can last without hitting anything.
- ``timer``: A float for tracking how long we've been alive.
- ``hit_something``: A boolean for tracking whether or not we've hit something.

With the exception of ``timer`` and ``hit_something``, all of these variables
change how the bullet interacts with the world.

.. note:: The reason we are using a kill timer is so we do not have a case where we
          get a bullet traveling forever. By using a kill timer, we can assure that
          no bullets will just travel forever and consume resources.

_________

In ``_ready`` we set the area's ``body_entered`` signal to ourself so that it calls
the ``collided`` function. Then we set ``_physics_process`` to ``true``.

_________

``_physics_process`` gets the bullet's local ``Z`` axis. If you look in at the scene
in local mode, you will find that the bullet faces the positive local ``Z`` axis.

Next we translate the entire bullet by that forward direction, multiplying in our speed and delta time.

After that we add delta time to our timer and check if the timer has as long or longer
than our ``KILL_TIME`` constant. If it has, we use ``queue_free`` to free ourselves.

_________

In ``collided`` we check if we've hit something yet or not.

Remember that ``collided`` is
only called when a body has entered the :ref:`Area <class_Area>` node. If we have not already collided with
something, we the proceed to check if the body we've collided with has a function/method
called ``bullet_hit``. If it does, we call it and pass in our damage and our position.

.. note:: in ``collided``, the passed in body can be a :ref:`StaticBody <class_StaticBody>`,
          :ref:`RigidBody <class_RigidBody>`, or :ref:`KinematicBody <class_KinematicBody>`

Then we set ``hit_something`` to ``true`` because regardless of whether or not the body
the bullet collided with has the ``bullet_hit`` function/method, it have hit something.

Then we free the bullet using ``queue_free``.

.. tip:: You may be wondering why we even have a ``hit_something`` variable if we
         free the bullet using ``queue_free`` as soon as it hits something.

         The reason we need to track whether we've hit something or not is because ``queue_free``
         does not immediately free the node, so the bullet could collide with another body
         before Godot has a chance to free it. By tracking if the bullet has hit something
         we can make sure that the bullet will only hit one object.


_________

Before we start programming the player again, let's take a quick look at ``Player.tscn``.
Open up ``Player.tscn`` again.

Expand ``Rotation_helper`` and notice how it has two nodes: ``Gun_fire_points`` and
``Gun_aim_point``.

``Gun_aim_point`` is the point that the bullets will be aiming at. Notice how it
is lined up with the center of the screen and pulled a distance forward on the Z
axis. ``Gun_aim_point`` will serve as the point where the bullets will for sure collied
with as it goes along.

.. note:: There is a invisible mesh instance for debugging purposes. The mesh is
          a small sphere that visually shows where the bullets will be aiming at.

Open up ``Gun_fire_points`` and you'll find three more :ref:`Spatial <class_Spatial>` nodes, one for each
weapon.

Open up ``Rifle_point`` and you'll find a :ref:`Raycast <class_Raycast>` node. This is where
we will be sending the raycasts for our rilfe's bullets.
The length of the raycast will dictate how far our the bullets will travel.

We are using a :ref:`Raycast <class_Raycast>` node to handle the rifle's bullet because
we want to fire lots of bullets quickly. If we use bullet objects, it is quite possible
we could run into performance issues on older machines.

.. note:: If you are wondering where the positions of the points came from, they
          are the rough positions of the ends of each weapon. You can see this by
          going to ``AnimationPlayer``, selecting one of the firing animations
          and scrubbing through the timeline. The point for each weapon should mostly line
          up with the end of each weapon.

Open up ``Knife_point`` and you'll find a :ref:`Area <class_Area>` node. We are using a :ref:`Area <class_Area>` for the knife
because we only care for all of the bodies close to us, and because our knife does
not fire into space. If we were making a throwing knife, we would likely spawn a bullet
object that looks like a knife.

Finally, we have ``Pistol point``. This is the point where we will be creating/instancing
our bullet objects. We do not need any additional nodes here, as the bullet handles all
of its own collision detection.

Now that we've seen how we will handle our other weapons, and where we will spawn the bullets,
let's start working on making them work.

.. note:: You can also look at the HUD nodes if you want. There is nothing fancy there and other
         than using a single :ref:`Label <class_Label>`, we will not be touching any of those nodes.
         Check :ref:`doc_design_interfaces_with_the_control_nodes` for a tutorial on using GUI nodes.

         The GUI provided in this tutorial is *very* basic. Maybe in a later part we will
         revise the GUI, but for now we are going to just use this GUI as it will serve our needs for now.

Making the weapons work
-----------------------

Lets start making the weapons work in ``Player.gd``.

First lets start by adding some global variables we'll need for the weapons:

::

    # Place before _ready
    var animation_manager;

    var current_gun = "UNARMED"
    var changing_gun = false

    var bullet_scene = preload("Bullet_Scene.tscn")

    var health = 100

    const RIFLE_DAMAGE = 4
    const KNIFE_DAMAGE = 40

    var UI_status_label

Let's go over what these new variables will do:

- ``animation_manager``: This will hold the :ref:`AnimationPlayer <class_AnimationPlayer>` node and its script, which we wrote previously.
- ``current_gun``: This is the name of the gun we are currently using. It has four possible values: ``UNARMED``, ``KNIFE``, ``PISTOL``, and ``RIFLE``.
- ``changing_gun``: A boolean to track whether or not we are changing guns/weapons.
- ``bullet_scene``: The bullet scene we worked on earlier, ``Bullet_Scene.tscn``. We need to load it here so we can create/spawn it when the pistol fires
- ``health``: How much health our player has. In this part of the tutorial we will not really be using it.
- ``RIFLE_DAMAGE``: How much damage a single rifle bullet causes.
- ``KNIFE_DAMAGE``: How much damage a single knife stab/swipe causes.
- ``UI_status_label``: A label to show how much health we have, and how much ammo we have both in our gun and in reserves.

_________

Next we need to add a few things in ``_ready``. Here's the new ``_ready`` function:

::

    func _ready():
        camera = get_node("Rotation_helper/Camera")
        rotation_helper = get_node("Rotation_helper")

        animation_manager = get_node("Rotation_helper/Model/AnimationPlayer")
        animation_manager.callback_function = funcref(self, "fire_bullet")

        set_physics_process(true)

        Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)
        set_process_input(true)

        # Make sure the bullet spawn point, the raycast, and the knife area are all aiming at the center of the screen
        var gun_aim_point_pos = get_node("Rotation_helper/Gun_aim_point").global_transform.origin
        get_node("Rotation_helper/Gun_fire_points/Pistol_point").look_at(gun_aim_point_pos, Vector3(0, 1, 0))
        get_node("Rotation_helper/Gun_fire_points/Rifle_point").look_at(gun_aim_point_pos, Vector3(0, 1, 0))
        get_node("Rotation_helper/Gun_fire_points/Knife_point").look_at(gun_aim_point_pos, Vector3(0, 1, 0))

        # Because we have the camera rotated by 180 degrees, we need to rotate the points around by 180
        # degrees on their local Y axis because otherwise the bullets will fire backwards
        get_node("Rotation_helper/Gun_fire_points/Pistol_point").rotate_object_local(Vector3(0, 1, 0), deg2rad(180))
        get_node("Rotation_helper/Gun_fire_points/Rifle_point").rotate_object_local(Vector3(0, 1, 0), deg2rad(180))
        get_node("Rotation_helper/Gun_fire_points/Knife_point").rotate_object_local(Vector3(0, 1, 0), deg2rad(180))

        UI_status_label = get_node("HUD/Panel/Gun_label")
        flashlight = get_node("Rotation_helper/Flashlight")

Let's go over what's changed.

First we get the :ref:`AnimationPlayer <class_AnimationPlayer>` node and assign it to our animation_manager variable. Then we set the callback function
to a :ref:`FuncRef <class_FuncRef>` that will call the player's ``fire_bullet`` function. Right now we haven't written our fire_bullet function,
but we'll get there soon.

Then we get all of the weapon points and call each of their ``look_at``.
This will make sure they all are facing the gun aim point, which is in the center of our camera at a certain distance back.

Next we rotate all of those weapon points by ``180`` degrees on their ``Y`` axis. This is because our camera is pointing backwards.
If we did not rotate all of these weapon points by ``180`` degrees, all of the weapons would fire backwards at ourselves.

Finally, we get the UI :ref:`Label <class_Label>` from our HUD.

_________

Lets add a few things to ``_physics_process`` so we can fire our weapons. Here's the new code:

::

    func _physics_process(delta):
        var dir = Vector3()
        var cam_xform = camera.get_global_transform()


        if Input.is_action_pressed("movement_forward"):
            dir += -cam_xform.basis.z.normalized()
        if Input.is_action_pressed("movement_backward"):
            dir += cam_xform.basis.z.normalized()
        if Input.is_action_pressed("movement_left"):
            dir += -cam_xform.basis.x.normalized()
        if Input.is_action_pressed("movement_right"):
            dir += cam_xform.basis.x.normalized()


        if is_on_floor():
            if Input.is_action_just_pressed("movement_jump"):
                vel.y = JUMP_SPEED

        if Input.is_action_just_pressed("flashlight"):
            if flashlight.is_visible_in_tree():
                flashlight.hide()
            else:
                flashlight.show()

        if Input.is_action_pressed("movement_sprint"):
            is_sprinting = true;
        else:
            is_sprinting = false;

        dir.y = 0
        dir = dir.normalized()

        var grav = norm_grav

        vel.y += delta*grav

        var hvel = vel
        hvel.y = 0

        var target = dir
        if is_sprinting:
            target *= MAX_SPRINT_SPEED
        else:
            target *= MAX_SPEED

        var accel
        if dir.dot(hvel) > 0:
            if is_sprinting:
                accel = SPRINT_ACCEL
            else:
                accel = ACCEL
        else:
            accel = DEACCEL

        hvel = hvel.linear_interpolate(target, accel*delta)
        vel.x = hvel.x
        vel.z = hvel.z
        vel = move_and_slide(vel,Vector3(0,1,0), 0.05, 4, deg2rad(MAX_SLOPE_ANGLE))


        if Input.is_action_just_pressed("ui_cancel"):
            if Input.get_mouse_mode() == Input.MOUSE_MODE_VISIBLE:
                Input.set_mouse_mode(Input.MOUSE_MODE_CAPTURED)
            else:
                Input.set_mouse_mode(Input.MOUSE_MODE_VISIBLE)

        # NEW CODE
        if changing_gun == false:
    		if Input.is_key_pressed(KEY_1):
    			current_gun = "UNARMED"
    			changing_gun = true
    		elif Input.is_key_pressed(KEY_2):
    			current_gun = "KNIFE"
    			changing_gun = true
    		elif Input.is_key_pressed(KEY_3):
    			current_gun = "PISTOL"
    			changing_gun = true
    		elif Input.is_key_pressed(KEY_4):
    			current_gun = "RIFLE"
    			changing_gun = true

        if changing_gun == true:
    		if current_gun != "PISTOL":
    			if animation_manager.current_state == "Pistol_idle":
    				animation_manager.set_animation("Pistol_unequip")
    		if current_gun != "RIFLE":
    			if animation_manager.current_state == "Rifle_idle":
    				animation_manager.set_animation("Rifle_unequip")
    		if current_gun != "KNIFE":
    			if animation_manager.current_state == "Knife_idle":
    				animation_manager.set_animation("Knife_unequip")

    		if current_gun == "UNARMED":
    			if animation_manager.current_state == "Idle_unarmed":
    				changing_gun = false

    		elif current_gun == "KNIFE":
    			if animation_manager.current_state == "Knife_idle":
    				changing_gun = false
    			if animation_manager.current_state == "Idle_unarmed":
    				animation_manager.set_animation("Knife_equip")

    		elif current_gun == "PISTOL":
    			if animation_manager.current_state == "Pistol_idle":
    				changing_gun = false
    			if animation_manager.current_state == "Idle_unarmed":
    				animation_manager.set_animation("Pistol_equip")

    		elif current_gun == "RIFLE":
    			if animation_manager.current_state == "Rifle_idle":
    				changing_gun = false
    			if animation_manager.current_state == "Idle_unarmed":
    				animation_manager.set_animation("Rifle_equip")


	# Firing the weapons
	if Input.is_action_pressed("fire"):
		if current_gun == "PISTOL":
    		if animation_manager.current_state == "Pistol_idle":
    			animation_manager.set_animation("Pistol_fire")

		elif current_gun == "RIFLE":
			if animation_manager.current_state == "Rifle_idle":
				animation_manager.set_animation("Rifle_fire")

		elif current_gun == "KNIFE":
			if animation_manager.current_state == "Knife_idle":
				animation_manager.set_animation("Knife_fire")

    # HUD (UI)
	UI_status_label.text = "HEALTH: " + str(health)


Lets go over it one chunk at a time:

_________

First we have a if check to see if ``changing_gun`` is equal to ``false``. If it is, we
then check to see if the number keys ``1`` through ``4`` are pressed. If one of the keys
are pressed, we set current gun to the name of each weapon assigned to each key and set
``changing_gun`` to ``true``.

Then we check if ``changing_gun`` is ``true``. If it is ``true``, we then go through a series of checks.
The first set of checks is checking if we are in a idle animation that is not the weapon/gun we are trying
to change to, as then we'd be stuck in a loop.

Then we check if we are in an unarmed state. If we are and the newly selected 'weapon'
is ``UNARMED``, then we set ``changing_gun`` to ``false``.

If we are trying to change to any of the other weapons, we first check if we are in the
desired weapon's idle state. If we are, then we've successfully changed weapons and set
``changing_gun`` to false.

If we are not in the desired weapon's idle state, we then check if we are in the idle unarmed state.
This is because all unequip animations transition to idle unarmed, and because we can transition to
any equip animation from idle unarmed.

If we are in the idle unarmed state, we set the animation to the equip animation for the
desired weapon. Once the equip animation is finished, it will change to the idle state for that
weapon, which will pass the ``if`` check above.

_________

For firing the weapons we first check if the ``fire`` action is pressed or not.
If the fire action is pressed, we then check which weapon we are using.

If we are in a weapon's idle state, we then call set the animation to the weapon's fire animation.

_________

Now, we just need to add one more function to the player, and then the player is ready to start shooting!

We just need to add ``fire_bullet``, which will be called when by the :ref:`AnimationPlayer <class_AnimationPlayer>` at those
points we set earlier in the :ref:`AnimationPlayer <class_AnimationPlayer>` function track:

::

    func fire_bullet():
        if changing_gun == true:
            return

        # Pistol bullet handling: Spawn a bullet object!
        if current_gun == "PISTOL":
            var clone = bullet_scene.instance()
            var scene_root = get_tree().root.get_children()[0]
            scene_root.add_child(clone)

            clone.global_transform = get_node("Rotation_helper/Gun_fire_points/Pistol_point").global_transform
            # The bullet is a little too small (by default), so let's make it bigger!
            clone.scale = Vector3(4, 4, 4)

        # Rifle bullet handeling: Send a raycast!
        elif current_gun == "RIFLE":
            var ray = get_node("Rotation_helper/Gun_fire_points/Rifle_point/RayCast")
            ray.force_raycast_update()

            if ray.is_colliding():
                var body = ray.get_collider()
                if body.has_method("bullet_hit"):
                    body.bullet_hit(RIFLE_DAMAGE, ray.get_collision_point())

        # Knife bullet(?) handeling: Use an area!
        elif current_gun == "KNIFE":
            var area = get_node("Rotation_helper/Gun_fire_points/Knife_point/Area")
            var bodies = area.get_overlapping_bodies()

            for body in bodies:
                if body.has_method("bullet_hit"):
                    body.bullet_hit(KNIFE_DAMAGE, area.global_transform.origin)


Lets go over what this function is doing:

_________

First we check if we are changing weapons or not. If we are changing weapons, we do not want shoot so we just ``return``.

.. tip:: Calling ``return`` stops the rest of the function from being called. In this case, we are not returning a variable
         because we are only interested in not running the rest of the code, and because we are not looking for a returned
         variable either when we call this function.

Next we check which weapon we are using.

If we are using a pistol, we first create a ``Bullet_Scene.tscn`` instance and assign it to
a variable named ``clone``. Then we get the root node in the :ref:`SceneTree <class_SceneTree>`, which happens to be a :ref:`Viewport <class_Viewport>`.
We then get the first child of the :ref:`Viewport <class_Viewport>` and assign it to the ``scene_root`` variable.

We then add our newly instanced/created bullet as a child of ``scene_root``.

.. warning:: As mentioned later below in the section on adding sounds, this method makes a assumption. This will be explained later
             in the section on adding sounds in :ref:`doc_fps_tutorial_part_three`

Next we set the global :ref:`Transform <class_Transform>` of the bullet to that of the pistol bullet spawn point we
talked about earlier.

Finally, we set the scale a little bigger because the bullet normally is too small to see.

_______

For the rifle, we first get the :ref:`Raycast <class_Raycast>` node and assign it to a variable called ``ray``.
Then we call :ref:`Raycast <class_Raycast>`'s ``force_raycast_update`` function.

``force_raycast_update`` sends the :ref:`Raycast <class_Raycast>` out and collects the collision data as soon as we call it,
meaning we get frame perfect collision data and we do not need to worry about performance issues by having the
:ref:`Raycast <class_Raycast>` enabled all the time.

Next we check if the :ref:`Raycast <class_Raycast>` collided with anything. If it has, we then get the collision body
it collided with. If the body has the ``bullet_hit`` method/function, we then call it and pass
in ``RIFLE_DAMAGE`` and the position where the :ref:`Raycast <class_Raycast>` collided.

.. tip:: Remember how we mentioned the speed of the animations for firing was faster than
         the other animations? By changing the firing animation speeds, you can change how
         fast the weapon fires bullets!

_______

For the knife we first get the :ref:`Area <class_Area>` node and assign it to a variable named ``area``.
Then we get all of the collision bodies inside the :ref:`Area <class_Area>`. We loop through each one
and check if they have the ``bullet_hit`` method/function. If they do, we call it and pass
in ``KNIFE_DAMAGE`` and the global position of :ref:`Area <class_Area>`.

.. note:: While we could attempt to calculate a rough location for where the knife hit, we
          do not bother because using the area's position works well enough and the extra time
          needed to calculate a rough position for each body is not really worth the effort.

_______


Before we are ready to test our new weapons, we still have just a little bit of work to do.

Creating some test subjects
---------------------------

Create a new script by going to the scripting window, clicking "file", and selecting new.
Name this script "RigidBody_hit_test" and make sure it extends :ref:`RigidBody <class_RigidBody>`.

Now we just need to add this code:

::

    extends RigidBody

    func _ready():
        pass

    func bullet_hit(damage, bullet_hit_pos):
        var direction_vect = self.global_transform.origin - bullet_hit_pos
        direction_vect = direction_vect.normalized()

        self.apply_impulse(bullet_hit_pos, direction_vect * damage)


Lets go over how ``bullet_hit`` works:

First we get the direction from the bullet pointing towards our global :ref:`Transform <class_Transform>`.
We do this by subtracting the bullet's hit position from the :ref:`RigidBody <class_RigidBody>`'s position.
This results in a :ref:`Vector3 <class_Vector3>` that we can use to tell the direction the bullet collided into the
:ref:`RigidBody <class_RigidBody>` at.

We then normalize it so we do not get crazy results from collisions on the extremes
of the collision shape attached to the :ref:`RigidBody <class_RigidBody>`. Without normalizing shots farther
away from the center of the :ref:`RigidBody <class_RigidBody>` would cause a more noticeable reaction than
those closer to the center.

Finally, we apply an impulse at the passed in bullet collision position. With the force
being the directional vector times the damage the bullet is supposed to cause. This makes
the :ref:`RigidBody <class_RigidBody>` seem to move in response to the bullet colliding into it.

_______

Now we just need to attach this script to all of the :ref:`RigidBody <class_RigidBody>` nodes we want to effect.

Open up ``Testing_Area.tscn`` and select all of the cubes parented to the ``Cubes`` node.

.. tip:: If you select the top cube, and then hold down ``shift`` and select the last cube, Godot will
         select all of the cubes in between!

Once you have all of the cubes selected, scroll down in the inspector until you get to the
the "scripts" section. Click the drop down and select "Load". Open your newly created ``RigidBody_hit_test.gd`` script.

With that done, go give your guns a whirl! You should now be able to fire as many bullets as you want on the cubes and
they will move in response to the bullets colliding into them.

In :ref:`doc_fps_tutorial_part_three`, we will add ammo to the guns, as well as some sounds!

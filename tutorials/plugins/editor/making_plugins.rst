.. _doc_making_plugins:

Making plugins
==============

About plugins
~~~~~~~~~~~~~

A plugin is a great way to extend the editor with useful tools. It can be made
entirely with GDScript and standard scenes, without even reloading the editor.
Unlike modules, you don't need to create C++ code nor recompile the engine.
While this makes plugins less powerful, there's still a lot of things you can
do with them. Note that a plugin is similar to any scene you can already
make, except it is created using a script to add functionality.

This tutorial will guide you through the creation of two simple plugins so
you can understand how they work and be able to develop your own. The first
will be a custom node that you can add to any scene in the project and the
other will be a custom dock added to the editor.

Creating a plugin
~~~~~~~~~~~~~~~~~

Before starting, create a new empty project wherever you want. This will serve
as base to develop and test the plugins.

The first thing you need to do is to create a new plugin the editor can
understand as such. You need two files for that: ``plugin.cfg`` for the
configuration and a custom GDScript with the functionality.

Plugins have a standard path like ``addons/plugin_name`` inside the project
folder. For this example, create a folder ``my_custom_node`` inside ``addons``.
You should end up with a directory structure like this:

.. image:: img/making_plugins-my_custom_mode_folder.png

Now, open the script editor, click the **File** menu, choose **New TextFile**,
then navigate to the plugin folder and name the file ``plugin.cfg``.
Add the following structure to ``plugin.cfg``::

    [plugin]

    name="My Custom Node"
    description="A custom node made to extend the Godot Engine."
    author="Your Name Here"
    version="1.0.0"
    script="custom_node.gd"

This is a simple INI file with metadata about your plugin. You need to set
the name and description so people can understand what it does. Don't forget
to add your own name so you can be properly credited. Add a version number
so people can see if they have an outdated version; if you are unsure on
how to come up with the version number, check out `Semantic Versioning <https://semver.org/>`_.
Finally, set the main script file to load when your plugin is active.

The script file
^^^^^^^^^^^^^^^

Open the script editor (F3) and create a new GDScript file called
``custom_node.gd`` inside the ``my_custom_node`` folder. This script is special
and it has two requirements: it must be a ``tool`` script and it has to
inherit from :ref:`class_EditorPlugin`.

It's important to deal with initialization and clean-up of resources.
A good practice is to use the virtual function
:ref:`_enter_tree() <class_Node_method__enter_tree>` to initialize your plugin and
:ref:`_exit_tree() <class_Node_method__exit_tree>` to clean it up. You can delete the
default GDScript template from your file and replace it with the following
structure:

.. _doc_making_plugins_template_code:
.. code-block:: python

    tool
    extends EditorPlugin

    func _enter_tree():
        # Initialization of the plugin goes here
        pass

    func _exit_tree():
        # Clean-up of the plugin goes here
        pass

This is a good template to use when creating new plugins.

A custom node
~~~~~~~~~~~~~

Sometimes you want a certain behavior in many nodes, such as a custom scene
or control that can be reused. Instancing is helpful in a lot of cases, but
sometimes it can be cumbersome, especially if you're using it in many
projects. A good solution to this is to make a plugin that adds a node with a
custom behavior.

To create a new node type, you can use the function
:ref:`add_custom_type() <class_EditorPlugin_method_add_custom_type>` from the
:ref:`class_EditorPlugin` class. This function can add new types to the editor
(nodes or resources). However, before you can create the type, you need a script
that will act as the logic for the type. While that script doesn't have to use
the ``tool`` keyword, it can be added so the script runs in the editor.

For this tutorial, we'll create a simple button that prints a message when
clicked. For that, we'll need a simple script that extends from
:ref:`class_Button`. It could also extend
:ref:`class_BaseButton` if you prefer::

    tool
    extends Button

    func _enter_tree():
        connect("pressed", self, "clicked")

    func clicked():
        print("You clicked me!")

That's it for our basic button. You can save this as ``button.gd`` inside the
plugin folder. You'll also need a 16×16 icon to show in the scene tree. If you
don't have one, you can grab the default one from the engine and save it in your
`addons/my_custom_node` folder as `icon.png`, or use the default Godot logo
(`preload("res://icon.png")`). You can also use SVG icons if desired.

.. image:: img/making_plugins-custom_node_icon.png

Now, we need to add it as a custom type so it shows on the **Create New Node**
dialog. For that, change the ``custom_node.gd`` script to the following::

    tool
    extends EditorPlugin

    func _enter_tree():
        # Initialization of the plugin goes here
        # Add the new type with a name, a parent type, a script and an icon
        add_custom_type("MyButton", "Button", preload("button.gd"), preload("icon.png"))

    func _exit_tree():
        # Clean-up of the plugin goes here
        # Always remember to remove it from the engine when deactivated
        remove_custom_type("MyButton")

With that done, the plugin should already be available in the plugin list in the
**Project Settings**, so activate it as explained in `Checking the results`_.

Then try it out by adding your new node:

.. image:: img/making_plugins-custom_node_create.png

When you add the node, you can see that it already have the script you created
attached to it. Set a text to the button, save and run the scene. When you
click the button, you can see some text in the console:

.. image:: img/making_plugins-custom_node_console.png

A custom dock
^^^^^^^^^^^^^

Sometimes, you need to extend the editor and add tools that are always available.
An easy way to do it is to add a new dock with a plugin. Docks are just scenes
based on Control, so they are created in a way similar to usual GUI scenes.

Creating a custom dock is done just like a custom node. Create a new
``plugin.cfg`` file in the ``addons/my_custom_dock`` folder, then
add the following content to it::

    [plugin]

    name="My Custom Dock"
    description="A custom dock made so I can learn how to make plugins."
    author="Your Name Here"
    version="1.0"
    script="custom_dock.gd"

Then create the script ``custom_dock.gd`` in the same folder. Fill it with the
:ref:`template we've seen before <doc_making_plugins_template_code>` to get a
good start.

Since we're trying to add a new custom dock, we need to create the contents of
the dock. This is nothing more than a standard Godot scene: just create
a new scene in the editor then edit it.

For an editor dock, the root node **must** be a :ref:`Control <class_Control>`
or one of its child classes. For this tutorial, you can create a single button.
The name of the root node will also be the name that appears on the dock tab,
so be sure to give it a short and descriptive name.
Also, don't forget to add some text to your button.

.. image:: img/making_plugins-my_custom_dock_scene.png

Save this scene as ``my_dock.tscn``. Now, we need to grab the scene we created
then add it as a dock in the editor. For this, you can rely on the function
:ref:`add_control_to_dock() <class_EditorPlugin_method_add_control_to_dock>` from the
:ref:`EditorPlugin <class_EditorPlugin>` class.

You need to select a dock position and define the control to add
(which is the scene you just created). Don't forget to
**remove the dock** when the plugin is deactivated.
The script could look like this::

    tool
    extends EditorPlugin

    # A class member to hold the dock during the plugin lifecycle
    var dock

    func _enter_tree():
        # Initialization of the plugin goes here
        # Load the dock scene and instance it
        dock = preload("res://addons/my_custom_dock/my_dock.tscn").instance()

        # Add the loaded scene to the docks
        add_control_to_dock(DOCK_SLOT_LEFT_UL, dock)
        # Note that LEFT_UL means the left of the editor, upper-left dock

    func _exit_tree():
        # Clean-up of the plugin goes here
        # Remove the dock
        remove_control_from_docks(dock)
         # Erase the control from the memory
        dock.free()

Note that while the dock will initially appear at its specified position,
the user can freely change its position and save the resulting layout.

Checking the results
^^^^^^^^^^^^^^^^^^^^

It's now time to check the results of your work. Open the **Project
Settings** and click on the **Plugins** tab. Your plugin should be the only one
on the list. If it is not showing, click on the **Update** button in the
top-right corner.

.. image:: img/making_plugins-project_settings.png

You can see the plugin is inactive on the **Status** column; click on the status
to select **Active**. The dock should become visible before you even close
the settings window. You should now have a custom dock:

.. image:: img/making_plugins-custom_dock.png

Going beyond
~~~~~~~~~~~~

Now that you've learned how to make basic plugins, you can extend the editor in
several ways. Lots of functionality can be added to the editor with GDScript;
it is a powerful way to create specialized editors without having to delve into
C++ modules.

You can make your own plugins to help yourself and share them in the
`Asset Library <https://godotengine.org/asset-library/>`_ so that people
can benefit from your work.

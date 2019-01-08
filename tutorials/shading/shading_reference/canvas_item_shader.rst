.. _doc_canvas_item_shader:

CanvasItem shaders
==================

CanvasItem shaders are used to draw all 2D elements in Godot. These include
all nodes that inherit from CanvasItems, and all GUI elements.

CanvasItem shaders contain less built-in variables and functionality than Spatial 
shaders, but they maintain the same basic structure with vertex, fragment, and 
light processor functions.

Render modes
^^^^^^^^^^^^

+---------------------------------+----------------------------------------------------------------------+
| Render mode                     | Description                                                          |
+=================================+======================================================================+
| **blend_mix**                   | Mix blend mode (alpha is transparency), default.                     |
+---------------------------------+----------------------------------------------------------------------+
| **blend_add**                   | Additive blend mode.                                                 |
+---------------------------------+----------------------------------------------------------------------+
| **blend_sub**                   | Subtractive blend mode.                                              |
+---------------------------------+----------------------------------------------------------------------+
| **blend_mul**                   | Multiplicative blend mode.                                           |
+---------------------------------+----------------------------------------------------------------------+
| **blend_premul_alpha**          | Pre-multiplied alpha blend mode.                                     |
+---------------------------------+----------------------------------------------------------------------+
| **unshaded**                    | Result is just albedo. No lighting/shading happens in material.      |
+---------------------------------+----------------------------------------------------------------------+
| **light_only**                  | Only draw on light pass.                                             |
+---------------------------------+----------------------------------------------------------------------+
| **skip_vertex_transform**       | VERTEX/NORMAL/etc need to be transformed manually in vertex function.|
+---------------------------------+----------------------------------------------------------------------+

Vertex built-ins
^^^^^^^^^^^^^^^^

Values marked as "in" are read-only. Values marked as "out" are for optional writing and will 
not necessarily contain sensible values. Values marked as "inout" provide a sensible default 
value, and can optionally be written to. Samplers are not subjects of writing and they are 
not marked.

Vertex data (``VERTEX``) is presented in local space (pixel coordinates, relative to the camera).
If not written to, these values will not be modified and be passed through as they came.

User can disable the built-in modelview transform (projection will still happen later) and do 
it manually with the following code:

.. code-block:: glsl

    shader_type canvas_item;
    render_mode skip_vertex_transform;

    void vertex() {

        VERTEX = (EXTRA_MATRIX * (WORLD_MATRIX * vec4(VERTEX, 0.0, 1.0))).xy;
    }

Other built-ins such as UV and COLOR are also passed through to the fragment function if not modified.

For instancing, the INSTANCE_CUSTOM variable contains the instance custom data. When using particles, this information
is usually:

* **x**: Rotation angle in radians.
* **y**: Phase during lifetime (0 to 1).
* **z**: Animation frame.

+--------------------------------+----------------------------------------------------------------+
| Built-in                       | Description                                                    |
+================================+================================================================+
| in mat4 **WORLD_MATRIX**       | Image to World transform.                                      |
+--------------------------------+----------------------------------------------------------------+
| in mat4 **EXTRA_MATRIX**       | Extra transform.                                               |
+--------------------------------+----------------------------------------------------------------+
| in mat4 **PROJECTION_MATRIX**  | World to view transform.                                       |
+--------------------------------+----------------------------------------------------------------+
| in float **TIME**              | Global time, in seconds.                                       |
+--------------------------------+----------------------------------------------------------------+
| in vec4 **INSTANCE_CUSTOM**    | Instance custom data.                                          |
+--------------------------------+----------------------------------------------------------------+
| in bool **AT_LIGHT_PASS**      | True if this is a light pass.                                  |
+--------------------------------+----------------------------------------------------------------+
| inout vec2 **VERTEX**          | Vertex, in image space.                                        |
+--------------------------------+----------------------------------------------------------------+
| inout vec2 **UV**              | UV.                                                            |
+--------------------------------+----------------------------------------------------------------+
| inout vec4 **COLOR**           | Color from vertex primitive.                                   |
+--------------------------------+----------------------------------------------------------------+
| inout float **POINT_SIZE**     | Point size for point drawing.                                  |
+--------------------------------+----------------------------------------------------------------+

Fragment built-ins
^^^^^^^^^^^^^^^^^^

Certain Nodes (for example, :ref:`Sprites <class_Sprite>`) display a texture by default. However, 
when a custom fragment function is attached to these nodes the texture lookup needs to be done 
manually. Godot does not provide the texture color in the ``COLOR`` built-in variable, to read 
the texture color for such nodes use:

.. code-block:: glsl

  COLOR = texture(TEXTURE, UV);

This differs from the behaviour of the built-in normal map. If a normal map is attached, Godot uses
it by default and assigns its value to the built-in ``NORMAL`` variable. If you are using a normal
map meant for use in 3D, it will appear inverted. In order to use it in your shader you must assign
it to the ``NORMALMAP`` property. Godot will handle converting it for use in 2D and overwriting ``NORMAL``.

.. code-block:: glsl

  NORMALMAP = texture(NORMAL_TEXTURE, UV).rgb;

+----------------------------------+----------------------------------------------------------------+
| Built-in                         | Description                                                    |
+==================================+================================================================+
| in vec4 **FRAGCOORD**            | Fragment coordinate, pixel adjusted.                           |
+----------------------------------+----------------------------------------------------------------+
| inout vec3 **NORMAL**            | Normal read from **NORMAL_TEXTURE**. Writable.                 |
+----------------------------------+----------------------------------------------------------------+
| out vec3 **NORMALMAP**           | Configures normal maps meant for 3D for use in 2D. If used,    |
|                                  | overwrites **NORMAL**.                                         | 
+----------------------------------+----------------------------------------------------------------+
| inout float **NORMALMAP_DEPTH**  | Normalmap depth for scaling.                                   |
+----------------------------------+----------------------------------------------------------------+
| in vec2 **UV**                   | UV from vertex function.                                       |
+----------------------------------+----------------------------------------------------------------+
| inout vec4 **COLOR**             | Color from vertex function and output fragment color. If       |
|                                  | unused, will be set to **TEXTURE** color.                      |
+----------------------------------+----------------------------------------------------------------+
| in sampler2D **TEXTURE**         | Default 2D texture.                                            |
+----------------------------------+----------------------------------------------------------------+
| in sampler2D **NORMAL_TEXTURE**  | Default 2D normal texture.                                     |
+----------------------------------+----------------------------------------------------------------+
| in vec2 **TEXTURE_PIXEL_SIZE**   | Normalized pixel size of default 2D texture.                   |
|                                  | For a Sprite with a texture of size 64x32px,                   |
|                                  | **TEXTURE_PIXEL_SIZE** = :code:`vec2(1/64, 1/32)`              |
+----------------------------------+----------------------------------------------------------------+
| in vec2 **SCREEN_UV**            | Screen UV for use with **SCREEN_TEXTURE**.                     |
+----------------------------------+----------------------------------------------------------------+
| in vec2 **SCREEN_PIXEL_SIZE**    | Size of individual pixels. Equal to inverse of resolution.     |
+----------------------------------+----------------------------------------------------------------+
| in vec2 **POINT_COORD**          | Coordinate for drawing points.                                 |
+----------------------------------+----------------------------------------------------------------+
| in float **TIME**                | Global time in seconds.                                        |
+----------------------------------+----------------------------------------------------------------+
| in bool **AT_LIGHT_PASS**        | True if this is a light pass.                                  |
+----------------------------------+----------------------------------------------------------------+
| in sampler2D **SCREEN_TEXTURE**  | Screen texture, mipmaps contain gaussian blurred versions.     |
+----------------------------------+----------------------------------------------------------------+

Light built-ins
^^^^^^^^^^^^^^^

Light processor functions work differently in 2D than they do in 3D. In CanvasItem shaders, the 
shader is called once for the object being drawn, and then once for each light touching that 
object in the scene. Use render_mode ``unshaded`` if you do not want any light passes to occur
for that object. Use render_mode ``light_only`` if you only want light passes to occur for
that object, this can be useful when you only want the object visible where it is covered by light. 

When the shader is on a light pass the ``AT_LIGHT_PASS`` variable will be ``true``.

+-------------------------------------+-------------------------------------------------------------------------------+
| Built-in                            | Description                                                                   |
+=====================================+===============================================================================+
| in vec4 **FRAGCOORD**               | Fragment coordinate of pixel center. Origin at lower left.                    |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec3 **NORMAL**                  | Input Normal. Although this value is passed in,                               |
|                                     | **normal calculation still happens outside of this function**.                |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec2 **UV**                      | UV from vertex function, equivalent to the UV in the fragment function.       |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec4 **COLOR**                   | Input Color.                                                                  |
|                                     | This is the output of the fragment function with final modulation applied.    |
+-------------------------------------+-------------------------------------------------------------------------------+
| sampler2D **TEXTURE**               | Current texture in use for CanvasItem.                                        |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec2 **TEXTURE_PIXEL_SIZE**      | Normalized pixel size of default 2D texture.                                  |
|                                     | For a Sprite with a texture of size 64x32px,                                  |
|                                     | **TEXTURE_PIXEL_SIZE** = :code:`vec2(1/64, 1/32)`                             |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec2 **SCREEN_UV**               | **SCREEN_TEXTURE** Coordinate (for using with screen texture).                |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec2 **POINT_COORD**             | UV for Point Sprite.                                                          |
+-------------------------------------+-------------------------------------------------------------------------------+
| in float **TIME**                   | Global time in seconds.                                                       |
+-------------------------------------+-------------------------------------------------------------------------------+
| inout vec2 **LIGHT_VEC**            | Vector from light to fragment, can be modified to alter shadow computation.   |
+-------------------------------------+-------------------------------------------------------------------------------+
| inout float **LIGHT_HEIGHT**        | Height of Light. Only effective when normals are used.                        |
+-------------------------------------+-------------------------------------------------------------------------------+
| inout vec4 **LIGHT_COLOR**          | Color of Light.                                                               |
+-------------------------------------+-------------------------------------------------------------------------------+
| in vec2 **LIGHT_UV**                | UV for Light texture.                                                         |
+-------------------------------------+-------------------------------------------------------------------------------+
| out vec4 **SHADOW_COLOR**           | Shadow Color of Light. **(not yet implemented)**                              |
+-------------------------------------+-------------------------------------------------------------------------------+
| inout vec4 **LIGHT**                | Value from the Light texture and output color. Can be modified. If not used,  |
|                                     | the light function is ignored.                                                |
+-------------------------------------+-------------------------------------------------------------------------------+

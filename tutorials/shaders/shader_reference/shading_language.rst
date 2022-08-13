.. _doc_shading_language:

Shading language
================

Introduction
------------

Godot uses a shading language similar to GLSL ES 3.0. Most datatypes and
functions are supported, and the few remaining ones will likely be added over
time.

If you are already familiar with GLSL, the :ref:`Godot Shader Migration
Guide<doc_converting_glsl_to_godot_shaders>` is a resource that will help you
transition from regular GLSL to Godot's shading language.

Data types
----------

Most GLSL ES 3.0 datatypes are supported:

+---------------------+---------------------------------------------------------------------------------+
| Type                | Description                                                                     |
+=====================+=================================================================================+
| **void**            | Void datatype, useful only for functions that return nothing.                   |
+---------------------+---------------------------------------------------------------------------------+
| **bool**            | Boolean datatype, can only contain ``true`` or ``false``.                       |
+---------------------+---------------------------------------------------------------------------------+
| **bvec2**           | Two-component vector of booleans.                                               |
+---------------------+---------------------------------------------------------------------------------+
| **bvec3**           | Three-component vector of booleans.                                             |
+---------------------+---------------------------------------------------------------------------------+
| **bvec4**           | Four-component vector of booleans.                                              |
+---------------------+---------------------------------------------------------------------------------+
| **int**             | Signed scalar integer.                                                          |
+---------------------+---------------------------------------------------------------------------------+
| **ivec2**           | Two-component vector of signed integers.                                        |
+---------------------+---------------------------------------------------------------------------------+
| **ivec3**           | Three-component vector of signed integers.                                      |
+---------------------+---------------------------------------------------------------------------------+
| **ivec4**           | Four-component vector of signed integers.                                       |
+---------------------+---------------------------------------------------------------------------------+
| **uint**            | Unsigned scalar integer; can't contain negative numbers.                        |
+---------------------+---------------------------------------------------------------------------------+
| **uvec2**           | Two-component vector of unsigned integers.                                      |
+---------------------+---------------------------------------------------------------------------------+
| **uvec3**           | Three-component vector of unsigned integers.                                    |
+---------------------+---------------------------------------------------------------------------------+
| **uvec4**           | Four-component vector of unsigned integers.                                     |
+---------------------+---------------------------------------------------------------------------------+
| **float**           | Floating-point scalar.                                                          |
+---------------------+---------------------------------------------------------------------------------+
| **vec2**            | Two-component vector of floating-point values.                                  |
+---------------------+---------------------------------------------------------------------------------+
| **vec3**            | Three-component vector of floating-point values.                                |
+---------------------+---------------------------------------------------------------------------------+
| **vec4**            | Four-component vector of floating-point values.                                 |
+---------------------+---------------------------------------------------------------------------------+
| **mat2**            | 2x2 matrix, in column major order.                                              |
+---------------------+---------------------------------------------------------------------------------+
| **mat3**            | 3x3 matrix, in column major order.                                              |
+---------------------+---------------------------------------------------------------------------------+
| **mat4**            | 4x4 matrix, in column major order.                                              |
+---------------------+---------------------------------------------------------------------------------+
| **sampler2D**       | Sampler type for binding 2D textures, which are read as float.                  |
+---------------------+---------------------------------------------------------------------------------+
| **isampler2D**      | Sampler type for binding 2D textures, which are read as signed integer.         |
+---------------------+---------------------------------------------------------------------------------+
| **usampler2D**      | Sampler type for binding 2D textures, which are read as unsigned integer.       |
+---------------------+---------------------------------------------------------------------------------+
| **sampler2DArray**  | Sampler type for binding 2D texture arrays, which are read as float.            |
+---------------------+---------------------------------------------------------------------------------+
| **isampler2DArray** | Sampler type for binding 2D texture arrays, which are read as signed integer.   |
+---------------------+---------------------------------------------------------------------------------+
| **usampler2DArray** | Sampler type for binding 2D texture arrays, which are read as unsigned integer. |
+---------------------+---------------------------------------------------------------------------------+
| **sampler3D**       | Sampler type for binding 3D textures, which are read as float.                  |
+---------------------+---------------------------------------------------------------------------------+
| **isampler3D**      | Sampler type for binding 3D textures, which are read as signed integer.         |
+---------------------+---------------------------------------------------------------------------------+
| **usampler3D**      | Sampler type for binding 3D textures, which are read as unsigned integer.       |
+---------------------+---------------------------------------------------------------------------------+
| **samplerCube**     | Sampler type for binding Cubemaps, which are read as floats.                    |
+---------------------+---------------------------------------------------------------------------------+

Casting
~~~~~~~

Just like GLSL ES 3.0, implicit casting between scalars and vectors of the same
size but different type is not allowed. Casting of types of different size is
also not allowed. Conversion must be done explicitly via constructors.

Example:

.. code-block:: glsl

    float a = 2; // invalid
    float a = 2.0; // valid
    float a = float(2); // valid

Default integer constants are signed, so casting is always needed to convert to
unsigned:

.. code-block:: glsl

    int a = 2; // valid
    uint a = 2; // invalid
    uint a = uint(2); // valid

Members
~~~~~~~

Individual scalar members of vector types are accessed via the "x", "y", "z" and
"w" members. Alternatively, using "r", "g", "b" and "a" also works and is
equivalent. Use whatever fits best for your needs.

For matrices, use the ``m[column][row]`` indexing syntax to access each scalar,
or ``m[idx]`` to access a vector by row index. For example, for accessing the y
position of an object in a mat4 you use ``m[3][1]``.

Constructing
~~~~~~~~~~~~

Construction of vector types must always pass:

.. code-block:: glsl

    // The required amount of scalars
    vec4 a = vec4(0.0, 1.0, 2.0, 3.0);
    // Complementary vectors and/or scalars
    vec4 a = vec4(vec2(0.0, 1.0), vec2(2.0, 3.0));
    vec4 a = vec4(vec3(0.0, 1.0, 2.0), 3.0);
    // A single scalar for the whole vector
    vec4 a = vec4(0.0);

Construction of matrix types requires vectors of the same dimension as the
matrix. You can also build a diagonal matrix using ``matx(float)`` syntax.
Accordingly, ``mat4(1.0)`` is an identity matrix.

.. code-block:: glsl

    mat2 m2 = mat2(vec2(1.0, 0.0), vec2(0.0, 1.0));
    mat3 m3 = mat3(vec3(1.0, 0.0, 0.0), vec3(0.0, 1.0, 0.0), vec3(0.0, 0.0, 1.0));
    mat4 identity = mat4(1.0);

Matrices can also be built from a matrix of another dimension. There are two
rules:

1. If a larger matrix is constructed from a smaller matrix, the additional rows
and columns are set to the values they would have in an identity matrix.
2. If a smaller matrix is constructed from a larger matrix, the top, left
submatrix of the larger matrix is used.

.. code-block:: glsl

	mat3 basis = mat3(MODEL_MATRIX);
	mat4 m4 = mat4(basis);
	mat2 m2 = mat2(m4);

Swizzling
~~~~~~~~~

It is possible to obtain any combination of components in any order, as long as
the result is another vector type (or scalar). This is easier shown than
explained:

.. code-block:: glsl

    vec4 a = vec4(0.0, 1.0, 2.0, 3.0);
    vec3 b = a.rgb; // Creates a vec3 with vec4 components.
    vec3 b = a.ggg; // Also valid; creates a vec3 and fills it with a single vec4 component.
    vec3 b = a.bgr; // "b" will be vec3(2.0, 1.0, 0.0).
    vec3 b = a.xyz; // Also rgba, xyzw are equivalent.
    vec3 b = a.stp; // And stpq (for texture coordinates).
    float c = b.w; // Invalid, because "w" is not present in vec3 b.
    vec3 c = b.xrt; // Invalid, mixing different styles is forbidden.
    b.rrr = a.rgb; // Invalid, assignment with duplication.
    b.bgr = a.rgb; // Valid assignment. "b"'s "blue" component will be "a"'s "red" and vice versa.

Precision
~~~~~~~~~

It is possible to add precision modifiers to datatypes; use them for uniforms,
variables, arguments and varyings:

.. code-block:: glsl

    lowp vec4 a = vec4(0.0, 1.0, 2.0, 3.0); // low precision, usually 8 bits per component mapped to 0-1
    mediump vec4 a = vec4(0.0, 1.0, 2.0, 3.0); // medium precision, usually 16 bits or half float
    highp vec4 a = vec4(0.0, 1.0, 2.0, 3.0); // high precision, uses full float or integer range (default)


Using lower precision for some operations can speed up the math involved (at the
cost of less precision). This is rarely needed in the vertex processor function
(where full precision is needed most of the time), but is often useful in the
fragment processor.

Some architectures (mainly mobile) can benefit significantly from this, but
there are downsides such as the additional overhead of conversion between
precisions. Refer to the documentation of the target architecture for further
information. In many cases, mobile drivers cause inconsistent or unexpected
behavior and it is best to avoid specifying precision unless necessary.

Arrays
------

Arrays are containers for multiple variables of a similar type.

Local arrays
~~~~~~~~~~~~

Local arrays are declared in functions. They can use all of the allowed
datatypes, except samplers. The array declaration follows a C-style syntax:
``[const] + [precision] + typename + identifier + [array size]``.

.. code-block:: glsl

    void fragment() {
        float arr[3];
    }

They can be initialized at the beginning like:

.. code-block:: glsl

    float float_arr[3] = float[3] (1.0, 0.5, 0.0); // first constructor

    int int_arr[3] = int[] (2, 1, 0); // second constructor

    vec2 vec2_arr[3] = { vec2(1.0, 1.0), vec2(0.5, 0.5), vec2(0.0, 0.0) }; // third constructor

    bool bool_arr[] = { true, true, false }; // fourth constructor - size is defined automatically from the element count

You can declare multiple arrays (even with different sizes) in one expression:

.. code-block:: glsl

    float a[3] = float[3] (1.0, 0.5, 0.0),
    b[2] = { 1.0, 0.5 },
    c[] = { 0.7 },
    d = 0.0,
    e[5];

To access an array element, use the indexing syntax:

.. code-block:: glsl

    float arr[3];

    arr[0] = 1.0; // setter

    COLOR.r = arr[0]; // getter

Arrays also have a built-in function ``.length()`` (not to be confused with the
built-in ``length()`` function). It doesn't accept any parameters and will
return the array's size.

.. code-block:: glsl

    float arr[] = { 0.0, 1.0, 0.5, -1.0 };
    for (int i = 0; i < arr.length(); i++) {
        // ...
    }

.. note::

    If you use an index either below 0 or greater than array size - the shader will
    crash and break rendering. To prevent this, use ``length()``, ``if``, or
    ``clamp()`` functions to ensure the index is between 0 and the array's
    length. Always carefully test and check your code. If you pass a constant
    expression or a number, the editor will check its bounds to prevent
    this crash.

Global arrays
~~~~~~~~~~~~~

You can declare arrays at global space like:

.. code-block:: glsl

    shader_type spatial;

    const lowp vec3 v[1] = lowp vec3[1] ( vec3(0, 0, 1) );

    void fragment() {
      ALBEDO = v[0];
    }

.. note::

    Global arrays have to be declared as global constants, otherwise they can be
    declared the same as local arrays.

Constants
---------

Use the ``const`` keyword before the variable declaration to make that variable
immutable, which means that it cannot be modified. All basic types, except
samplers can be declared as constants. Accessing and using a constant value is
slightly faster than using a uniform. Constants must be initialized at their
declaration.

.. code-block:: glsl

    const vec2 a = vec2(0.0, 1.0);
    vec2 b;

    a = b; // invalid
    b = a; // valid

Constants cannot be modified and additionally cannot have hints, but multiple of
them (if they have the same type) can be declared in a single expression e.g

.. code-block:: glsl

    const vec2 V1 = vec2(1, 1), V2 = vec2(2, 2);

Similar to variables, arrays can also be declared with ``const``.

.. code-block:: glsl

    const float arr[] = { 1.0, 0.5, 0.0 };

    arr[0] = 1.0; // invalid

    COLOR.r = arr[0]; // valid

Constants can be declared both globally (outside of any function) or locally
(inside a function). Global constants are useful when you want to have access to
a value throughout your shader that does not need to be modified. Like uniforms,
global constants are shared between all shader stages, but they are not
accessible outside of the shader.

.. code-block:: glsl

    shader_type spatial;

    const float PI = 3.14159265358979323846;

Constants of the ``float`` type must be initialized using ``.`` notation after the
decimal part or by using the scientific notation. The optional ``f`` post-suffix is
also supported.

.. code-block:: glsl

    float a = 1.0;
    float b = 1.0f; // same, using suffix for clarity
    float c = 1e-1; // gives 0.1 by using the scientific notation

Constants of the ``uint`` (unsigned int) type must have a ``u`` suffix to differentiate them from signed integers.
Alternatively, this can be done by using the ``uint(x)`` built-in conversion function.

.. code-block:: glsl

    uint a = 1u;
    uint b = uint(1);

Structs
-------

Structs are compound types which can be used for better abstraction of shader
code. You can declare them at the global scope like:

.. code-block:: glsl

    struct PointLight {
        vec3 position;
        vec3 color;
        float intensity;
    };

After declaration, you can instantiate and initialize them like:

.. code-block:: glsl

    void fragment()
    {
        PointLight light;
        light.position = vec3(0.0);
        light.color = vec3(1.0, 0.0, 0.0);
        light.intensity = 0.5;
    }

Or use struct constructor for same purpose:

.. code-block:: glsl

    PointLight light = PointLight(vec3(0.0), vec3(1.0, 0.0, 0.0), 0.5);

Structs may contain other struct or array, you can also instance them as global
constant:

.. code-block:: glsl

    shader_type spatial;

    ...

    struct Scene {
        PointLight lights[2];
    };

    const Scene scene = Scene(PointLight[2](PointLight(vec3(0.0, 0.0, 0.0), vec3(1.0, 0.0, 0.0), 1.0), PointLight(vec3(0.0, 0.0, 0.0), vec3(1.0, 0.0, 0.0), 1.0)));

    void fragment()
    {
        ALBEDO = scene.lights[0].color;
    }

You can also pass them to functions:

.. code-block:: glsl

    shader_type canvas_item;

    ...

    Scene construct_scene(PointLight light1, PointLight light2) {
        return Scene({light1, light2});
    }

    void fragment()
    {
        COLOR.rgb = construct_scene(PointLight(vec3(0.0, 0.0, 0.0), vec3(1.0, 0.0, 0.0), 1.0), PointLight(vec3(0.0, 0.0, 0.0), vec3(1.0, 0.0, 1.0), 1.0)).lights[0].color;
    }

Operators
---------

Godot shading language supports the same set of operators as GLSL ES 3.0. Below
is the list of them in precedence order:

+-------------+------------------------+------------------+
| Precedence  | Class                  | Operator         |
+-------------+------------------------+------------------+
| 1 (highest) | parenthetical grouping | **()**           |
+-------------+------------------------+------------------+
| 2           | unary                  | **+, -, !, ~**   |
+-------------+------------------------+------------------+
| 3           | multiplicative         | **/, \*, %**     |
+-------------+------------------------+------------------+
| 4           | additive               | **+, -**         |
+-------------+------------------------+------------------+
| 5           | bit-wise shift         | **<<, >>**       |
+-------------+------------------------+------------------+
| 6           | relational             | **<, >, <=, >=** |
+-------------+------------------------+------------------+
| 7           | equality               | **==, !=**       |
+-------------+------------------------+------------------+
| 8           | bit-wise AND           | **&**            |
+-------------+------------------------+------------------+
| 9           | bit-wise exclusive OR  | **^**            |
+-------------+------------------------+------------------+
| 10          | bit-wise inclusive OR  | **|**            |
+-------------+------------------------+------------------+
| 11          | logical AND            | **&&**           |
+-------------+------------------------+------------------+
| 12 (lowest) | logical inclusive OR   | **||**           |
+-------------+------------------------+------------------+

Flow control
------------

Godot Shading language supports the most common types of flow control:

.. code-block:: glsl

    // `if` and `else`.
    if (cond) {

    } else {

    }

    // Ternary operator.
    // This is an expression that behaves like `if`/`else` and returns the value.
    // If `cond` evaluates to `true`, `result` will be `9`.
    // Otherwise, `result` will be `5`.
    int result = cond ? 9 : 5;

    // `switch`.
    switch (i) { // `i` should be a signed integer expression.
        case -1:
            break;
        case 0:
            return; // `break` or `return` to avoid running the next `case`.
        case 1: // Fallthrough (no `break` or `return`): will run the next `case`.
        case 2:
            break;
        //...
        default: // Only run if no `case` above matches. Optional.
            break;
    }

    // `for` loop. Best used when the number of elements to iterate on
    // is known in advance.
    for (int i = 0; i < 10; i++) {

    }

    // `while` loop. Best used when the number of elements to iterate on
    // is not known in advance.
    while (cond) {

    }

    // `do while`. Like `while`, but always runs at least once even if `cond`
    // never evaluates to `true`.
    do {

    } while (cond);

Keep in mind that, in modern GPUs, an infinite loop can exist and can freeze
your application (including editor). Godot can't protect you from this, so be
careful not to make this mistake!

Also, when comparing floating-point values against a number, make sure to
compare them against a *range* instead of an exact number.

A comparison like ``if (value == 0.3)`` may not evaluate to ``true``.
Floating-point math is often approximate and can defy expectations. It can also
behave differently depending on the hardware.

**Don't** do this.

.. code-block:: glsl

    float value = 0.1 + 0.2;

    // May not evaluate to `true`!
    if (value == 0.3) {
        // ...
    }

Instead, always perform a range comparison with an epsilon value. The larger the
floating-point number (and the less precise the floating-point number, the
larger the epsilon value should be.

.. code-block:: glsl

    const float EPSILON = 0.0001;
    if (value >= 0.3 - EPSILON && value <= 0.3 + EPSILON) {
        // ...
    }

See `floating-point-gui.de <https://floating-point-gui.de/>`__ for more
information.

.. warning::

    When exporting a GLES2 project to HTML5, WebGL 1.0 will be used. WebGL 1.0
    doesn't support dynamic loops, so shaders using those won't work there.

Discarding
----------

Fragment and light functions can use the **discard** keyword. If used, the
fragment is discarded and nothing is written.

Functions
---------

It is possible to define functions in a Godot shader. They use the following
syntax:

.. code-block:: glsl

    ret_type func_name(args) {
        return ret_type; // if returning a value
    }

    // a more specific example:

    int sum2(int a, int b) {
        return a + b;
    }


You can only use functions that have been defined above (higher in the editor)
the function from which you are calling them.

Function arguments can have special qualifiers:

* **in**: Means the argument is only for reading (default).
* **out**: Means the argument is only for writing.
* **inout**: Means the argument is fully passed via reference.
* **const**: Means the argument is a constant and cannot be changed, may be
  combined with **in** qualifier.

Example below:

.. code-block:: glsl

    void sum2(int a, int b, inout int result) {
        result = a + b;
    }

Varyings
~~~~~~~~

To send data from the vertex to the fragment (or light) processor function, *varyings* are
used. They are set for every primitive vertex in the *vertex processor*, and the
value is interpolated for every pixel in the *fragment processor*.

.. code-block:: glsl

    shader_type spatial;

    varying vec3 some_color;

    void vertex() {
        some_color = NORMAL; // Make the normal the color.
    }

    void fragment() {
        ALBEDO = some_color;
    }

    void light() {
        DIFFUSE_LIGHT = some_color * 100; // optionally
    }

Varying can also be an array:

.. code-block:: glsl

    shader_type spatial;

    varying float var_arr[3];

    void vertex() {
        var_arr[0] = 1.0;
        var_arr[1] = 0.0;
    }

    void fragment() {
        ALBEDO = vec3(var_arr[0], var_arr[1], var_arr[2]); // red color
    }

It's also possible to send data from *fragment* to *light* processors using *varying* keyword. To do so you can assign it in the *fragment* and later use it in the *light* function.

.. code-block:: glsl

    shader_type spatial;

    varying vec3 some_light;

    void fragment() {
        some_light = ALBEDO * 100.0; // Make a shining light.
    }

    void light() {
        DIFFUSE_LIGHT = some_light;
    }

Note that varying may not be assigned in custom functions or a *light processor* function like:

.. code-block:: glsl

    shader_type spatial;

    varying float test;

    void foo() {
        test = 0.0; // Error.
    }

    void vertex() {
        test = 0.0;
    }

    void light() {
        test = 0.0; // Error too.
    }

This limitation was introduced to prevent incorrect usage before initialization.

Interpolation qualifiers
~~~~~~~~~~~~~~~~~~~~~~~~

Certain values are interpolated during the shading pipeline. You can modify how
these interpolations are done by using *interpolation qualifiers*.

.. code-block:: glsl

    shader_type spatial;

    varying flat vec3 our_color;

    void vertex() {
        our_color = COLOR.rgb;
    }

    void fragment() {
        ALBEDO = our_color;
    }

There are two possible interpolation qualifiers:

+-------------------+---------------------------------------------------------------------------------+
| Qualifier         | Description                                                                     |
+===================+=================================================================================+
| **flat**          | The value is not interpolated.                                                  |
+-------------------+---------------------------------------------------------------------------------+
| **smooth**        | The value is interpolated in a perspective-correct fashion. This is the default.|
+-------------------+---------------------------------------------------------------------------------+


Uniforms
~~~~~~~~

Passing values to shaders is possible. These are global to the whole shader and
are called *uniforms*. When a shader is later assigned to a material, the
uniforms will appear as editable parameters in it. Uniforms can't be written
from within the shader.

.. note::
    Uniform arrays are not implemented yet.

.. code-block:: glsl

    shader_type spatial;

    uniform float some_value;

You can set uniforms in the editor in the material. Or you can set them through
GDScript:

::

  material.set_shader_param("some_value", some_value)

.. note:: The first argument to ``set_shader_param`` is the name of the uniform
          in the shader. It must match *exactly* to the name of the uniform in
          the shader or else it will not be recognized.

Any GLSL type except for *void* can be a uniform. Additionally, Godot provides
optional shader hints to make the compiler understand for what the uniform is
used, and how the editor should allow users to modify it.

.. code-block:: glsl

    shader_type spatial;

    uniform vec4 color : source_color;
    uniform float amount : hint_range(0, 1);
    uniform vec4 other_color : source_color = vec4(1.0);

It's important to understand that textures that are supplied as color require
hints for proper sRGB->linear conversion (i.e. ``hint_albedo``), as Godot's 3D
engine renders in linear color space.

Full list of hints below:

+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| Type                 | Hint                                           | Description                                                                 |
+======================+================================================+=============================================================================+
| **vec3, vec4**       | source_color                                   | Used as color.                                                              |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **int, float**       | hint_range(min, max[, step])                   | Restricted to values in a range (with min/max/step).                        |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | source_color                                   | Used as albedo color.                                                       |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | hint_normal                                    | Used as normalmap.                                                          |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | hint_default_white                             | As value or albedo color, default to opaque white.                          |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | hint_default_black                             | As value or albedo color, default to opaque black.                          |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | hint_default_transparent                       | As value or albedo color, default to transparent black.                     |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | hint_anisotropy                                | As flowmap, default to right.                                               |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | hint_roughness[_r, _g, _b, _a, _normal, _gray] | Used for roughness limiter on import (attempts reducing specular aliasing). |
|                      |                                                | ``_normal`` is a normal map that guides the roughness limiter,              |
|                      |                                                | with roughness increasing in areas that have high-frequency detail.         |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | filter[_nearest, _linear][_mipmap][_aniso]     | Enabled specified texture filtering.                                        |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+
| **sampler2D**        | repeat[_enable, _disable]                      | Enabled texture repeating.                                                  |
+----------------------+------------------------------------------------+-----------------------------------------------------------------------------+

GDScript uses different variable types than GLSL does, so when passing variables
from GDScript to shaders, Godot converts the type automatically. Below is a
table of the corresponding types:

+-----------------+-----------+
| GDScript type   | GLSL type |
+=================+===========+
| **bool**        | **bool**  |
+-----------------+-----------+
| **int**         | **int**   |
+-----------------+-----------+
| **float**       | **float** |
+-----------------+-----------+
| **Vector2**     | **vec2**  |
+-----------------+-----------+
| **Vector3**     | **vec3**  |
+-----------------+-----------+
| **Color**       | **vec4**  |
+-----------------+-----------+
| **Transform3D** | **mat4**  |
+-----------------+-----------+
| **Transform2D** | **mat4**  |
+-----------------+-----------+

.. note:: Be careful when setting shader uniforms from GDScript, no error will
          be thrown if the type does not match. Your shader will just exhibit
          undefined behavior.

Uniforms can also be assigned default values:

.. code-block:: glsl

    shader_type spatial;

    uniform vec4 some_vector = vec4(0.0);
    uniform vec4 some_color : source_color = vec4(1.0);

Built-in variables
------------------

A large number of built-in variables are available, like ``UV``, ``COLOR`` and
``VERTEX``. What variables are available depends on the type of shader
(``spatial``, ``canvas_item``, ``particle``, ``sky``, or ``fog``) and the
function used (``vertex``, ``fragment``, ``light``, ``start``, ``process,
``sky``, or ``fog``). For a list of the built-in variables that are available,
please see the corresponding pages:

- :ref:`Spatial shaders <doc_spatial_shader>`
- :ref:`Canvas item shaders <doc_canvas_item_shader>`
- :ref:`Particle shaders <doc_particle_shader>`
- :ref:`Sky shaders <doc_sky_shader>`
- :ref:`Fog shaders <doc_fog_shader>`

Built-in functions
------------------

Godot supports a large number of built-in functions, conforming roughly to the
GLSL 4.00 specification.

When vec_type, ivec_type, uvec_type, or bvec_type nomenclature is used, the type
can be scalar or vector. For example vec_type can be float, vec2, vec3, or vec4.

.. note:: For a list of the functions that are not available in the OpenGL
          backend, please see the :ref:`Differences between GLES2 and GLES3 doc
          <doc_gles2_gles3_differences>`.

+-------------+---------------------------------------------------------------------------------------------------------+
| Return Type |                                                Function                                                 |
+=============+=========================================================================================================+
| vec_type    | :js:attr:`radians <radians>` (vec_type degrees)                                                         |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`degrees <degrees>` (vec_type radians)                                                         |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`sin <sin>` (vec_type x)                                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`cos <cos>` (vec_type x)                                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`tan <tan>` (vec_type x)                                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`asin <asin>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`acos <acos>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`atan <atan>` (vec_type y_over_x)                                                              |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`atan <atan>` (vec_type y, vec_type x)                                                         |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`sinh <sinh>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`cosh <cosh>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`tanh <tanh>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`asinh <asinh>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`acosh <acosh>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`atanh <atanh>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`pow <pow>` (vec_type x, vec_type y)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`exp <exp>` (vec_type x)                                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`exp2 <exp2>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`log <log>` (vec_type x)                                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`log2 <log2>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`sqrt <sqrt>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`inversesqrt <inversesqrt>` (vec_type x)                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`abs <abs>` (vec_type x)                                                                       |
|             |                                                                                                         |
| ivec_type   | :js:attr:`abs <abs>` (ivec_type x)                                                                      |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`sign <sign>` (vec_type x)                                                                     |
|             |                                                                                                         |
| ivec_type   | :js:attr:`sign <sign>` (ivec_type x)                                                                    |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`floor <floor>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`round <round>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`roundEven <roundEven>` (vec_type x)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`trunc <trunc>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`ceil <ceil>` (vec_type x)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`fract <fract>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`mod <mod>` (vec_type x, vec_type y)                                                           |
|             |                                                                                                         |
| vec_type    | :js:attr:`mod <mod>` (vec_type x, float y)                                                              |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`modf <modf>` (vec_type x, out vec_type i)                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`min <min>` (vec_type a, vec_type b)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`max <max>` (vec_type a, vec_type b)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`clamp <clamp>` (vec_type x, vec_type min, vec_type max)                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| float       | :js:attr:`mix <mix>` (float a, float b, float c)                                                        |
|             |                                                                                                         |
| vec_type    | :js:attr:`mix <mix>` (vec_type a, vec_type b, float c)                                                  |
|             |                                                                                                         |
| vec_type    | :js:attr:`mix <mix>` (vec_type a, vec_type b, bvec_type c)                                              |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`fma <fma>` (vec_type a, vec_type b, vec_type c)                                               |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`step <step>` (vec_type a, vec_type b)                                                         |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`step <step>` (float a, vec_type b)                                                            |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`smoothstep <smoothstep>` (vec_type a, vec_type b, vec_type c)                                 |
|             |                                                                                                         |
| vec_type    | :js:attr:`smoothstep <smoothstep>` (float a, float b, vec_type c)                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`isnan <isnan>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`isinf <isinf>` (vec_type x)                                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`floatBitsToInt <floatBitsToInt>` (vec_type x)                                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| uvec_type   | :js:attr:`floatBitsToUint <floatBitsToUint>` (vec_type x)                                               |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`intBitsToFloat <intBitsToFloat>` (ivec_type x)                                                |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`uintBitsToFloat <uintBitsToFloat>` (uvec_type x)                                              |
+-------------+---------------------------------------------------------------------------------------------------------+
| float       | :js:attr:`length <length>` (vec_type x)                                                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| float       | :js:attr:`distance <distance>` (vec_type a, vec_type b)                                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| float       | :js:attr:`dot <dot>` (vec_type a, vec_type b)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec3        | :js:attr:`cross <cross>` (vec3 a, vec3 b)                                                               |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`normalize <normalize>` (vec_type x)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec3        | :js:attr:`reflect <reflect>` (vec3 I, vec3 N)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec3        | :js:attr:`refract <refract>` (vec3 I, vec3 N, float eta)                                                |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`faceforward <faceforward>` (vec_type N, vec_type I, vec_type Nref)                            |
+-------------+---------------------------------------------------------------------------------------------------------+
| mat_type    | :js:attr:`matrixCompMult <matrixCompMult>` (mat_type x, mat_type y)                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| mat_type    | :js:attr:`outerProduct <outerProduct>` (vec_type column, vec_type row)                                  |
+-------------+---------------------------------------------------------------------------------------------------------+
| mat_type    | :js:attr:`transpose <transpose>` (mat_type m)                                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| float       | :js:attr:`determinant <determinant>` (mat_type m)                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| mat_type    | :js:attr:`inverse <inverse>` (mat_type m)                                                               |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`lessThan <lessThan>` (vec_type x, vec_type y)                                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`greaterThan <greaterThan>` (vec_type x, vec_type y)                                           |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`lessThanEqual <lessThanEqual>` (vec_type x, vec_type y)                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`greaterThanEqual <greaterThanEqual>` (vec_type x, vec_type y)                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`equal <equal>` (vec_type x, vec_type y)                                                       |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`notEqual <notEqual>` (vec_type x, vec_type y)                                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| bool        | :js:attr:`any <any>` (bvec_type x)                                                                      |
+-------------+---------------------------------------------------------------------------------------------------------+
| bool        | :js:attr:`all <all>` (bvec_type x)                                                                      |
+-------------+---------------------------------------------------------------------------------------------------------+
| bvec_type   | :js:attr:`not <not>` (bvec_type x)                                                                      |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec2       | :js:attr:`textureSize <textureSize>` (gsampler2D s, int lod)                                            |
|             |                                                                                                         |
| ivec3       | :js:attr:`textureSize <textureSize>` (gsampler2DArray s, int lod)                                       |
|             |                                                                                                         |
| ivec3       | :js:attr:`textureSize <textureSize>` (gsampler3D s, int lod)                                            |
|             |                                                                                                         |
| ivec2       | :js:attr:`textureSize <textureSize>` (samplerCube s, int lod)                                           |
|             |                                                                                                         |
| ivec2       | :js:attr:`textureSize <textureSize>` (samplerCubeArray s, int lod)                                      |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`texture <texture>` (gsampler2D s, vec2 p [, float bias])                                      |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`texture <texture>` (gsampler2DArray s, vec3 p [, float bias])                                 |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`texture <texture>` (gsampler3D s, vec3 p [, float bias])                                      |
|             |                                                                                                         |
| vec4        | :js:attr:`texture <texture>` (samplerCube s, vec3 p [, float bias])                                     |
|             |                                                                                                         |
| vec4        | :js:attr:`texture <texture>` (samplerCubeArray s, vec4 p [, float bias])                                |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`textureProj <textureProj>` (gsampler2D s, vec3 p [, float bias])                              |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureProj <textureProj>` (gsampler2D s, vec4 p [, float bias])                              |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureProj <textureProj>` (gsampler3D s, vec4 p [, float bias])                              |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`textureLod <textureLod>` (gsampler2D s, vec2 p, float lod)                                    |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureLod <textureLod>` (gsampler2DArray s, vec3 p, float lod)                               |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureLod <textureLod>` (gsampler3D s, vec3 p, float lod)                                    |
|             |                                                                                                         |
| vec4        | :js:attr:`textureLod <textureLod>` (samplerCube s, vec3 p, float lod)                                   |
|             |                                                                                                         |
| vec4        | :js:attr:`textureLod <textureLod>` (samplerCubeArray s, vec4 p, float lod)                              |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`textureProjLod <textureProjLod>` (gsampler2D s, vec3 p, float lod)                            |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureProjLod <textureProjLod>` (gsampler2D s, vec4 p, float lod)                            |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureProjLod <textureProjLod>` (gsampler3D s, vec4 p, float lod)                            |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`textureGrad <textureGrad>` (gsampler2D s, vec2 p, vec2 dPdx, vec2 dPdy)                       |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureGrad <textureGrad>` (gsampler2DArray s, vec3 p, vec2 dPdx, vec2 dPdy)                  |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureGrad <textureGrad>` (gsampler3D s, vec3 p, vec2 dPdx, vec2 dPdy)                       |
|             |                                                                                                         |
| vec4        | :js:attr:`textureGrad <textureGrad>` (samplerCube s, vec3 p, vec3 dPdx, vec3 dPdy)                      |
|             |                                                                                                         |
| vec4        | :js:attr:`textureGrad <textureGrad>` (samplerCubeArray s, vec3 p, vec3 dPdx, vec3 dPdy)                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`texelFetch <texelFetch>` (gsampler2D s, ivec2 p, int lod)                                     |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`texelFetch <texelFetch>` (gsampler2DArray s, ivec3 p, int lod)                                |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`texelFetch <texelFetch>` (gsampler3D s, ivec3 p, int lod)                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| gvec4_type  | :js:attr:`textureGather <textureGather>` (gsampler2D s, vec2 p [, int comps])                           |
|             |                                                                                                         |
| gvec4_type  | :js:attr:`textureGather <textureGather>` (gsampler2DArray s, vec3 p [, int comps])                      |
|             |                                                                                                         |
| vec4        | :js:attr:`textureGather <textureGather>` (samplerCube s, vec3 p [, int comps])                          |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`dFdx <dFdx>` (vec_type p)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`dFdy <dFdy>` (vec_type p)                                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`fwidth <fwidth>` (vec_type p)                                                                 |
+-------------+---------------------------------------------------------------------------------------------------------+
| uint        | :js:attr:`packHalf2x16 <packHalf2x16>` (vec2 v)                                                         |
|             |                                                                                                         |
| vec2        | :js:attr:`unpackHalf2x16 <unpackHalf2x16>` (uint v)                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| uint        | :js:attr:`packUnorm2x16 <packUnorm2x16>` (vec2 v)                                                       |
|             |                                                                                                         |
| vec2        | :js:attr:`unpackUnorm2x16 <unpackUnorm2x16>` (uint v)                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| uint        | :js:attr:`packSnorm2x16 <packSnorm2x16>` (vec2 v)                                                       |
|             |                                                                                                         |
| vec2        | :js:attr:`unpackSnorm2x16 <unpackSnorm2x16>` (uint v)                                                   |
+-------------+---------------------------------------------------------------------------------------------------------+
| uint        | :js:attr:`packUnorm4x8 <packUnorm4x8>` (vec4 v)                                                         |
|             |                                                                                                         |
| vec4        | :js:attr:`unpackUnorm4x8 <unpackUnorm4x8>` (uint v)                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| uint        | :js:attr:`packSnorm4x8 <packSnorm4x8>` (vec4 v)                                                         |
|             |                                                                                                         |
| vec4        | :js:attr:`unpackSnorm4x8 <unpackSnorm4x8>` (uint v)                                                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`bitfieldExtract <bitfieldExtract>` (ivec_type value, int offset, int bits)                    |
|             |                                                                                                         |
| uvec_type   | :js:attr:`bitfieldExtract <bitfieldExtract>` (uvec_type value, int offset, int bits)                    |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`bitfieldInsert <bitfieldInsert>` (ivec_type base, ivec_type insert, int offset, int bits)     |
|             |                                                                                                         |
| uvec_type   | :js:attr:`bitfieldInsert <bitfieldInsert>` (uvec_type base, uvec_type insert, int offset, int bits)     |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`bitfieldReverse <bitfieldReverse>` (ivec_type value)                                          |
|             |                                                                                                         |
| uvec_type   | :js:attr:`bitfieldReverse <bitfieldReverse>` (uvec_type value)                                          |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`bitCount <bitCount>` (ivec_type value)                                                        |
|             |                                                                                                         |
| uvec_type   | :js:attr:`bitCount <bitCount>` (uvec_type value)                                                        |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`findLSB <findLSB>` (ivec_type value)                                                          |
|             |                                                                                                         |
| uvec_type   | :js:attr:`findLSB <findLSB>` (uvec_type value)                                                          |
+-------------+---------------------------------------------------------------------------------------------------------+
| ivec_type   | :js:attr:`findMSB <findMSB>` (ivec_type value)                                                          |
|             |                                                                                                         |
| uvec_type   | :js:attr:`findMSB <findMSB>` (uvec_type value)                                                          |
+-------------+---------------------------------------------------------------------------------------------------------+
| void        | :js:attr:`imulExtended <imulExtended>` (ivec_type x, ivec_type y, out ivec_type msb, out ivec_type lsb) |
|             |                                                                                                         |
| void        | :js:attr:`umulExtended <umulExtended>` (uvec_type x, uvec_type y, out uvec_type msb, out uvec_type lsb) |
+-------------+---------------------------------------------------------------------------------------------------------+
| uvec_type   | :js:attr:`uaddCarry <uaddCarry>` (uvec_type x, uvec_type y, out uvec_type carry)                        |
+-------------+---------------------------------------------------------------------------------------------------------+
| uvec_type   | :js:attr:`usubBorrow <usubBorrow>` (uvec_type x, uvec_type y, out uvec_type borrow)                     |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`ldexp <ldexp>` (vec_type x, out ivec_type exp)                                                |
+-------------+---------------------------------------------------------------------------------------------------------+
| vec_type    | :js:attr:`frexp <frexp>` (vec_type x, out ivec_type exp)                                                |
+-------------+---------------------------------------------------------------------------------------------------------+

.. js:function:: radians( vec_type degrees )

    ``radians`` converts a quantity specified in degrees into radians.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/radians.xhtml>`_

    :param vec_type degrees:
        Specify the quantity, in degrees, to be converted to radians.

    :return:
        The return value is ``(π * degrees) / 180``.

    :rtype: vec_type

.. js:function:: degrees (vec_type radians)    

    ``degrees`` converts a quantity specified in radians into degrees.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/degrees.xhtml>`_.

    :param vec_type radians:
        Specify the quantity, in radians, to be converted to degrees.

    :return:
        The return value is ``(radians * 180) / π``.

    :rtype: vec_type

.. js:function:: sin (vec_type angle)

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/sin.xhtml>`_.

    :param vec_type angle:
        The quantity, in radians, of which to return the sine

    :return:
        The return value is the trigonometric sine of ``angle``.

    :rtype: vec_type

.. js:function:: cos (vec_type angle)

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/cos.xhtml>`_.

    :param vec_type angle:
        The quantity, in radians, of which to return the cosine.

    :return:
        The return value is the trigonometric cosine of ``angle``.

    :rtype: vec_type

.. js:function:: tan (vec_type angle)

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/tan.xhtml>`_.

    :param vec_type angle:
        The quantity, in radians, of which to return the tangent.

    :return:
        The return value is the trigonometric tangent of ``angle``.

    :rtype: vec_type

.. js:function:: asin (vec_type x)

    Calculates the angle whose sine is ``x``.
    The result is undefined if ``x < -1`` or ``x > 1``.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/asin.xhtml>`_.

    :param vec_type x:
        The value whose arccosine to return.
    :return:
        The return value is the angle whose trigonometric sine is ``x`` and is 
        in the range ``[-π/2, π/2]``.

    :rtype: vec_type

.. js:function:: acos (vec_type x)

    Calculates the angle whose cosine is ``x``.
    The result is undefined if ``x < -1`` or ``x > 1``.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/acos.xhtml>`_.

    :param vec_type x:
        The value whose arccosine to return.

    :return:
        The return value is the angle whose trigonometric cosine is ``x`` and
        is in the range ``[0, π]``.

    :rtype: vec_type

.. js:function:: atan (vec_type y_over_x)

    Calculate the arctangent given a tangent value of ``y/x``. Note: becuase of
    the sign ambiguity, the function cannot determine with certainty in which
    quadrant the angle falls only by its tangent value. If you need to know the
    quadrant, use the other overload of ``atan``.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/atan.xhtml>`_.

    :param vec_type y_over_x:
        The fraction whose arctangent to return.

    :return:
        The return value is the trigonometric arc-tangent of ``y_over_x`` and is
        in the range ``[−π/2, π/2]``.

    :rtype: vec_type

.. js:function:: atan (vec_type y, vec_type x)

    Calculate the arctangent given a numerator and denominator. The signs of
    ``y`` and ``x`` are used to determine the quadrant that the angle lies in.
    The result is undefined if ``x == 0``.
    
    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/atan.xhtml>`_.

    :param vec_type y:
        The numerator of the fraction whose arctangent to return.

    :param vec_type x:
        The denominator of the fraction whose arctangent to return.

    :return:
        The return value is the trigonometric arc-tangent of ``y/x`` and is in
        the range ``[−π, π]``.

    :rtype: vec_type

.. js:function:: sinh (vec_type x)

    Calculates the hyperbolic sine using ``(e^x - e^-x)/2``.
    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/sinh.xhtml>`_.

    :param vec_type x:
        The value whose hyperbolic sine to return.

    :return:
        The return value is the hyperbolic sine of ``x``.

    :rtype: vec_type

.. js:function:: cosh (vec_type x)

    Calculates the hyperbolic cosine using ``(e^x + e^-x)/2``.
    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/cosh.xhtml>`_.

    :param vec_type x:
        The value whose hyperbolic cosine to return.

    :return:
        The return value is the hyperbolic cosine of ``x``.

    :rtype: vec_type

.. js:function:: tanh (vec_type x)

    Calculates the hyperbolic tangent using ``sinh(x)/cosh(x)``.
    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/tanh.xhtml>`_.

    :param vec_type x:
        The value whose hyperbolic tangent to return.

    :return:
        The return value is the hyperbolic tangent of ``x``.

    :rtype: vec_type

.. js:function:: asinh (vec_type x)

    Calculates the arc hyperbolic sine of a value.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/asinh.xhtml>`_.

    :param vec_type x:
        The value whose arc hyperbolic sine to return.

    :return:
        The return value is the arc hyperbolic sine of ``x`` which is the
        inverse of sinh.

    :rtype: vec_type

.. js:function:: acosh (vec_type x)

    Calculates the arc hyperbolic cosine of a value.
    The result is undefined if ``x < 1``.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/acos.xhtml>`_.

    :param vec_type x:
        The value whose arc hyperbolic cosine to return.

    :return:
        The return value is the arc hyperbolic cosine of ``x`` which is the
        inverse of cosh.

    :rtype: vec_type

.. js:function:: atanh (vec_type x)

    Calculate the arctangent given a tangent value of ``y/x``. Note: becuase of
    the sign ambiguity, the function cannot determine with certainty in which
    quadrant the angle falls only by its tangent value. If you need to know the
    quadrant, use the other overload of ``atan``.

    The result is undefined if ``x < -1`` or ``x > 1``.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/atan.xhtml>`_.

    :param vec_type y_over_x:
        The fraction whose arc hyperbolic tangent to return.

    :return:
        The return value is the arc hyperbolic tangent of ``x`` which is the
        inverse of tanh.

    :rtype: vec_type

.. js:function:: pow (vec_type x, vec_type y)

    Raises ``x`` to the power of ``y``.

    The result is undefined if ``x < 0`` or  if ``x == 0`` and ``y <= 0``.

    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/pow.xhtml>`_.

    :param vec_type x:
        The value to be raised to the power ``y``.

    :param vec_type y:
        The power to which ``x`` will be raised.

    :return:
        Returns the value of ``x`` raised to the ``y`` power.

    :rtype: vec_type

.. js:function:: exp (vec_type x)


    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: exp2 (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: log (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: log2 (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: sqrt (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: inversesqrt (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: abs (vec_type x)

    `GLSL documentation <>`_.
 
      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: abs (ivec_type x)

    `GLSL documentation <>`_.

      :param ivec_type x:

      :return:
The return value is 

      :rtype: ivec_type

.. js:function:: sign (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: sign (ivec_type x)

    `GLSL documentation <>`_.

      :param ivec_type x:

      :return:
The return value is 

      :rtype: ivec_type

.. js:function:: floor (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: round (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: roundEven (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: trunc (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: ceil (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: fract (vec_type x)

    `GLSL documentation <>`_.

      :param vec_type radians:

      :return:
The return value is 

      :rtype: vec_type

.. js:function:: mod (vec_type x, vec_type y)

.. js:function:: mod (vec_type x, float y)

.. js:function:: modf (vec_type x, out vec_type i)

.. js:function:: min (vec_type a, vec_type b)

.. js:function:: max (vec_type a, vec_type b)

.. js:function:: clamp (vec_type x, vec_type min, vec_type max)

.. js:function:: mix (float a, float b, float c)

.. js:function:: mix (vec_type a, vec_type b, float c)

.. js:function:: mix (vec_type a, vec_type b, bvec_type c)

.. js:function:: fma (vec_type a, vec_type b, vec_type c)

.. js:function:: step (vec_type a, vec_type b)

.. js:function:: step (float a, vec_type b) 

.. js:function:: smoothstep (vec_type a, vec_type b, vec_type c)

.. js:function:: smoothstep (float a, float b, vec_type c)

.. js:function:: isnan (vec_type x)

.. js:function:: isinf (vec_type x)

.. js:function:: floatBitsToInt (vec_type x)

.. js:function:: floatBitsToUint (vec_type x)

.. js:function:: intBitsToFloat (ivec_type x)

.. js:function:: uintBitsToFloat (uvec_type x)

.. js:function:: length (vec_type x)

.. js:function:: distance (vec_type a, vec_type b)

.. js:function:: dot (vec_type a, vec_type b)

.. js:function:: cross (vec3 a, vec3 b)

.. js:function:: normalize (vec_type x)

.. js:function:: reflect (vec3 I, vec3 N)

.. js:function:: refract (vec3 I, vec3 N, float eta)

.. js:function:: faceforward (vec_type N, vec_type I, vec_type Nref)

.. js:function:: matrixCompMult (mat_type x, mat_type y)

.. js:function:: outerProduct (vec_type column, vec_type row)

.. js:function:: transpose (mat_type m)

.. js:function:: determinant (mat_type m)

.. js:function:: inverse (mat_type m)

.. js:function:: lessThan (vec_type x, vec_type y)

.. js:function:: greaterThan (vec_type x, vec_type y)

.. js:function:: lessThanEqual (vec_type x, vec_type y)

.. js:function:: greaterThanEqual (vec_type x, vec_type y)

.. js:function:: equal (vec_type x, vec_type y)

.. js:function:: notEqual (vec_type x, vec_type y)

.. js:function:: any (bvec_type x)

.. js:function:: all (bvec_type x)

.. js:function:: not (bvec_type x)

    ``degrees`` converts a quantity specified in radians into degrees.
    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/degrees.xhtml>`_

      :param bvec_type x:
Specify the quantity, in radians, to be converted to degrees.

      :return:
The return value is ``(radians * 180) / π``.

      :rtype: bool

.. js:function:: textureSize (gsampler2D s, int lod)
.. js:function:: textureSize (gsampler2DArray s, int lod)
.. js:function:: textureSize (gsampler3D s, int lod)
.. js:function:: textureSize (samplerCube s, int lod)
.. js:function:: textureSize (samplerCubeArray s, int lod)

    ``degrees`` converts a quantity specified in radians into degrees.
    `GLSL documentation <https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/degrees.xhtml>`_

      :param sampler_type s:
Specify the quantity, in radians, to be converted to degrees.

      :param int lod:
Specify the quantity, in radians, to be converted to degrees.

      :return:
The return value is ``(radians * 180) / π``.

      :rtype: vec_type

.. js:function:: texture (gsampler2D s, vec2 p [, float bias])
.. js:function:: texture (gsampler2DArray s, vec3 p [, float bias])
.. js:function:: texture (gsampler3D s, vec3 p [, float bias])
.. js:function:: texture (samplerCube s, vec3 p [, float bias])
.. js:function:: texture (samplerCubeArray s, vec4 p [, float bias])

.. js:function:: textureProj (gsampler2D s, vec3 p [, float bias])
.. js:function:: textureProj (gsampler2D s, vec4 p [, float bias])
.. js:function:: textureProj (gsampler3D s, vec4 p [, float bias])

.. js:function:: textureLod (gsampler2D s, vec2 p, float lod)
.. js:function:: textureLod (gsampler2DArray s, vec3 p, float lod)
.. js:function:: textureLod (gsampler3D s, vec3 p, float lod)
.. js:function:: textureLod (samplerCube s, vec3 p, float lod)
.. js:function:: textureLod (samplerCubeArray s, vec4 p, float lod)

.. js:function:: textureProjLod (gsampler2D s, vec3 p, float lod)
.. js:function:: textureProjLod (gsampler2D s, vec4 p, float lod)
.. js:function:: textureProjLod (gsampler3D s, vec4 p, float lod)

.. js:function:: textureGrad (gsampler2D s, vec2 p, vec2 dPdx, vec2 dPdy)
.. js:function:: textureGrad (gsampler2DArray s, vec3 p, vec2 dPdx, vec2 dPdy)
.. js:function:: textureGrad (gsampler3D s, vec3 p, vec2 dPdx, vec2 dPdy)
.. js:function:: textureGrad <textureGrad>` (samplerCube s, vec3 p, vec3 dPdx, vec3 dPdy)
.. js:function:: textureGrad <textureGrad>` (samplerCubeArray s, vec3 p, vec3 dPdx, vec3 dPdy)

.. js:function:: texelFetch <texelFetch>` (gsampler2D s, ivec2 p, int lod)
.. js:function:: texelFetch <texelFetch>` (gsampler2DArray s, ivec3 p, int lod)
.. js:function:: texelFetch <texelFetch>` (gsampler3D s, ivec3 p, int lod)

.. js:function:: textureGather <textureGather>` (gsampler2D s, vec2 p [, int comps])
.. js:function:: textureGather <textureGather>` (gsampler2DArray s, vec3 p [, int comps])
.. js:function:: textureGather <textureGather>` (samplerCube s, vec3 p [, int comps])

.. js:function:: dFdx (vec_type p)

.. js:function:: dFdy (vec_type p)

.. js:function:: fwidth (vec_type p)

.. js:function:: packHalf2x16 (vec2 v)
.. js:function:: unpackHalf2x16 (uint v)

.. js:function:: packUnorm2x16 (vec2 v)
.. js:function:: unpackUnorm2x16 (uint v)

.. js:function:: packSnorm2x16 (vec2 v)
.. js:function:: unpackSnorm2x16 (uint v)

.. js:function:: packUnorm4x8 (vec4 v)
.. js:function:: unpackUnorm4x8 (uint v)

.. js:function:: packSnorm4x8 (vec4 v)
.. js:function:: unpackSnorm4x8 (uint v)

.. js:function:: bitfieldExtract (ivec_type value, int offset, int bits)
.. js:function:: bitfieldExtract (uvec_type value, int offset, int bits)

.. js:function:: bitfieldInsert (ivec_type base, ivec_type insert, int offset, int bits)     
.. js:function:: bitfieldInsert (uvec_type base, uvec_type insert, int offset, int bits)     

.. js:function:: bitfieldReverse (ivec_type value)
.. js:function:: bitfieldReverse (uvec_type value)

.. js:function:: bitCount (ivec_type value)
.. js:function:: bitCount (uvec_type value)

.. js:function:: findLSB (ivec_type value)
.. js:function:: findLSB (uvec_type value)

.. js:function:: findMSB (ivec_type value)
.. js:function:: findMSB (uvec_type value)

.. js:function:: imulExtended (ivec_type x, ivec_type y, out ivec_type msb, out ivec_type lsb) 
.. js:function:: umulExtended (uvec_type x, uvec_type y, out uvec_type msb, out uvec_type lsb) 

.. js:function:: uaddCarry (uvec_type x, uvec_type y, out uvec_type carry)

.. js:function:: usubBorrow (uvec_type x, uvec_type y, out uvec_type borrow)

.. js:function:: ldexp (vec_type x, out ivec_type exp)

.. js:function:: frexp (vec_type x, out ivec_type exp)
.. _doc_gles2_gles3_differences:

Differences between GLES2 and GLES3
===================================

This page documents the differences between GLES2 and GLES3 that are by design and are not the result
of bugs. There may be differences that are unintentional, but they should be reported as bugs.

.. note:: "GLES2" and "GLES3" are the names used in Godot for the two OpenGL-based rendering backends.
          In terms of graphics APIs, the GLES2 backend maps to OpenGL 2.1 on desktop, OpenGL ES 2.0 on
          mobile and WebGL 1.0 on the web. The GLES3 backend maps to OpenGL 3.3 on desktop, OpenGL ES
          3.0 on mobile and WebGL 2.0 on the web.

Particles
---------

GLES2 cannot use the :ref:`Particles <class_Particles>` or :ref:`Particles2D <class_Particles2D>` nodes
as they require advanced GPU features. Instead, use :ref:`CPUParticles <class_CPUParticles>` or
:ref:`CPUParticles2D <class_CPUParticles2D>`, which provides a similar interface to a
:ref:`ParticlesMaterial <class_ParticlesMaterial>`.

.. tip:: Particles and Particles2D can be converted to their CPU equivalent node with the "Convert to
         CPUParticles" option in the editor.

``SCREEN_TEXTURE`` mip-maps
---------------------------

In GLES2, ``SCREEN_TEXTURE`` (accessed via a :ref:`ShaderMaterial <class_ShaderMaterial>`) does not have
computed mip-maps. So when accessing at a different LOD, the texture will not appear blurry. 

``DEPTH_TEXTURE``
-----------------

While GLES2 supports ``DEPTH_TEXTURE`` in shaders, it may not work on some old hardware (especially mobile).

Color space
-----------

GLES2 and GLES3 are in different color spaces. This means that colors will appear slightly
different between them  especially when lighting is used. 

If your game is going to use both GLES2 and GLES3, you can use an ``if`` 
statement check and see if the output is in sRGB, using ``OUTPUT_IS_SRGB``. ``OUTPUT_IS_SRGB`` is 
``true`` in GLES2 and ``false`` in GLES3.

HDR
---

GLES is not capable of using High Dynamic Range (HDR) rendering features. If HDR is set for your 
project, or for a given viewport, Godot will still user Low Dynamic Range (LDR) which limits 
viewport values to the ``0-1`` range.

SpatialMaterial features
------------------------

In GLES2, the following advanced rendering features in the :ref:`SpatialMaterial <class_SpatialMaterial>` are missing:

- Refraction
- Subsurface scattering
- Anisotropy
- Clearcoat
- Depth mapping

When using SpatialMaterials they will not even appear in the editor.

In custom :ref:`ShaderMaterials <class_ShaderMaterial>`, you can set values for these features but they 
will be non-functional. For example, you will still be able to set the ``SSS`` built-in (which normally adds 
subsurface scattering) in your shader, but nothing will happen.

Environment features
--------------------

In GLES2, the following features in the :ref:`Environment <class_Environment>` are missing:

- Auto exposure
- Tonemapping
- Screen space reflections
- Screen space ambient occlusion
- Depth of field
- Glow (also known as bloom)
- Adjustment
 
That means that in GLES2 environments you can only set:

- Sky (including procedural sky)
- Ambient light
- Fog

GIProbes
--------

:ref:`GIProbes <class_GIProbe>` do not work in GLES2. Instead use :ref:`Baked Lightmaps <class_BakedLightmap>`. 
For a description of how baked lightmaps work see the :ref:`Baked Lightmaps tutorial <doc_baked_lightmaps>`.

Contact shadows
---------------

The ``shadow_contact`` property of :ref:`Lights <class_Light>` is not supported in GLES2 and so does nothing.

Light performance
-----------------

In GLES2, performance scales poorly with several lights, as each light is processed in a separate render 
pass (in opposition to GLES3 which is all done in a single pass). Try to limit scenes to as few lights as 
possible in order to achieve greatest performance. 

Texture compression
-------------------

On mobile, GLES2 requires ETC texture compression, while GLES3 requires ETC2. ETC2 is enabled by default, 
so if exporting to mobile using GLES2 make sure to set the project setting 
``rendering/vram_compression/import_etc`` and then reimport textures.

Blend shapes
------------

Blend shapes are not supported in GLES2.

Shading language
----------------

GLES3 provides many built-in functions that GLES2 does not. Below is a list of functions 
that are not available or are have limited support in GLES2.

For a complete list of built-in GLSL functions see the :ref:`Shading Language doc <doc_shading_language>`.

+------------------------------------------------------------------------+--------------------------------------------------+
| Function                                                               | Note                                             |
+========================================================================+==================================================+
| vec_type **sinh** ( vec_type )                                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **cosh** ( vec_type )                                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **tanh** ( vec_type )                                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **asinh** ( vec_type )                                        |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **acosh** ( vec_type )                                        |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **atanh** ( vec_type )                                        |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **round** ( vec_type )                                        |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **roundEven** ( vec_type )                                    |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **trunc** ( vec_type )                                        |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **modf** ( vec_type x, out vec_type i )                       |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_bool_type **isnan** ( vec_type )                                   |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_bool_type **isinf** ( vec_type )                                   |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_int_type **floatBitsToInt** ( vec_type )                           |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_uint_type **floatBitsToUint** ( vec_type )                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **intBitsToFloat** ( vec_int_type )                           |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **uintBitsToFloat** ( vec_uint_type )                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| mat_type **outerProduct** ( vec_type, vec_type )                       |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| mat_type **transpose** ( mat_type )                                    |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| float **determinant** ( mat_type )                                     |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| mat_type **inverse** ( mat_type )                                      |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| ivec2 **textureSize** ( sampler2D_type s, int lod )                    |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| ivec2 **textureSize** ( samplerCube s, int lod )                       |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **texture** ( sampler2D_type s, vec2 uv [, float bias] )     | **bias** not available in vertex shader          |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **texture** ( samplerCube s, vec3 uv [, float bias] )        | **bias** not available in vertex shader          |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **textureProj** ( sampler2D_type s, vec3 uv [, float bias] ) | **bias** not available in vertex shader          |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **textureProj** ( sampler2D_type s, vec4 uv [, float bias] ) | **bias** not available in vertex shader          |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **textureLod** ( sampler2D_type s, vec2 uv, float lod )      | Only available in vertex shader on some hardware |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **textureLod** ( samplerCube s, vec3 uv, float lod )         | Only available in vertex shader on some hardware |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **textureProjLod** ( sampler2D_type s, vec3 uv, float lod )  | Only available in vertex shader                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **textureProjLod** ( sampler2D_type s, vec4 uv, float lod )  | Only available in vertex shader                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec4_type **texelFetch** ( sampler2D_type s, ivec2 uv, int lod )       |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **dFdx** ( vec_type )                                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **dFdy** ( vec_type )                                         |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+
| vec_type **fwidth** ( vec_type )                                       |                                                  |
+------------------------------------------------------------------------+--------------------------------------------------+

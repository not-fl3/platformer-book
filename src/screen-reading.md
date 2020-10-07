# Screen reading shaders

Some effects requires reading from the same target the shader is writing to.

Good example - glass shader with some distortion and refraction. Or water shader in our case!

![water](https://user-images.githubusercontent.com/910977/94856759-0bd0bf00-03f6-11eb-9b13-05335fb1e34d.gif)

The idea here:
- render whole scene
- render the mesh for water with refraction shader
- make refractions realistic - read screen data in the shader

**First step: render water layer with default material**

```rust
impl World {
    ...
    fn draw(&mut self) {
        // draw the whole world
        ...
   
       // draw tiled layer with water
       self.tiled_map.draw_tiles("water", dest_rect, level.area);
   }
}
```

![water_plain](https://user-images.githubusercontent.com/910977/94857384-07f16c80-03f7-11eb-8246-f0659167f3a2.png)

**Second step: add some refractions**

Just as in previous chapter we need a material with custom shader
```rust
let water_material = load_material(WATER_VERTEX_SHADER, WATER_FRAGMENT_SHADER, Default::default()).unwrap();

...
// use that material

gl_use_material(self.water_material);
// draw tiled layer with water
self.tiled_map.draw_tiles("water", dest_rect, level.area);
gl_use_default_material();
```

This was pretty much the same as in previous, post-processing chapter.  
The most intresting part is in the shader.

In macroquad there are bunch of built-in shader variables.

TODO: link to a list of builtins

For water shader we are going to use two of them:

```C
uniform vec4 _Time;
uniform sampler2D _ScreenTexture;
```

`_Time` contains time passed since game start.
`_ScreenTexture` is a very special texture. When material with `_ScreenTexture` in the shader is used for the first time in current frame, macroquad will take a "snapshot" of current active render target and will place a copy of it into _ScreenTexture.

So in the sahder we will be able to read the frame how it is rendered so far and use the frame data while rendering water mesh itself.

[water.glsl](https://gist.github.com/not-fl3/066dfb1e63d3c28d1b5519854d59afe8)

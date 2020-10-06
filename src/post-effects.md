# Post processing

## Step 0: No post processing

```
#[macroquad::main("Post processing")]
async fn main() {
    loop {
        set_camera(Camera2D {
            zoom: vec2(0.01, 0.01),
            target: vec2(0.0, 0.0),
            ..Default::default()
        });
        
        clear_background(RED);
        draw_line(-30.0, 45.0, 30.0, 45.0, 3.0, BLUE);
        draw_circle(-45.0, -35.0, 20.0, YELLOW);
        draw_circle(45.0, -35.0, 20.0, GREEN);
        
        next_frame().await;
    }
}
```

![](https://i.imgur.com/X4mew1P.png)

## Step 1: Pixelisation

```
let render_target = render_target(320, 150);

set_texture_filter(render_target.texture, FilterMode::Nearest);

loop {
  // drawing to the texture

  // add "render_target" field to camera setup
  // now it will render to texture 
  set_camera(Camera2D {
    zoom: vec2(0.01, 0.01),
    target: vec2(0.0, 0.0),
    render_target: Some(render_target),
    ..Default::default()
  });

  clear_background(RED);

  // draw our game!
  draw_line(-30.0, 45.0, 30.0, 45.0, 3.0, BLUE);
  draw_circle(-45.0, -35.0, 20.0, YELLOW);
  draw_circle(45.0, -35.0, 20.0, GREEN);

  // drawing to the screen

  // 0..1, 0..1 camera
  set_camera(Camera2D {
    zoom: vec2(1.0, 1.0),
    target: vec2(0.0, 0.0),
    ..Default::default()
  });

  // draw full screen quad with previously rendered scene
  draw_texture_ex(
      render_target.texture,
      0.,
      0.,
      WHITE,
      DrawTextureParams {
          dest_size: Some(vec2(1.0, 1.0)),
          ..Default::default()
      },
  );

```

We got nicely pixelized, old-schoold game.  

![](https://i.imgur.com/shyFuNp.png)

## Step 3: Shaders

But just pixelization is not enough.  
Let glean some cool post-processing effect from shadertoy.

https://www.shadertoy.com/view/XtlSD7 CRT effect!  

```
// same render target setup from previous step
let render_target = render_target(320, 150);
set_texture_filter(render_target.texture, FilterMode::Nearest);

// 
let material = load_material(CRT_VERTEX_SHADER, CRT_FRAGMENT_SHADER, Default::default()).unwrap();

// main loop is going to be exactly the same 
loop {
   // drawing to the texture
   ...
   
  // draw full screen quad with previously rendered scene
  // but before drawing texture, default macroquad material
  // needs to be replaced to the new one with CRT shader
  gl_use_material(material);

  draw_texture_ex(
      render_target.texture,
      0.,
      0.,
      WHITE,
      DrawTextureParams {
          dest_size: Some(vec2(1.0, 1.0)),
          ..Default::default()
      },
  );
  // switch back to default material
  gl_use_default_material();
 
}
```

![](https://i.imgur.com/Cx6GhWm.png)

Entire code for this example is available here: https://github.com/not-fl3/macroquad/blob/master/examples/post_processing.rs
Web build: https://not-fl3.github.io/miniquad-samples/post_processing.html

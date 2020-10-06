# Water level

Lets implement a very special levels with water.  
For how to draw water take a look on "screen reading shaders" chapter.

The goal here - implement very unique physics and game rules for being underwater.  
Bonus points for keeping all the old code clean from underwater special cases.

**Barebone swimming state**

```rust
impl Player {
    const ST_NORMAL: usize = 0;
    const ST_DASH: usize = 1;
    const ST_DEATH: usize = 2;
    const ST_WIN: usize = 3;
    // The new state
    const ST_SWIM: usize = 4;

   pub fn new(...) {
   ...
       // state machine configuration for the new state
       // very similar to all the same custom state from previous chapters
       state_machine.insert(Self::ST_SWIM, State::new().update(Self::update_swim));
   }
   
    fn update_swim(&mut self, room: &mut Room, dt: f32) {
        // "swim_check" will look is the middle of the player sprite collides water tile on "water" level
        // TODO: when collision code will be cleaned up - show swim_check contents
        if self.swim_check(room) == false {
            // not in water, back to normal
            self.state_machine.set_state(Self::ST_NORMAL);
        }

        // simple water "physics"
        // if down button is not pushed - player is going up
        // if is pushed - player going to sink 
        let mut floating_speed = self.swim_afloat_speed;
        if is_key_down(KeyCode::Down) {
            floating_speed = self.swim_sink_speed;
        }
        self.spd.y = floating_speed;

        if is_key_down(KeyCode::Right) {
          ..
        }

        if is_key_down(KeyCode::Left) {
          ..
        }
    }
    
    fn update(&mut self, room: &mut Room) {
        ...
        // if player is in the normal state and in the water - switch for water state
        if self.state_machine.state() == Self::ST_NORMAL && self.swim_check(room) {
            self.state_machine.set_state(Self::ST_SWIM);
        }
        ...
    }
}
```

![water-simple](https://user-images.githubusercontent.com/910977/94893803-a27c9a80-044d-11eb-892b-bed255d62e35.gif)

**Oxygen**

```rust
    fn update_swim(&mut self, room: &mut Room, dt: f32) {
        ...
        self.oxygen -= self.oxygen_consumption * dt;

        if self.oxygen <= 0.0 {
            // while there is no cutscene state for dead player underwater - jsut ask room to reload level
            room.lose();
        }
        ...
    }
```

But oxygen recovery is going to happen in any other non-swimming states. So here for the first time swimming-specific behavior is going to leak just from swimming state to game logic in general.  
It is possible to justify it, though: oxygen is a fundamental law of the game now, so it is fine to do something about it on each frame in main player's update function:


```rust
impl Player {
    ...
    
    pub fn update(&mut self, room: &mut Room) {
        ...
        if self.swim_check(room) == false {
            self.oxygen = self.max_oxygen.min(self.oxygen + self.oxygen_recovery * dt);
        } else {
            self.oxygen -= self.oxygen_consumption * dt;
        }        
        ...
    }
}
```

**Oxygen level UI**

```rust
impl Player {
    ...
    
    fn draw(&mut self) {
        // even if player is not swimming state - it would be nice to see how oxygen is recovering
        if self.state_machine.state() == Self::ST_SWIM || self.oxygen != self.max_oxygen {
            draw_rectangle(self.pos.x() - 2.3, self.pos.y() - 0.1, 2.6, 6.2, BLACK);
            draw_rectangle(
                self.pos.x() - 2.0,
                self.pos.y(),
                2.0,
                6.0 * self.oxygen / self.max_oxygen,
                BLUE,
            );
        }
    }
}
```

Result of that magic constants and hand-adjusted positions in "draw" function:

![water_bar](https://user-images.githubusercontent.com/910977/94889259-23816500-0441-11eb-9fdd-8634feb6d9e5.gif)

**Out of oxygen post-effect**

For more pressure on the player from running out of oxygen situation lets add some vignette post effect.  
Shader used is going to be very similar to the one used in "post-effects" chapter, but this time the amount of post effect is going to depend on in-game content.

```rust
    let material = load_material(
        VIGNETTE_SHADER,
        VIGNETTE_SHADER,
        MaterialParams {
            uniforms: vec![
                ("Target".to_string(), UniformType::Float2),
                ("Amount".to_string(), UniformType::Float1),
            ],
            ..Default::default()
        },
    )
    .unwrap();
```

This way macroquad will know that this material have two uniform variables. And now it is possible to set those variables at runtime: 

```rust
        vignette_material.set_uniform("Target", room.vignette_center);
        vignette_material.set_uniform("Amount", room.vignette_amount);

        gl_use_material(vignette_material);
        
        // full-screen quad from "post-processing" chapter
        draw_texture_ex(
            render_target.texture,
            0.,
            0.,
            WHITE,
            DrawTextureParams {
                dest_size: Some(vec2(screen_width(), screen_height())),
                ..Default::default()
            },
        );
        gl_use_default_material();

```

![water_vignette](https://user-images.githubusercontent.com/910977/94890904-43675780-0446-11eb-9a7d-cd5610c20f94.gif)


To speed up game tempo we can allow dashing under water.  
Dash can consume significant amount of oxygen, so spending non-optimal amount of dashes or going even a slightly wrong direction will result of fast fail.  
However now it is possible for a player to take some risks and finish level a bit faster!

```rust
    fn update_swim(&mut self, room: &mut Room, dt: f32) {
        ...
        if self.can_dash() {
            self.oxygen -= self.dash_oxygen_cost;
            
            // the same function that was used in "update_normal"
            // so its going to be exactly the same dash as in underwater state
            self.start_dash();
            return;
        }
        ...
    }

```

![water_vignette_dash](https://user-images.githubusercontent.com/910977/94891424-9f7eab80-0447-11eb-9787-542aba3a09e6.gif)




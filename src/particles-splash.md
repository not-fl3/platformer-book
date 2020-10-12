# Particles system: water splash

```rust
fn new() -> Player {
    Player {
        ...
        water_particles: Emitter::new(EmitterConfig {
            emitting: false,
            lifetime: 0.8,
            lifetime_randomness: 0.5,
            initial_size: 0.3,
            initial_velocity: 50.0,
            initial_velocity_randomness: 0.3,
            initial_direction_spread: 0.5,
            gravity: vec2(0.0, 150.0),
            material: Some(ParticleMaterial::new(VERTEX, FRAGMENT)),
            ..Default::default()
        }),
    }
}
```

There are two options of using emitters: set emit speed, amount etc parameters to EmitterConfig and let emitter emit by itself.  
Or just manually emit some particles in certain place.  

```rust
    fn water_splash_effect(&mut self) {
        let amount = 7;

        // Spawn some particles in equally spreaded straight line
        for i in 0..amount {
            self.water_particles.emit(
                self.pos + vec2(i as f32 / amount as f32 * self.width, 0.0),
                1,
            );
        }
    }

```

Vertex shader:  
```C
#include "particles.glsl"

PARTICLE_MESH_DECL

varying lowp vec2 texcoord;
varying lowp vec4 particle_data;

void main() {
    gl_Position = particle_transform_vertex();
    texcoord = particle_transform_uv();
    
    particle_data = in_attr_inst_data;
}
```

Fragment shader:  
```C
#include "particles.glsl"

precision lowp float;
varying lowp vec2 texcoord;
varying lowp vec4 particle_data;

uniform sampler2D texture;

void main() {
    // particle_ix is uniquad id of each particle
    float randomize_initial_color = 0.5 + rand(vec2(particle_ix(particle_data), 0)) * 0.5;

    // particle_lifetime is 0..1 value with 0 at the beginning of particle life and 1 just before particle removal
    float fade_during_lifetime = 0.5 + (1.0 - particle_lifetime(particle_data));

    gl_FragColor = texture2D(texture, texcoord) * randomize_initial_color * fade_during_lifetime;
}

```

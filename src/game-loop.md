# Game loop

```rust
#[macroquad::main("Platformer")]
async fn main() {
    let mut player = Vec2::new(screen_width() / 2, screen_height() / 2);

    loop {
        clear_background();

        // update game logic

        if is_key_down(KeyCode::Right) {
            player.x += 1.0;
        }
        if is_key_down(KeyCode::Left) {
            player.x -= 1.0;
        }

        // draw game world, quite simple so far
        draw_circle(player, 5., RED);

        next_frame().await
    }
}
```

GIF



# Animation controller

```rust
// wat really
async fn play_animation(&mut self) {
    self.sprite = 322;
    for _ in 0 .. 9i32 {
        self.sprite += 1;
        wait_seconds(0.05).await;
    }
}
```

```rust
async fn death_coroutine(&mut self, room: Room) {
    self.spd = vec2(0., 0.);
    self.play_animation().await;
    room.lose();
}
```


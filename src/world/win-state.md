# Win state

Small state with cutscene at the end of each level.

![cutscene](https://i.imgur.com/tVu5FQR.gif)

```rust
impl Player {
    const ST_NORMAL: usize = 0;
    const ST_DASH: usize = 1;
    const ST_DEATH: usize = 2;
    const ST_WIN: usize = 3;

    pub fn new(...) -> Player {
        let mut state_machine = StateMachine::new();

        state_machine.insert(Self::ST_NORMAL, State::new().update(Self::update_normal));
        state_machine.insert(
            Self::ST_DASH,
            State::new()
                .update(Self::update_dash)
                .coroutine(Self::dash_coroutine),
        );
        state_machine.insert(
            Self::ST_DEATH,
            State::new().coroutine(Self::death_coroutine),
        );
        
        // New state added: Win state
        state_machine.insert(Self::ST_WIN, State::new().coroutine(Self::win_coroutine));
        ... 
    }
}
```

```rust
impl Player {
    // this function is supposed to be called by collision detection code and signal the player to start win cutscene
    pub fn win(&mut self, flag_position: Vec2) {
        self.flag_position = flag_position;

        self.state_machine.set_state(Self::ST_WIN);
    }
}
```

```rust
impl Player {
    ...
    async fn win_coroutine(&mut self, room: &mut Room) -> Coroutine {
        let start = self.pos;
        let end = self.flag_position;

        room.in_cutscene = true;

        // here we start 3 independent parallel coroutines moving some player params at the same time
        let rotate = start_coroutine(tweens::linear(&mut self.rotation, 5.0, 0.7));
        let scale = start_coroutine(tweens::linear(&mut self.scale, vec2(0.1, 0.1), 0.9));
        let slowdown = start_coroutine(tweens::linear(&mut self.spd, vec2(0.0, 0.0), 0.3));

        // while coroutines are now runned on background
        // here we can wait while all 3 of them will finish
        // and animate the player with those params
        while !rotate.is_done() || !slowdown.is_done() || !scale.is_done() {
            self.pos += self.spd * delta;
            self.pos = self.pos.lerp(end, 0.02);

            next_frame().await;
        }

        room.in_cutscene = false;
    }
}
```

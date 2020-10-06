# Player state machine

```rust
pub struct Player {
    pos: Vec2,
    spd: Vec2,
    
    ...
    
    state_machine: StateMachine,
}
```


```rust
impl Player {
    fn new() -> Player {
        let mut state_machine = StateMachine::new();
        
        // update function will be called on each state_machine.update()
        state_machine.insert(
            Self::ST_NORMAL, 
            State::new()
                .update(Self::update_normal));

        // coroutine will be started each time player enter dash state 
        state_machine.insert(
            Self::ST_DASH,
            State::new()
                .update(Self::update_dash)
                .coroutine(Self::dash_coroutine),
        );

    }
}
```

Dash state: both "update" and "coroutine" are used.   
"dash_coroutine" is responsible for changing player state once at the beginning of the dash and than switching back to "normal".   
"update" may apply some optional physics or game rules that works only in dash state. Is just an empty function right now.

```rust
impl Player {
    // this will be started on each coroutine state enter
    async fn dash_coroutine(&mut self, room: &mut Room) {
        // change the speed with dash speed
        self.spd = self.last_aim * self.dash_speed;
        // and just wait for dash_duration seconds
        // during wait period "update_dash" function will be called on each frame
        wait_seconds(self.dash_duration).await;
        // dash is over, going back to normal
        self.state_machine.set_state(Self::ST_NORMAL);
     
    }

    // dash update is empty: nothing is affecting player during the dash 
    fn update_dash(&mut self, room: &mut Room, dt: f32) {}
}
```

Player's "normal" state update: check controls available in "normal" state and maybe switch to "dash" state:

```rust
impl Player {
    fn start_dash(&mut self) {
        self.dashes = 0;
        // during the dash player has completely different behaviour
        // changing the playr's state to DASH
        self.state_machine.set_state(Self::ST_DASH);
    }
    
    fn jump(&mut self) {
        self.spd.y = self.jump_speed;
        // during jump player is behaving exactly as usual
        // so the state do not change here
    }

    fn update_normal(&mut self, room: &mut Room, dt: f32) {
        if is_key_pressed(KeyCode::A) {
            self.start_dash();
            return;
        }
        
        if is_key_pressed(KeyCode::S) {
            self.jump();
            return;
        }
        
        // running
        ...
        
        // gravity
        ...
    }
}
```

Main player's update function: apply general, state independent game rules and update state machine:

```rust
impl Player {
    fn update(&mut self, room: &mut Room) {
        // physics: apply spd to self.pos with some collisions
        ...
        
        // win conditions check
        ...
        
        // lose conditions check
        ...
        
        // camera update
        ...
        
        self.state_machine.update(room);
    }
}
```
# Dash

# Spring

![spring](https://i.imgur.com/7aIobSo.gif)

```rust
impl Spring {
    pub fn new(...) -> Spring {
        let mut state_machine = StateMachine::new();
        
        state_machine.insert(Self::ST_NORMAL, State::new().update(Self::update_normal));
        state_machine.insert(
            Self::ST_JUMP,
            State::new()
                .update(Self::update_jump)
                .coroutine(Self::jump_coroutine),
        );

        Spring {
            state_machine: StateMachineContainer::Ready(state_machine),
            
            ...
        }
    }

    // normal state update: when spring is ready to bounce some players
    pub fn update_normal(&mut self, room: &mut Room, _dt: f32) {
        for object in room.objects.iter() {
            if let GameObject::Player(ref mut player) = object.data {
                if self.collide(player) {
                    player.bounce();
                    self.state_machine.set_state(Self::ST_JUMP);
                }
            }
        }
    }

    // during jump animation spring do not work as a spring
    pub fn update_jump(&mut self, _room: &mut Room, _dt: f32) {}

    // this coroutine will be started on enter to jump state
    pub async fn jump_coroutine(&mut self, _room: &mut Room) {
       // change a sprite to a compressed spring
       self.spr = 865;
       // wait a little bit
       wait_seconds(2.0).await;
       // and change sprite back to normal, uncompressed spring
       self.spr = 864;
       // and now the spring will work as a spring again
       self.state_machine.set_state(Self::ST_NORMAL);
    }
}
```

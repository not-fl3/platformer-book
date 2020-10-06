# Tweens

```rust
async fn dash_coroutine(&mut self, room: &mut Room) {
    // change current speed to maximum dash speed
    self.spd = self.last_aim * self.dash_speed;

    // and than wait for some time to move forward with dash
    wait_seconds(self.dash_duration).await;

    // release the controls and go back to normal mode
    self.state_machine.set_state(Self::ST_NORMAL);
}
```

This is cool, but looks not so awesome. 
To make it better it would be nice to slowly accelerate from current speed to dashspeed instead of instant acceleration. 

Tweens are going to be used. Tweens are special coroutines made specifically for this: change some variable for some time.  

```rust
async fn dash_coroutine(&mut self, room: &mut Room) {
    let target_dash_speed = self.last_aim * self.dash_speed;

    // accelerate from current speed to dash speed for some time
    tweens::linear(&mut self.spd, target_dash_speed, self.dash_accel_duration).await;

    // and than keep moving with dash speed
    wait_seconds(self.dash_duration).await;

    // release the controls and go back to normal mode
    self.state_machine.set_state(Self::ST_NORMAL);
}
```

# Dead Reckoning for Aerial Robotics

## What is Dead Reckoning?

Dead reckoning is a method used to estimate the current position of a drone based on a previously known position and measurements of its movement. 
Instead of depending completely on GPS, the drone uses onboard sensors such as accelerometers and gyroscopes to estimate where it is.
Similar ideas were used in NASA's Ingenuity Mars Helicopter, where onboard sensors helped estimate motion and position because GPS is not available on Mars.

In simple terms:

> If I know where I started and I know how I moved, I can estimate where I am now.

---
## How It Works

The flight controller doesn't work continuously. It samples the IMU at fixed intervals, so every quantity is tracked one sample at a time:

Dead reckoning starts from a known state:

- Initial position *p*<sub>0</sub> 
- Initial velocity *v*<sub>0</sub> 
- Initial orientation *θ*<sub>0</sub> 

The drone then uses data from its Inertial Measurement Unit (IMU), which contains:

- Accelerometers (measure acceleration)
- Gyroscopes (measure rotation rate)

The gyroscope measures angular velocity *ω* (how fast the drone is rotating). Multiply by Δ*t* to get how much it turned during one tick, then add that to the current angle:

*θ*<sub>1</sub> = *θ*<sub>0</sub> + *ω*<sub>0</sub> · Δ*t*

where *θ* is orientation and *ω* is angular velocity.

This angle is needed for the next step.

## Step 2: Estimate Velocity

The accelerometer measures acceleration in the drone's own frame, which tilts as the drone tilts. Using the orientation *θ* from Step 1, the controller rotates this reading into the world frame and subtracts gravity. What's left is the drone's true acceleration *a*, which is integrated the same way:

*v*<sub>1</sub> = *v*<sub>0</sub> + *a* · Δ*t*

where *v* is velocity and *a* is the gravity-corrected acceleration.

## Step 3: Estimate Position

Velocity from Step 2 is integrated once more to get position:

*p*<sub>1</sub> = *p*<sub>0</sub> + *v*<sub>1</sub> · Δ*t*

where *p* is position.

The controller repeats Steps 1 to 3 every tick, so each new estimate builds on the previous one, starting from the known initial state.


```text
Gyroscope ------> Orientation (θ)
                       |
                       v   rotate reading + remove gravity
Accelerometer --> Acceleration (a)
                       |
                       v
                  Velocity (v)
                       |
                       v
                  Position (p)
```

### Why Errors Grow

A small sensor error can become a large position error because integration is performed repeatedly.

For example:

`Position Error ≈ Velocity Error × Time`

If the velocity estimate is off by only 0.2 m/s for 30 seconds:

`Position Error ≈ 0.2 × 30 = 6 m`

As flight time increases, the estimated position gradually moves away from the true position unless another sensor is used to correct it.
This accumulated error is known as drift.

## Why Is It Useful?

Dead reckoning becomes important when GPS is unavailable or unreliable.

Examples include:

- Flying indoors
- Under bridges
- Dense tree cover
- Areas with signal interference
- Flying on Mars

The drone can continue navigating for a short period even without GPS.

---

## Main Challenge

The biggest problem with dead reckoning is **drift**. Small sensor errors build up over time.
A tiny error in acceleration measurement can eventually become a large position error after continuous calculations.
Because of this, dead reckoning is usually reliable for short periods but not for long-term navigation.

---

## How Modern Drones Solve This

Modern flight controllers combine dead reckoning with other sensors such as:

- GPS
- Barometer
- Magnetometer
- Optical Flow Sensors
- Cameras
- LiDAR

An Extended Kalman Filter (EKF) is commonly used to combine information from these sensors and reduce accumulated errors.

---

## Conclusion

Dead reckoning is an important part of drone navigation because it allows a UAV to continue estimating its position when GPS is unavailable.
However, sensor errors accumulate over time, making dead reckoning unsuitable as a standalone long-term navigation solution.
For this reason, modern drones combine dead reckoning with other sensors to achieve more accurate and reliable navigation.

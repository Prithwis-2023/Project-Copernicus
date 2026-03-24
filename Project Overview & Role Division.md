Every component of our project has two aspects, one is the theoretical ground -up and second is the technical implementation, which is due to the fact that our edge system, and our CubeSat in general, is going to operate at Low Earth Orbit with resource constraints. 

## Power Division

- Assigned to: **Snow**, *Orbital Power Management Engineer* 
### Task 1: The Orbital Power

Power is everything for any spacecraft, without it the spacecraft can neither talk to base station nor can it correct its trajectory; overall the craft will be doomed! Therefore, before our system assembly,  we need to prove the math. 

-  **Edge Device Power Requirement:** A Jetson Nano running an active inference pulls 5 to 10 Watts of power.
-  **LEO Challenge:** In Low Earth Orbit  (LEO), a satellite experiences roughly 45 minutes of sunlight and 45 minutes of total darkness (eclipse) per orbit.
-  We need to calculate the required surface area of solar panels to not only power the Edge device during the "sunlight" phase, but also simultaneously charge our backup power supply, the onboard battery bank (LiPo or 18650s), enough to survive the 45 minute eclipse phase.

### Task 2: Telemetry and Intelligent Power Management

This is where we bridge the gap between power supply and the intelligent decision making related to power.

- **The Sensor:** We need to integrate INA219 or INA3221 DC Current/Voltage Sensor Module.
-  This module connects to the Edge Device's I2C pins.
-  A Python Script will constantly monitor the battery voltage and incoming solar wattage. If the voltage drops too low (which will indicate we have a degraded solar panel or a micro-meteorite strike), the edge device will autonomously enter "Safe Mode" - throttling the CPU and pausing our AI inference system to conserve power.

## Astro-Mechanical Division 

- Assigned to: **May** & **Prithwis**, *Deployable Structures Engineer*

The work of this division is is solving the high-stakes mechanical constraints. The primary goal is to ensure the **Miura-ori** array deploys 100% of the time without tearing or jamming.

### Task 1: Geometry and Kinematics

The Miura-ori is a "rigid-foldable" pattern, meaning the panels stay flat while hinges do all the work.
-  **The Task:** Must determine the "Packing Factor". This is the ratio of the deployable area to the stowed volume.
-  **The Math:** Need to calculate the necessary torque required from a servo to overcome the friction of the hinges and the "memory" of the material.
-  **Goal:** A functional Miura-ori template scaled to fit our prototype.

### Task 2: Materials & Prototyping

Space hardware uses Kapton or specialized composites. For our project we need to find suitable material, thin enough to fold but stiff enough to retain shape.

- The Task: Testing materials like 0.5mm Polypropylene sheets, with living hinges.
-  Innovation (TBD): We use conductive ink or thin copper tape along the fold lines to simulate real electrical traces that carry power from the panels to the main bus.

### Task 3: Actuation System

- Designing the **Deployment Sequence**. How will the solar array deploy?
-  Must provide a single trigger pin so that when the pin goes high, the array will deploy smoothly.

## AI Division

Assigned to: **Noge**, *AI Systems Engineer*

I recommend utilizing **Isolation Forest**, which is a sophisticated choice for performing **Anomaly Detection**. In the context of orbital mechanics, most things follow a predictable, "normal" path. Debris—or a satellite that has lost its station-keeping—is essentially a mathematical outlier.

- **The Concept:** Instead of training a model of a large number of pictures of debris, the Isolation Forest focuses on what "Normal Space" looks like. Anything object that moves with irregular velocity, a different spectral signature, or an unplanned trajectory is "isolated" as a threat.
-  Isolation Forest (IF) is computationally cheap compared to deep neural networks. On an edge device, this leaves more GPU headroom for the primary vision pipeline.

#Note Be wary of **Noise**. In space simulations, background stars can look like distant debris. We need to implement a **Temporal Filter** (tracking objects across multiple frames) before feeding the data into the Isolation Forest.


| Phase               | Task                         | Description                                                                                                                      |
| ------------------- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Feature Engineering | Define the "Normal" scenario | Extract features like $x, y$ coordinates, velocity vectors $(\Delta x, \Delta y)$, and brightness (albedo) from the camera feed. |
| Model Training      | Scikit-Learn Loop            | Need to tune ```n_estimators``` and ```contamination``` levels to avoid false positives (like distant stars)                     |
| Inference Engine    | Real-Time Gateway            | Model must perform processing the vision metadata under 10ms. If the "Anomaly Score" hits the threshold, it triggers the alert.  |
| Hybridization       | The Safety Net               | Use a lightweight Object Detector (like SSD-Mobilenet) to find known satellites and use Isolation Forest to find the anomalies.  |

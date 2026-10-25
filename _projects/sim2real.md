---
layout: page
title: Sim2Real Autonomous Driving Platform
description: A simulation-to-reality platform for autonomous driving research, from virtual agents to real robots.
img: assets/img/sim2real/hero.jpg
importance: 1
category: research
related_publications: false
toc:
  sidebar: left
---

My Ph.D. shifted from vehicle dynamics and optimal control toward learning-based **motion
prediction**, but my interest in control engineering remained. Around the time I finished my
Ph.D., I built a project at Linköping University that let me revisit all of it hands-on: a
**sim-to-real platform for autonomous driving (AD) research**.

The goal was a platform where an entire AD software stack could be developed and validated in
simulation, and then deployed onto real robotic vehicles while preserving consistent behavior.

{% include figure.liquid loading="eager" path="assets/img/sim2real/hero.jpg" class="img-fluid rounded z-depth-1" zoomable=true caption="Small-scale robotic vehicles used for real-world experiments. The platform was designed so the same autonomous driving stack drove both the simulated and the physical cars." %}

I was interviewed about the platform by Linköping University. You can read the
full article here:

<div class="text-center mt-2 mb-4">
  <a class="btn btn-outline-primary" href="https://liu.se/en/news-item/fran-simulering-till-verklighet-med-sjalvkorande-bilar" target="_blank" rel="noopener noreferrer">Read the interview (liu.se) →</a>
</div>

## Objective

The platform was built around a single guiding objective: **develop a sim-to-real platform
for autonomous driving research that supports a smooth transition from simulation to
real-world deployment.** Three requirements followed from it:

1. The simulation had to model robot **kinematics and sensor behavior** with sufficient accuracy.
2. The platform had to support **realistic, rule-based traffic scenarios** that translate to real-world conditions.
3. The system had to include a **comprehensive AD stack** that runs consistently in simulation and in the real world.

Everything was built on **ROS 2** and deployed using **Docker**, which kept the software
environment identical across machines and across the sim-to-real boundary.

## A modular framework

The software stack followed a **modular framework**: a set of submodules, each with a
distinct responsibility, connected into a full perception, planning, and control pipeline.
This was the standard design used in industry, and it was deliberate here. Modularity
promotes interpretability, reusability, and maintainability, and a complete modular stack is
an excellent starting point for targeted research into any single functionality.

{% include figure.liquid loading="eager" path="assets/img/sim2real/architecture.png" class="img-fluid rounded z-depth-1" zoomable=true caption="The modular autonomous driving stack: perception feeds a behavioral and route-planning layer, which feeds motion planning and local feedback control. Control architecture after Paden et al., IEEE Trans. Intelligent Vehicles, 1(1), 2016." %}

The sections below walk through the stack from state estimation out to the low-level
controllers that drove the wheels.

## State estimation

In simulation, the full system state was available for free. On the real robots it had to be
**estimated** from noisy sensors (onboard IMU, wheel odometry, and a camera-based positioning
system) fused together.

Motion was described by a **kinematic single-track (bicycle) model**, which served as the
prediction model for the estimator and as ground-truth dynamics in simulation:

$$
\begin{aligned}
\dot{x} &= v \cos\theta \\
\dot{y} &= v \sin\theta \\
\dot{\theta} &= \frac{v}{L}\tan\delta \\
\dot{v} &= a
\end{aligned}
$$

Here $$x, y, \theta$$ are the vehicle's pose, $$v$$ is velocity, $$a$$ is acceleration, and
$$\delta$$ is the steering angle. The parameters were matched to the physical robot; for
example, a wheelbase of $$L = 0.275\,\text{m}$$. An **Extended Kalman Filter (EKF)** fused
the sensor measurements against this model to produce the state estimates used during
real-world experiments.

{% include figure.liquid loading="lazy" path="assets/img/sim2real/state_estimation.png" class="img-fluid rounded z-depth-1" zoomable=true caption="State estimation: the kinematic single-track model was used both as the simulator's ground truth and as the prediction step of an EKF that fused IMU, odometry, and camera-based positioning on the real robots." %}

## Route planning

The **route planner** solved the planning problem at the highest level: given a start and a
goal, it computed the best path through the road network. The network was represented as a
**graph** whose nodes were categorized (stop, yield, parking, regular); a mission, such as
driving from one parking space to another, was defined over it, and the optimal plan was
found with **A\* search**. The resulting waypoints were handed down to the behavior layer.

{% include figure.liquid loading="lazy" path="assets/img/sim2real/route_planner.png" class="img-fluid rounded z-depth-1" zoomable=true caption="The road network as a categorized graph. A* search computed an optimal sequence of waypoints from start to goal, which was passed to the behavior layer." %}

## Behavior layer

The **behavior layer** continuously adapted the robot's actions to the current environment
and mission. It was implemented as a **Behavior Tree (BT)**, a modular, scalable structure
that executed logical decisions such as checking for stops, handling yields, and managing
intersections. It sent subtasks as waypoints down to the motion planner, communicated
*driving intent* (go, stop, slow down) to the controller, and, because it was hierarchical,
was easy to extend with new behaviors.

{% include figure.liquid loading="lazy" path="assets/img/sim2real/behavior.png" class="img-fluid rounded z-depth-1" zoomable=true caption="The behavior layer as a Behavior Tree: hierarchical, interpretable decision-making that was straightforward to extend with new behaviors." %}

## Motion planning

The **motion planner** was optimization-based. It found a time-optimal path near a set of
reference waypoints $$\mathcal{W} = \{(w_{x_i}, w_{y_i})\}_{i=1}^{M}$$ subject to the vehicle's
differential constraints $$f(\cdot)$$:

$$
\min_{u}\quad T + R\sum_{k=0}^{N-1}\lVert u_k \rVert^2
   + Q_1 \sum_{k=0}^{N} g_{\text{path}}(x_k, s_k, \mathcal{W})
   + Q_2 \lVert \sigma \rVert_1
$$

$$
\begin{aligned}
\text{s.t.}\quad & x_{k+1} = f(x_k, u_k), && \forall k \in \{0,\dots,N-1\}, \\
   & s_{k+1} \geq s_k,\; s_0 = 0,\; s_N = 1, && \text{(path-progress constraints)} \\
   & u_{\min} \leq u_k \leq u_{\max}, && \forall k, \\
   & h(x_0, x_N, \sigma) \leq 0, && \text{(soft boundary constraints)} \\
   & 0 \leq \sigma \leq \sigma_{\max}. && \text{(slack bounds)}
\end{aligned}
$$

$$R, Q_1, Q_2$$ are tunable weights and $$g_{\text{path}}$$ penalizes deviation from the
reference waypoints. Increasing $$Q_1$$ pulled the optimized trajectory closer to the
reference path, trading off against time-optimality.

{% include figure.liquid loading="lazy" path="assets/img/sim2real/motion_planner.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Optimization-based motion planning. Larger Q₁ weights pulled the optimized path closer to the reference waypoints; smaller weights favored a more time-optimal trajectory." %}

## Motion control

The **motion controller** executed the planned path by regulating steering and velocity. It
used a **decoupled** architecture: lateral and longitudinal control were handled
independently, which simplified tuning and reduced complexity.

- **Lateral control** used a **Pure-Pursuit** algorithm to compute the steering angle $$\delta$$, with rate-limiting for smooth steering transitions.
- **Longitudinal control** used a cascaded structure: the **Intelligent Driver Model (IDM)** set the velocity (used directly in simulation), and a **PI controller** converted velocity commands into PWM signals for the real robots.

{% include figure.liquid loading="lazy" path="assets/img/sim2real/motion_controller.png" class="img-fluid rounded z-depth-1" zoomable=true caption="Decoupled motion control: Pure-Pursuit for lateral steering, and a cascaded IDM and PI structure for longitudinal velocity control." %}

## Demonstration

The platform ran a full AD stack (state estimation, route planning, motion planning, and
control) in a simulated traffic scenario and then transitioned to real-world deployment on
hardware. The video below shows the system in action, highlighting the consistency of system
behavior across the sim-to-real boundary.

{% include video.liquid path="assets/video/video_pres_link_v3b.mp4" class="img-fluid rounded z-depth-1" controls=true muted=true width="100%" caption="The Sim2Real platform in action: a full AD stack running in a real-world-inspired traffic scenario in simulation, followed by real-world deployment on the physical robots." %}

The technical write-ups and additional material are also on LinkedIn, and the project was
featured in an interview by Linköping University:

<div class="text-center mt-3 mb-2">
  <a class="btn btn-outline-primary mb-2" href="https://liu.se/en/news-item/fran-simulering-till-verklighet-med-sjalvkorande-bilar" target="_blank" rel="noopener noreferrer">Interview article</a>
  <a class="btn btn-outline-primary mb-2" href="https://www.linkedin.com/feed/update/urn:li:activity:7323254891517227008/" target="_blank" rel="noopener noreferrer">Technical details (LinkedIn)</a>
  <a class="btn btn-outline-primary mb-2" href="https://www.linkedin.com/in/theodor-westny/recent-activity/all/" target="_blank" rel="noopener noreferrer">Demonstration post (LinkedIn)</a>
</div>

---

*Acknowledgements: this work was carried out at Linköping University, where it supported
teaching and research in autonomous driving.*

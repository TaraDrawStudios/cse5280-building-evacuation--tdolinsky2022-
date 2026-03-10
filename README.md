# CSE5280 Assignment Report: Evacuation Simulation in a Three-Floor Building

## Name
Your Name Here

##Course
CSE5280

##Assignment
Evacuation Simulation (Three-Floor Building)

---

#1.Introduction

In this assignment, I built a multi-agent evacuation simulation for a three-floor building using a cost-function minimization framework. The main goal was to model how a group of agents can evacuate a building while moving through rooms, avoiding walls, using ramps to change floors, and selecting between two different exits on the ground floor.

A major requirement of this assignment was that all motion had to emerge from a scalar cost function. This means that I could not use hard-coded path planning, explicit collision handling as the main navigation system, or rule-based floor switching. Instead, the behavior of the agents had to come from the shape of the cost landscape and from gradient-based updates over time.

The simulation was visualized in 3D using the `vedo` library. I designed a building with three stacked floors, semi-transparent floor plates, interior walls, ramps between floors, and moving particles represented as spheres.

---
#2. Goals of the Assignment

The main goals of this project were:

1. To create a three-floor evacuation environment in 3D.
2. To simulate at least 20 agents starting from different floors.
3. To allow agents to evacuate using gradient descent on a total cost function.
4. To include two exits on the ground floor.
5. To use a soft-min formulation for exit selection instead of a hard minimum.
6. To design ramps as continuous geometric surfaces connecting floors.
7. To keep agents attached to the building surfaces so they do not fall through floors or ceilings.
8. To produce a notebook that reads like a report and not just a code dump.

---

#3. Theory and Mathematical Background

## 3.1 Cost-Function-Based Motion

The simulation is based on the idea that each agent moves in the direction that reduces a total cost function. If the position of an agent is written as

\[
p = (x, y, z),
\]

then its motion is determined by the gradient of a total cost:

\[
C(p) = C_{\text{goal}} + C_{\text{walls}} + C_{\text{height}} + C_{\text{smooth}} + C_{\text{repulsion}} + C_{\text{ramp-related}}.
\]

Each term serves a different purpose:
- \(C_{\text{goal}}\): pulls the agent toward exits or toward useful transition regions
- \(C_{\text{walls}}\): discourages entering walls or getting too close to boundaries
- \(C_{\text{height}}\): keeps the agent on a valid floor or ramp surface
- \(C_{\text{smooth}}\): reduces jitter and sudden jumps
- \(C_{\text{repulsion}}\): discourages agents from overlapping
- \(C_{\text{ramp-related}}\): helps agents find and move along ramps

The movement update is based on gradient descent with momentum:

\[
v_{k+1} = \beta v_k - \eta \nabla C(p_k),
\]

\[
p_{k+1} = p_k + v_{k+1}.
\]

Here, \(\beta\) is the momentum coefficient and \(\eta\) is the time-step or learning-rate-like parameter.

---

## 3.2 Soft-Min Exit Selection

A major requirement was that the two-exit decision could not be hard-coded. Instead, the exit choice had to emerge from a smooth cost term.

If the exits are located at \(e_1\) and \(e_2\), then the squared costs to each exit are:

\[
C_1(p) = \|p - e_1\|^2, \qquad C_2(p) = \|p - e_2\|^2.
\]

A hard minimum,

\[
\min(C_1, C_2),
\]

It is not ideal because it is non-smooth. So I used a soft-min form:

\[
C_{\text{goal}}(p) = -\tau \log\left(\exp\left(-\frac{C_1(p)}{\tau}\right) + \exp\left(-\frac{C_2(p)}{\tau}\right)\right).
\]

This makes the choice differentiable and allows agents to naturally favor the exit that is more attractive, while still being influenced by both exits in a smooth way.

---

## 3.3 Ramp Modeling

The ramps were modeled as continuous surfaces in 3D. Each ramp was defined by:

- two endpoints \(A\) and \(B\) in the 2D floor plan,
- a width \(r\),
- a lower floor height \(z_0\),
- an upper floor height \(z_1\).
For a point inside the ramp footprint, the height on the ramp is found by projecting the point onto the ramp centerline and linearly interpolating between \(z_0\) and \(z_1\).

This was important because floor transitions were not allowed to be rule-based. Agents had to move onto ramps because the cost function made that path favorable, not because of a hard-coded floor switch.

---

#4. Building Design

I designed a building with the following layout:

- **Ground floor** at \(z = 0\)
- **First floor** at \(z = H\)
- **Second floor** at \(z = 2H\)

The building uses line-segment walls to create interior structure. I kept the floors semi-transparent so that the motion of the agents could still be seen even when they were inside the building. I also left visible open space around the ramps so the vertical transitions were easier to inspect.

The simulation included:

- two exits on the ground floor,
- at least 20 agents,
- ramps connecting floors,
- moving spheres for agents,
- a 3D visualization using `vedo`.

---

#5.Methods

## 5.1 Agent Initialization

I initialized 20 agents with randomized starting positions across the three floors. I also added checks so that agents would not spawn:

- directly on walls,
- too close to each other,
- directly inside a ramp footprint.

This created a more realistic initial distribution.

---

## 5.2 Cost Terms Used

### Goal Cost
The goal term guided agents either to exits or toward useful ramp/waypoint regions depending on their floor and location.

### Wall Cost
I used wall penalties based on distance to line segments. This discouraged agents from crossing walls or hugging them too closely.

### Height Cost
The height cost was very important in the 3D setting. It kept agents attached to either:

- the flat floor height, or
- the local ramp surface height.
Without this term, agents could drift vertically in unrealistic ways.

### Smoothness Cost
A smoothness term was included to reduce jitter and unstable motion.

### Agent Repulsion
A repulsion term was used so that agents would not overlap too much when crowding near hallways, ramps, or exits.

### Ramp Guidance
Additional ramp-related logic was included through the cost design so that agents would move toward ramps when needed and continue making progress once on a ramp.

---

## 5.3 Optimization Method

The simulation used finite-difference gradient descent. For each agent, I approximated the gradient numerically by perturbing each coordinate and measuring the change in cost.

I also used:

- momentum smoothing,
- a maximum step cap,
- clamping to building bounds,
- collision checks against wall segments.

This helped make the simulation more stable.

---

#6. Experiments

To improve the simulation, I tested multiple design and parameter choices.

## Experiment 1: Exit Selection Behavior

First, I checked whether agents would use both exits rather than collapsing toward only one. Since the exit term was based on a soft-min rather than a hard-coded choice, agents were able to split more naturally depending on where they started.

### Observation
Both exits were used, which suggests that the soft-min formulation worked as intended.

### Conclusion
This experiment satisfied the assignment requirement that exit assignment not be hard-coded.

---
## Experiment 2: Ramp Geometry and Floor Transitions

One of the biggest challenges in the simulation was ramp behavior. At first, some agents got stuck near the ramp entrance or in the middle of the ramp. I found that the geometry of the ramp itself mattered a lot.

In particular, rotating one of the left ramps improved the flow significantly. Before that change, agents tend to bunch up or get trapped in awkward positions. After rotating the ramp, more agents were able to descend successfully.

### Observation
Ramp orientation strongly affected evacuation performance.

### Conclusion
Even though the motion comes from the cost function, the geometry of the environment still has a greater effect on the resulting trajectories.

---

## Experiment 3: Height-Adherence Weight

I also tested the effect of the height cost weight. If this weight is too small, agents can detach from the surface too easily or behave unrealistically near ramps. If it is too large, the optimization becomes stiff and the motion can become too constrained.

### Observation
A moderate-to-large height adhesion weight worked best because it kept agents on floors and ramps without freezing the motion.

### Conclusion
The height term is essential in a multi-floor environment and must be tuned carefully.

---

## Experiment 4: Stability and Smoothness

I tuned parameters such as:

- time step,
- momentum,
- smoothness weight,
- repulsion strength,
- ramp progress weight.

At earlier stages, I observed occasional instability, including some visible teleport-like motion in a few agents. After tuning the model, the overall behavior became much better, although the simulation still remained imperfect in a realistic multi-agent sense.

### Observation
Smaller step sizes and smoother updates improved stability.

### Conclusion
The stability of the simulation depends strongly on balancing the descent step against the wall, height, and crowd-repulsion terms.

---
#7.Results

The final version of the simulation evacuated **15 out of 20 agents** successfully.

This result improved after I adjusted the ramp orientation, especially on the left side of the building. Before that geometry change, fewer agents were able to transition floors effectively.

## Summary of final behavior

- Agents started from multiple floors.
- Agents used ramps to move downward.
- Both exits on the ground floor were used.
- The choice of exit emerged from the cost function.
- The simulation produced believable crowd motion in many cases.
- Some congestion and local trapping still occurred, which is expected in a complex multi-agent gradient-based system.

---

#8. Discussion

Overall, I think the simulation met the main goals of the assignment. The strongest parts of the project were:

- the 3D building visualization,
- the use of multiple floors and ramps,
- the soft-min exit selection,
- the cost-based multi-agent motion.

The most difficult part was making the ramps work well. In 2D, wall avoidance and goal attraction are already challenging, but in 3D the ramp adds another layer because agents have to remain on a valid surface while still moving through the environment naturally.

One important thing I learned is that even when the model is fully cost-based, geometry still matters a lot. Small changes to ramp orientation or placement can change how easy it is for agents to transition between floors. I also learned that stability is not only about the descent method itself, but also about the relative scaling of all cost terms.

Another important lesson was that soft-min is a good compromise for multi-target behavior. It makes the optimization smoother and avoids unnatural switching.

Although the final simulation was not perfect, I believe it demonstrates the intended ideas of the assignment: multi-agent evacuation, two-exit competition, ramp-based floor transitions, and surface-adhering motion driven by a differentiable cost framework.

---
#9.Direct Answers to Assignment Questions

## How were the two exits handled?
The two exits were handled through a soft-min goal cost. Instead of assigning each agent to a specific exit, I computed attraction to both exits and combined them using a differentiable soft-min. This allowed agents to split naturally based on position and local geometry.

##How were floor transitions handled?
Floor transitions were handled using ramps modeled as continuous geometric surfaces. The agent height was matched to the surface height of either the floor or the ramp, depending on the local position.

## How were agents prevented from going through walls?
I used wall penalty terms based on distance to wall segments. I also included wall crossing checks and clamped trial positions when necessary.

## How were agents prevented from falling through floors or ceilings?
This was done with the height-adherence cost, which penalized deviation from the valid surface height. The valid surface could be either a flat floor or a ramp surface.

## What experiments were performed?
I tested exit behavior, ramp geometry, stability parameters, and the height-adherence weight. The most important experimental improvement came from rotating a ramp so that agents could flow through it more effectively.

## Was the behavior fully hard-coded?
No. The main motion was generated from the total cost function and gradient descent. I did use practical stabilization steps such as clamping, bounded steps, and wall checks, but the evacuation behavior itself emerged from the cost design.

---

#10.Limitations

This simulation still has some limitations:

1. Some agents can become trapped in local minima.
2. Crowd congestion near ramps can slow evacuation.
3. The final evacuation was incomplete.
4. Parameter tuning had a strong effect on the quality of motion.
5. A few unstable artifacts appeared during development before tuning was improved.

These limitations are expected in a gradient-based multi-agent simulation and show that cost design is powerful but also sensitive.

---
#11. Conclusion

In this project, I created a three-floor evacuation simulation using cost-function minimization in 3D. The system included 20 agents, two exits, ramps between floors, wall avoidance, surface adhesion, and multi-agent interactions. The exit choice was handled through a soft-min formulation, and the building was visualized using `vedo`.

The final simulation evacuated 13 out of 20 agents and demonstrated the main concepts required by the assignment. The project showed how a relatively simple set of cost terms can produce complex group motion, while also highlighting how sensitive the results are to geometry and parameter tuning.

Overall, this assignment helped me better understand how optimization, geometry, and simulation interact in a multi-agent environment.

---

# 12. Suggested Figures to Insert in the Notebook

You should place images or screenshots in the notebook around this section.

## Figure ideas
1. A full 3D screenshot of the building.
2. A screenshot showing agents on different floors.
3. A screenshot of agents using both exits.
4. A screenshot focusing on one ramp transition.
5. A final frame showing evacuated vs. non-evacuated agents.

Example caption style:

**Figure 1.** Three-floor building visualization with semi-transparent floors, interior walls, and visible ramps.

**Figure 2.** Agents distributed across the ground, first, and second floors at the start of the simulation.

**Figure 3.** Mid-simulation frame showing agents selecting between the two ground-floor exits.

---
# 14. Code/video

The full simulation code is included in the notebook.

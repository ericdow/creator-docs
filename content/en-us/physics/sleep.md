---
title: Sleep System
description: The sleep system automatically prevents unnecessary simulation of non-moving parts.
---

Each [assembly](../physics/assemblies.md) in the Roblox Engine corresponds to a single **rigid body**. The position and velocity of each rigid body describe where it's located and how fast it's moving, and one of the primary tasks of the Engine is to update the positions and velocities of each assembly. Assemblies can be connected together with [mechanical constraints](../physics/mechanical-constraints.md) and [mover constraints](../physics/mover-constraints.md) to form **mechanisms**, such as cars or airplanes. 

As the number of assemblies and constraints in a mechanism increases, the time required to simulate the mechanism also increases. If we know that particular assembly is not moving, we can save time and improve performance by not simulating it. We can further improve performance by not simulating constraints between any non-moving assemblies if these constraints won't cause the attached assemblies to move. The **sleep system** is responsible for determining which assemblies and constraints should be simulated during each **worldstep**.

## Sleep States

Each assembly can be in one of three states:
1. **Awake**: assembly is moving or accelerating and is therefore currently being simulated.
2. **Sleeping**: assembly is not moving and not accelerating and therefore not being simulated.
3. **Sleep-checking**: assembly is not moving or accelerating and not being simulated, but shares a constraint with at least one awake assembly.

An assembly is determined to be not moving by checking the deviation of it's position. The deviation of the position is calculated as the maximum deviation from the average position of the point furthest from an assembly's center of mass over the most recent set of worldsteps. If this deviation is greater than the **displacement threshold**, this assembly is considered to be moving.

There are cases where just checking that an assembly is not moving would cause the assembly to be incorrectly put to sleep. For example, consider a ball thrown straight up. When the ball reaches it's maximum height, the position barely changes for a number of worldsteps, and would be considered non-moving and put into the sleeping state. Since the ball is no longer simulated, its position and velocity will never be updated, and it will never fall back down.

To handle such cases, the net force acting on the assembly is used to determine if it is accelerating. If the product of the assembly's acceleration and current timestep size is greater than the **velocity threshold**, the assembly is considered to be accelerating. Both the linear and angular velocities of each assembly are compared to the velocity threshold to determine if a body is accelerating.

Non-moving assemblies that share a constraint with at least one awake assembly are put into the sleep-checking state. These assemblies are not simulated. Each worldstep, they check the position deviation and acceleration of every awake assembly that they share a constraint with. If the position deviation of any neighboring assembly is greater than the **neighbor displacement threshold**, sleep-checking assemblies will be put into the awake state. Similarly, if the product of a neighboring assembly's acceleration and current timestep size is greater than the **neighbor velocity threshold**, the assembly will be put into the awake state.

<table>
<thead>
	<tr>
		<th>Threshold</th>
		<th>Value</th>
	</tr>
</thead>
<tbody>
	<tr>
		<td>Displacement</td>
		<td>0.001 studs</td>
	</tr>
	<tr>
		<td>Neighbor Displacement</td>
		<td>0.01 studs</td>
	</tr>
	<tr>
		<td>Linear Velocity</td>
		<td>0.1 studs/s</td>
	</tr>
	<tr>
		<td>Neighbor Linear Velocity</td>
		<td>0.2 studs/s</td>
	</tr>
	<tr>
		<td>Angular Velocity</td>
		<td>0.1 rad/s</td>
	</tr>
	<tr>
		<td>Neighbor Angular Velocity</td>
		<td>0.2 rad/s</td>
	</tr>
</tbody>
</table>

## Debugging Visualization

During testing, it may be useful to visualize sleeping, sleep-checking and awake assemblies. To enable this option, first ensure that the sleep system is enabled:

1. Open the Studio settings window (**File** &rarr; **Studio Settings**).
2. From the **Physics** tab, enable **Allow&nbsp;Sleep**.

   <img
   src="../assets/physics/sleep/Settings-Allow-Sleep.png"
   width="500" />

Then, enable the visualization:

1. Open the Studio settings window (**File** &rarr; **Studio Settings**).
2. From the **Physics** tab, enable **Are&nbsp;Awake&nbsp;Parts&nbsp;Highlighted**.

   <img
   src="../assets/physics/sleep/Settings-Are-Awake-Parts-Highlighted.png"
   width="500" />

Once enabled, simulated parts will be outlined by their current sleep state. Awake parts are outlined in red, sleep-checking parts are outlined in orange, and sleeping parts will not be outlined.

<figure>
  <video src=TODO controls width="100%"></video>
  <figcaption>Simulated parts outlined by the color representing their current sleep state</figcaption>
</figure>

## TODO
- Debugging tips
- Update link in https://create.roblox.com/docs/physics/adaptive-timestepping
- Center table
- Video for sleep state





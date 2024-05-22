---
title: Sleep System
description: The sleep system automatically prevents unnecessary simulation of non-moving parts.
---

Each [assembly](../physics/assemblies.md) in the Roblox Engine corresponds to a single **rigid body**. The position and velocity of each rigid body describe where it's located and how fast it's moving, and one of the primary tasks of the Engine is to update the positions and velocities of each assembly. Assemblies can be connected together with [mechanical constraints](../physics/mechanical-constraints.md) and [mover constraints](../physics/mover-constraints.md) to form **mechanisms**, such as cars or airplanes. 

As the number of assemblies and constraints in a mechanism increases, the time required to simulate the mechanism also increases. If we know that particular assembly is not moving, we can save time and improve performance by not simulating it. Moreover, we can further improve performance by not simulating constraints between any non-moving assemblies if these constraints won't cause the attached assemblies to move. The **sleep system** is responsible for determining which assemblies and constraints should be simulated during each **worldstep**.

## Sleep States

Each assembly can be in one of three states:
1. **Awake**: assembly is moving and is therefore currently being simulated.
2. **Sleeping**: assembly is not moving and therefore not being simulated.
3. **Sleep-checking**: assembly is not moving and not being simulated, but shares a constraint with at least one awake assembly.

Due to finite floating point and numerical precision, an assembly's velocity is unlikely to be exactly zero. To ensure that assemblies that are moving very slowly can be put to sleep, we introduce a **velocity threshold**. If an assembly's velocity is below this threshold, it is put into the **sleep** state. TODO history?

Non-moving assemblies that share a constraint with at least one awake assembly are put into the **sleep-checking** state. These assemblies are not simulated. Each frame, they check the velocity of every awake assembly that they share a constraint with, and are put into the **awake** state if this velocity is above the **neighbor velocity threshold**. TODO history?

<table size="small">
<thead>
	<tr>
		<th>Threshold</th>
		<th>Value</th>
	</tr>
</thead>
<tbody>
	<tr>
		<td>Velocity</td>
		<td>TODO studs/second</td>
	</tr>
	<tr>
		<td>Neighbor Velocity</td>
		<td>TODO studs/second</td>
	</tr>
  <tr>
		<td>Impulse</td>
		<td>TODO Rowtons [RMU stud/s&sup2;]</td>
	</tr>
  <tr>
		<td>Neighbor Impulse</td>
		<td>TODO Rowtons [RMU stud/s&sup2;]</td>
	</tr>
</tbody>
</table>

## Debugging Visualization

During testing, it may be useful to visualize sleeping, sleep-checking and awake assemblies. To enable this option:

1. Open the Studio settings window (**File** &rarr; **Studio Settings**).
2. From the **Physics** tab, enable **Are&nbsp;Awake&nbsp;Parts&nbsp;Highlighted**.

   <img
   src=TODO
   width="500" />

Once enabled, simulated parts will be outlined by their current sleep state. Awake parts are outlined in red, sleep-checking parts are outlined in orange, and sleeping parts will not be outlined.

<figure>
  <video src=TODO controls width="100%"></video>
  <figcaption>Simulated parts outlined by the color representing their current sleep state</figcaption>
</figure>

## TODO
- Velocity thresholds values: velocity threshold and neighbor velocity threshold
- Update link in https://create.roblox.com/docs/physics/adaptive-timestepping

Like other engines, Roblox had been using a traditional heuristic for determining when bodies fall asleep. Each assembly keeps a “history” of its past positions and orientations, and as long as the deviation from the average position stays within the threshold, the assembly stays awake. Essentially if the velocity is below a set value for a few frames, the assembly sleeps. I’ll refer to this value as the velocity threshold.

By trying to use this same logic for causing assemblies to wake up, we encounter a small problem. We can’t check the last impulse value of a sleeping assembly, since the whole point of something being asleep is that it’s not in the solver. For that, we still need to rely on contacts and property change events. However, the mechanism relationship can help in a variety of cases. In fact, an assembly’s history is also used to help determine when other assemblies in the same mechanism should wake up.






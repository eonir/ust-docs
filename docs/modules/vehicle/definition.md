# Vehicle Definition

Part of [Vehicle](index.md).

## Vehicle Definition

*Create > UST > Vehicle > Vehicle Definition*. Several prefabs can share one.

| Field | Default | Meaning |
|---|---|---|
| Display Name, Icon, Description | | For your UI and the prompts |
| **Engine** | | |
| Engine Power | 90 kW | How fast the vehicle gets to speed. 75 for a family car, 250 for a sports car |
| Max Speed, Max Reverse Speed | 140, 25 km/h | |
| Brake Strength | 1 g | Braking deceleration |
| Coast Deceleration | 1.2 m/s² | Slowing down with no pedal pressed |
| Drive Type | Front Wheel | Front, Rear or All Wheel |
| **Steering** | | |
| Max Steer Angle | 32° | At low speed |
| Steer Angle At Max Speed | 10° | Lower is more stable at high speed |
| Steer Speed | 160°/s | How fast the wheels turn |
| **Grip** | | |
| Grip | 1 | Tire friction, as a multiple of the weight on the tire. 0.7 heavy truck, 1.3 racing tires |
| Side Stiffness | 14 | How fast a tire kills its sideways slide. Higher feels planted, lower feels loose |
| Handbrake Grip | 1 | Side grip of the rear wheels with the handbrake. 1: the handbrake only brakes. Lower lets the rear slide: 0.7 for a sports car, 0.4 for an arcade drift |
| Anti Roll | 0.4 | Leans less in turns. 0: off |
| **Body** | | |
| Mass | 1300 kg | |
| Center Of Mass | (0, 0.3, 0.1) | From the pivot, on the ground under vehicles made by the tools. Lower is harder to roll over |
| Auto Flip Delay | 3 s | Puts the vehicle back on its wheels after this long upside down and nearly stopped. 0: never |
| **Wheels and suspension** | | |
| Wheel Radius | 0.36 m | |
| Suspension Length | 0.3 m | Travel |
| Suspension Stiffness | 1 | 1 rests at half travel. Higher sits higher and feels firmer |
| Suspension Damping | 0.45 | Fraction of critical damping. 0.3 bounces a little, 1 never bounces |
| Ground Layers | Everything | Layers the wheels drive on |
| **Fuel** | | |
| Fuel Capacity | 50 L | 0: no fuel needed |
| Start Fuel | 1 | Fraction of the tank for a new vehicle |
| Idle Consumption, Full Consumption | 0.1, 1.5 L/min | Engine running and standing still, at top speed |
| **Health** | | |
| Max Health | 500 | |
| Resistances | | By damage type, see [Damage types](../combat/index.md#damage-types) |
| Default Hit Damage | 5 | Damage from a melee hit by an item without damage of its own |
| Damage From Players | off | Weapons and tools of players damage it. Repairs always work |
| Damage From Enemies | on | Enemies damage it, for example when it blocks their way |
| **Crashes** | | |
| Crash Safe Speed | 18 km/h | Impact speed below which a crash does nothing |
| Crash Damage Per Kmh | 3 | Damage to the vehicle per km/h above the safe speed |
| Ramming Damage Per Kmh | 1.5 | Damage to an enemy or a player run over, per km/h. 0: none |
| Ramming Min Speed | 12 km/h | |
| Ramming Damage Type | | Empty: untyped |
| **Camera** | | |
| Camera Distance | 7 m | Third person camera distance while in the vehicle |

The settings are scaled by the mass, so a stiffness of 1 or a side stiffness of 14 feel the same on a kart and on a
truck: change the mass without retuning the rest.

### The three presets

| | Sport Car | Family Car | Truck |
|---|---|---|---|
| Power, top speed | 260 kW, 220 km/h | 95 kW, 160 km/h | 220 kW, 110 km/h |
| Drive | Rear | Front | All |
| Mass | 1250 kg | 1400 kg | 4500 kg |
| Grip, side stiffness | 1.25, 18 | 1, 14 | 0.85, 10 |
| Suspension | 0.2 m, firm | 0.3 m | 0.4 m |
| Seats, trunk slots | 2, 8 | 4, 16 | 2, 40 |
| Fuel tank | 60 L | 50 L | 120 L |
| Health | 400 | 500 | 900 |

### Tuning tips

| Feels | Change |
|---|---|
| Slow to pick up speed | Engine Power up |
| Spins the wheels at launch, slides out of turns | Grip up, or Engine Power down on a rear drive |
| Floaty, drifts in every turn | Side Stiffness up |
| Too twitchy at speed | Steer Angle At Max Speed down |
| Rolls over in turns | Center Of Mass Y down, Anti Roll up |
| Bounces after a bump | Suspension Damping up |
| Body hits the ground | Suspension Stiffness up, or Suspension Length up |
| Stops dead when releasing the pedal | Coast Deceleration down |
| Handbrake spins the car | Handbrake Grip up, 1 to only brake |

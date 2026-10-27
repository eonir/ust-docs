# Add a vital

Example: a *Temperature* vital that kills the player when it reaches zero.

1. *Create > UST > Vitals > Vital Definition*, name it Temperature.
2. Authority **Server**, Max 100, Start 100.
3. Decay 1 every 10 s.
4. Depletion Effects: Target Health, Drain Amount 1, Interval 2.
5. Add it to your `VitalsProfile`.

No code needed. To make it drop faster in the cold, call `SetDecayMultiplier(temperature, this, 3f)` on the server
when the player enters a cold zone, and `RemoveDecayMultiplier(temperature, this)` when he leaves.

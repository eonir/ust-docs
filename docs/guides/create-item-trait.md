# Create an item trait

A trait adds data to an item. Your systems then look for that trait.

```csharp
using System;
using UST.Inventory;

[Serializable, ItemTraitMenu("Survival/Food")]
public class FoodTrait : ItemTrait
{
    public float hunger = 25f;
    public float thirst;
}
```

The trait appears in **Add Trait** on every item. Read it anywhere:

```csharp
if (item.TryGetTrait(out FoodTrait food))
{
    vitals.Modify(hungerVital, food.hunger);
}
```

!!! warning "Renaming a trait class"
    Traits are stored with `[SerializeReference]`. If you rename or move the class, add
    `[UnityEngine.Scripting.APIUpdating.MovedFrom]` on it, or existing items lose the trait.

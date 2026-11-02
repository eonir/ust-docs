# Create an item trait

A trait adds data to an item. Your systems then look for that trait.

```csharp
using System;
using System.Collections.Generic;
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

## Warnings in the Item Database window

Override `CollectIssues` to report what is wrong with the trait. Each message shows as a warning on the item in the
Item Database window, and the item is counted with the items with issues.

```csharp
public override void CollectIssues(ItemDefinition owner, ItemDatabase database, List<string> issues)
{
    if (hunger <= 0f && thirst <= 0f) issues.Add("Food that feeds nothing.");
}
```

`database` is the database selected in the window, or null.

!!! warning "Renaming a trait class"
    Traits are stored with `[SerializeReference]`. If you rename or move the class, add
    `[UnityEngine.Scripting.APIUpdating.MovedFrom]` on it, or existing items lose the trait.

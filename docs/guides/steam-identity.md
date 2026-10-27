# Use Steam ids for saves

Implement `IPlayerIdentityProvider` and assign the component to **Identity Provider** on the `SaveManager`.

Example with Steamworks.NET:

```csharp
using Steamworks;
using UnityEngine;
using UST.Core;

public class SteamIdentityProvider : MonoBehaviour, IPlayerIdentityProvider
{
    public string GetLocalPlayerId() => SteamUser.GetSteamID().ToString();
}
```

!!! warning "Trust"
    The id is sent by the client. A modified client could send someone else's id. For public servers, verify the
    id on the server with Steam authentication tickets before loading the save.

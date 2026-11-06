# Shop

The Shop module adds traders: characters, stalls or machines players buy from and sell to. A price is a list of items,
so a coin item makes a currency and any other item makes a barter: 50 coins for an axe, or 3 hides for an axe. Each
trader has its own catalog and its own stock, shared by every player and saved with the world.

The module is optional: add `UST_DISABLE_SHOP` to the Scripting Define Symbols to remove it, see
[Modules](../architecture/modules.md#disabling-an-optional-module).

## Setup

1. Create a catalog: *Create > UST > Shop > Shop Catalog*, and fill its offers.
2. Run *right click in the Hierarchy > UST > Trader* for a test trader, or add a `NetworkObject` and a `Trader` to your own
   character or object. It needs a collider so that the interaction finds it.
3. Assign the catalog and the item database to the trader.
4. Run *UST Tools > Advanced > Shop > Add Shop Window*. *Create Game UI* adds it too.

For a currency, create an item such as **Coin**, stackable, with a small weight or none, and use it in the prices.

## Shop Catalog

| Field | Meaning |
|---|---|
| Display Name | Name of the trader in the window, unless the trader has its own |
| Greeting | Line under the name |
| Buys | What the trader sells |
| Sells | What the trader buys from players |

**Buy offer**

| Field | Default | Meaning |
|---|---|---|
| Item, Count | | What the player gets for one purchase |
| Price | | Items the player pays for one purchase |
| Stock | -1 | Purchases available before the trader runs out. -1: never runs out. |
| Restock Minutes | 10 | Minutes for one unit to come back, up to the stock. 0: never restocks. Sleeping through the night restocks too. |

**Sell offer**

| Field | Meaning |
|---|---|
| Item, Count | What the player gives for one sale |
| Reward | Items the player gets for one sale |

Several traders can share a catalog: each keeps its own stock.

!!! warning "Set the stock of every new offer"
    An offer added with the **+** of the inspector starts with a **Stock** of 0: the trader is out of stock and the Buy
    button stays greyed out. Set it to **-1** for an offer that never runs out, or to the number of purchases
    available.

## Trader

| Field | Default | Meaning |
|---|---|---|
| Catalog, Database | | The offers, and the item database shared by every peer |
| Display Name | | Empty: the name of the catalog |
| Use Range | 3 | Distance to trade, checked on the server |
| Prompt Format | Trade with {0} | Interaction text |
| Max Per Request | 99 | Most purchases or sales one request can ask for |
| Messages | | Too far, out of stock, cannot pay, missing items |

The server checks the distance, the stock and the price before every trade. What does not fit in the inventory is
dropped at the feet of the player. The stock of traders placed in the scene is saved with the world.

## Shop Window

Buy and Sell tabs, the offers with their price and the stock left, and for the selected offer: what the player owns
of each price item (green when enough, red when not), a quantity (Shift for steps of 10, Max) and the Buy or Sell
button. Offers the player cannot afford are faded. Escape, B on a gamepad, the X button or walking away closes it.

## Scripting

```csharp
trader.RequestBuy(offerIndex, 5);                 // local player
trader.RequestSell(offerIndex, 1);
string refusal = trader.ServerBuy(clientId, offerIndex, 1);   // server: null on success
trader.ServerRestockAll();
int left = trader.StockOf(offerIndex);            // -1: never runs out
Trader.OpenRequested += t => MyShopUI.Open(t);    // your own window instead of ShopWindow
```

## Not in this version

- Prices that change with supply and demand.
- An invisible wallet: the currency is an item that takes inventory room, as in Valheim.

[README.md](https://github.com/user-attachments/files/34245866/README.md)
# HideoutAutoBuy

One-key auto-buy for Hideout construction — press a hotkey and your missing building materials are purchased from the Flea Market at the lowest available price, then construction starts automatically.

**Compatibility:** SPT `4.1.5` (EFT `0.16.9`)

---

## ✨ Features

- 🔑 **One-key auto-buy** — Select a hideout area and press `F9`: the mod calculates exactly which materials are missing, finds the cheapest Flea Market offers for them and buys them with your Roubles.
- 🏗️ **Auto-start construction** — Once the materials are in your stash, construction/upgrade begins automatically. No manual clicking, no re-pressing the hotkey.
- 🔓 **No more lock-outs** — Locked areas can be built at any time. Area-prerequisite, trader-loyalty and skill-level conditions are removed; only the materials are required.
- ✅ **No more found-in-raid grind** — Hideout construction no longer requires "found in raid" items. Items bought from the Flea Market (or obtained any other way) can be used directly.
- 🧠 **Smart market logic** — Offers are filtered (your own listings, locked offers and buy-limit-exhausted offers are excluded) and sorted by price, so you always buy the cheapest first.
- 🖥️ **BepInEx config panel** — Press `F12` in-game to rebind the hotkey, toggle auto-start, toggle notifications and set a maximum auto-spend limit. Changes apply instantly.
- 🛡️ **Read-only server** — The server mod only answers price quotes and patches hideout data in memory. All purchases and construction go through the game's own native requests — your profile stays consistent.

---

## 📦 Installation

1. Make sure **SPT 4.1.5** is installed and closed.
2. Extract the archive into your SPT game folder, so that:
   - `SPT_Runtime\user\mods\HideoutAutoBuy\HideoutAutoBuy.dll`
   - `BepInEx\plugins\HideoutAutoBuy.Client.dll`
   end up in their respective locations.
3. Start the SPT server, then the game.

> Both files are required: the **client plugin** triggers purchases and construction, the **server mod** provides flea-market price quotes and removes build restrictions.

---

## 🎮 Usage

1. Enter your **Hideout**.
2. Click an area you want to build or upgrade (locked areas work too).
3. Press `F9` (or your configured hotkey).
4. Done — the mod:
   - computes the missing materials for the next level,
   - quotes the cheapest Flea Market offers,
   - buys everything automatically,
   - unlocks the area and starts construction.

Notifications keep you informed: buying progress, total cost, "Insufficient funds!" and any items that cannot be bought.

### ⚙️ Configuration (F12 in-game)

| Option | Default | Description |
|---|---|---|
| Hotkey | `F9` | Click to rebind |
| Auto-start construction | On | Turn off to only buy materials |
| Show notifications | On | Turn off to keep results in the log only |
| Max auto-spend | 0 (unlimited) | Cancel the purchase if the quote exceeds this amount |

Config file: `BepInEx\config\com.ds.hideoutautobuy.client.cfg`

---

## 🔧 How it works

- **Client plugin** (`BepInEx`): reads the selected area's requirements (`Stage.Requirements`), asks the server for a purchase plan, buys every offer via the game's native `RagfairPurchaseOffers()` call (stash updates through the official pipeline), then unlocks the area status and calls the native `AreaData.UpgradeAction()`.
- **Server mod** (`SPT_Runtime/user/mods`): registers a `POST /client/hideout/autobuy/quote` route that picks the cheapest valid offers via `RagfairOfferHolder`, and on startup:
  - removes `Area` / `TraderLoyalty` / `Skill` build conditions from the hideout database,
  - removes the `isSpawnedInSession` (found-in-raid) flag from all construction requirements.

---

## 🛠️ Building from source

No .NET SDK required — the project builds with the in-box `csc.exe` plus assemblies shipped with SPT:

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force
& ".\build-hideout-autobuy.ps1" -Server -Client -GameDir "D:\YourSPTFolder"
```

Output lands in `build\`. The script also patches the server assembly with Mono.Cecil so the `IModMetadata` record-style interface is implemented correctly by the legacy compiler.

---

## 📄 License

MIT

## ⚠️ Disclaimer

This is a mod for **SPT (Single Player Tarkov)**. It is not affiliated with, or endorsed by, Battlestate Games. Do not use it with the live game — mods are for offline/single-player use only.


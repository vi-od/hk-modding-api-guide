# Comprehensive Guide to the Hollow Knight Modding API

Welcome to the comprehensive guide for the [Hollow Knight Modding API](https://github.com/hk-modding/api). This document aims to provide a thorough understanding of how to create mods for the beloved game [Hollow Knight](https://www.hollowknight.com/), leveraging the powerful tools and framework provided by the modding community. Whether you are a complete beginner curious about modifying the game or an experienced developer looking to delve deeper into Hollow Knight's internals, this guide will walk you through the essential concepts, tools, and techniques.

Hollow Knight, developed by [Team Cherry](https://www.teamcherry.com.au/), is renowned for its beautiful art, challenging gameplay, and intricate world. Built on the [Unity engine](https://unity.com/) (specifically version 2020.2.2f1 for game patch 1.5.x.x onwards) and utilizing C# with the [.NET Framework 4.7.2](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472), the game offers a solid foundation that the modding community has expertly built upon. Mods in Hollow Knight are essentially Dynamic-Link Libraries (.dll files) that inject custom code and logic into the game, allowing for a vast range of modifications – from simple quality-of-life tweaks and cosmetic changes to entirely new gameplay mechanics, enemies, and areas.

The backbone of this vibrant modding scene is the [Hollow Knight Modding API](https://github.com/hk-modding/api), often referred to simply as the Mod API or Modding API. Maintained by the dedicated [`hk-modding` group](https://github.com/hk-modding) and contributors, this API acts as a crucial loader and intermediary layer. It utilizes the sophisticated [`MonoMod` library](https://github.com/MonoMod/MonoMod), a general-purpose .NET assembly patching framework, to safely and efficiently hook into the game's code at runtime. This approach allows mods to interact with and alter game behavior without modifying the original game files directly, ensuring better compatibility and stability.

This guide will cover everything from setting up your development environment and creating your first simple mod to exploring advanced topics like hooking game methods, managing mod data, creating custom menus, and interacting with complex game systems like [PlayMaker FSMs](https://hutonggames.com/). We will also discuss essential tools for development and debugging, best practices for creating robust and compatible mods, and the process for distributing your creations to the wider Hollow Knight community. Prepare to embark on a journey into the heart of Hallownest's code and unleash your creativity!

## Setting Up Your Development Environment

Before you can start crafting your own modifications for Hollow Knight, you need to set up a proper development environment. This involves installing the necessary software, obtaining the game files, and configuring your tools. This section will guide you through the essential steps to get you ready for mod development.

### Essential Software

1.  **[Hollow Knight](https://www.hollowknight.com/)**: Naturally, you need a legitimate copy of Hollow Knight installed on your computer. The modding tools and API are designed to work with the latest version available on platforms like [Steam](https://store.steampowered.com/app/367520/Hollow_Knight/) or [GOG](https://www.gog.com/en/game/hollow_knight). Ensure your game is up-to-date.
2.  **.NET Development Environment**: Since Hollow Knight mods are written in C#, you need the .NET Framework SDK. Specifically, the Modding API targets [.NET Framework 4.7.2](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472). 
    *   **Windows**: You should install the [.NET Framework 4.7.2 Developer Pack](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net472) from Microsoft's official website. This provides the necessary libraries and tools for building mods targeting this framework, even if you use an IDE other than Visual Studio.
    *   **macOS/Linux**: The .NET Framework 4.7.2 Developer Pack is Windows-only. However, Hollow Knight on these platforms runs using the Mono runtime, which is compatible with .NET Framework assemblies. Your development environment (like Visual Studio for Mac, Rider, or VS Code with extensions) should be configured to target `.NET Framework 4.7.2` when building your C# Class Library project. The necessary reference assemblies are typically included with compatible IDEs or can be managed through them. You do not install the Developer Pack directly on macOS or Linux.
3.  **Integrated Development Environment (IDE)**: While you *could* write C# code in a simple text editor, using an IDE significantly enhances productivity. The most common choices for Hollow Knight modding are:
    *   **[Microsoft Visual Studio](https://visualstudio.microsoft.com/vs/community/)**: The Community Edition is free for individual developers and offers a powerful, feature-rich environment for C# development, including excellent debugging tools, IntelliSense (code completion), and project management features. This is often the recommended choice for beginners.
    *   **[JetBrains Rider](https://www.jetbrains.com/rider/)**: A cross-platform .NET IDE known for its intelligent code analysis, refactoring tools, and seamless integration with Unity. It's a paid product, but offers a free trial and may be preferred by developers already familiar with JetBrains tools.
4.  **Decompiler**: Understanding how the original game code works is crucial for effective modding. You'll need a decompiler to inspect the game's compiled assemblies (`.dll` files). The highly recommended tool is:
    *   **[dnSpy-Unity](https://github.com/dnSpyEx/dnSpy)**: A specific fork of the popular dnSpy decompiler, tailored for Unity games. It allows you to browse the game's C# code, view IL (Intermediate Language), set breakpoints, and even debug the game live. Alternatives like [ILSpy](https://github.com/icsharpcode/ILSpy) exist, but dnSpy-Unity is generally preferred within the community.

### Getting the Game Files (Platform Specific)

You need access to the game's core files, particularly the managed assemblies where the game logic resides. Locate your Hollow Knight installation directory. The exact path varies depending on your operating system and how you installed the game ([Steam](https://store.steampowered.com/app/367520/Hollow_Knight/) or [GOG](https://www.gog.com/en/game/hollow_knight)). Common locations for the crucial `Managed` folder are:

*   **Windows**:
    *   **Steam**: `C:/Program Files (x86)/Steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/`
    *   **GOG**: `C:/Program Files (x86)/GOG Galaxy/Games/Hollow Knight/hollow_knight_Data/Managed/` (or similar GOG installation path)
*   **macOS**:
    *   **Steam**: `~/Library/Application Support/Steam/steamapps/common/Hollow Knight/hollow_knight.app/Contents/Resources/Data/Managed/`
    *   **GOG**: `/Applications/Hollow Knight.app/Contents/Resources/Data/Managed/` (or similar GOG installation path)
*   **Linux**:
    *   **Steam**: `~/.steam/steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/` (or `~/.local/share/Steam/...` depending on installation)
    *   **GOG**: Check your GOG installation directory, typically something like `~/GOG Games/Hollow Knight/game/hollow_knight_Data/Managed/`

**Note:** You may need to enable viewing hidden files/folders (like `Library` on macOS or dot-folders on Linux) to navigate to these paths.

The most important file here is `Assembly-CSharp.dll`. This contains the bulk of the game's code. You will frequently decompile this file using tools like [dnSpy-Unity](https://github.com/dnSpyEx/dnSpy) or [ILSpy](https://github.com/icsharpcode/ILSpy) to understand how game systems work and identify methods to hook into.

### Setting Up a Mod Project

While you can manually create project files, the recommended approach is to use mod templates, which provide a basic project structure and necessary references:

1.  **[Hollow Knight Mod Templates (NuGet)](https://marketplace.visualstudio.com/items?itemName=BadMagic.HKModdingExtension)**: The community provides NuGet packages containing project templates for Visual Studio. Search for `HollowKnight.Modding` templates in the [Visual Studio Marketplace](https://marketplace.visualstudio.com/) or via the [NuGet Package Manager](https://learn.microsoft.com/en-us/nuget/consume-packages/install-use-packages-visual-studio) within Visual Studio. Installing these templates allows you to create a new Hollow Knight Mod project (`File -> New -> Project`) with the basic structure, necessary references (like the Modding API itself), and a basic `Mod` class implementation already set up.
2.  **Manual Setup (Alternative)**: If not using templates, you would create a new C# Class Library project targeting .NET Framework 4.7.2. You would then need to manually add references to:
    *   `Assembly-CSharp.dll` (from the game's `Managed` folder)
    *   `UnityEngine.CoreModule.dll` (and potentially other `UnityEngine` modules from the `Managed` folder)
    *   `HollowKnight.Modding.API.dll` (the [Modding API](https://github.com/hk-modding/api) itself, obtained from a release or built from source)
    *   `0Harmony.dll` ([MonoMod/Harmony library](https://github.com/MonoMod/MonoMod), usually bundled with the API)

Using the templates is strongly recommended as it simplifies this setup process considerably.

### Installing the [Modding API](https://github.com/hk-modding/api) (for Testing)

To test your mods, you need the Modding API installed in your game. While developers often build the API from source (as described in the [hk-modding/api GitHub repository](https://github.com/hk-modding/api)), users and testers typically use a mod installer:

*   **[Scarab](https://github.com/fifty-six/Scarab)**: A popular, community-developed mod installer for Hollow Knight. It handles downloading, installing, and managing the Modding API and various mods.
*   **[Lumafly](https://themulhima.github.io/Lumafly/)**: Another mod installer option.

Download and run one of these installers. They will typically auto-detect your game installation and allow you to install the Modding API with a single click. This ensures your game is ready to load the mods you develop.

With your IDE installed, the .NET Framework ready, a decompiler at hand, and a project structure set up (preferably using templates), you are now equipped to start writing your first Hollow Knight mod. The next sections will delve into the core concepts of the Modding API itself.

## Core Concepts of the Modding API

Now that your development environment is set up, it's time to dive into the fundamental concepts that underpin Hollow Knight modding using the API. Understanding these core elements is essential for building functional and well-behaved mods.

### The `Mod` Base Class: Your Mod's Foundation

At the heart of nearly every Hollow Knight mod lies a class that inherits from the abstract `Mod` class, found within the `HollowKnight.Modding` namespace. This base class provides the essential structure and entry points that the Modding API uses to load, initialize, and manage your mod. Think of it as the main blueprint for your modification.

To create your mod, you'll define a public class that inherits from `Mod`. For example:

```csharp
using HollowKnight.Modding;
using System.Collections.Generic;
using UnityEngine;

namespace MyFirstMod
{
    // Inherit from the Mod class
    public class MyFirstMod : Mod
    {
        // Internal reference to the mod instance
        internal static MyFirstMod Instance;

        // Constructor (optional, but can be useful)
        public MyFirstMod()
        {
            // Mod constructor logic, if any
        }

        // Mandatory: Return the mod's name
        public override string GetName() => "My First Mod";

        // Mandatory: Return the mod's version
        public override string GetVersion() => "1.0.0.0";

        // Optional: Specify assets to preload
        public override List<(string, string)> GetPreloadNames()
        {
            return new List<(string, string)>
            {
                // Example: Preload the "Crawler" enemy prefab from the "Crossroads_01" scene
                // ("SceneName/Path/To/GameObject", "ExpectedNameInDictionary")
                // ("Crossroads_01/Crawler", "Enemy_Crawler") 
            };
        }

        // Main initialization logic
        public override void Initialize(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)
        {
            Log("Initializing My First Mod");

            // Store instance for easy access
            Instance = this;

            // Access preloaded objects if needed
            // GameObject crawlerPrefab = preloadedObjects["Crossroads_01"]["Enemy_Crawler"];

            // Hook into game events, apply patches, etc.
            ModHooks.HeroUpdateHook += OnHeroUpdate;

            Log("My First Mod Initialized");
        }

        // Optional: Cleanup logic when the mod is disabled/unloaded
        public override void Unload()
        {
            Log("Unloading My First Mod");

            // Unhook events to prevent errors
            ModHooks.HeroUpdateHook -= OnHeroUpdate;

            Log("My First Mod Unloaded");
        }

        // Example hook method
        private void OnHeroUpdate()
        {
            // Code to run every frame the hero updates
            // LogDebug("Hero Updated!"); // Use LogDebug for frequent messages
        }

        // Optional: Control load order (higher numbers load later)
        public override int LoadPriority() => 10; 
    }
}
```

Key methods and properties you'll frequently interact with in your `Mod`-derived class include:

*   **`GetName()` (Abstract, Mandatory)**: Must return a `string` representing the user-facing name of your mod.
*   **`GetVersion()` (Abstract, Mandatory)**: Must return a `string` representing the mod's version. Semantic versioning (e.g., 
`1.0.0.0`) is highly recommended.
*   **`Initialize(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)` (Override, Mandatory Logic)**: This is the primary entry point where your mod's setup logic resides. It's called by the API after the mod is loaded. The `preloadedObjects` dictionary contains any game assets you requested via `GetPreloadNames()`, organized by scene name and then asset name. This is where you typically subscribe to events (hooks), patch methods, load settings, and prepare your mod to run.
*   **`Unload()` (Override, Optional but Recommended)**: Called when the mod is disabled by the user or when the game is shutting down. It's crucial to perform cleanup here, especially unsubscribing from any events or hooks you registered in `Initialize`. Failing to unhook can lead to errors or memory leaks if the mod is disabled and re-enabled or if the game continues running after unloading.
*   **`GetPreloadNames()` (Override, Optional)**: If your mod needs frequent access to specific game assets (like enemy prefabs, particle effects, or UI elements), you can specify them here. Return a `List<(string scene, string asset)>` where `scene` is the name of the scene containing the asset and `asset` is the path to the GameObject within that scene's hierarchy (e.g., `"Cliffs_01/Grub"`). The API will load these assets before calling `Initialize`, making them readily available in the `preloadedObjects` dictionary. This is more efficient than searching for objects manually during gameplay.
*   **`LoadPriority()` (Override, Optional)**: Returns an `int` that determines the loading order relative to other mods. Mods with lower numbers load first. The default priority is typically 0. You might increase this if your mod depends on another mod being fully initialized first, or decrease it if other mods might depend on yours.
*   **Logging Methods (`Log`, `LogDebug`, `LogError`, `LogFine`)**: The `Mod` base class provides these convenient instance methods for writing messages to the `ModLog.txt` file. Use `Log` for general information, `LogError` for errors, `LogDebug` for verbose messages helpful during development (often disabled in release builds), and `LogFine` for even more detailed tracing.

### The Mod Lifecycle: From Installation to Unloading

Understanding the lifecycle of a mod helps in structuring your code correctly:

1.  **Installation**: The user places the mod's `.dll` file (and any associated assets or dependency `.dll`s) into the `Mods` directory within the game's `Managed` folder, usually via a mod installer like [Scarab](https://github.com/fifty-six/Scarab) or [Lumafly](https://themulhima.github.io/Lumafly/). The exact path to the `Mods` folder is typically:
    *   **Windows**: `C:/Program Files (x86)/Steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/Mods/` (or similar GOG path)
    *   **macOS**: `~/Library/Application Support/Steam/steamapps/common/Hollow Knight/hollow_knight.app/Contents/Resources/Data/Managed/Mods/` (or similar GOG path)
    *   **Linux**: `~/.steam/steam/steamapps/common/Hollow Knight/hollow_knight_Data/Managed/Mods/` (or `~/.local/share/Steam/...` or similar GOG path)
2.  **API Initialization**: When the game starts, the Modding API itself initializes.
3.  **Mod Discovery**: The API scans the `Mods` folder for valid `.dll` files containing classes that inherit from `Mod`.
4.  **Dependency Resolution & Sorting**: The API checks dependencies listed in mod manifests (if available) and sorts mods based on `LoadPriority()`.
5.  **Preloading**: For each mod, the API calls `GetPreloadNames()` and loads the requested assets.
6.  **Instantiation**: The API creates an instance of the mod's main class (the one inheriting from `Mod`).
7.  **Initialization**: The API calls the `Initialize()` method on the mod instance, passing the preloaded objects.
8.  **Runtime**: The mod is now active. Its code runs in response to game events triggered via hooks or other mechanisms set up during initialization.
9.  **Unloading**: If the user disables the mod via a menu or quits the game, the API calls the `Unload()` method on the mod instance, allowing for cleanup.

### Hooking into the Game: Modifying Behavior

Hooks are the primary mechanism for mods to interact with and modify the game's behavior. They allow you to execute your custom code when specific events occur or when certain game methods are called.

*   **`ModHooks`**: This static class (`HollowKnight.Modding.ModHooks`) is the first place to look for common hooking needs. It exposes numerous C# events corresponding to high-level game actions. You subscribe your methods to these events in `Initialize` and unsubscribe in `Unload`. Examples include:
    *   `ModHooks.HeroUpdateHook += OnHeroUpdate;` // Runs every frame the player updates.
    *   `ModHooks.TakeDamageHook += OnTakeDamage;` // Runs when an entity takes damage, allows modifying damage.
    *   `ModHooks.LanguageGetHook += OnLanguageGet;` // Allows changing translated text.
    *   `ModHooks.SavegameLoadHook += OnSaveGameLoad;` // Runs after a save is loaded.
    *   `ModHooks.NewGameHook += OnNewGame;` // Runs when a new game starts.
    *   `ModHooks.BeforeSceneLoadHook += OnBeforeSceneLoad;` // Runs just before a scene starts loading.
    *   `ModHooks.AfterSceneLoadHook += OnAfterSceneLoad;` // Runs after a scene has finished loading.

    Using `ModHooks` is generally safer and easier than manual patching when an appropriate hook exists.

*   **`On.` Hooks (MonoMod/Harmony)**: For more granular control or hooking methods not covered by `ModHooks`, you can use `On.` hooks provided by the underlying MonoMod/Harmony library. This allows you to intercept calls to almost *any* method in the game's code (or even other mods). You need to know the exact namespace, class, and method signature.

    ```csharp
    // Example: Hooking the 'Update' method of the 'Crawler' enemy class
    On.Crawler.Update += Crawler_Update_Hook;

    // ... in Unload():
    On.Crawler.Update -= Crawler_Update_Hook;

    // The hook method itself:
    private void Crawler_Update_Hook(On.Crawler.orig_Update orig, Crawler self)
    {
        // 'orig' is a delegate to call the original method
        // 'self' is the instance of the Crawler class being updated

        // Add custom logic before the original method runs:
        Log("Crawler is about to update!");

        // Call the original method (important!)
        orig(self);

        // Add custom logic after the original method runs:
        if (self.currentState == Crawler.CrawlerState.Turning)
        {
            Log("Crawler is turning!");
        }
    }
    ```
    Finding the correct method signatures requires using a decompiler like dnSpy.

*   **IL Hooks (MonoMod/Harmony)**: The most advanced hooking technique involves directly manipulating the Intermediate Language (IL) instructions of a method. This offers ultimate flexibility, allowing you to insert, remove, or modify instructions within a method's body. It's powerful but complex and requires a good understanding of CIL (Common Intermediate Language). IL hooks are typically used for optimizations or modifications that are difficult or impossible with `On.` hooks.

    ```csharp
    // Example (Conceptual - IL manipulation is complex)
    IL.HeroController.Attack += ModifyAttackLogic;

    // ... in Unload():
    IL.HeroController.Attack -= ModifyAttackLogic;

    private void ModifyAttackLogic(ILContext il)
    {
        ILCursor cursor = new ILCursor(il);
        // Use the cursor to find specific instructions and modify them
        // e.g., find where damage is calculated and multiply it
    }
    ```

Choosing the right hooking method depends on the task. Start with `ModHooks`, use `On.` hooks for more specific method interception, and reserve IL hooks for advanced scenarios.

### Logging: Your Debugging Compass

Effective logging is indispensable during mod development and for diagnosing issues reported by users. The Modding API provides a straightforward logging system.

*   **How it Works**: The API directs log messages to a file named `ModLog.txt`, located in the game's data directory (e.g., `Hollow Knight_Data/Managed/Mods/ModLog.txt` or sometimes near the save files depending on the installer/API version). This file accumulates logs from the API itself and all loaded mods.
*   **Using Logger Methods**: As mentioned earlier, your `Mod`-derived class inherits logging methods:
    *   `Log(object message)`: For standard informational messages.
    *   `LogDebug(object message)`: For detailed messages useful during development. These can often be filtered out by users or configuration.
    *   `LogError(object message)`: For critical errors or exceptions.
    *   `LogFine(object message)`: For highly granular tracing, even more detailed than `LogDebug`.
    *   **Static Access**: You can also use the static `Logger` class: `Logger.Log("Message")`, `Logger.LogError("Error")`, etc., from anywhere in your code.

*   **Best Practices**:
    *   Log key events in your mod's lifecycle (initialization, unloading).
    *   Log errors comprehensively, including stack traces if possible (`LogError(ex)` where `ex` is an Exception).
    *   Use `LogDebug` or `LogFine` for messages that shouldn't clutter the log for regular users (e.g., per-frame updates, detailed state changes).
    *   Prefix your log messages with your mod's name (`Log("[MyModName] Initializing...")`) to make them easily identifiable in the shared `ModLog.txt`.

### Saving and Loading Mod Data: Persistence Matters

Most non-trivial mods need to save data, whether it's user configuration or mod-specific progress. The API offers two primary mechanisms for this, simplifying persistence by handling the file I/O and serialization (to JSON) automatically.

*   **Global Settings (`IGlobalSettings`)**: Use this for data that should apply across all save files for a user. Common examples include keybinds, UI preferences, or difficulty settings specific to your mod.
    1.  Create a plain C# class (POCO) to hold your settings.
    2.  Make your main `Mod` class implement the `IGlobalSettings<YourSettingsClass>` interface.
    3.  The API will automatically load data from `ModName.GlobalSettings.json` into an instance of `YourSettingsClass` when the mod initializes and save it when the game closes.
    4.  Access the settings via a property (e.g., `this.GlobalSettings`).

    ```csharp
    // 1. Define the settings class
    public class MyModGlobalSettings
    {
        public bool FeatureEnabled = true;
        public float DifficultyMultiplier = 1.0f;
        public KeyCode SpecialActionKey = KeyCode.F5;
    }

    // 2. Implement the interface in your Mod class
    public class MyFirstMod : Mod, IGlobalSettings<MyModGlobalSettings>
    {
        // ... other Mod methods ...

        // Instance to hold the loaded settings
        public static MyModGlobalSettings GlobalSettings { get; private set; } = new MyModGlobalSettings();

        // Method called by API to load settings
        public void OnLoadGlobal(MyModGlobalSettings s) => GlobalSettings = s;

        // Method called by API to save settings
        public MyModGlobalSettings OnSaveGlobal() => GlobalSettings;

        public override void Initialize(Dictionary<string, Dictionary<string, GameObject>> preloadedObjects)
        {
            Instance = this;
            Log($"Initializing {GetName()}");

            // Settings are loaded automatically before Initialize by the API
            Log($"Feature Enabled setting: {GlobalSettings.FeatureEnabled}");

            // ... rest of initialization ...
        }
        // ...
    }
    ```

*   **Local Settings (`ILocalSettings`)**: Use this for data that is specific to a particular save file. Examples include quest progress, collected items added by your mod, or character states relevant only to that playthrough.
    1.  Create a plain C# class for your save-specific data.
    2.  Make your main `Mod` class implement the `ILocalSettings<YourLocalSettingsClass>` interface.
    3.  The API automatically saves this data within the game's save file structure (often in a `userX.modded.json` file alongside `userX.dat`) when the game saves, and loads it when the save file is loaded.
    4.  Access via a property (e.g., `this.LocalSettings`).

    ```csharp
    // 1. Define the local settings class
    public class MyModLocalSettings
    {
        public int CustomItemsCollected = 0;
        public bool OptionalBossDefeated = false;
    }

    // 2. Implement the interface in your Mod class
    public class MyFirstMod : Mod, ILocalSettings<MyModLocalSettings>
    {
        // ... other Mod methods and Global Settings if applicable ...

        // Instance to hold the loaded local settings
        public static MyModLocalSettings LocalSettings { get; private set; } = new MyModLocalSettings();

        // Method called by API to load settings for the current save
        public void OnLoadLocal(MyModLocalSettings s) => LocalSettings = s;

        // Method called by API to save settings for the current save
        public MyModLocalSettings OnSaveLocal() => LocalSettings;

        // You might reset local settings on a new game
        public MyFirstMod()
        {
            ModHooks.NewGameHook += ResetLocalSettings;
        }

        public override void Unload()
        {
            ModHooks.NewGameHook -= ResetLocalSettings;
            // ... other unload logic ...
        }

        private void ResetLocalSettings()
        {
            Log("Resetting local settings for new game.");
            LocalSettings = new MyModLocalSettings();
        }
        // ...
    }
    ```

### Creating Mod Menus (`IMenuMod`, `ITogglableMod`)

Providing configuration options to users greatly enhances a mod's usability. The API allows mods to add their own menus to the game's Options screen.

*   **`IMenuMod`**: Implement this interface in your `Mod` class to add a custom menu.
    *   Requires implementing `List<MenuEntry> GetMenuData(MenuScreen modlistMenu)` (or similar signature depending on API version/helpers). This method should return a list of UI elements (toggles, sliders, buttons, text) that define your menu's content.
    *   Helper libraries like `Satchel` are *highly* recommended for simplifying menu creation, providing easier ways to define layouts and elements.
*   **`ITogglableMod`**: Implementing this simple interface alongside `IMenuMod` tells the API that your mod can be enabled or disabled. The API will automatically add an enable/disable toggle button for your mod in the main Mod List menu, calling your `Unload()` method when disabled and re-initializing when enabled.

```csharp
using Modding.Menu;
using Modding.Menu.Config;
// Potentially using Satchel or other helpers for menu creation

public class MyFirstMod : Mod, IMenuMod, ITogglableMod // Implement both interfaces
{
    // ... other Mod code, including settings classes ...

    public bool ToggleButtonInsideMenu => false; // Where to put the main enable/disable toggle

    // Method required by IMenuMod
    public List<IMenuMod.MenuEntry> GetMenuData(IMenuMod.MenuEntry? toggleButtonEntry)
    {
        // Using hypothetical helper functions for clarity 
        // (Real implementation depends on API version / Satchel)
        return new List<IMenuMod.MenuEntry>
        {
            new IMenuMod.MenuEntry
            {
                Name = "Enable Feature",
                Description = "Toggle the main feature of this mod.",
                Values = new [] {"On", "Off"},
                Saver = opt => GlobalSettings.FeatureEnabled = opt == 0, // Index 0 is "On"
                Loader = () => GlobalSettings.FeatureEnabled ? 0 : 1
            },
            new IMenuMod.MenuEntry
            {
                Name = "Difficulty",
                Description = "Adjust the mod's difficulty.",
                Values = new [] {"Easy", "Normal", "Hard"},
                Saver = opt => GlobalSettings.DifficultyMultiplier = (opt + 1) * 0.5f, // Example mapping
                Loader = () => (int)((GlobalSettings.DifficultyMultiplier / 0.5f) - 1) // Reverse mapping
            },
            // ... more menu options ...
        };
    }

    // ... rest of Mod class ...
}
```
Creating menus often involves understanding the specific UI helper methods provided by the API version or libraries like Satchel, but the core principle is implementing `IMenuMod` and defining your options.

## Advanced Topics and Techniques

Beyond the core concepts, successful Hollow Knight modding often requires delving into more advanced techniques to interact deeply with the game's systems and create complex features. This section explores some of these crucial areas.

### Interacting with Game Objects and Components

Hollow Knight is built in Unity, and understanding basic Unity concepts like GameObjects and Components is essential. GameObjects are the fundamental entities in a Unity scene (like the player, enemies, platforms, triggers), and Components define their behavior and properties (like `Rigidbody` for physics, `Collider` for collision detection, `SpriteRenderer` for visuals, or custom scripts like `HeroController` or `HealthManager`).

*   **Accessing GameObjects**:
    *   **Preloading**: As discussed earlier, `GetPreloadNames()` is the most efficient way to get references to frequently used prefabs (template GameObjects).
    *   **`GameObject.Find()`**: You can search for active GameObjects in the current scene by name (`GameObject.Find("Object Name")`). This can be slow and unreliable if names are not unique or objects are inactive. Use sparingly.
    *   **Scene Traversal**: You can iterate through the root GameObjects in a scene (`UnityEngine.SceneManagement.SceneManager.GetActiveScene().GetRootGameObjects()`) and search through their children (`transform.Find("ChildName")`, `GetComponentInChildren<T>()`). This is more robust than `GameObject.Find` but still requires care.
    *   **Hooks**: Often, the easiest way to get a reference to a specific instance (like the enemy that was just hit) is through the parameters provided by hooks (e.g., the `GameObject self` parameter in an `On.` hook, or the `GameObject go` parameter in `ModHooks.HitInstanceHook`).
*   **Accessing Components**: Once you have a reference to a GameObject (`go`), you can access its components using `GetComponent<T>()`, `GetComponents<T>()`, `GetComponentInChildren<T>()`, or `GetComponentInParent<T>()`.

    ```csharp
    // Example: Get the HealthManager component from an enemy GameObject
    GameObject enemyGO = /* ... get enemy GameObject reference ... */ ;
    HealthManager hm = enemyGO.GetComponent<HealthManager>();
    if (hm != null)
    {
        Log($"Enemy Health: {hm.hp}");
        // hm.TakeDamage(...); // Interact with the component
    }
    ```
*   **Instantiating Objects**: You can create new instances of GameObjects, often based on preloaded prefabs, using `GameObject.Instantiate(prefab)`. Remember to set the position, rotation, and potentially parent the new object correctly within the scene hierarchy.

### Understanding `PlayerData`

The static class `HollowKnight.PlayerData` is a central hub for most of the player's progression and state information. It's a singleton, accessible via `PlayerData.instance`. Modifying its fields directly is often how mods grant items, change health/soul, or check game completion flags.

*   **Key Fields (Examples)**:
    *   `health`, `maxHealth`, `healthBlue` (Lifeblood hearts)
    *   `MPCharge`, `maxMP` (Soul)
    *   `geo`
    *   `equippedCharm_X`, `gotCharm_X`, `charmCost_X` (Charm related booleans and costs)
    *   `hasDash`, `hasWalljump`, `hasSuperDash`, `hasDoubleJump`, etc. (Ability flags)
    *   `killedBossName`, `completedBossName` (Boss kill/completion flags)
    *   Inventory items, map completion flags, dream nail state, and much more.
*   **Accessing and Modifying**: `PlayerData.instance.geo += 100;` or `bool hasMonarchWings = PlayerData.instance.hasDoubleJump;`. Changes are typically saved automatically when the game saves.
*   **Caution**: Directly modifying `PlayerData` can have unintended consequences if not done carefully. Always check conditions (e.g., `if (!PlayerData.instance.gotCharm_1)`) before granting items to avoid duplicates or breaking progression.

### Working with PlayMaker Finite State Machines (FSMs)

Hollow Knight uses PlayMaker, a visual scripting tool for Unity, extensively for controlling NPC behavior, enemy AI, scene events, UI interactions, and more. Understanding and interacting with FSMs is often necessary for complex mods.

*   **What are FSMs?**: An FSM consists of States. Only one State is active at a time. States contain Actions (like Move, Wait, Play Sound, Set Variable). Transitions connect States, triggered by Events (like "FINISHED", "ATTACK", "PLAYER_NEAR"). FSMs also have Variables (Int, Float, Bool, GameObject, String, etc.) that store data used by Actions and Transitions.
*   **Inspecting FSMs**: You *cannot* see FSMs directly in a standard decompiler. You need specialized tools:
    *   **FSM Viewer Mod**: An in-game mod that lets you select GameObjects and view their attached FSMs, including active states, actions, transitions, and variable values.
    *   **Runtime Inspector Mods (e.g., Unity Explorer)**: These often include FSM inspection capabilities.
    *   **Decompiling Actions**: The *code* for the Actions used by FSMs *can* be found in the decompiled `Assembly-CSharp.dll` (often in namespaces like `HutongGames.PlayMaker.Actions`).
*   **Interacting with FSMs**: Direct modification is complex. Helper libraries are almost always used:
    *   **`FsmUtil` / `PlayMakerFSM` extensions**: Libraries like SFCore, FrogCore, or Satchel often provide extension methods or utility classes to find FSMs (`gameObject.LocateMyFSM("FSM Name")`), get/set FSM variables (`fsm.FsmVariables.GetFsmInt("VarName").Value = 10;`), get/set state (`fsm.SetState("StateName")`), add new states/actions, or hook into state transitions (`FsmHook`).

    ```csharp
    // Example using hypothetical FsmUtil helpers
    GameObject bossGO = GameObject.Find("MegaMossCharger");
    PlayMakerFSM controlFSM = bossGO.LocateMyFSM("Moss Control");
    if (controlFSM != null)
    {
        // Get a variable
        int chargeCounter = controlFSM.FsmVariables.GetFsmInt("Charge Counter").Value;
        Log($"Moss Charger charge count: {chargeCounter}");

        // Set a variable
        controlFSM.FsmVariables.GetFsmBool("Can Charge").Value = false;

        // Hook into a state's entry
        FsmHook hook = new FsmHook(controlFSM, "Charge Antic", state => {
            Log("Moss Charger entering Charge Antic state!");
            // Modify actions or variables within this state if needed
        });
    }
    ```
    Working with FSMs requires patience, investigation using inspector tools, and leveraging helper libraries.

### The Importance of Decompilation

Since official source code isn't available, decompiling the game's `Assembly-CSharp.dll` (and other relevant DLLs like `UnityEngine.*.dll`) is non-negotiable for serious modding. Tools like dnSpy-Unity are essential for:

*   **Understanding Game Logic**: Seeing how vanilla features are implemented.
*   **Finding Methods to Hook**: Identifying the exact class, method name, and signature needed for `On.` or IL hooks.
*   **Locating Variables**: Finding the names of fields in classes like `PlayerData` or components like `HeroController`.
*   **Debugging**: dnSpy allows attaching to the running Hollow Knight process, setting breakpoints in the decompiled code (or your mod's code), stepping through execution, and inspecting variable values live.

### Managing Dependencies

Many mods rely on other mods or libraries to function. Proper dependency management is crucial for users.

*   **API Dependency**: All mods implicitly depend on the Modding API itself.
*   **Common Libraries**: Many mods depend on:
    *   **Satchel**: UI helpers, FSM utilities, general QoL functions.
    *   **SFCore/FrogCore**: FSM utilities, API extensions, helper methods.
    *   **HKMirror**: Provides "mirrored" versions of game classes, simplifying reflection and access.
    *   **Vasi**: Various utility functions.
    *   **ItemChanger**: A powerful framework for randomizers and item modifications.
*   **Declaring Dependencies**: When publishing your mod for use with installers like Scarab, you declare dependencies in the `Modlinks.xml` manifest. The installer will then ensure these dependencies are also installed.

    ```xml
    <Manifest>
        <Name>MyAwesomeMod</Name>
        <!-- ... other fields ... -->
        <Dependencies>
            <Dependency>Satchel</Dependency> <!-- Declare dependency by mod name -->
            <Dependency>SFCore</Dependency>
        </Dependencies>
        <!-- ... -->
    </Manifest>
    ```
*   **Optional Integrations**: The `<Integrations>` tag in the manifest can be used for soft dependencies – features that are enabled only if another specific mod is present, but not required for the core functionality.

Mastering these advanced topics requires practice and exploration, often involving reading the code of other mods and experimenting within the game using debugging and inspection tools.

## Best Practices and Common Pitfalls

Developing mods can be a rewarding experience, but following best practices ensures your mods are stable, compatible, and well-received by the community. Here are some key guidelines and common pitfalls to avoid:

### Code Quality and Style

*   **Readability**: Write clear, well-commented code. Use meaningful variable and method names. Follow standard C# naming conventions (e.g., PascalCase for methods and properties, camelCase for local variables).
*   **Organization**: Structure your mod logically. Separate concerns into different classes or methods where appropriate. Avoid putting all your logic into the main `Mod` class file if it becomes too large.
*   **Error Handling**: Anticipate potential errors. Use `try-catch` blocks where operations might fail (e.g., file I/O, complex game interactions), and log errors using `LogError`. Check for null references before accessing object members, especially when dealing with game components that might not always exist (`if (component != null)`).
*   **Namespaces**: Use a unique namespace for your mod (e.g., `YourName.YourModName`) to prevent naming conflicts with the game or other mods.

### Performance Considerations

*   **Avoid Heavy Logic in Frequent Hooks**: Hooks like `ModHooks.HeroUpdateHook` or `On.SomeClass.Update` run every frame. Performing complex calculations, allocations (creating new objects), or searches (`GameObject.Find`) within these hooks can significantly impact game performance. Optimize critical code paths or find less frequent hooks if possible.
*   **Caching**: If you frequently need access to a specific component or GameObject, get the reference once (e.g., in `Initialize` or `OnSceneLoad`) and store it in a variable instead of searching for it repeatedly.
*   **Object Pooling**: If your mod frequently creates and destroys objects (like custom projectiles or effects), consider implementing an object pooling system to reuse objects instead of constantly instantiating and destroying them, which can cause garbage collection spikes.
*   **Logging**: Excessive logging, especially `LogDebug` or `LogFine` in per-frame hooks, can impact performance. Use it judiciously during development and consider adding a configuration option to disable verbose logging in release versions.

### Compatibility and Mod Interactions

*   **Hook Carefully**: Be mindful when hooking methods that other mods might also hook. Ensure your hook calls the original method (`orig(...)` in `On.` hooks) unless you explicitly intend to replace its functionality entirely. Modifying method behavior drastically can break other mods relying on the original behavior.
*   **Unsubscribe from Hooks**: *Always* unsubscribe from events and hooks (`ModHooks`, `On.`, `IL.`) in your `Unload` method. Failure to do so is a common cause of errors and memory leaks, especially if mods are toggled on/off or the game state changes.
*   **Dependencies**: Clearly declare dependencies in your `Modlinks.xml` manifest. If you rely on specific features of another mod, check if that mod is loaded and potentially check its version before using those features.
*   **Avoid Modifying Core Game Prefabs Directly**: If possible, avoid directly altering properties on shared game prefabs obtained via preloading. Instantiate the prefab first and modify the instance, or use hooks to alter behavior at runtime. Modifying the shared prefab can affect all instances in the game, potentially breaking vanilla behavior or other mods.

### Data Management

*   **Reset Local Settings**: Remember to reset `ILocalSettings` data when a new game is started (using `ModHooks.NewGameHook`) to ensure players start fresh without carrying over progress from a previous save.
*   **Settings Validation**: When loading settings (especially `IGlobalSettings`), consider validating the values to ensure they are within expected ranges or formats, as the JSON file could potentially be edited manually or become corrupted.

### Community Interaction

*   **Source Code**: Consider making your mod's source code public (e.g., on GitHub). This fosters transparency, allows others to learn from your code, and potentially contribute fixes or improvements.
*   **Issue Tracking**: Use GitHub Issues or a similar platform to track bug reports and feature requests from users.
*   **Modding Discord**: Engage with the Hollow Knight Modding Discord server. It's an invaluable resource for asking questions, getting help, sharing progress, and staying updated on API changes and community developments.

### Common Pitfalls

*   **Forgetting to Unhook**: Leads to `NullReferenceException`s or unexpected behavior when scenes change or mods are unloaded.
*   **Blocking the Main Thread**: Performing long-running operations (like complex calculations or synchronous network requests) directly in hooks can freeze the game. Use asynchronous programming (`async/await`) or background threads for such tasks if necessary, being careful about thread safety when interacting with Unity objects.
*   **Incorrect Hook Signatures**: `On.` and `IL` hooks require precise matching of method signatures. Errors here often lead to the hook silently failing or causing runtime crashes.
*   **Modifying `PlayerData` Recklessly**: Granting items or setting flags without checking existing state can break game progression or cause unexpected side effects.
*   **Case Sensitivity**: C# is case-sensitive. Ensure class, method, and variable names match exactly, especially when using reflection or string-based lookups (like `GameObject.Find` or FSM variable names).

By keeping these practices and potential pitfalls in mind, you can create more robust, performant, and community-friendly mods for Hollow Knight.

## Useful Resources and Links

As you continue your journey into Hollow Knight modding, the following resources will be invaluable:

*   **Hollow Knight Modding Discord**: The central hub for the community. Ask questions, share progress, find help, and stay updated.
    *   [Join the Discord](https://discord.gg/hollowknight) (Look for the modding channels)
*   **Official Modding API Repository (hk-modding/api)**: Source code for the API, examples, and issue tracking.
    *   [GitHub Repository](https://github.com/hk-modding/api)
*   **Community Modding Documentation (ModdingDocs)**: A comprehensive, community-maintained documentation site covering many aspects of modding.
    *   [ModdingDocs Site](https://prashantmohta.github.io/ModdingDocs/)
*   **Radiance API Documentation**: An older but still potentially useful documentation site for the API.
    *   [Radiance Docs](https://radiance.synthagen.net/apidocs/_images/index.html)
*   **Modlinks Repository**: The central list (`Modlinks.xml`) used by installers like Scarab to find and download mods.
    *   [Modlinks GitHub](https://github.com/hollow-knight-modding/Modlinks)
*   **Scarab Mod Installer**: A popular tool for users to install and manage mods.
    *   [Scarab Releases](https://github.com/hollow-knight-modding/Scarab)
*   **dnSpy-Unity Fork**: Essential decompiler and debugger for Unity games.
    *   (Search for "dnSpyEx/dnSpy" or specific Unity forks - direct links change)
*   **Unity Scripting Reference**: Official documentation for the Unity engine API, crucial for understanding core game components and functions.
    *   [Unity Scripting API](https://docs.unity3d.com/ScriptReference/index.html)
*   **Microsoft .NET API Browser**: Reference for the .NET Framework classes.
    *   [.NET API Browser](https://docs.microsoft.com/en-us/dotnet/api/)
*   **Satchel Library**: A popular utility library simplifying many common modding tasks (UI, FSMs, etc.).
    *   [Satchel GitHub](https://github.com/PrashantMohta/Satchel)

## Conclusion

The Hollow Knight Modding API opens up a world of possibilities for customizing and expanding upon the already incredible experience of Hollow Knight. While the learning curve can be steep, especially when dealing with advanced concepts like FSMs and IL hooking, the supportive community and the powerful tools available make it an accessible and rewarding endeavor. By understanding the core concepts, utilizing the right tools, following best practices, and leveraging the wealth of community resources, you can bring your own unique ideas to life within the kingdom of Hallownest. Happy modding!


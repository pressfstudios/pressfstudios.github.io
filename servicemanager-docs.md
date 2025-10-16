---
layout: default
title: Press F Studios
description: Bootstrap and Service Manager Tool Docs
permalink: /tools/service-manager
---

# Bootstrap and Service Manager

The bootstrap and service manager is a Unity tool which provides both:
- A quick, easy to use, safe, and easy to manage solution for your in-game 'singleton objects'.
- A simple but flexible custom per-scene bootstrap solution to ensure you never have issues of missing managers, services, or other required gameobjects when you boot from a random scene in your project.

*NB: The service and bootstrap side of the plugin are completely separate and do not require each other's use. If you just want to use our Service model and handle your own bootstrapping you can easily do so.*

## Service Manager

The service manager is extremely simple to use.

Firstly, add an empty gameobject to the first scene which runs in your project build. Add the `F Service Manager` component to it. It is good practice to name this object `Service Manager`.

Secondly, in your service script (what you otherwise would have considered to be a singleton), inherit from `FService` instead of `MonoBehaviour`. `FService` is within the `using F.Services.Base` namespace.
You can now add this script as a component to any `GameObject` in any scene, and as soon as this object is loaded it will be automatically registered with the `F Service Manager`.

To get a reference to this object from another script at runtime, you can use the following code:
```c#
public F serviceReference;

void MyFunction()
{
    serviceReference = FServiceManager.GetService<F>();
} 
```

Please note that `F` can be any type that inherits from `FService`.
The `GetService<T>()` method returns `null` if no object of the specified type is registered with `FServiceManager`.

Lastly, if you need to write code in `Awake()` or `OnDestroy()` on a script that inherits from `FService`, please ALWAYS CALL `base.Awake()` and `base.OnDestroy()` WITHIN THE OVERRIDE, as follows:
```c#
public class MyService: FService
{
    protected override void Awake()
    {
        base.Awake();

        //Your code here...
    }

    protected override void OnDestroy()
    {
        base.OnDestroy();

        //your code here...
    }
}
```

That is all for the service side of the tool.

## Bootstrapper

### The Editor

You can open the bootstrap window by going to `Tools/PressF/FBootstrapper Setup`.

![Bootstrap Editor](https://pressfstudios.github.io/assets/images/BootstrapperEditor.png)

As seen above, there are 4 sections to this editor. The first are is the list at the top of the window under Default Spawn Prefabs. In this list you can drag and drop any `GameObject` prefabs you want to spawn MOST OF THE TIME. These can be things like an Audio Manager, a HUD prefab, or anything else like that.

The section under this will have a region for every scene added to your build settings. Under each of these you can control 3 things:
- Whether or not on this scene you want to spawn the common, "defaut", prefabs you defined above in the general section (this can be done by toggling the `Spawn Default Prefabs` option).
- What GameObject prefabs you want to spawn in this scene specifically (these can be dragged and dropped to the `Scene-Specific Definition` list area).
- What prefab Packages you want to load in this scene specifically (more on this later, but these can be dragged and dropped in the `Scene-Specific Packages` list area).

### The scene detail viewer

You can see a detailed breakdown of every gameobject that will be spawned in a scene in list format if you click the `i` button next to the scene name in the Editor window.

![Bootstrap Scene Detail View](https://pressfstudios.github.io/assets/images/BootstrapperSceneDetails.png)

Here, you will see a list of all the objects that will be spawned in the scene, as well as what they are included in. There is also a search bar at the top if you want to quickly query if an object is being spawned.

### Packages

<!-- [back](./) -->

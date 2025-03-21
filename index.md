---
layout: default
---

Welcome to the Press F Studios public asset documentation page.

* * *

# FId - Unique Object ID Reference System

FId is a tool designed to replace direct object referencing where necessary, for example:

```c#
public GameObject directObjectReference; //Only supports scene-scope references
public IdentifiableObjectReference idBasedReference; //Supports across-scene references  
```

The editor difference is as follows:

![Comparison](https://pressfstudios.github.io/assets/images/DefaultVsIdRef.png)


## Getting Started

1.  Import the _Unique Object ID Reference System_ package from your Unity Package Manager.
2.  In the first scene to load in your project (for example a base/service scene or a main menu), add an empty GameObject and add the ObjectQuerier component to it.
![Querier](https://pressfstudios.github.io/assets/images/Querier.png)
3.  To begin referencing objects by ID, you can now add the _IdentifiableObject_ component to any scene object which you wish to add a unique ID to.
![IdObject](https://pressfstudios.github.io/assets/images/IdObject.png)
4.  Copy the ID of the identifiable object through the component you have just applied.
![IdCopy](https://pressfstudios.github.io/assets/images/IdObject_Copy.png)
5.  Alternatively, you can open the _Object Identifier Search_ window, search, and copy the code of any identifiable object in your project.
![SearchCopy](https://pressfstudios.github.io/assets/images/CopyFromSearch.png)
6.  Write a new line of code to create an instance of _IdentifiableObjectReference_ that will draw in the inspector. This can be done on any _MonoBehaviour_:
```c#
public IdentifiableObjectReference exampleObjectReference;
```
7.  Paste the code you previously copied into the inspector drawer of this newly created field.
![ValidRef](https://pressfstudios.github.io/assets/images/ValidReference.png)


## Code

Using FId in code is extremely simple. Every interaction follow a one-line approach.

#### Object from ID
Usually, objects are referenced and accessed as follows:
```c#
public GameObject gameObjectReference;

void Start()
{
  gameObjectReference.transform.position = Vector3.zero;
}
``` 
You can similarly access the object from its reference at any point in your code as follows by using `referenceInstance.Object()`:
```c#
public IdentifiableObjectReference objectReference;

void Start()
{
  objectReference.Object().transform.position = Vector3.zero;
}
```
_It is suggested that you do not cache objectReference.Object() as it is already cached behind the scenes and doing so will just result in creating redundant references to the same GameObject._

**If the object is in a scene not currently loaded, calling `.Object()` will return `null`, therefore it is suggested to wrap any executions on `.Object()` inside a `if (reference.Object())`.**

#### ID from GameObject
If an object has an ID assigned to it (if it has an `IdentifiableObject` component on it), you can get its ID at any point by using hte following code:
```c#
public GameObject directObjectReference;
public IdentifiableObjectReference idBasedReference;

private void Start()
{
  ObjectQuerier.GetIdForObject(directObjectReference, out idBasedReference);
      
  //use idBasedReference here, serialize it, or whatever else you want to do...
  if (idBasedReference.Object())
  {
    idBasedReference.Object().transform.position = Vector3.zero;    
  }
}
```
_using `ObjectQuerier.GetIdForObject(GameObject obj, out IdentifiableObjectReference outRef)` is much more performant than doing a `GetComponent` check and therefore it is the suggested method._

#### Referencing runtime spawned Identifiable Objects
Usually, GameObjects are spawned in a scene by getting/storing a reference to a prefab and instantiating that into the scene. It is suggested, however, when spawning an object with an `IdentifiableObejct` component assigned to it, you rather store a reference to the `IdentifiableObject` component and instantiate that instead, which will allow you to store an ID reference without having to do a `GetComponent` call:
```c#
public IdentifiableObject objectPrefab;

  private void Start()
  {
    IdentifiableObjectReference runtimeObjectReference = Instantiate(objectPrefab);
  }
```

For debugging or structural purposes, you may want to have an `IdentifiableObjectReference` as a public (or anyways visible) field. In this case - should the reference not be meant for setup in the editor but only meant for assigning at runtime - it is suggested (but not required) to set the `IdentifiableObjectReference` to `IdentifiableObjectReferenceType.Runtime` as follows:
```c#
public IdentifiableObject objectPrefab;
public IdentifiableObjectReference runtimeObjectReference = new IdentifiableObjectReference(IdentifiableObjectReference.IdentifiableObjectReferenceType.Runtime);

private void Start()
{
  runtimeObjectReference = Instantiate(objectPrefab);
}
```
**This will make the field unassignable in editor and will prevent the reference from showing up as "invalid" in the validator.**

#### Spawning Identifiable Objects with a specific ID
Especially when working with save systems, you might want to keep persistent the ID of objects that are spawned in scene throughout gameplay so that they can correctly match and reflect the ID of objects in your save data/save files (or you just simply want to use your own runtime object ID assigning logic).
If you instantiate an `IndentifiableObject` normally as seen above in the _"Referencing runtime spawned Identifiable Objects"_ section, a random ID will be assigned, which might not be what you want.
You can instead use the `SpawnInSceneWithDefinedId(string id)` call to make sure the object is assigned a specific ID:
```c#
public IdentifiableObject objectPrefab;
public IdentifiableObjectReference runtimeObjectReference = new IdentifiableObjectReference(IdentifiableObjectReference.IdentifiableObjectReferenceType.Runtime);

private void Start()
{
  runtimeObjectReference = objectPrefab.SpawnInSceneWithDefinedId("your-wanted-object-id");
}
```
**Be careful when using this. Assigning the same ID to multiple objects is not supported and trying to do so will cause errors. It is suggested that you always let IDs be randomised uless you are trying to re-spawn previously assigned IDs which you already know are unique (e.g. object IDs from a previous game session).**

* * *

## Netcode for GameObjects (NGO) support

**Important note:** Netcode for GameObjects (NGO) forces all objects with `IdentifiableObject` to become `NetworkObjects`. NGO support makes sure that objects maintain the same ID both on host/server and clients. It is **very important to note** that this adds a 1-frame+RTT ID sync delay depending on network conditions. This means all ID generation and sync code is moved from `Awake()` to `Start()` to account for how `NetworkObject` syncing works in NGO, which will cause all `IdentifiableObjectReference` queries to be `null` for frame 1 of the object's instantiation. Please keep this in mind and follow good coding standards by verifying object reference validity before executing commands on it, by encapsulating any code in a `if (objectReference.Object()) {...}` check.

### Additional Setup
To ensure correct behavior and to account for some possible but unlikely design choices with regards to how networking and connections are handled on a per-project/per-game basis, there is an additional boolean flag in `ObjectQuerier` which needs to be set through code for NGO support to work correctly: `ObjectQuerier.IsHost = true/false` needs to be set before any Host/Server/Client connection is started.

### Getting Started
To enable Netcode for GameObjects support, first, make sure the Netcode for GameObjects package is in your project and that it is compiling with no errors, then click the _Enable Netcode for GameObjects Support_ button on the `ObjectQuerier`. Then wait for the project to recompile. If the button state changes to _Disable Netcode for GameObjects Support_, the process has been carried out correctly.

![NetcodeEnable](https://pressfstudios.github.io/assets/images/NGOEnable.png)

**Note that doing this changes the `IdentifiableObject` and `ObjectQuerier` components to `NetworkBehaviour`, and will require `NetworkObject` components to be assigned to any object with either of those scripts assigned.**

#### Spawn with ID over Network

You can spawn an `IdentifiableObject` on all clients through a single spawn call (from Host or Server):

```c#
public NetworkObject networkObjectPrefab;

public IdentifiableObjectReference runtimeObjectReference = new IdentifiableObjectReference(IdentifiableObjectReference.IdentifiableObjectReferenceType.Runtime);

private void Start()
{
  runtimeObjectReference = 
      networkObjectPrefab.SpawnNetworkObjectWithDefinedId(Vector3.zero, Quaternion.identity, "your-wanted-object-id");
}

private void Update()
{
  if (runtimeObjectReference.Object())
  {
    Debug.Log($"Object Spawned with ID: {runtimeObjectReference.Identifier}");
  }
}
```
**It is suggested to always check if `Object()` is not null as network spawning can take multiple frames depending on RTT and other networking factors**


## Search and Validate Window

To simplify the ID search and copy/paste process, there is a helper window which can be accessed from _Tools > Press F Studios > Object Identifier Search_ which has 2 tabs:

#### Search

The Search tab shows a list of all identifiable objects in the project. 

_**DEBUGGING NOTE:** Should this list for whatever reason glitch or behave/display incorrectly, find the following file and set its list length to 0: _Assets > Resources > ObjectIdentifierSearchDatabase. After that re-open each scene in your project individually **or** execute the Validate process TWICE (the first will show errors and the second should fix everything as it finds the references again)._

Objects that appear will show the ID, name of the GameObject in the scene, and name of the scene the object is in. Objects in other scenes only have 1 button to copy the ID (bottom right corner), while objects in a currently open scene will have a second button which allows for quick heirarchy object selection.

The search bar allows you to search both by name and/or ID.

#### Validate

The Validate tab allows you to scan for missing/incorrect references in your project.
Firstly, click _Validate_.

Leaving an `IndentifiableObjectReference` blank or with an incorrect code (unless it is marked as `Runtime` in its constrctor), will result in it showing up in the invalid list. For this reason marking all references that should be empty by default as `Runtime` (as mentioned above in the runtime spawning section) is highly suggested. 

If the incorrect reference in the list is in the current scene, a button to quickly select the object will appear for each entry.

In order for elements to go away from the list, you need to run the _Validate_ process once more.

**It is highly suggested to run a validity check before building to help spot any referencing errors that might be present**

## Debugger

The debugger can be opened through _Tools > Press F Studios > Object Identifier Debugger_.

This window shows every `IdentifiableObject` that is currently reachable (in a loaded scene).

This can be helpful to see if an object is not registering correctly.

## Help

If you have any questions, run into any issues, or have pre-purchase questions about any product, please do not hesitate to contact us via <a href="mailto:info@pressfstudio.com">email</a> , join our [Discord](https://discord.gg/r4QcysF93v), or reach us through our [Website](https://pressfstudio.com/)
---
layout: default
---

Welcome to the Press F Studios public asset documentation page.

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

Using FId in code is extremely simple.

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

## Netcode for GameObjects support
### Getting Started
To enable Netcode for GameObjects support, first, make sure the Netcode for GameObjects package is in your project and that it is compiling with no errors, then click the _Enable Netcode for GameObjects Support_ button on the ObjectQuerier. Then wait for the project to recompile. If the button state changes to Disable Netcode for GameObjects Support_, the process has been carried out correctly.

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

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```

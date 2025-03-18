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


### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

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

## The Remote debugger

### How to use the remote debugger?

This section was written, based on this version of Pharo: `Pharo-12.0.0+SNAPSHOT.build.1525.sha.8bc5c91f2aed76c584f65c81481b870aed28ab2d (64 Bit)`.

#### How to load the remote debugger?

To use the remote debugger, you will have to open 2 images: the server image that contain the program you want to debug, and the client image from which you will debug the program of the server image.

In both images, load the following baseline in a playground:

```Smalltalk
Metacello new
	baseline: 'Red';
	repository: 'github://adri09070/RED:Pharo12';
	load
```

This will load the remote debugger server and client in both images, with their dependencies.

In a playground, in the **server** image, execute this code:

```Smalltalk
PharoBridge start.
PharoBridgeObject baseTargetUrl: 'http://127.0.0.1:4321/'.
RedDebuggerServer current
```

Then, in a playground, in the **client** image, execute this code:

```Smalltalk
PharoBridge startAtPort: 4321.
```

In the **client** image, in the World toolbar at the top, click on the `Debug > Remote Debugger Sessions` menu entry (Figure *@fig:red-client-opening@*).

![Launching Red Debugger client](./figures/red-client-opening.png width=70&label=fig:red-client-opening)

This will connect the client to the server and will open a presenter listing the different remote debug sessions you can debug (which is empty, for now).

In the **server** image, open the Pharo settings (`Pharo > Settings` menu entry in the World toolbar) on `Tools > Debugging > Debuggers > StDebugger` and set the priority of the `StDebugger` to `9` (Figure *@fig:stdebugger-priority-change@*).

![Changing StDebugger priority](./figures/stdebugger-priority.png width=70&label=fig:stdebugger-priority-change)

To check that everything works correctly, execute `1/0` in the **server** image, and check that the session appears in the **client** image (Figure *@fig:red-client-check@*)

![The remote session appears in the red client presenter](./figures/red-client-check.png width=70&label=fig:red-client-check)

It is then possible to click on the _Debug_ button of the client presenter to open a debugger on the remote program.

#### Example

In the **server** image, open a class browser, and create this class:

```Smalltalk
Object << #A
	slots: { #myInstVar };
	sharedVariables: { #mySharedVar };
	tag: '';
	package: 'MyPackage'
```

In the class `A` in the **server** image, compile the following methods **instance-side**:

```Smalltalk
myInstVar

	^ myInstVar
```

```Smalltalk
myInstVar: anObject

	myInstVar := anObject
```

```Smalltalk
myMethod

	^ 42 + 666
```

The, in the class `A`, compile the following methods **class-side**:

```Smalltalk
mySharedVar

	^ mySharedVar
```

```Smalltalk
mySharedVar: anObject

	mySharedVar := anObject
```

```Smalltalk
exampleMethod

	| obj instVar sharedVar |
	obj := self new.
	instVar := obj myInstVar.
	obj myInstVar: 42.
	instVar := obj myInstVar.
	sharedVar := obj class mySharedVar.
	obj class mySharedVar: obj myMethod.
	sharedVar := obj class mySharedVar.
	obj unkwownMethod
```

The class `A` has been installed in the **server** image, and we will show how to debug it from the **client** image, in which the class `A` does not exist.

In a playground in the **server image**, debug this code: 

```Smalltalk
A exampleMethod
```

Debug the new session that appears in the **client** image.
This will open a debugger on the remote method.
Click on _Into_ to enter the method.
Now, you can see what happens when performing _Over_, _Restart_ and _Into_.
For instance, you can execute _Over_ twice then execute _Into_ once to enter the method `myInstVar`, execute _Over_ again.
Then you can click on _Restart_ to restart the whole context from the beginning.
Finally, you can click on _Proceed_ to resume the execution.
This will close the debugger in the client image but you will also see a new remote session appear in the **client** image because an exception has been raised as an object `A` does not understand the message `#unkwownMethod`.

#### Pharo Bridge example

This is how this remote debugger can be used.
For now, the remote objects cannot be inspected.
Here, I give examples of what can be done in the `RedDebuggerClient` to recover these remote objects.

To recover the remote objects in the **client** image, `PharoBridge` should be used.

It is possible to recover a proxy for the class `A` in the **client** image by executing this code:

```Smalltalk
aClass := PharoBridge loadClass: #A.
```

Inspecting `aClass` will give you an `objectId` (Figure *@fig:A-proxy-inspect@*).

![Inspecting proxy for `A` class.](./figures/A-proxy-inspect.png width=70&label=fig:A-proxy-inspect).

This `objectId` is mapped to the real object (here, the class `A`) in the **server** image, which you can see by inspecting `PharoBridgeObject reverseServer instanceMap` (Figure *@fig:instanceMap-mapping-A-class@*).

![Instance map, showing the objectId of proxy of `A` class.](./figures/instanceMap-mapping-A-class.png width=70&label=fig:instanceMap-mapping-A-class).

Then, you can create a new instance from the loaded class in your **client** image:

```Smalltalk
aInstance := aClass new.
```

This creates an instance in the **server** image, which is mapped in its `insatnceMap` to an `objectId` that is used in the **client** image as a proxy.

This proxy can then be used in the **client** to send messages to the real remote object:

```Smalltalk
aInstance myInstVar: 'abcdef'.
aInstance myInstVar
```

Inspecting `aInstance myInstVar` will inspect a real string in the **client** image, because it is a literal (Figure *@fig:remote-instvar-literal@*)

![Inspecting a remote instance variable returns a local literal.](./figures/remote-instvar-literal.png width=70&label=fig:remote-instvar-literal)

However, if the value of a remote instance variable is not a literal, inspecting it in the **client** image will inspect a new proxy, whose `objectId` would have automatically been registered in the **server** `instanceMap`.
> I am not sure about that. I cannot reproduce it because of the 4th Pharo Bridge problem ([See section](#known-pharobridge-problems)).

#### Discussion

Now, it could be possible to inspect remote objects in the debugger in the **client** image.

It could be done by mapping in the **server** image a new `objectId` for each accessible object in the selected `context` in the **client** image.
Then, this `objectId` could be used to get the inspector nodes of the remote object.
However, we should think about what inspector nodes should be displayed and how.
Because the values of the instance variables of the remote object will created other proxies in the **client** image and we may have to get the remote values of their own instance variables also...
This can get complicated as not all objects are serializable (`CompiledMethod`, `CompiledBlock`, etc.), so what inspector nodes should be displayed and how? I did not manage to answer this question.

### Known problems

- If you open several images that run the server (resp. several images that run the client), you can close the image that should not run the server (resp. client), then in the other image whose server (resp. client) is stopped, you can inspect `RedDebuggerServer current` (resp. `RedDebuggerClient current`) and execute `server start` in the inspector code pane.

- Sometimes, the code does not seem to be highlighted in the client image when debugging a remote `DoIt`.

### Known PharoBridge problems

- A `PharoBridgeClass` object is not printed correctly when inspecting the object. However, it is printed correctly when sending the message `printString` to it.

- A `PharoBridgeObject` object is not printed correctly when inspecting the object. It is not even printed correctly when sending the message `printString` to it.

- A new `objectId` is created each time a remote object is accessed, which is not normal because `PharoBridge` has been designed to create a proxy only once when the remote object is accessed for the first time in the **client** image. This floods the `instanceMap` of the `PharoBridge` reverse server.

- The `objectId` for the remote object is kept in the **server** instanceMap while the object is not garbage collected.
So when creating a new remote instance in the **client** image, and when sending messages to it, you may have `KeyNotFound in Dictionary` as the object may be garbage collected at any time in the **server** image.
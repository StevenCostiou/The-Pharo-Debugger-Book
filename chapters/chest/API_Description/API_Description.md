### API Description

This section details the _API_ of _Chest_ and give examples of its usage. 

#### Chest class-side API

This subsection describes the _API_ that can be used on the class `Chest`.

##### How to create instances

- `Chest class>>#new` creates a chest with a default name that is in the form of `Chest_autoIncrementedNumber`.
	
	In the examples below, if no other chest has been created before, the names of the created chests are respectively _Chest\_1_ and _Chest\_2_: 

	```language=Pharo
	Chest new. "Chest_1"
	Chest new. "Chest_2"
	```

- `Chest class>>#newNamed:` creates a chest with the name given as argument if no other chest is already named so, else an exception `ChestKeyAlreadyInUseError` is raised.

	For example, if no other chest is already called "toto", the piece of code below creates a chest that is named as "toto":

	```language=Pharo
	Chest newNamed: 'toto'. "its name is 'toto'"
	```

	On the contrary, if another chest is already called "toto", the same piece of code would raise a `ChestKeyAlreadyInUseError` because two chests cannot have the same name:

	```language=Pharo
	Chest newNamed: 'toto'. "ChestKeyAlreadyInUseError as a chest named 'toto' already exists"
	```

##### Accessors

- `Chest class>>#allChests` returns an ordered collection that contains all chest instances.

If we suppose that there are no other chests than the ones created above, the piece of code *@code:chest-allChests@* returns a collection with two chests: the default chest and the chest named _toto_:

```language=Pharo&label=code:chest-allChests
Chest allChests "{DefaultChest. totoChest}"
```

- `Chest class>>#chestDictionary` returns a dictionary containing all chests with their name as key.

	If we suppose that there are no other chests than the ones created above, the piece of code *@code:chest-chestDictionary@* returns a dictionary with two chests: the default chest, with `'Default'` as a key, and the chest named _toto_, with `'toto'` as a key:

	```language=Pharo&label=code:chest-chestDictionary
	Chest chestDictionary "{'Default' -> DefaultChest. 'toto' -> totoChest}"
	```

- `Chest class>>#named:` returns the chest that is named as the string given as argument if it exists, else raises an exception `KeyNotFound`.

	The expression below can be used to get the chest named 'toto', if it exists:

	```language=Pharo
	Chest named: 'toto' "chest named 'toto'"
	```

	However, as no chest is named _'titi'_, the expression below would raise a `KeyNotFound`:

	```language=Pharo
	Chest named: 'titi' "KeyNotFound"
	```

- `Chest class>>#defaultInstance` (or `Chest class>>#default`)

	Most of `Chest`'s instance-side _API_ can be used on the class `Chest` itself. These methods return the default chest that is used when you use `Chest`'s instance-side API directly on `Chest class`.
	If the default chest doesn't already exist, calling these methods lazily create it.

	For example, the expression below returns `true`:

	```language=Pharo
	Chest defaultInstance == Chest default "true"
	```

- `Chest class>>#announcer` is an helper method that returns the unique instance of `ChestAnnouncer`, which propagates changes related to chests to any subscriber.

	The example below subscribes `self` to the `ChestAnnouncer` singleton to 3 different events; when a new chest chest has been created, when the content of a chest has changed and when a chest has been removed:

	```language=Pharo
	Chest announcer weak when: ChestCreated send: #eventNewChest: to: self;
							when: ChestUpdated send: #eventContentOfChestUpdated: to: self.
							when: ChestRemoved send: #eventChestRemoved: to: self.
	```

	When the event occurs, the methods that are called (in the example: `#eventNewChest:`, `#eventContentOfChestUpdated:` and `#eventChestRemoved:`) take the related event as an argument.

- `Chest class>>#weak` returns the class `WeakChest`, subclass of `Chest`. You can use the same _API_ on this class as you would on `Chest class`, in order to create or access chests that hold weak references to objects. This means, that storing your objects in a weak chest doesn't prevent them from being garbage-collected.

	For example, the expression below creates a new weak chest named as "wtiti", if it doesn't already exist:

	```language=Pharo
	Chest weak newNamed: 'wtiti' "weak chest named as 'wtiti'"
	```

##### How to perform actions

- `Chest class>>#inChest:at:put:` puts the object given as third argument with the name given as second argument in the chest named as first argument. This chest is lazily created if it doesn't exist yet.

	In the example below, the object `42` is stored as _toto_ in the chest named _toto_:

	```language=Pharo
	(Chest named: 'toto') at: 'toto' put: 42. "stores 42 as 'toto' in chest named as 'toto'.
	```

	If the chest _toto_ didn't exist, it would be created automatically.
	_Also, please note that a chest cannot contain the same object twice. So, if the chest named as "toto" already contained the object `42`, it would simply be renamed as "toto" in this chest._

- `Chest class>>#unsubscribe:` is an helper method that unsubscribes its argument from the unique instance of `ChestAnnouncer`.

	For example, the expression below unsubscribes `self` from the `ChestAnnouncer` singleton:

	```language=Pharo
	Chest unsubscribe: self
	```

#### Chest instance-side API

This subsection describes the _API_ that can be used on an instance of `Chest`. Methods marked by the symbol `(*)` can also be used on the class `Chest` itself. In this case, it is equivalent to using the _API_ on the default instance of `Chest`. 

##### Accessors

- `Chest>>#name` returns the receiver chest's name.

	For example, the expression below evaluates to `true`:

	```language=Pharo
	(Chest named: 'toto') name = 'toto' "true"
	```

- `Chest>>#contents` (\*) returns a copy of the receiver chest's contents, as a dictionary that contains all objects in the chest with their name as key.

	For example, evaluating the piece of code below returns a dictionary that associates `42` to `'toto'` and `144` to `'titi'`:

	```language=Pharo
	| c |
	c := Chest new.
	Chest inChest: c name at: 'toto' put: 42.
	Chest inChest: c name at: 'titi' put: 144.
	c contents "{'toto' -> 42. 'titi' -> 144}"
	```

- `Chest>>#at: ` (\*) returns the object, contained in the receiver chest, whose name is the string given as argument if it exists, else an exception `KeyNotFound` is raised.

	For example, if the chest named as _toto_ stores `42` as _titi_, then the expression below returns `42`:

	```language=Pharo
	(Chest named: 'toto') at: 'titi'. "42"
	```

	On the contrary, if no object is named as _titi_ in the chest named as _toto_, then the same expression raises `KeyNotFound`:

	```language=Pharo
	(Chest named: 'toto') at: 'titi' "KeyNotFound"
	```

##### How to perform actions

- `Chest>>#add:` (\*) adds the object, given as argument, to the receiver chest, with a default name that is in the form of `chestName_autoIncrementedNumber`.

	In the piece of code below, we add `42` then `144` to a new chest named _MyChest_. Thus, `42` is stored with the key _MyChest\_1_ and `144` is stored with the key _MyChest\_2_:

	```language=Pharo
	| c |
	c := Chest newNamed: 'MyChest'.
	c add: 42. "MyChest_1 -> 42"
	c add: 144 "MyChest_2 -> 144"
	```

- `Chest>>#at:put:` (\*) adds the object igiven as second argument to the receiver chest with the name in first argument if no other object is already named so, else an exception `ChestKeyAlreadyInUseError` is raised.

	In the example below, in a new chest, `42` is stored as _toto_. Then, trying to store `144` as _toto_ raises a `ChestKeyAlreadyInUseError` because two objects cannot have the same name:

	```language=Pharo
	| c |
	c := Chest new.
	c at: 'toto' put: 42. "toto -> 42"
	c at: 'toto' put: 144 "ChestKeyAlreadyInUseError"
	```

	*Also, please note that a chest cannot contain the same object twice. So, if the chest named as "toto" already contained the object 42, it would simply be renamed as "toto" in this chest.*

- `Chest>>#at:put:ifPresent:` (\*) adds the object given as second argument to the receiver chest with the name given as first argument if no other object is already named so, else the block in third argument is evaluated with zero argument.

	In the exemple below, in a new chest, 42 is stored as "toto". Then, we try to store 144 as "toto" and if the key is already use, we try to store it as "titi" instead. As the key "toto" is already used by 42, 144 is stored as "titi":

	```language=Pharo
	| c |
	c := Chest new.
	c at: 'toto' put: 42 ifPresent: [Chest at: 'titi' put: 42]. "toto -> 42"
	c at: 'toto' put: 144 ifPresent: [ Chest at: 'titi' put: 144 ] "titi -> 144"
	```

- `Chest>>#remove:` (\*) removes the object given as argument from the receiver chest if it is there, else an exception `KeyNotFound` is raised.

	For example, if the chest named _toto_ contains the object `42`, then the following code snippet successfully removes `42` from this chest. Then it tries to remove 42 from this chest a second time, in which case a `KeyNotFound` is raised because the object is not there anymore:

	```language=Pharo
	| c |
	c := Chest named: 'toto'.
	c contents includes: 42 "true"

	c remove: 42. "42 is removed from c"

	c contents includes: 42. "false"

	c remove: 42 "KeyNotFound"
	```

- `Chest>>#removeObjectNamed:` (\*) removes from the receiver chest the object named as the argument if the key exists, else an exception `ObjectNotInChestError` is raised.

	For example, if the chest named _toto_ stores an object named as _tata_, then the following code snippet removes the object named as _tata_ from this chest. Then it tries to remove a second time the object named as _tata_ from this chest, in which case an `ObjectNotInChestError` is raised because the key is not used anymore.

	```language=Pharo
	| c |
	c := Chest named: 'toto'.
	c contents includesKey: 'tata' "true"

	c removeObjectNamed: 'tata'. "obj named 'tata' is removed from c"

	c contents includesKey: 'tata'. "false"

	c removeObjectNamed: 'tata' "ObjectNotInChestError"
	```

- `Chest>>#empty` removes the entire contents of the receiver chest:

	```language=Pharo
	| c |
	c := Chest new.
	c at: 'toto' put: 42.
	c at: 'titi' put 144.

	c contents isEmpty. "false"

	c empty.

	c contentns isEmpty "true"
	```

- `Chest>>#remove` completely deletes the receiver chest from the list of existing chests. It cannot be accessed afterwards:

	```language=Pharo
	| c cname |
	c := Chest new.
	cname := c name.
	Chest chestDictionary includesKey: cname. "true"

	c remove.

	Chest chestDictionary includesKey: cname. "false"
	```

- `Chest>>#name:` renames the receiver chest as the string in argument if no other chest is already named so, else an exception `ChestKeyAlreadyInUseError` is raised.

	For example, if there exist only two chests named "toto" and "titi", then it is possible to rename "toto" into "tata" but not into "titi", in which case a `ChestKeyAlreadyInUseError` is raised:

	```language=Pharo
	| toto titi |
	toto := Chest named: 'toto'.
	titi := Chest named: 'titi'.

	toto name: 'tata'.
	toto name "tata".

	toto name: 'titi'. "ChestKeyAlreadyInUseError"
	toto name "tata"
	```

- `Chest>>#renameObject:into:`, inside the receiver chest, renames the object given as first argument into the string given as second argument if the object is in the chest, else an exception `ObjectNotInChestError` is raised, and if no other object is already named so else an exception `ChestKeyAlreadyInUseError` is raised.

	If the chest named as _toto_ contains only `42` as _titi_ and `144` as _tata_, then the piece of code below renames `42` to _tutu_:

	```language=Pharo
	(Chest named: 'toto') at: 'titi'. "42"
	(Chest named: 'toto') renameObject: 42 into: 'tutu'.
	(Chest named: 'toto') at: 'tutu'. "42"
	(Chest named: 'toto') at: 'titi' "KeyNotFound"
	```

	Under the same circumstances, the following code snippet raises an `ObjectNotInChestError` because it tries to rename `666` that is not in the chest named as _toto_:

	```language=Pharo
	(Chest named: 'toto) contents includes: 666 "false"
	(Chest named: 'toto') renameObject: 666 into 'anyValidName' "ObjectNotInChestError"
	```

	Finally, trying to rename 144 into "tutu" will raise a `ChestKeyAlreadyInUseError` because 42 is already named so:

	```language=Pharo
	(Chest named: 'toto') renameObject: 144 into: 'tutu' "ChestKeyAlreadyInUseError"
	```

- `Chest>>#inspectAt:` inspect the object named as the string given as argument in the receiver chest:

	For example, the following expression opens an inspector on the object stored as _titi_ in the chest named as _toto_:

	```language=Pharo
	(Chest named: 'toto') inspectAt: 'titi'
	```
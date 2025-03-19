## Scripting the Debugger with Sindarin



Here refering to *@fig:sindarin-script-1@*

```language=Pharo&caption=Defining the name of your debugger extension&label=fig:sindarin-script-1
sindarin stepUntil: [ 
	sindarin selector = #method: 
		and:[sindarin receiver class = TheClass 
			and: [ sindarin arguments first = 5 ]]]
```
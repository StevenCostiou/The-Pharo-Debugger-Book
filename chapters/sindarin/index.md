## Scripting the Debugger with Sindarin





```language=Pharo&caption=Defining the name of your debugger extension&label=fig:debugger-extension-name
sindarin stepUntil: [ 
	sindarin selector = #method: 
		and:[sindarin receiver class = TheClass 
			and: [ sindarin arguments first = 5 ]]]
```
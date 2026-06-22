```maude
mod LIGHT-SWITCH
	Sort State .
	ops on off : -> State .
	rl [turn-on] : off => on . 
	rl [turn-off] : on => off .
```
# Fallback

This family of nodes are known as "Selector" or "Priority"
in other frameworks.

They are used to iterate through children, running each one, until a child node 
returns __SUCCESS__ or all children have returned __FAILURE__

The framework provides two nodes:

- Fallback
- ReactiveFallback

Although similar, the two nodes differ in one important way: 

- Fallback ticks children in a linear sequence.

- ReactiveFallback ticks children in a loop.  

## Shared Rules

They share the following rules:

- Before ticking the first child, the node status becomes __RUNNING__.

- If a child returns __FAILURE__, the fallback ticks the next child.

- If the __last__ child returns __FAILURE__ too, all the children are halted and
 the fallback returns __FAILURE__.
 
- If a child returns __SUCCESS__, it stops and returns __SUCCESS__.
  All the children are halted. 

## Difference in __RUNNING__ behavior

The following table describes the difference in control sequence when a child 
returns __RUNNING__:

| Type of ControlNode | Child returns RUNNING |
|---|:---:|
| Fallback | Tick again  |
| ReactiveFallback  |  Restart |

- "__Tick again__" means that on the next tick, the 
  same child is ticked again. Previous siblings
  are not ticked again.
  
- "__Restart__" means that the iteration through the sequence of children is 
  restarted on the next tick.

## Fallback



In this example, we try different strategies to open the door. 
Check first (and once) if the door is open.

![FallbackNode](images/FallbackSimplified.png)

??? example "See the pseudocode"
	``` c++
		// index is initialized to 0 in the constructor
		status = RUNNING;

		while( _index < number_of_children )
		{
			child_status = child[index]->tick();
			
			if( child_status == RUNNING ) {
				// Suspend execution and return RUNNING.
				// At the next tick, _index will be the same.
				return RUNNING;
			}
			else if( child_status == FAILURE ) {
				// continue the while loop
				_index++;
			}
			else if( child_status == SUCCESS ) {
				// Suspend execution and return SUCCESS.
   			    HaltAllChildren();
				_index = 0;
				return SUCCESS;
			}
		}
		// all the children returned FAILURE. Return FAILURE too.
		index = 0;
		HaltAllChildren();
		return FAILURE;
	```	

## ReactiveFallback

This ControlNode is used when you want to interrupt an __asynchronous__
child if one of the previous Conditions changes its state from 
FAILURE to SUCCESS.

In the following example, the character will sleep *up to* 8 hours. If he/she has fully rested, then the node `areYouRested?` will return SUCCESS and the asynchronous nodes `Timeout (8 hrs)` and `Sleep` will be interrupted.

In each loop, "areYouRested?" will be ticked. If it returns __FAILURE__, the "Timeout (8 Hours)" will then be ticked. If instead, "areYouRested?" returns __SUCCESS__, "Timeout (8 Hours)" will be interrupted by calling its halt() method. If "Timeout (8 Hours)" returns __SUCCESS__ at any point, all other nodes will be killed. 

![ReactiveFallback](images/ReactiveFallback.png)


??? example "See the pseudocode"
	``` c++
		status = RUNNING;

		for (int index=0; index < number_of_children; index++)
		{
			child_status = child[index]->tick();
			
			if( child_status == RUNNING ) {
				// Suspend all subsequent siblings and return RUNNING.
				HaltSubsequentSiblings();
				return RUNNING;
			}
			
			// if child_status == FAILURE, continue to tick next sibling
			
			if( child_status == SUCCESS ) {
				// Suspend execution and return SUCCESS.
   				HaltAllChildren();
				return SUCCESS;
			}
		}
		// all the children returned FAILURE. Return FAILURE too.
		HaltAllChildren();
		return FAILURE;
	```	


 

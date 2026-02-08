# VoidScript Language Specification

Version: 0.1

## Overview

VoidScript is a compiled language.

The compiler emits:
- native assembly (ASM)
- WebAssembly (WASM)
- Void Assembly bytecode (VASM)

## Void Assembly

Void Assembly (VASM) is an intermediate format that maps well to native assembly, WebAssembly, and Void Assembly bytecode.

Due to limitations of WebAssembly and low-level targets, VASM contains a custom instruction set.

### About Void Assembly

As mentioned above, due to platform limitations, Void Assembly uses a custom instruction set.

The instruction style is inspired by IC10 chip instructions from the game *Stationeers*.

### Instructions and Format
reg* = r0..99 <br>
valuetype* = `const int | const bool | const float | const enum | reg`

| Op 			| Arg1 				| Arg2 			| Arg3 								| Description 																											|
|---------------|-------------------|---------------|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| **add** 		| out reg 			| in valuetype 	| in valuetype 						| Arithmetic addition																									|
| **sub** 		| out reg 			| in valuetype 	| in valuetype 						| Arithmetic subtraction 																								|
| **div** 		| out reg 			| in valuetype 	| in valuetype 						| Arithmetic division 																									|
| **mul** 		| out reg 			| in valuetype 	| in valuetype 						| Arithmetic multiplication 																							|
| **rem** 		| out reg 			| in valuetype 	| in valuetype 						| Remainder / modulo 																									|
| **jump** 		| out label 		| — 			| — 								| Unconditional jump 																									|
| **jump** 		| in valuetype 		| out label 	| — 								| Conditional jump 																										|
| **select** 	| in valuetype 		| out valuetype | const valuetype[] \| valuetype... | Selects from constant array or variadic arguments. Type is inherited from the first element; all args must match. 	|
| **load** 		| out reg 			| in var 		| — 								| Load variable into register; may be optimized to move or discarded 													|
| **store** 	| in reg 			| out var 		| — 								| Store register into variable; may be optimized to move or discarded 													|
| **move** 		| out reg 			| in valuetype 	| — 								| Move value; discarded if source and destination are the same register 												|
| **copy** 		| in reg 			| in reg 		| in valuetype 						| Copy memory A[0..arg3] → B[0..arg3]; unsafe and unchecked 															|
| **fill** 		| in reg 			| in valuetype 	| - 								| Fill memory A[0..arg2]; unsafe and unchecked 																			|
| **new** 		| in reg 			| in valuetype 	| — 								| Allocate memory blob A[arg1]; size in QWORDs; leaks if not tracked 													|
| **free** 		| in reg 			| in valuetype 	| — 								| Free memory blob A[arg1]; size in QWORDs; leaks if not tracked 														|
| **cast** 		| in reg 			| out reg 		| — 								| Type-safe cast 																										|
| **rcast** 	| in reg 			| out reg 		| — 								| Reinterpret (bitwise) cast 																							|
| **min** 		| out reg 			| in valuetype 	| in valuetype 						| Minimum 																												|
| **max** 		| out reg 			| in valuetype 	| in valuetype 						| Maximum 																												|
| **nearest** 	| out reg 			| in valuetype 	| in valuetype 						| Nearest value 																										|
| **ceil** 		| out reg 			| in valuetype 	| — 								| Ceiling 																												|
| **floor** 	| out reg 			| in valuetype 	| — 								| Floor 																												|
| **trunc** 	| out reg 			| in valuetype 	| — 								| Truncate 																												|
| **abs** 		| out reg 			| in valuetype 	| — 								| Absolute value 																										|
| **neg** 		| out reg 			| in valuetype 	| — 								| Negation 																												|
| **sqrt** 		| out reg 			| in valuetype 	| — 								| Square root 																											|
| **copysign** 	| out reg 			| in valuetype 	| in valuetype 						| Copy sign from second operand 																						|
| **equal** 	| out reg 			| in valuetype 	| in valuetype 						| Equality comparison 																									|
| **nequal** 	| out reg 			| in valuetype 	| in valuetype 						| Inequality comparison 																								|
| **gthan** 	| out reg 			| in valuetype 	| in valuetype 						| Greater-than 																											|
| **lthan** 	| out reg 			| in valuetype 	| in valuetype 						| Less-than 																											|
| **gtequal** 	| out reg 			| in valuetype 	| in valuetype 						| Greater-than-or-equal 																								|
| **ltequal** 	| out reg 			| in valuetype 	| in valuetype 						| Less-than-or-equal 																									|
| **and** 		| out reg 			| in valuetype 	| in valuetype 						| Bitwise AND 																											|
| **or** 		| out reg 			| in valuetype 	| in valuetype 						| Bitwise OR 																											|
| **xor** 		| out reg 			| in valuetype 	| in valuetype 						| Bitwise XOR 																											|
| **lshift** 	| out reg 			| in valuetype 	| in valuetype 						| Logical left shift 																									|
| **rshift** 	| out reg 			| in valuetype 	| in valuetype 						| Logical right shift 																									|
| **yield** 	| — 				| — 			| — 								| No-op; wastes one CPU cycle if executed else it can be used to optimize a function 									|
| **ret** 		| — 				| — 			| — 								| Return from node/function 																							|
| **call** 		| in node \| event 	| args... 		| — 								| Call node or event; all args must be specified; use `_` to discard outputs 											|

## Void Script Format

A VoidScript program is composed of:
- libraries
- graphs
- enums
- structs

### Library

A library is a collection of static functions and global variables.

```
library <Name> <: BaseLibrary>
{
    asm ...
    macro ...
    node ...
    event ...
}
```

Libraries may contain:
- asm nodes (Void Assembly blocks)
- macro nodes (macros)
- node nodes (functions)
- event nodes (function pointers)
- `<type> <name> = <default>` global variables

### Graph

A graph is a collection of functions and variables.

Graphs are reference-counted shared objects.  
A graph is not copied unless `<Name>.Copy(graph)` is called.

Graphs must implement:
- Make
- Copy
- Destroy

Graphs support a special keyword `softref` when used as a variable; a softref does not keep a hard reference to another graph.

```
graph <Name> <: interface>
{
    asm ...
    macro ...
    node ...
    event ...
    node Make() {}
    node Copy(in <Name>) {}
    node Destroy() {}
}
```

Graphs may contain:
- asm nodes
- macro nodes
- node nodes
- event nodes
- `<type> <name> = <default>` variables

### Interface

An interface is a collection of function declarations without bodies.  
Interfaces may only be applied to graphs.

```
interface <Name>
{
    asm ...
    node ...
}
```

### Struct

A struct is a collection of variables.

Structs are copied by default unless passed by reference.

```
struct <Name>
{
}
```

Structs may contain:
- `<type> <name> = <default>` variables

### Enum

An enum is a collection of constant values.

Enums are copied by default.  
Enum values cannot be mixed between types.

```
enum <Name> : <float | int>
{
}
```

Enums may contain:
- `<name> = <default>` values

## Nodes

Nodes are defined as:

```
nodetype <name>(params...) <const> <pure>
```

### const

The `const` modifier restricts a function from writing to:
- graph variables
- struct variables
- global variables

`const` is currently a programmer hint only.  
The compiler does not enforce it yet but will emit warnings.

### pure

The `pure` modifier marks a function as pure:
- it does not execute by itself
- implicit `in exec` and `out exec then` are stripped
- it cannot modify variables

`pure` functions are limited by design.

`pure` may be applied to macro nodes, but macros marked as pure cant have exec type as in or out.

### Node Types

- **macro** — copy/paste graph expansion
- **event** — function pointer with no body
- **node** — normal function
- **asm** — Void Assembly core node

### Recursive Functions

Recursive functions are not allowed.

VoidScript does not have traditional stack frames due to WASM and Void Assembly constraints.

Recursion may only exist in external/native calls.

### Parameters

Node parameters follow this order:

1. direction (`in`, `out`)
2. optional `const`
3. type
4. optional `[]` (array, must be const)
5. optional `&` (pass by reference)
6. optional default value (`=`)

Arrays are always passed by reference.

The `&` ref modifier is recommended for struct parameters.

Example:

```
node FindItem(
    in itemType type = ItemType.Neutral,
    in const MyStruct[] arr = [],
    out MyStruct& result,
    out int ID
) const pure
```

### Node Execution Context

Non-pure nodes implicitly have:
- `in exec execute`
- `out exec then`

When used inside a graph, nodes also implicitly receive:
- `in graph self`

If a node is called outside of a graph context, the target graph must be explicitly specified, even if the node is pure.

## Node Body

Only **`node`** types may declare local variables using a `local {}` block.

**`macro`** types **must not** contain `local {}` blocks.

This restriction exists because macros are expanded inline and do not own storage. Any state created inside a macro must be explicitly constructed in the parent node scope.

### Local Variables (node only)

Nodes may declare local variables using a `local` block.

```
local
{
    <type> <name> = <default>
}
```

Rules:
- Default values are **mandatory**
- Local variables are owned by the node instance
- Local variables persist for the lifetime of the node execution context

### State Creation Inside Macros

Macros **cannot** declare local variables.

To create state inside a macro, use `<type>.Make()` explicitly.

```
node localVar = <type>.Make();
```

This creates a state variable in the **parent node scope**, not inside the macro.

This behavior is required due to how macros are expanded and how storage ownership works.

### Summary

- **node**
  - May declare `local {}` blocks
  - May call `<type>.Make()`

- **macro**
  - Must NOT declare `local {}`
  - May ONLY create state via `<type>.Make()`


## Calling a Node

Nodes are invoked explicitly by name using the following form:

```
node <result> = <Type>.<Name>(input arguments)
```

### Call Semantics

- The call creates a node invocation in the execution graph
- All input arguments must be explicitly specified
- Outputs must be explicitly captured or discarded
- There is no implicit evaluation or execution order

If an output value is not needed, it must be explicitly discarded using `_`.

### Discarding Outputs

When a node has multiple outputs, unused outputs must be explicitly discarded.

```
discard Math.Add(a, b);
```

or for multiple outputs:

```
node n = SomeNode(a, b);
discard n.val 
```

### Calling Nodes on Graph Instances

When calling a node that belongs to a graph instance, the target is automatically set to self.

```
node ovalue = myGraph.Compute(x, y);
```
can be overwriten by doing
```
node ovalue = myGraph.Compute(target,x, y);
```

If the call is made outside of a graph context, the target graph is always required, even if the node is marked as `pure` or `const`.

### Calling Pure Nodes

Pure nodes:
- have no implicit `exec` flow
- do not mutate state
- may be freely reordered by the compiler

```
node sum = Math.Add(a, b);
```

### Calling Impure Nodes

Non-pure nodes implicitly include execution flow:

- `in exec execute`
- `out exec then`

These connections are created implicitly.

- Node calls are always explicit
- All inputs must be provided
- All outputs must be captured or discarded
- Graph instance targets must be specified when required

## Execution Flow (exec)

Execution flow in VoidScript is explicit and graph-based.  
There is no implicit control flow, sequencing, or branching.

The `exec` type represents execution flow only and does not carry data.

### exec Pins and Shapes

only macro or asm node types, may define exec pins with the following forms:

- `in exec execute`  
  Single execution input.

- `in exec[] do`  
  Multiple execution inputs (fan-in).

- `out exec then`  
  Single execution output.

- `out exec[] then`  
  Multiple execution outputs (fan-out).

These shapes are explicit in the node signature and preserved in the execution graph.

### Implicit exec Pins

For **non-pure nodes**, the following exec pins are implicitly present unless overridden:

- `in exec execute`
- `out exec then`

If the node defines:
- `out exec[] then` → the implicit single `then` is replaced
- `out exec finaly` → a final execution path is added

Pure nodes:
- do NOT have exec pins
- do NOT participate in execution flow
- are evaluated only through data dependencies

### exec Arrays

Exec arrays (`exec[]`) represent:
- multiple entry points (`in exec[]`)
- or multiple exit points (`out exec[]`)

They are indexed, ordered, and explicitly connected.

Example conceptual usage:
- a `Sequence` node exposes `out exec[] then`
- a `Combine` node may expose `in exec[] do`

There is no implicit behavior associated with exec arrays beyond explicit connections.

### Explicit exec Blocks

nodes and macros may define **named exec blocks** corresponding to exec pins.

```
execute:
{
    // primary entry
}

do 0:
{
    // first entry
}

do 1:
{
    // second entry
}

finaly:
{
    // final execution path
}
```

Each block maps directly to an exec pin.

### Calling Nodes with exec

Calling a non-pure node creates an execution edge from the current exec context to the node’s `execute` pin.

```
SomeNode(a, b);
```

This implicitly connects the current exec flow to `SomeNode.execute`.

If the node exposes `in exec[] do`, the caller must explicitly select which entry index to connect.

### Calling Macro Nodes with exec

Macros may define:
- multiple exec inputs
- multiple exec outputs
- exec arrays
- final execution paths

When a macro is called:
- its exec flow is inlined
- all exec pins are resolved at expansion time

Example:

```
MyMacro(exec);
```

This expands the macro body and connects the current exec flow to the macro’s entry exec pin.

### Multiple exec Inputs (Merging)

Nodes and macros may have **multiple exec inputs**, including exec arrays.

Example:
- inputA → A → B
- inputB → B

Both exec flows explicitly connect to `B.execute` (or `B.do[i]`).

There is no implicit merge operation.  
Merging occurs naturally when multiple exec edges target the same exec pin.

### Explicit Goto

Execution flow may be redirected explicitly using `goto`.

```c
goto TargetNode.execute;
```

Rules:
- `goto` creates an explicit exec edge
- there is no fallthrough
- execution continues only at the target exec pin

`goto` may target:
- `<name>`
- `<naem>[TargetID]`

### Branching, Branching Completion and Termination

A somfing like a `branch` node may splits execution into multiple exec outputs, taking 1 input (for example  `in exec execute, in bool cnd , out exec true , out exec false`).

There is no implicit merge after a branch.

After a branch splits execution into multiple exec paths, each path proceeds independently. The compiler does **not** insert an automatic merge node.

Instead, the compiler tracks active exec paths:

- If an exec path reaches an explicit continuation (for example a `goto`, a node call, or another exec input), execution continues normally.
- If an exec path reaches the end of its scope with no outgoing exec connection, that path is considered **completed**.

When **all exec paths spawned from a branch are completed**, the compiler considers the sequencing finished.

- If there is an enclosing execution context, the compiler inserts an implicit `goto` to the nearest valid entry point.
- If no valid entry point exists, execution terminates and the program exits.
- No implicit merge semantics
- No dangling exec paths

Execution always ends explicitly or by exhaustion of all exec paths.

### Sequencing
Sequencing is explicit and typically expressed using nodes that expose `out exec[] then`.
Each index in `then[]` represents a distinct ordered execution output.

## Example
```cs
library bool
{
    asm Make(out bool value) pure
    {
        move r0, 0;
        store r0, value;
    }

    asm Make(in const bool const_value, out bool value) pure
    {
        move r0, const_value;
        store r0, value;
    }

    asm Assign(in bool var, in bool value)
    {
        load r0, var;
        store r0, value;
    }

    asm And(in bool A, in bool B, out bool value) pure
    {
        load r0, A;
        load r1, B;
        and r3, r0, r1;
        store r3, value;
    }

    asm Or(in bool A, in bool B, out bool value) pure
    {
        load r0, A;
        load r1, B;
        or r3, r0, r1;
        store r3, value;
    }

    asm Not(in bool A, out bool value) pure
    {
        load r0, A;
        not r1, r0;
        store r1, value;
    }
}

library int
{
    asm Make(out int value) pure
    {
        move r0, 0;
        store r0, value;
    }

    asm Make(in const int const_value, out int value) pure
    {
        move r0, const_value;
        store r0, value;
    }

    asm Assign(in int var, in int value)
    {
        load r0, var;
        store r0, value;
    }

    asm Add(in int A, in int B, out int value) pure
    {
        load r0, A;
        load r1, B;
        add r3, r0, r1;
        store r3, value;
    }

    asm Subtract(in int A, in int B, out int value) pure
    {
        load r0, A;
        load r1, B;
        sub r3, r0, r1;
        store r3, value;
    }

    asm Multiply(in int A, in int B, out int value) pure
    {
        load r0, A;
        load r1, B;
        mul r3, r0, r1;
        store r3, value;
    }

    asm Equal(in int A, in int B, out bool value) pure
    {
        load r0, A;
        load r1, B;
        equal r3, r0, r1;
        store r3, value;
    }

    asm LessThenEqual(in int A, in int B, out bool value) pure
    {
        load r0, A;
        load r1, B;
        ltequal r3, r0, r1;
        store r3, value;
    }
}

library FlowControl : global
{
	//core node executes code in sequence
	asm Sequence(in exec execute,out exec[Count] then)
	{
	execute:
		#for then.Count,i
		#coppy jump then[i];
		#end
        ret;
	}

	//core node if
	asm Branch(in exec execute,in bool Condition,out exec true,out exec false)
	{
	execute:
		load r0, Condition;
		jump r0, true;
		jump false;
	true:
		jump Branch.true;		    //link to exec var
        ret;
	false:
        jump Branch.false;		    //link to exec var
        ret;
	}

    macro ForLoopWithBreak(in exec execute,in int FirstIndex,in int LastIndex,in exec Break,out exec LoopBody,out int Index,out exec Completed)
    {
    	node index  = int.Make();
    	node broken = bool.Make(false);
    	node control = Branch(execute,bool.And(bool.Not(broken),int.LessThenEqual(index, LastIndex)))
    	{
    		true:
    		{
    			Sequence(control.true)
    			{
    				then 0:
    				{
    					goto LoopBody;
    				}
    				then 1:
    				{
    					index = int.Add(index, 1);
    					Index = index;
    					goto control.execute;
    				}
    			}
    		}
    		false:
    		{
    			goto Completed;
    		}
    	}
    	
    	execute:
    	{
    		index = FirstIndex;
    		Index = index;
    		goto control.execute;
    	}
    	Break:
    	{
    		broken = true;
    		goto control.execute;
    	}
    }

    macro While(in exec execute,in bool Condition,out exec LoopBody,out exec Completed)
    {
        node control = Branch(execute, Condition)
        {
            true:
            {
                Sequence(control.true)
                {
                    then 0:
                    {
                        goto LoopBody;
                    }
                    then 1:
                    {
                        goto control.execute;
                    }
                }
            }
            false:
            {
                goto Completed;
            }
        }
    
        execute:
        {
            goto control.execute;
        }
    }

    macro ForLoop(in exec execute,in int FirstIndex = 0,in int LastIndex = 1,out exec LoopBody,out int Index,out exec Completed)
    {
        node index = int.Make();
        node control = Branch(execute,int.LessThenEqual(index, LastIndex))
        {
            true:
            {
                Sequence(control.true)
                {
                    then 0:
                    {
                        goto LoopBody;
                    }
                    then 1:
                    {
                        index = int.Add(index, 1);
                        Index = index;
                        goto control.execute;
                    }
                }
            }
            false:
            {
                goto Completed;
            }
        }
    
        execute:
        {
            index = FirstIndex;
            Index = index;
            goto control.execute;
        }
    }

    macro FlipFlop(in exec execute,out exec A,out exec B,out bool IsA)
    {
        node state = bool.Make();
        node control = Branch(execute, state)
        {
            true:
            {
                goto A;
            }
            false:
            {
                goto B;
            }
        }
    
        execute:
        {
            state = bool.Not(state);
            IsA = state;
            goto control.execute;
        }
    }
```

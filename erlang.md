# Distributed Erlang

Building distributed systems in Erlang has support built directly into the Erlang Virtual Machine. Distributed systems are acheived in Erlang by employing nodes, which are named Erlang runtime systems. An Erlang runtime system is name by passing it the `--name` (long name), or `--sname`(short name) flag, however it should be noted that short name systems cannot communicate with long name systems and vice versa. 

e.g. if an Erlang system were started with the name foo `sys -name foo`, the built-in-function `node()` would report the name as an atom `:foo@gamma.csc.uvic.ca` with the pattern name@full_host_name. If a short name is used, `sys -sname foo`, then only the first portion of the host name is reported: `:foo@gamma`

Using Erlang's message passing functions in a distributed environment is very similar to using them within a single instance of a run time system; the same built in functions, such as spawn and receive are used, just with a different arity - an atom representing the node is included as a function parameter. When a pid is used, the use of nodes are completely transparent. Connections to nodes are transitive, that is, when a node connects to another node it will connect to all nodes that that node is connected to. To illustrate:
e.g. Let A, B and C be Nodes. Suppose B is connected to C, if A connects to B then A will also connect to C.
This behavior is enabled by deault, unless the system is started with the `-connect_all false` option. Nodes may be created that are 'hidden'. A hidden node will not be returned on a `nodes()` function, as an activly running node, and connections between standard nodes and hidden nodes are not transitive. Hidden nodes are useful when a running system needs to be inspected without disturbing it. C code can be interopted with running nodes by creating a C executable binary that makes use Erl_interface library, to allow it to behave as an erlang node (called a C Node).  [5].

Message passing between nodes in the vm is implemented over TCP sockets, but vm is extendable and documentation exists that provides a guide to creating a third party carrier driver to allow nodes to communicate differently. A driver is native code, written in c that is either statically linked when the vm is compiled (as is preferable when using an open source version), or is dynamically loaded in the the vm's address space (in the case of using a precompiled release). Drivers implement a number of callback function that provide read-write functions on file descriptors [6].

# Language features to support Concurrency

## Creating Processes
`spawn` has 1,2,3 and 4 arity versions, that allows a process to be spawned on another Node, executing a function defined in a particular module, with a set of arguments. In all variation of the function a pid primitive is returned, that can then be registered to a name, or used to send messages to.

## Receiving Data

``` erlang
receive
        Pattern1 [when GuardSeq1] ->
                Body1;
        %...;
        PatternN [when GuardSeqN] ->
                BodyN
after
        ExprT ->
                BodyT
end.
``` 

The `receive` primitive takes any number of patterns to match against, followed by by function bodies to excute if the pattern matches, alternatively with an additional guard statement (Guard statements are much like they are in Prolog, a boolean expression that will only allow the function body to execute if the expression resoluves to true.Guard statements, after, and even patterns are all optional. A receive statement goes through queued messages in the processes mailbox, and sequentially attempts to match the message with each pattern listed. When a pattern is matched the message is consumed and removed from the queue. A receive statement can block indefitnelty, to get around this an after statement can include an expression (ExprT in the example the will evaluate to an integer that represents the number of miliseconds until a timeoutoccurs.

## Sending Data
The ! (bang) operator is used to send data to a process

``` erlang
Expr1 ! Expr2
```

Expr1 must evaluate be a pid, registered named (atom), or a tuple { Name, Node }. Inline with Erlangs idea of fault tolerant applications, the send action will happily send messages to a dead process without reporting an error. Expr2 is the data to be sent to the process, that will be consumed and matched to a pattern in the the `receive` primitive [4].

## Error Handling
It's worth noting that there are a number of features built into the language to support error handling. Erlang allows processes to be 'linked' either with the `spawn_link` (same signature as spawn) or `link(pid)`. Normally, a process that encounters an error dies quitely, but a linked process's errors will bubble to the link procces. An alternative linking is to spawn 'monitors' which are processes that check the health of a monitored process, and when it dies, performs an action - such as restarting it [8].

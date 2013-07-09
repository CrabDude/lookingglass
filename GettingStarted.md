```
There is not much info available. These are the rough hints for you (use cs.chromium.org to navigate Blink source). Then we could chat over VC / hangouts / Skype.

[Compile]
InspectorController is a container for remote debugging protocol agents. We have one for the Page (InspectorController.cpp) and we have one for the workers (WorkerInspectorController.cpp that has limited functionality). Given that Node is more like a worker (pure JS execution environment), take a look at that file for the list of agents to compile in your module (InspectorDebuggerAgent, InspectorConsoleAgent, InspectorProfileAgent, etc.). So you would need to compile all of them.

These agents would pull ScriptDebugServer.cpp, ScriptProfiler.cpp, Script* etc. These are wrappers around v8, should compile against v8.h. Some of them (ScriptProfiler) and most of the agents will reference Node and that's what we need to fix. In fact, most of them will pull Page, Document, Node for no good reason and that's what we need to fix upstream, in Blink.

You would also need to generate InspectorBackendDispatcher and InspectorFrontend from devtools/protocol.json. First one will parse incoming JSON messages and will dispatch them on controller's agents, second will generate typed API for agents to send messages back. Some tweaking would be necessary here as well since currently we generate a giant dispatch and giant front-end instead of per-domain files. Having them per-domain would let you only take what you really need.

[Wire]
Lets assume it all compiled. Now you need to instantiate this controller and call dispatchMessageFromFrontend on it with every message from the front-end. Front-end will issue them using web socket transport, so you need to have a server web socket as a part of your Node module that would accept connection and do the right thing to the messages (call dispatchMessageFromFrontend). And for the way back, your m_frontendChannel would need to send info back using that web socket.

[Debugging]
After compiling and wiring, console will start working. But you also need to make things work while on breakpoint. For that, you would need to implement runMessageLoopOnPause in your version of WorkerScriptDebugServer.

As I mentioned, it is quite some work both downstream (in the Node module) and upstream (to remove poor Blink dependencies from agents). But it might be worth it.

Regards
Pavel
```
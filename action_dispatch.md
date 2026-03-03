# `ActionDispatch`

## ==Action Pack -- From request to response==

Action Pack is a framework for handling and responding to web requests. It provides mechanisms for _routing_ (mapping request URLs to actions), defining _controllers_ that implement actions, and generating responses. In short, Action Pack provides the controller layer in the MVC paradigm.

It consists of several modules:

* **Action Dispatch**, which parses information about the web request, handles routing as defined by the user, and does advanced processing related to HTTP such as MIME-type negotiation, decoding parameters in POST, PATCH, or PUT bodies, handling HTTP caching logic, cookies and sessions.
* **Action Controlle**r, which provides a base controller class that can be subclassed to implement filters and actions to handle requests. ==The result of an action is typically content generated from views.==

==Rails users only directly interface with the Action Controller module.== Necessary Action Dispatch functionality is activated by default and Action View rendering is implicitly triggered by Action Controller.

## ActionDispatch

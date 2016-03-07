# General Conventions

## Packets

> The following section is tentative, and reflects my intent to refactor the
> debugger protocol to adhere to the JSON-RPC 2.0 specification. This needs to
> be agreed on by the rest of the team before we move forward.
>
> JSON-RPC 2.0 has the advantage that it is standard, so there is more tooling
> available for it. It also helps to make our protocol match that of chrome`s
> more closely (though not completely). In addition, each request/response pair
> has a unique identifier, freeing the client from the responsibility of
> matching up each response with a request.

The debugger protocol adheres to the JSON-RPC 2.0 specification.

Each request has the form:

    {
      "jsonrpc": "2.0",
      "id": id,
      "method": method,
      "params": params
    }

where `id` is an identifier for the request, `method` is the name of the method
to be called, and `params` is an object to be passed as argument.

The `params` object has the form:

    {
      "to": actor,
      ...
    }

where `actor` is the identifier for the actor on which the method is to be
called. If the method is to be called on the root actor, `actor` must be
omitted. The `params` object may contain additional properties, depending on the
method to be called.

Each response has one of two forms. If the request was handled succesfully,
the response has the form:

    {
      "jsonrpc": "2.0",
      "id": id,
      "result": result
    }

where `id` is the identifier for the request, and `result` is the value returned
by the call.

If the request failed for some reason, the response instead has the form:

    {
      "jsonrpc": "2.0",
       "id": id,
      "error": error
    }

where `id` is the identifier for the request, and `error` is an object
representing the error that was thrown.

The `error` object has the form:

    {
      "name": name,
      "message": message
    }

where `name` is a name for the type of error, and `message` is a human-readable
description of the error.

> There exists a straightforward mapping from packets in the existing debugger
> protocol to JSON-RPC 2.0. For request packets, we can simply use the
> existing packet for the `params` property in a JSON-RPC 2.0 packet, and use
> the `type` property in the existing packet for the `method` property in the
> JSON-RPC 2.0 packet.
>
> For response packets, we can simply use the existing packet for the `result`
> property in a JSON-RPC 2.0 packet. If the response is an error packet
> (indicated by the presence of an `error` property), we use the existing packet
> for the `error` property.

# The Root Actor

To get a list of tabs in the browser, a client sends a request of the form:

    {
      "jsonrpc": "2.0",
      "id": id,
      "method": "Root.getTabs"
    }

The server replies by sending a response of the form:

    {
      "jsonrpc": "2.0",
      "id": id,
      "result": {
        "tabs": [tab, ...],
        "selected": selected
      }
    }

where each `tab` describes a single tab, and `selected` is the index of the
currently selected tab in the array of tabs.

Each `tab` has the form:

    {
      "title": title,
      "url": url
    }

where `title` and `url` are the title and url of the currently displayed
document in the tab.

> Is this request actually useful? It is used heavily in most of our tests, but
> the debugger itself seems to primarily rely on the getTab request, which
> obtains a tabActor for a specific tab. I can`t think of any scenarios where
> being able to list all tabs is actually useful to us.

> If we do want to be able to list all tabs, what happens if the currently
> displayed document in a tab changes? The server sends a one-shot notification
> when a new tab appears or an existing tab is closed. This allows the client to
> request an up to date list of tabs. However, the server does not send this
> notification when the currently displayed document in a tab changes.

# Coordinator

![[README#^95df96]]

In the code, the coordinator is the leader. It organizes the master channel, slave
channels and related data structures for master channel and slave channel.

## Data Structures

```python
profile: str = "default"

mutex: threading.Lock = threading.Lock()

master: MasterChannel

slaves: Dict[ModuleID, SlaveChannel] = dict()

middlewares: List[Middleware] = list()

master_thread: Optional[threading.Thread] = None

slave_threads: Dict[ModuleID, threading.Thread] = dict()
```

As we can see, coordinator maintains some important data structure for the whole EFB framework.

## Functions

### Register Related

It provides `add_channel` function which will register the channel.

```python
def add_channel(channel: Channel):
    global master, slaves
    if isinstance(channel, MasterChannel):
        master = channel
    if isinstance(channel, SlaveChannel):
        slaves[channel.channel_id] = channel
    else:
        raise TypeError("Channel instance is expected")
```

Here, we can conclude many things:

1. There is only one master channel in the EFB. See [[README#^2b0b56|Master Channel Definition]].
2. There could be many slave channels in EFB. See [[README#^d26159|Slave Channel Definition]].
3. The channel id for slave channels must be unique.

Also, it provides `add_middlerware` function which will register the middleware.

```python
def add_middleware(middleware: Middleware):
    global middlewares
    if isinstance(middleware, Middleware):
        middlewares.append(middleware)
    else:
        raise TypeError("Middleware instance is expected")
```

And we could use id to get the corresponding module:

```python
def get_module_by_id(module_id: ModuleID) -> Union[Channel, Middleware]:
    with suppress(NameError):
        if master.channel_id == module_id:
            return master
        if module_id in slaves:
            return slaves[module_id]
        for i in middlewares:
            if i.middleware_id == module_id:
                return i
        raise NameError("Module ID {} is not found".format(module_id))
```

We could know that for channels and middlewares, the id of them should be unique, thus we could identify.

Now it provides a wonderful method for sending messages both for master channel and slave channel.

```python
def send_message(msg: 'Message') -> Optional['Message']:
    global middlewares, master, slaves

    if msg is None:
        return

    for i in middlewares:
        m = i.process_message(msg)
        if m is None:
            return None
        msg = m

    msg.verify()

    if msg.deliver_to.channel_id == master.channel_id:
        return master.send_message(msg)
    elif msg.deliver_to.channel_id in slaves:
        return slaves[msg.deliver_to.channel_id].send_message(msg)
    else:
        raise EFBChannelNotFound()
```

From above code, we can see `send_message` will do the following things:

1. It will process `msg` for every middleware and verify it at last.
2. Send the message to the corresponding channel.

At last, coordinator provides `send_status` method.

```python
def send_status(status: 'Status'):
    global middlewares, master

    if status is None:
        return

    s: 'Optional[Status]' = status

    for i in middlewares:
        s = i.process_status(cast('Status', s))
        if s is None:
            return
        status = cast('Status', s)

    status.verify()

    status.destination_channel.send_status(status)
```

## Reference

1. [coordinator.py source code](https://github.com/ehForwarderBot/ehForwarderBot/blob/master/ehforwarderbot/coordinator.py)

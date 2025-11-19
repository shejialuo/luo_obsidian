# Channel

There are two kinds of channel in ehForwarderBot.

+ [[OpenSource/ehForwarderBot/README#^2b0b56|Master Channel]].
+ [[OpenSource/ehForwarderBot/README#^d26159|Slave Channels]].

The code first defines the abstract class `Channel` like the following:

```python
class Channel(ABC):
    channel_name: str = "Empty channel"
    channel_emoji: str = ""
    channel_id: ModuleID = ModuleID("efb.empty_channel")
    instance_id: Optional[InstanceID] = None

    def __init__(self, instance_id: InstanceID = None):
        if instance_id:
            self.instance_id = InstanceID(instance_id)
            self.channel_id = ModuleID(self.channel_id + "#" + instance_id)

    @abstractmethod
    def send_message(self, msg: 'Message') -> 'Message':
        raise NotImplementedError()

    @abstractmethod
    def poll(self):
        raise NotImplementedError()

    @abstractmethod
    def send_status(self):
        raise NotImplementedError()

    @abstractmethod
    def stop_polling(self):
        raise NotImplementedError()

    @abstractmethod
    def get_message_by_id(self, chat: 'Chat', msg_id: MessageID):
        raise NotImplementedError()
```

The EFB framework will call `poll` method for each channel in each thread.

<div align="center">
  
  ![Memphis light logo](https://github.com/memphisdev/memphis-broker/blob/staging/logo-white.png?raw=true#gh-dark-mode-only)
  
</div>

<div align="center">
  
  ![Memphis light logo](https://github.com/memphisdev/memphis-broker/blob/staging/logo-black.png?raw=true#gh-light-mode-only)
  
</div>

<div align="center">
<h1>A Powerful Messaging Platform For Devs</h1>
<a target="_blank" href="https://twitter.com/intent/tweet?text=Probably+The+Easiest+Message+Broker+In+The+World%21+%0D%0Ahttps%3A%2F%2Fgithub.com%2Fmemphisdev%2Fmemphis-broker+%0D%0A%0D%0A%23MemphisDev"><img src="https://user-images.githubusercontent.com/70286779/174467733-e7656c1e-cfeb-4877-a5f3-1bd4fccc8cf1.png" width="60"></a> 
</div>
 
 <p align="center">
  <a href="https://memphis.dev/docs/">Docs</a> - <a href="https://twitter.com/Memphis_Dev">Twitter</a> - <a href="https://www.youtube.com/channel/UCVdMDLCSxXOqtgrBaRUHKKg">YouTube</a>
</p>

<p align="center">
<a href="https://discord.gg/WZpysvAeTf"><img src="https://img.shields.io/discord/963333392844328961?color=6557ff&label=discord" alt="Discord"></a> <a href=""><img src="https://img.shields.io/github/issues-closed/memphisdev/memphis-broker?color=6557ff"></a> <a href="https://github.com/memphisdev/memphis-broker/blob/master/CODE_OF_CONDUCT.md"><img src="https://img.shields.io/badge/Code%20of%20Conduct-v1.0-ff69b4.svg?color=ffc633" alt="Code Of Conduct"></a> <a href="https://github.com/memphisdev/memphis-broker/blob/master/LICENSE"><img src="https://img.shields.io/github/license/memphisdev/memphis-broker?color=ffc633" alt="License"></a> <img alt="GitHub release (latest by date)" src="https://img.shields.io/github/v/release/memphisdev/memphis-broker?color=61dfc6"> <img src="https://img.shields.io/github/last-commit/memphisdev/memphis-broker?color=61dfc6&label=last%20commit">
</p>

**[Memphis{dev}](https://memphis.dev)** is a Go-based message broker for developers made out of devs' struggles develop around message brokers.<br>Enables devs to achieve all other message brokers' benefits in a fraction of the time.<br>
Focusing on automatic optimization, schema management, inline processing,  and troubleshooting abilities. All under the same hood.
Utilizing NATS core.
## Installation

```sh
$ pip3 install memphis-py
```

## Importing

```python
from memphis import Memphis
from memphis import retention_types, storage_types
```

### Connecting to Memphis

First, we need to create Memphis `object` and then connect with Memphis by using `memphis.connect`.

```python
async def main():
  try:
    memphis = Memphis()
    await memphis.connect(
      host="<memphis-host>",
      username="<application-type username>",
      connection_token="<broker-token>",
      port="<port>", # defaults to 6666
      reconnect=True, # defaults to True
      max_reconnect=3, # defaults to 3
      reconnect_interval_ms=1500, # defaults to 1500
      timeout_ms=1500 # defaults to 1500
      )
    ...
  except Exception as e:
    print(e)
  finally:
    await memphis.close()

if __name__ == '__main__':
  asyncio.run(main())
```

Once connected, the entire functionalities offered by Memphis are available.

### Disconnecting from Memphis

To disconnect from Memphis, call `close()` on the memphis object.

```python
await memphis.close()
```

### Creating a Factory

```python
factory = memphis.factory(name="<factory-name>", description="")
```

### Destroying a Factory

Destroying a factory will remove all its resources (stations/producers/consumers)

```python
factory.destroy()
```

### Creating a Station

```python
station = memphis.station(
  name="<station-name>",
  factory_name="<factory-name>",
  retention_type=retention_types.MAX_MESSAGE_AGE_SECONDS, # MAX_MESSAGE_AGE_SECONDS/MESSAGES/BYTES. Defaults to MAX_MESSAGE_AGE_SECONDS
  retention_value=604800, # defaults to 604800
  storage_type=storage_types.FILE, # torage_types.FILE/torage_types.MEMORY. Defaults to MEMORY
  replicas=1, # defaults to 1
  dedup_enabled=False, # defaults to false
  dedup_window_ms: 0, # defaults to 0
)
```

### Retention types

Memphis currently supports the following types of retention:

```python
memphis.retention_types.MAX_MESSAGE_AGE_SECONDS
```

Means that every message persists for the value set in retention value field (in seconds)

```python
memphis.retention_types.MESSAGES
```

Means that after max amount of saved messages (set in retention value), the oldest messages will be deleted

```python
memphis.retention_types.BYTES
```

Means that after max amount of saved bytes (set in retention value), the oldest messages will be deleted

### Storage types

Memphis currently supports the following types of messages storage:

```python
memphis.storage_types.FILE
```

Means that messages persist on the file system

```python
memphis.storage_types.MEMORY
```

Means that messages persist on the main memory

### Destroying a Station

Destroying a station will remove all its resources (producers/consumers)

```python
station.destroy()
```

### Produce and Consume messages

The most common client operations are `produce` to send messages and `consume` to
receive messages.

Messages are published to a station and consumed from it by creating a consumer.
Consumers are pull based and consume all the messages in a station unless you are using a consumers group, in this case messages are spread across all members in this group.

Memphis messages are payload agnostic. Payloads are `Uint8Arrays`.

In order to stop getting messages, you have to call `consumer.destroy()`. Destroy will terminate regardless
of whether there are messages in flight for the client.

### Creating a Producer

```python
producer = await memphis.producer(station_name="<station-name>", producer_name="<producer-name>")
```

### Producing a message

```python
await prod.produce(
  message=bytearray(msg, 'utf-8')), # Uint8Arrays
  ack_wait_sec=15, # defaults to 15
```

### Destroying a Producer

```python
producer.destroy()
```

### Creating a Consumer

```python
consumer = await memphis.consumer(
  station_name="<station-name>",
  consumer_name="<consumer-name>",
  consumer_group="<group-name>", # defaults to the consumer name
  pull_interval_ms=1000, # defaults to 1000
  batch_size=10, # defaults to 10
  batch_max_time_to_wait_ms=5000, # defaults to 5000
  max_ack_time_ms=30000, # defaults to 30000
  max_msg_deliveries=10, # defaults to 10
)
```

### Processing messages

Once all the messages in the station were consumed the msg_handler will receive error: `Memphis: TimeoutError`.

```python
async def msg_handler(msgs, error):
  for msg in msgs:
    print("message: ", msg.get_data())
    await msg.ack()
  if error:
    print(error)
consumer.consume(msg_handler)
```

### Acknowledge a message

Acknowledge a message indicates the Memphis server to not re-send the same message again to the same consumer / consumers group

```python
await message.ack()
```

### Destroying a Consumer

```python
consumer.destroy()
```

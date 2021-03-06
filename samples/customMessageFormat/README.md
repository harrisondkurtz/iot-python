# Custom Message Format Sample
Sample code demonstrating how to send and process events in a custom format.  This sample extends the HelloWorld sample but uses a custom message formar for the events.

## Building the custom codec
The custom codec module must implement two methods: encode and decode.  The encode method must return a string representation of the message and accept two parameters:
* a Python dict containing the data to be encoded
* an optional timestamp for the event that will be transmitted as part of the encoded message.

```python
def encode(data=None, timestamp=None):
    return data['hello'] + "," + str(data['x'])
```

The decode message must be able to convert the string generated by encode() into an ibmiotf.Message object instance.  In this simple example you can see that no timestamp information is transmitted in the event, the decoder simply uses the current time as the timestamp for the message.

```python
def decode(message):
    (hello, x) = message.payload.split(",")
    
    data = {}
    data['hello'] = hello
    data['x'] = x
    
    timestamp = datetime.now(pytz.timezone('UTC'))
    
    return Message(data, timestamp)
```

## Registering the custom codec with the application client
Using your custom message format encoder is simple.  the setMessageEncoderModule() method in both the Application and Device clients allows you to define a different module for each message format string.

```python
# Initialize the application client.
try:
	appCli = ibmiotf.application.Client(appOptions)
except Exception as e:
	print(str(e))
	sys.exit()

# Connect and configuration the application
# - subscribe to live data from the device we created, specifically to "greeting" events
# - use the myAppEventCallback method to process events
appCli.setMessageEncoderModule("custom", myCustomCodec)
appCli.connect()
appCli.subscribeToDeviceEvents(deviceOptions['type'], deviceOptions['id'], "greeting")
appCli.deviceEventCallback = myAppEventCallback
```


## Registering the custom codec with the device client
```python
# Initialize the device client.
try:
	deviceCli = ibmiotf.device.Client(deviceOptions)
	deviceCli.setMessageEncoderModule("custom", myCustomCodec)
except Exception as e:
	print(str(e))
	sys.exit()
```

## Usage

```
me@localhost ~ $ python customCodecSample.py
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:27: hello=world x=0
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:28: hello=world x=1
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:29: hello=world x=2
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:30: hello=world x=3
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:31: hello=world x=4
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:32: hello=world x=5
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:33: hello=world x=6
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:34: hello=world x=7
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:35: hello=world x=8
Received live data from ca72af10ef14 (helloWorldDevice) sent at 19:33:36: hello=world x=9
```


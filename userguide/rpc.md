# RPC - Remote Procedure Calls

RPC means Remote Procedure Call. This is the way to send commands to devices
and receive replies, i.e. call remote procedures.
Mongoose OS uses
[JSON-RPC 2.0](https://en.wikipedia.org/wiki/JSON-RPC).

Since the RPC mechanism uses JSON-RPC, that means that the "procedure",
or an "RPC service" that device implements, is a function written in C/C++
or JavaScript with the following properties:

- Has a name, for example `GPIO.Toggle`,
- Takes a `params` JSON object,
- Returns a `result` JSON object.

For example, an RPC request to set GPIO pin 2 to high voltage looks like this:

```json
{ "id": 1932, "method": "GPIO.Write", "params": {"pin": 2, "value": 1} }
```

The successful reply from a device looks like this:

```json
{ "id": 1932, "result": true }
```

The failure reply looks like this:

```json
{ "id": 1932, "error": { "code": 400, "message": "error setting pin mode"} }
```

Note that the `"jsonrpc": "2.0"` attribute in the request frame can be omitted.

Mongoose OS libraries implement a large set of ready-to-go RPC services,
like managing hardware peripherals (GPIO, SPI, I2C), managing files,
remote updates, etc. It is easy to add custom RPC services too,
see "RPC Core" library in the
[RPC Services](../api/rpc.md) section for more details.

The JSON-RPC messages could be carried out by many different channels:
serial (UART), HTTP/Restful, WebSocket, MQTT, Bluetooth. RPC API allows
to add support for other channels. `mos` tool provides an easy way to call
device's RPC services over the serial connection or over the network.
Alternatively, RPC services can be invoked programmatically.

That means, you can write a function in C or JavaSciprt,
and call it remotely via multiple protocols. Example: 
an RPC service `Sum` that adds two numbers `a` and `b`:

```javascript
RPC.addHandler('Sum', function(args) {
  return args.a + args.b;
});
```

Here is how you can call it:

#### Via serial connection, if a device is attached to a serial port:
<pre class="command-line language-bash" data-user="chris" data-host="localhost" data-output="2"><code>mos --port /dev/ttyUSB0 call Sum '{"a":1, "b": 2}'
3</code></pre>

#### Via WebSocket, if a device is in the local network:
<pre class="command-line language-bash" data-user="chris" data-host="localhost" data-output="2"><code>mos --port ws://192.168.0.206/rpc call Sum '{"a":1, "b": 2}'
3</code></pre>

#### Via RESTful call, if a device is in the local network:
<pre class="command-line language-bash" data-user="chris" data-host="localhost" data-output="2"><code>curl -d '{"a":1, "b": 2}' 192.168.0.206/rpc/Sum
3</code></pre>

#### Via an MQTT server, talking to an device idendified by its ID esp32_6732ac:
<pre class="command-line language-bash" data-user="chris" data-host="localhost" data-output="2"><code>mos --port mqtt://my.mqtt.server:1883/esp32_6732ac call Sum '{"a":1, "b": 2}'
3</code>
or 
mosquitto_pub -h my.mqtt.server -t esp32_6732ac/rpc -m '{"src":"out-topic","id":123,"method":"sum","args":{"a": 1, "b":2}}'
</pre>

To see a list of all RPC services implemented by a device, call `RPC.List`:

<pre class="command-line language-bash" data-user="chris" data-host="localhost" data-output="2-100"><code>mos call RPC.List
[
  "I2C.WriteRegW",
  "I2C.WriteRegB",
  "I2C.ReadRegW",
  "I2C.ReadRegB",
  ...</code></pre>


## Remote management

Mongoose OS has many built-in RPC services: for managing files, accessing
hardware peripherals, performing over-the-air updates, etc. In fact,
the majority of `mos` command talks to a device via an RPC call.

See [Secure remote device management with Mongoose OS and AWS IoT](https://mongoose-os.com/blog/secure-remote-device-management-with-mongoose-os-and-aws-iot-for-esp32-esp8266-ti-cc3200-stm32/)
for an example of remote management over the secure AWS IoT cloud. In that
case, Mongoose OS uses secure MQTT RPC channel, authenticated via the
mutual TLS - per AWS IoT standards.

An ability to call RPC services via network add an incredible level of
control, implementing remote management capabilities at the highest levels.

Example: scan I2C bus and return addresses of I2C peripherals from a device
at IP address 192.168.1.4 using `mos` tool in a command line mode:

<pre class="command-line language-bash" data-user="chris" data-host="localhost" data-output="2-100"><code>mos --port ws://192.168.1.4/rpc call I2C.Scan
[
  31
]</code></pre>

Example: connect to a remote device via the MQTT server and manage it
using `mos` tool Web UI:

![](images/mos3.gif)


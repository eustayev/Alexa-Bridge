# ha-bridge
Emulates Philips Hue api to other home automation gateways such as an Amazon Echo.  The Bridge has helpers to build devices for the gateway for the Logitech Harmony Hub, Vera, Vera Lite or Vera Edge. Alternatively the Bridge supports custom calls as well. The Bridge handles basic commands such as "On", "Off" and "brightness" commands of the hue protocol. 
## Build
To customize and build it yourself, build a new jar with maven:  
```
mvn install
```
Otherwise, downloads are available at https://github.com/bwssytems/ha-bridge/releases.  
## Run
Then locate the jar and start the server with:  
```
java -jar -Dvera.address=X.Y.Z.A -Dharmony.address=X.Y.Z.A -Dharmony.user=myself -Dharmony.pwd=passwd ha-bridge-0.X.Y.jar
```
## Available Arguments
### -Dvera.address=`<ip address>`
The argument for the vera address should be given as it the system does not have a way to find the address. Supply -Dvera.address=X.Y.Z.A on the command line to provide it. If a vera is not used, do not set it.
### -Dupnp.config.address=`<ip address>`
The server defaults to the first available address on the host. Replace the -Dupnp.config.address=`<ip address>` value with the server ipv4 address you would like to use as the address that any upnp device will call after discovery. 
### -Dserver.port=`<port>`
The server defaults to running on port 8080. If you're already running a server (like openHAB) on 8080, -Dserver.port=`<port>` on the command line.
### -Dupnp.device.db=`<filepath>`
The default location for the db to contain the devices as they are added is "data/devices.db". If you would like a different filename or directory, specify -Dupnp.devices.db=`<directory>/<filename> or <filename>` if it is the same directory.
### -Dupnp.response.port=`<port>`
The upnp response port that will be used. The default is 50000.  
### -Dharmony.address=`<ip address>`
The argument for the Harmony Hub address should be given as the system does not have a way to find the address. Supply -Dharmony.address=X.Y.Z.A on the command line to provide it. If a Harmony Hub is not used, do not set it.
### -Dharmony.user=`<username>`
The user name of the MyHarmony.com account for the Harmony Hub. This needs to be given if you are using the Harmony Hub Features, provide -Dharmony.user=`<username>` on the command line.
### -Dharmony.pwd=`<password>`
The password for the user name of the MyHarmony.com account for the Harmony Hub. This needs to be given if you are using the Harmony Hub Features, provide -Dharmony.pwd=`<password>` on the command line.
### -Dupnp.strict=`<true|false>`
Upnp has been very closed on this platform to try and respond as a hue and there is now a setting to control if it is more open or strict, Add -Dupnp.strict=`<true|false>` to your command line to have the emulator respond to what it thinks is an echo to a hue or any other device. The default is upnp.strict=true.
### -Dtrace.upnp=`<true|false>`
Turn on tracing for upnp discovery messages. The default is false.
## Web Config
Configure by going to the url for the host you are running on or localhost with port you have assigned: 
```
http://<ip address>:<port>
```
## Configuration REST API usage
This section will describe the REST api available for configuration. The REST body examples are all formatted for easy reading, the actual body usage should be like this:
```
{"var1":"value1","var2":"value2","var3:"value3"}
```
The body should be all in one string and not separated by returns, tabs or spaces. FYI, GET items do not require a body element.  If you would like to see example return of json data for full Harmony Hub configuration if configured, which includes activities and devices, take a look at the resource file config.data. If you are interested in how the json data looks for the HA bridge configuration, after creating a device, look at the data directory for the device.db.
These calls can be accomplished with a REST tool using the following URLs and HTTP Verb types:
### Add a device 
Add a new device to the HA Bridge configuration. This is the basic examples and the next 3 headinds describe alternate items to add. 
```
POST http://host:8080/api/devices
{
"name" : "bedroom light",
"deviceType" : "switch",
  "onUrl" : "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=1&DeviceNum=41",
  "offUrl" : "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=41"
}
```
### Dimming and value passing control
Dimming is also supported by using the expressions ${intensity.percent} for 0-100 or ${intensity.byte} for 0-255 or custom values using ${intensity.math(<your expression using "X" as the value to operate on>)} i.e. "${intensity.math(X/4)}".    
e.g.
```
POST http://host:8080/api/devices
{
    "name": "entry light",
    "deviceType": "switch",
    "offUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=31",
    "onUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&DeviceNum=31&serviceId=urn:upnp-org:serviceId:Dimming1&action=SetLoadLevelTarget&newLoadlevelTarget=${intensity.percent}"
}
```
See the echo's documentation for the dimming phrase.

### POST/PUT support
added optional fields
 * contentType (currently un-validated)
 * httpVerb (POST/PUT/GET only supported)
 * contentBody your post/put body for onUrl here
 * contentBodyOff your post/put body for offUrl here

This will allow control of any other application that may need more then GET.  You can also use the dimming and value control commands within the URLs as well.
e.g: 
```
POST http://host:8080/api/devices
{
    "name": "test device",
    "deviceType": "switch",
    "offUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=31",
    "onUrl": "http://192.168.1.201:3480/data_request?id=action&output_format=json&DeviceNum=31&serviceId=urn:upnp-org:serviceId:Dimming1&action=SetLoadLevelTarget&newLoadlevelTarget=${intensity.percent}",
  "contentType" : "application/json",
  "httpVerb":"POST",
  "contentBody" : "{\"fooBar\":\"baz_on\"}"
  "contentBodyOff" : "{\"fooBar\":\"baz_off\"}"
}
```
### Custom Usage URLs
Anything that takes an action as a result of an HTTP request will probably work and you can also use the dimming and value control commands within the URLs as well - like putting Vera in and out of night mode:  
```
POST http://host:8080/api/devices
{
  "name": "night mode",
  "deviceType": "switch",
  "offUrl": "http://192.168.1.201:3480/data_request?id=lu_action&serviceId=urn:micasaverde-com:serviceId:HomeAutomationGateway1&action=SetHouseMode&Mode=1",
  "onUrl": "http://192.168.1.201:3480/data_request?id=lu_action&serviceId=urn:micasaverde-com:serviceId:HomeAutomationGateway1&action=SetHouseMode&Mode=3"
}
```
### Update a device 
Update an existing device using it's ID that was given when the device was created and the update could contain any of the fields that are used and shown in the previous examples when adding a device. 
```
POST http://host:8080/api/devices/`<id>`
{
"name" : "bedroom light",
"deviceType" : "switch",
  "onUrl" : "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=1&DeviceNum=41",
  "offUrl" : "http://192.168.1.201:3480/data_request?id=action&output_format=json&serviceId=urn:upnp-org:serviceId:SwitchPower1&action=SetTarget&newTargetValue=0&DeviceNum=41"
}
```
### Get all devices 
Get all devices saved in the HA bridge configuration. 
```
GET http://host:8080/api/devices
```
### Get a specific device 
Get a device by ID assigned from creation and saved in the HA bridge configuration. 
```
GET http://host:8080/api/devices/`<id>'
```
### Delete a specific device 
Delete a device by ID assigned from creation and saved in the HA bridge configuration. 
```
DELETE http://host:8080/api/devices/`<id>'
```
### Get HA Bridge Version 
Get current version of the HA bridge software. 
```
GET http://host:8080/api/devices/habridge/version
```
### Get Vera Devices 
Get the list of devices available from the Vera, VeraLite or VeraEdge if configured. Please refer to the <a href="http://wiki.micasaverde.com/index.php/Luup_Sdata">Luup Sdata Structure</a> for the explanation of the devices list returned.
```
GET http://host:8080/api/devices/vera/devices
```
### Get Vera Scenes 
Get the list of scenes available from the Vera, VeraLite or VeraEdge if configured.  Please refer to the <a href="http://wiki.micasaverde.com/index.php/Luup_Sdata">Luup Sdata Structure</a> for the explanation of the scenes list returned.
```
GET http://host:8080/api/devices/vera/scenes
```
### Get Harmony Activities
Get the list of activities available from the Harmony Hub if configured. 
```
GET http://host:8080/api/devices/harmony/activities
```
### Get Harmony Devices
Get the list of devices available from the Harmony Hub if configured. 
```
GET http://host:8080/api/devices/harmony/devices
```
### Show Harmony Current Activity
Show the Harmony Hub's current activity.
```
GET http://host:8080/api/devices/harmony/show
```
## HUE REST API usage
This section will describe the REST api available for controlling the bridge based off of the HUE API. This Bridge does not support the full HUE API, only the calls that are supported with the HA Bridge are shown. The REST body examples are all formatted for easy reading, the actual body usage should be like this:
```
{"var1":"value1","var2":"value2","var3:"value3"}
```
### Get all lights
GET	http://host:port/api/<username>/lights
#### Description
Gets a list of all lights that have been discovered by the bridge.

#### Response
Returns a list of all lights in the system. 

If there are no lights in the system then the bridge will return an empty object, {}.

#### Response example
```
{
    "1": {
        "state": {
            "on": true,
            "bri": 144,
            "hue": 0,
            "sat": 254,
            "xy": [0,0],
            "ct": 0,
            "alert": "none",
            "effect": "none",
            "colormode": "",
            "reachable": true
        },
        "type": "Dimmable light",
        "name": "Table Lamp 1",
        "modelid": "LWB004",
        "swversion": "65003148",
        "pointsymbol": {
            "1": "none",
            "2": "none",
            "3": "none",
            "4": "none",
            "5": "none",
            "6": "none",
            "7": "none",
            "8": "none"
        }
    },
    "2": {
        "state": {
            "on": false,
            "bri": 0,
            "hue": 0,
            "sat": 0,
            "xy": [0,0],
            "ct": 0,
            "alert": "none",
            "effect": "none",
            "colormode": "",
            "reachable": true
        },
        "type": "Dimmable light",
        "name": "Table Lamp 2",
        "modelid": "LWB004",
        "swversion": "65003148",
        "pointsymbol": {
            "1": "none",
            "2": "none",
            "3": "none",
            "4": "none",
            "5": "none",
            "6": "none",
            "7": "none",
            "8": "none"
        }
    }
}
```
### Get light attributes and state
GET	http://host:port/api/<username>/lights/<id>

#### Description
Gets the attributes and state of a given light.

####  Response
Name |	Type |	Description
-----|-------|-------------
state |	state object |	Details the state of the light, see the state table below for more details.
type |	string |	A fixed name describing the type of light which will be "Dimmable light".
name |	string 0, 32 |	A unique, editable name given to the light.
modelid |	string 6, 6 |	The hardware model of the light which will be "LWB004".
uniqueid |	string 6, 32 |	Unique id of the device.
manufacturername |	string 6, 32 |	The manufacturer name will be "Philips".
luminaireuniqueid |	string 6, 32 |	This will be empty.
swversion |	string 8, 8 |	An identifier for the software version running on the light which will be "65003148".
Pointsymbol |	object |	This parameter is reserved for future functionality and returns a preset hashmap.

The state object contains the following fields

Name |	Type |	Description
-----|-------|-------------
on |	bool |	On/Off state of the light. On=true, Off=false
bri |	uint8 |	Brightness of the light. This will be set to 254 as a default.
hue |	uint16 |	This will be empty.
sat |	uint8 |	This will be set to 254.
xy |	list 2..2 of float 4 |	This will be empty.
ct |	uint16 |	This will be empty.
alert |	string |	This will be set to "none".
effect |	string |	This will be set to "none".
colormode |	string 2, 2 |	This will be empty.
reachable |	bool |	Indicates if a light can be reached by the bridge and will be set to true.
#### Response example
```
{
	"state": {
		"hue": 0,
		"on": true,
		"effect": "none",
		"alert": "none",
		"bri": 254,
		"sat": 254,
		"ct": 0,
		"xy": [0, 0],
		"reachable": true,
		"colormode": ""
	},
	"type": "Dimmable light",
	"name": "Kitchen Ceiling",
	"modelid": "LWB004",
	"swversion": "65003148", 	
	"pointsymbol": {
		"1": "none",
		"2": "none",
		"3": "none",
		"4": "none",
		"5": "none",
		"6": "none",
		"7": "none",
		"8": "none"
	}
}
```
### Set light state
PUT	http://host:port/api/<username>/lights/<id>/state
#### Description
Allows the user to turn the light on and off, modify the hue and effects. Please see the FAQ for info on performance considerations.

#### Body arguments
Name |	Type |	Description	 
-----|-------|-------------
on |	bool |	On/Off state of the light. On=true, Off=false. Optional
bri |	uint8 |	The brightness value to set the light to. Brightness is a scale from 1 (the minimum the light is capable of) to 254 (the maximum). Note: a brightness of 1 is not off. e.g. �brightness�: 60 will set the light to a specific brightness. Optional
#### Body example
```
{
	"on": true,
	"bri": 200
}
```
#### Response
A response to a successful PUT request contains confirmation of the arguments passed in. Note: If the new value is too large to return in the response due to internal memory constraints then a value of `�Updated.�` is returned.

#### Response example
```
[
	{"success":{"/lights/1/state/bri":200}},
	{"success":{"/lights/1/state/on":true}},
]
```
## Ask Alexa
After this Tell Alexa: "Alexa, discover my devices"  

Then you can say "Alexa, Turn on the office light" or whatever name you have given your configured devices.  

To view or remove devices that Alexa knows about, you can use the mobile app Menu / Settings / Connected Home  
## Debugging
To turn on debugging for the bridge, use the following extra parm in the command line:
```
-Dorg.slf4j.simpleLogger.defaultLogLevel=DEBUG
```
## Development Mode
To turn on development mode so that it will not need an Harmony Hub for testing, use the following extra parm in the command line and the harmony ip and login info will not be needed:
```
java -jar -Ddev.mode=true ha-bridge-0.X.Y.jar
```
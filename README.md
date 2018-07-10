# WallPanel
WallPanelis an Android application for displaying web based dashboards which has 
other features that integrate into your home automation platform.

## Quick Start
You can either side load the application to your device from the release section or install the application from the Google Play store. The application will open to the welcome page with a link to update the settings. Go to settings, and setup the link to your web page or home automation platform. You may also update additional settings for Motion, Face Detection, and for publishing device sensor data. 

## Sensors
If MQTT is enabled, the application can publish data and states for various device sensors and camera.

### Device Sensors
The application will post device sensors data per the API description and Sensor Reading Frequency. Curerntly device sensors for Pressure, Temperature, Light, and Battery Level are published. 

#### Sensor Data
Sensor | Keys | Example | Notes
-|-|-|-
battery | unit, value, charging, acPlugged, usbPlugged | ```{"unit":"%", "value":"39", "acPlugged":false, "usbPlugged":true, "charging":true}``` |
brightness | unit, value | ```{"unit":"lx", "value":"920"}``` |
pressure | unit, value | ```{"unit":"??", "value":"21"}``` |
temperature | unit, value | ```{"unit":"??", "value":"70"}``` |

*NOTE:* Sensor values are device specific. Not all devices will publish all sensor values.

* Sensor values are constructued as JSON per the above table
* For MQTT
  * WallPanel publishes all sensors to MQTT under ```[baseTopic]/sensor```
  * Each sensor publishes to a subtopic based on the type of sensor
    * Example: ```wallpanel/mywallpanel/sensor/battery```
    
#### Home Assistant Examples
```YAML
sensor:
  - platform: mqtt
    state_topic: "wallpanel/mywallpanel/sensor/battery"
    name: "wallpanel battery"
    unit_of_measurement: "%"
    value_template: '{{ value_json.value }}'

  - platform: mqtt
    state_topic: "wallpanel/mywallpanel/sensor/brightness"
    name: "wallpanel brightness"
    unit_of_measurement: "lx"
    value_template: '{{ value_json.value }}'

  - platform: mqtt
    state_topic: "wallpanel/mywallpanel/sensor/pressure"
    name: "wallpanel pressure"
    unit_of_measurement: "mb"
    value_template: '{{ value_json.value }}'
```

### Motion, Face, and QR Codes 
In additional to device sensor data publishing. The application can also publish states for Motion detection and Face detection, as well as the data from QR Codes derived from the device camera.  

#### Camera Data
Sensor | Keys | Example | Notes
-|-|-|-
motion | value | ```{"value": false}``` | Published immediately when motion detected
face | value | ```{"value": false}``` | Published immediately when face detected

#### Home Assistant Examples

```YAML
binary_sensor:
  - platform: mqtt
    state_topic: "wallpanel/mywallpanel/sensor/motion"
    name: "Motion"
    payload_on: '{"value":true}'
    payload_off: '{"value":false}'
    device_class: motion 
    
binary_sensor:
  - platform: mqtt
    state_topic: "wallpanel/mywallpanel/sensor/face"
    name: "Face Detected"
    payload_on: '{"value":true}'
    payload_off: '{"value":false}'
    device_class: motion 
```


## MJPEG Video Streaming

Use the device camera as a live MJPEG stream. Just connect to the stream using the device IP address and end point. Be sure to turn on the camera streaming options in the settings and set the number of allowed streams. Note that performance depends upon your device (older devices will be slow).

Example stream URL (replace the IP address with your device's IP): 

```http://192.168.1.1/camera/stream```

## MQTT and HTTP Remote Control
You can control the app remotely via MQTT or HTTP (REST). 

### Commands
Key | Value | Example Payload | Description
-|-|-|-
clearCache | true | ```{"clearCache": true}``` | Clears the browser cache
eval | JavaScript | ```{"eval": "alert('Hello World!');"}``` | Evaluates Javascript in the dashboard
audio | URL | ```{"audio": "http://<url>"}``` | Play the audio specified by the URL immediately
relaunch | true | ```{"relaunch": true}``` | Relaunches the dashboard from configured launchUrl
reload | true | ```{"reload": true}``` | Reloads the current page immediately 
url | URL | ```{"url": "http://<url>"}``` | Browse to a new URL immediately
wake | true | ```{"wake": true}``` | Wakes the screen if it is asleep

* Commands are constructed via valid JSON. It is possible to string multiple commands together:
  * eg, ```{"clearCache":true, "relaunch":true}```
* For REST
  * POST the JSON to URL ```http://[mywallpanel]:2971/api/command```
* For MQTT
  * WallPanel subscribes to topic ```[baseTopic]/command```
    * Default Topic: ```wallpanel/mywallpanel/command```
  * Publish a JSON payload to this topic

### State
Key | Value | Example | Description
-|-|-|-
currentUrl | URL String | ```{"currentUrl":"http://hasbian:8123/states"}``` | Current URL the Dashboard is displaying
screenOn | true/false | ```{"screenOn":true}``` | If the screen is currently on

* State values are presented together as a JSON block
  * eg, ```{"currentUrl":"http://hasbian:8123/states","screenOn":true}```
* For REST
  * GET the JSON from URL ```http://[mywallpanel]:2971/api/state```
* For MQTT
  * WallPanel publishes state to topic ```[baseTopic]/state```
    * Default Topic: ```wallpanel/mywallpanel/state```


<!-- ## Default Appication Configuration
Key | Value | Behavior | Default
-|-|-|-
app.deviceId | String | The unique identifier for this WallPanel device | mywallpanel
app.preventSleep | true/false | Prevents the screen from turning off | false
app.launchUrl | URL | The URL the Dashboard launches at | Tutorial Webpage 
app.showActivity | true/false | On-screen indication of browser activity | true
camera.cameraId | int | The camera ID to attach to | 0
camera.motionEnabled | true/false | If the device camera is used for motion detection | false
camera.motionCheckInterval | int | The interval the camera is polled for motion in milliseconds | 500
camera.motionLeniency | int | The leniency on changes in pictures between polls | 20
camera.motionMinLuma | int | The minimum light needed to perform motion detection | 1000
camera.motionWake | true/false | If motion activity should wake the device | true
PLANNED: camera.webcamEnabled | true/false | If the device camera is used as a webcam | false
http.enabled | true/false | Switch for REST(HTTP) being enabled | false
http.port | int | The port to listen on for REST(HTTP) | 2971
mqtt.enabled | true/false | Switch for MQTT being enabled | false 
mqtt.serverName | String | The hostname/IP of the MQTT server | mqtt 
mqtt.serverPort | Int | The port number for TCP MQTT | 1883 
mqtt.baseTopic | String | The root topic WallPanel will pub/sub under | wallpanel/{app.deviceId}/ 
mqtt.clientId | String | The client ID to connect to MQTT with | {app.deviceId}  
mqtt.username | String | The username to connect to MQTT with (or blank) |  
mqtt.password | String | The password to connect to MQTT with (or blank) | 
mqtt.sensorFrequency | Int | The frequency to post sensor data in seconds, or 0 to never post | 0 -->


## Credits

WallPanel (Formerly HomeDash) is a fork from the [original WallPanel project](https://github.com/WallPanel-Project/wallpanel-android) developed by [quadportnick](https://github.com/quadportnick).

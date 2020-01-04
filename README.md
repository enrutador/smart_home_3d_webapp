# First time usage
1. host this project on your local raspberry pi or see ['Local host vs Remote host' section](#Local-host-vs-remote-host) on how to use the live demo
2. create your own glTF model from blender (optional)

## activate hue bridge connection
<img src="./media/hue_register.gif" width="600">

1. adjust the light names to match your hue lights names
2. press the Hue Gateway authorisation button (on the real device)
3. Click on the 3d model of the hue gateway
4. An alert will apear on the screen to wait for confirmation
5. The user creation will proceed and the username will be stored as local storage (ctrl+j in chrome to oben the debug window)


## activate mqtt connection
<img src="./media/mqtt_activate.gif" width="600">

1. adjust the mqtt topic names to match your mqtt devices
2. install a mosquitto version with websocket support
3. Click on the 3d model of the mosquitto gateway

## Live demo
Link to [Github .io live demo](https://homesmartmesh.github.io/smart_home_3d_webapp/)

## Demo light control with power log

<img src="./media/demo1.gif" width="600">

We see in this demo a home 3d model augmented with interactive mesh items. Some meshes represent light bulbs and generate events when the user clicks on them. Those events are handled by a hue light client that interacts with the real hue gateway. The hue client publishes as events the actual state of the switched light, and that state is updated by the mesh bulb color and the associated 3d light.

As a demonstration for the reaction time of the real light switching on and off, we can see in the gif animation the power consumption log of the light switched. This log comes from a [shelly 2.5 device](https://shelly.cloud/shelly-25-wifi-smart-relay-roller-shutter-home-automation/) with power monitoring capabilites. This measure device is itself powering up the hue light. Note that the slow power up and down ramp are due to the hue effect of slow variation when switching on and off.

# Concept
<img src="./media/concept.png" width="600">

Home automation connected to 3d events in javascript

```javascript
send_custom_event("three_param",{name:"Kitchen", color:0.3});
send_custom_event("three_param",{name:"Kitchen", light:0.3});
send_custom_event("three_param",{name:"Kitchen", anim:0.3});
send_custom_event("three_param",{name:"Kitchen", pull:0.3});
send_custom_event("three_param",{name:"Kitchen", push:0.3});
```

<img src="https://github.com/HomeSmartMesh/web_three_interface/raw/master/12_multiple_parameters/media/demo.gif" width="400">

See `three_param` running in a [live demo](https://homesmartmesh.github.io/web_three_interface/12_multiple_parameters/)

# Hue lights usage
Note : This project is not related to the hue devices supplier and is a simple user of the provided API.

## Hue config in blender
<img src="./media/hue_blender.png" width="500">

configuration of hue light name in blender Light bulb object Custom properties.

## Hue events application code

```javascript
window.addEventListener( 'hue_lights_on_startup', onHueStartup, false );
window.addEventListener( 'hue_light_state', onHueLightState, false );

function onHueLightState(e){
    const name = hue_mesh_name[e.detail.name];
    send_custom_event("three_param",{name:name, light:e.detail.on});
}
```

lights broadcast their state on startup and as a feedback when updated from javascript

## Hue light groups
* Lightgroup are optional and the individual light items configuration does not differ depending on if they are part of a light group or not.
* only the hue gateway lightgroup is being used. That means a lightgroup has to be created with the hue app before it can be used in this webapp.

<img src="./media/hue_lightgroup_structure.png" width="300">

Indiviual lights have to be attached to a parent mesh that have these custom properties

<img src="./media/hue_light_groups.png" width="500">

* a lightgroup shall also have a `hue` field, the only differerence is that it has no blender light item and does have a `type:lightgroup` property.
* as the hue gateway allow it, a normal light and a lightgroup can have the same name.

## demo
<img src="./media/lightgroup.gif" width="600">

## Mqtt config in blender

<img src="./media/mqtt_blender.png" width="500">

The mqtt topic has to be assigned to an `mqtt` custom property

## Mqtt events application code

```javascript
window.addEventListener( 'mqtt_message', onMqttMessage, false);

function onMqttMessage(e){
    ...
	if(obj.userData.type == "heating"){
		const heating_demand = e.detail.payload.pi_heating_demand;
		send_custom_event('three_param',{name:obj_name,color:heating_demand});
	}
}
```

# blender model

<img src="./media/blender_model.png" width="600">

[blender file download from google drive](https://drive.google.com/drive/folders/1DFyGKp_6VMN4Vp36PCXglEsV0zEX9iyz?usp=sharing)

The blender model export in .glTF is already part of this repo. So you would only need the blender file in case you'd like a template to start with and customise.

Files within the [project's google drive folder](https://drive.google.com/drive/folders/1DFyGKp_6VMN4Vp36PCXglEsV0zEX9iyz?usp=sharing) will have commit ids to match them with this repo.

# Dependencies

* [three.js](https://threejs.org/)
* [jsHue.js](https://github.com/blargoner/jshue)
* [mqttws31](https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js)
* [web_three_interface](https://github.com/HomeSmartMesh/web_three_interface)

## web_three_interface
The [web_three_interface](https://github.com/HomeSmartMesh/web_three_interface) is not a direct dependency but is used as a boiler plate for 3d interactive meshes. It is still a useful repo that helps understand the workflow step by step with increasingly complex demo, and also debug in case one step is failing.


# Features
* 3d interactions with click and hold
* glTF model with custom properties including interactive meshes and hue light names
* Hue query on startup to set correct light states, on/off and reachability
* Consistent state on startup and on actions with the real light bulbs (but no polling in between)
* mqtt events forward to javascript events
* stats and fps. (can be disabled in [config.json](./config.js))
* hue lights on-off
* heaters pi_heating_demand on colors
* rooms temperature on floor color

<img src="./media/floor_temperature.png" width="500">

## Planned Features

* use 3d slide with world coordinate on scaled objects
* adjust light brightness with 3d slider
* adjust heating with 3d slider
* switch light groups on / off
* handle both hue light groups and model made groups with parenting
* directional light to reflect room sensors ambient light
* add windows open close with colors
* fall back on interactive demo mode from within the same app
* use lower poly mesh
* add coap support through node server (split, back+front) but keep the same front
* add direct mqtt support wtihout websockets through node server
* add android native app with sockets and coap permissions

# Features description
## Hue
* light structure
* light custom properties
 
  * hue:hue light name
  * mouseEvent:true
* lightgroup custom properties

  * type:lightgroup
  * hue:hue group name
  * mouseEvent:true


# Interaction models with glTF custom properties
The currently provided 3d interaction types are :
* Light
* Lightgroup
* Animted : glTF Mesh Animation
* Color : Properties Color Animation

## issues and limitations
* no material animation
* No Custom Properties export for Light Object Data Properties (green), only Object Properties (orange)
* gltf-blender : No area light export possible [github issue](https://github.com/KhronosGroup/glTF-Blender-IO/issues/786)
* three.js : RectAreaLight [has limitations and conditions](https://threejs.org/docs/#api/en/lights/RectAreaLight) [RectAreaLight demo](https://threejs.org/examples/webgl_lights_rectarealight.html)

## Create your own home model

The created home model should have custom properties. It is also possible to use the existing home model file and edit it as it is a json format : `./3d_models/home.gltf`. It is possible to rename the `hue: ` field with your own hue lights names as known by the hue gateway.

Example below :

```json
    "extras" : {
        "mouseEvent" : "true",
        "type" : "light",
        "hue" : "Bathroom main"
```

## Local host vs remote host
* preffered and recommended way : host this repo on your own local rapsberry pi webserver. A vpn is recommended as a solution to remotely get into your local netwrok. Any sort of port exposing and https secure hosting will require a complete security check and would be run with an unknown risk.

* non preferred way : but can be used for test and demonstration purpose, directly from github.io : https://homesmartmesh.github.io/smart_home_3d_webapp/

Limitation when using from github .io :
 - Github .io are the exact deployment of this same repo, the master branch is deployed so is not stable and might not run as it did the last time you used it.
 - Network safety : As the hue.js script is using http, a mixed content http/https error will happen when using from github .io as it is an https server. It is still possible to override this safety.
 - It is not possible to customize your own 3d model
 - It is not possible to update the name of your hue lights (you could still rename some to match the `./3d_models/home.gltf` content)

<a style="color:red">So proceed with this only if you know what you're doing. This might expose you at risk if done through an untrusted website :</a>

<img src="./media/unsafe1.png" width="300">
<br>
<br>
<img src="./media/unsafe2.png" width="300">

then reload the page

# Performance
[webgl report website](https://webglreport.com/)

test model : 8000 triangls

* PC wtih NVIDIA GeForce GTX 745 : 30 FPS, 5 ms
* Samsung galaxy S10 : 60 FPS
* Samsung galaxy S5 : 14 FPS
* Samsung galaxy Tab pro (SM-P900) :  N.A (does not fully load, simplified scene with 3000 tri at 2 FPS)

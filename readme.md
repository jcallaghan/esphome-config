# ESPHome Configs
This is a repo of all my ESPHome configurations with details about some of my ESPHome related projects. 
The ESPHome project makes it incredibly easy to use ESP8266/ESP32 microcontrollers in Smart Home or Home Automation projects. With a simple YAML based configuration to configure sensors ESPHome makes it super easy to get started and manages the build and deployment of the software that is deployed onto the ESP8266/ESP32 devices.

In the [Wiki](https://github.com/jcallaghan/esphome-config/wiki) you will also find links to useful resources and examples of code and config I have found helpful.


## Devices
I have a number of ESPHome devices around my Smart Home and I've to provide a description and readme for each of these devices to share how I built the device and what I use it for. I also use these readme to document any special components I've used with the device and any specific build steps I followed. 

| Config | Description | Readme |
|--------|-------------|--------|
| [esph_area_kitchen.yaml](/esph_area_kitchen.yaml)| IR integration with hob/stove, used for area specific sensors and to obtain data from local Xiaomi temperature sensors via BLE.| ~~[Readme](/readme/esph_area_kitchen.md)~~|
| [esph_bedroom.yaml](/esph_bedroom.yaml) | This ESP32 board connects via two separate GPO pins two pressure pads that are placed under my mattress for presence detection. This is incredibly useful to trigger automations at bed time or in the morning when getting up. |[Readme](/readme/esph_bedroom.md)|
| [esph_ground_water.yaml](/esph_ground_water) | ESP32 with an ultrasonic distance sensor fitted inside a plastic tube. The tube is fixed to my basement wall and monitors the depth of water down there. I place a table-tennis ball inside the plastic tube which floats as water rises. This provides us with a constant view of how bad the flooding is in the local area.|~~[Readme](/readme/esph_ground_water.md)~~|
| [esph_hot_tub_energy.yaml](/esph_hot_tub_energy.yaml) | I use a Sonoff POWR2 to monitor the energy my hot tub uses while also providing a remote power switch to turn the hot tub power on and off.|~~[Readme](/readme/esph_hot_tub_energy.md)~~|
| [esph_area_upstairs.yaml](/esph_area_upstairs.yaml) | I use this ESP32 board to read bluetooth data such as the temperature from a Xiaomi LYWSD02 clock. |~~[Readme](/readme/esph_area_upstairs.md)~~|


## Patterns & Practices

### Substitutions
Typically I leverage three substitutions in my configs. These ensure I have a common hostname, readable and friendly name for entities and a useful description displayed in the ESPHome dashboard for each of my devices.
```yaml
substitutions:
  system_name: <hostname all lowercase with underscores as spaces i.e living_room>
  friendly_name: <usually CAML case version of the hostname i.e Living Room>
  device_description: <useful description of the config and or project>
```
To ensure all my ESPHome nodes have a consistent hostname I use the following when defining my ESPHome config. You can see from the code block below I prefix all my nodes with ```esph_``` and I include the ```{$device_description}``` to define the device comment.
```yaml
esphome:
  name: "esph_${system_name}"
  comment: ${device_description}
  platform: ESP32
  board: pico32
```
To ensure my entities are easy to find once the node has been added to Home Assistant I use the following format when defining my entity names.
```yaml
name: "ESPH ${friendly_name} <name of the entity (CAML case)>"
```
Here is an example of a binary_sensor where you can see the ESPH prefix on the sensor name. 
```yaml
binary_sensor:
  - platform: gpio
    pin: 
      number: GPIO4
      mode: INPUT_PULLUP
      inverted: True      
    name: "ESPH ${friendly_name} Bed Left Occupied"
```
This results in all the entities from this node are returned when searching for an entity in Home Assistant with a text search of ```esph_bedroom*```. This also means I don't need to rename the entity or change the display name of each entity when the devices is added to Home Assistant.

## Onboarding and Build process
1. When I want to run ESPHome on a new device I first determine the most appropriate device to use. 
1. I then create a new config where I define the components I want to use for the project.
1. Once I have established a config I will compile if via the ESPHome Dashboard and download it.
1. I then use ESPHome Flasher to upload the firmware to the ESP device.
1. Note that specific drivers or in some cases (particularly M5 Stick and M5 Atom) require an earlier version of the ESPHome Flasher tool. See device readme or Wiki for further information.
1. Once the device in running I make futher config edits and update OTA until I am happy with the project. 
1. I leverage the device webpage and logs in my testing. Once everything is working fine I then add the device to Home Assistant.#

## Splitting up the ESPHome configuration
A pattern I have seen used by a others in the community which makes an ESPHome config much more readable is to split out common components to their own ```.yaml``` files. More recently (early May) [@frenck](https://github.com/frenck/home-assistant-config/tree/master/config/esphome) refactored his config and stepped up the split configuration even further by seperating ESP boards, types of devices and the common components used. 

In this repo you will see I use this same approach of splitting out the configuration and I hope this explanation does it justice.

```
|-- ESPHome-Config
    |-- .build (local directory not pushed to this repo)
    |-- .esphome (local directory not push to this repo)
    |-- assets (storage for images and fonts used in my configs)
    |   |-- fonts
    |   |   |-- segoeui.ttf
    |   |-- image.png
    |-- boards (definitions for each board type)
    |   |-- .esphome.yaml
    |   |-- board_definition.yaml
    |-- components (used to organised yaml files)
    |   |-- binary_sensors
    |   |   |-- status_sensor.yaml
    |   |-- colors
    |   |   |-- red.yaml
    |   |   |-- green.yaml
    |   |   |-- blue.yaml
    |   |-- common
    |   |   |-- api.yaml    
    |   |   |-- captive_portal.yaml    
    |   |   |-- logger.yaml    
    |   |   |-- ota.yaml    
    |   |   |-- restart.yaml    
    |   |   |-- secrets.yaml (references root secrets file)  
    |   |   |-- status.yaml    
    |   |   |-- time.yaml    
    |   |   |-- web_server.yaml    
    |   |   |-- wifi.yaml    
    |   |-- core
    |   |   |-- i2c_esp32.yaml
    |   |-- fonts
    |   |   |-- segoe_ui_12.yaml
    |   |   |-- segoe_ui_32.yaml
    |   |-- sensors
    |   |   |-- cse7766.yaml
    |   |   |-- uptime_timestamp.yaml
    |   |   |-- uptime.yaml
    |   |   |-- wifi_signal_percentage.yaml
    |   |   |-- wifi_signal.yaml
    |   |-- switches
    |   |   |-- restart.yaml
    |   |-- text_sensors
    |   |   |-- version.yaml
    |   |   |-- wifi_info.yaml
    |   |-- time
    |   |   |-- homeassistant.yaml
    |   |   |-- sntp.yaml
    |-- devices (device definitions)
    |   |-- esp32_._pico.yaml
    |   |-- m5_stick_c.yaml
    |   |-- sonoff_basic.yaml
    |   |-- sonoff_pow_r2.yaml
    |-- readme (collection of readme files for each of my configs shared in this repo)
    |   |-- assets (storage for pictures included in my readme files)
    |   |   |-- image.png
    |   |-- readme.md
    |-- .gitignore
    |-- esph_device_configuration_1.yaml (definition for each ESP board I run ESPHome on)
    |-- esph_device_configuration_2.yaml
    |-- esph_device_configuration_3.yaml
    |-- readme.md (this readme)
    |-- secrets.yaml (where passwords and other secrets are stored and not pushed to this repo)

```
### Boards
A ```boards``` directory exists where each different type of ESP board type is defined in a separate ```board definition``` file like the example shown below.  
```yaml
esphome:
  <<: !include .esphome.yaml
  platform: ESP32
  board: m5stick-c
```
There is also a shared definition ```.esphome.yaml``` where common attributes are maintained like the device name, comment and build path.
```yaml
name: "esph_${system_name}"
comment: "${device_description}"
build_path: "./.build/esph_${system_name}/"
```
The ```build_path``` attribute in the above codeblock places the build directory for each device in the ```.build``` folder in my config root and is not shared in this repo.

### Devices
In the ```devices``` folder there is a template ```device definition``` for each type of ESP device. This allows a device definition to be created for a specific manufacturers device for example a Sonoff Basic or M5 Stick C. Each device definition includes the components that are available for that device. This allows the device to defined once and referenced in multiple ESPHome configs.

```yaml
packages:
  <<: !include_dir_named ../components/common
  board: !include ../boards/esp_01.yaml
```
The device configuration inherits the ```board definition``` described above as well as ```common components``` describe below. This is where ```binary_sensor```, ```switch```, ```sensor``` components for that device can be included.

### Common Components
To standardise a number of common components each configuration defined in ```/components/common/``` folder is included in each ```device definition``` via the ```<<: !include_dir_named ../components/common``` include. This means the following components only need to be defined once:

1. API
1. Captive portal
1. Logger
1. OTA
1. Restart switch
1. Status
1. Time
1. Web Server
1. Wifi

Alongside the ```common components``` other components can be added to specific boards such as ```i2c```, ```font``` and ```colors```. 

*@Frenck makes a great point in his repo in that we should not be pushing sensor date like uptime to Home Assistant and he has made some great changes to the ```uptime.yaml``` and ```uptime_timestamp.yaml``` sensors to prevent this unnecessary data being processed by Home Assistant.*

```yaml
packages:
  <<: !include_dir_named ../components/common
  board: !include ../boards/m5_stick_c.yaml
  i2c: ../components/core/i2c_esp32.yaml
  <<: !include_dir_named ../components/fonts
  <<: !include_dir_named ../components/colors
```

### Config
A config file now just needs to include ```substitutions```, the package block to inherit the correct ```device definition``` and any specific components the project needs. 

Remember these configs need to exist in the root of the config directory for them to appear on the ESPHome dashboard. If you add a ```.``` prefix to the config file name they will be hidden. In my case anything in my repo with a ```~``` prefix doesn't get pushed to this repo.

```yaml
substitutions:
  system_name: device_name
  friendly_name: Device Name
  device_description: "Useful description."

packages:
  device: !include devices/esp32_pico.yaml
```

Additional attribute can be defined in the config for example add something to ```on_boot``` this can be added after the packages block. 
```yaml
esphome:
  on_boot:
    priority: -10  
    then:
      - switch.turn_on: device_relay
```
Or to change the device name.

```yaml
wifi:
  use_address: old_device_name
```
> Note: When renaming devices don't forget to clean up old device directory from the .build folder and json config in .esphome directory.
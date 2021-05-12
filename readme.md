# ESPHome Configs
This is a repo of all my ESPHome configurations with details about some of my ESPHome related projects. 

In the [Wiki](https://github.com/jcallaghan/esphome-config/wiki) you will find links to useful resources and examples of code and config I have found helpful.

## Devices

| Config | Description |
|--------|-------------|
| [esph_area_kitchen.yaml](/esph_area_kitchen.yaml)| IR integration with hob/stove, used for area specific sensors and to obtain data from local Xiaomi temperature sensors via BLE.|
| [esph_bedroom.yaml](/esph_bedroom.yaml) | This ESP32 board connects via two separate GPO pins two [pressure pads](https://www.amazon.co.uk/gp/product/B0045U4MNC) that are placed under my mattress for presence detection. This is incredibly useful to trigger automations at bed time or in the morning when getting up. |
| [esph_ground_water.yaml](/esph_ground_water) | ESP32 with an ultrasonic distance sensor fitted inside a plastic tube. The tube is fixed to my basement wall and monitors the depth of water down there. I place a table-tennis ball inside the plastic tube which floats as water rises. This provides us with a constant view of how bad the flooding is in the local area.|
| [esph_hot_tub_energy.yaml](/esph_hot_tub_energy.yaml) | I use a Sonoff POWR2 to monitor the energy my hot tub uses while also providing a remote power switch to turn the hot tub power on and off.|
| [esph_area_upstairs.yaml](/esph_area_upstairs.yaml) | I use this ESP32 board to read bluetooth data such as the temperature from a Xiaomi LYWSD02 clock. |

## Patterns & Practices

### Substitutions
Typically I leverage three substitutions in my configs. These ensure I have a common hostname, readable and friendly name for entities and a useful description displayed in the ESPHome dashboard for each of my devices.
```yaml
substitutions:
  system_name: <hostname all lowercase with underscores as spaces i.e living_room>
  friendly_name: <usually CAML case version of the hostname i.e Living Room>
  device_description: <useful description of the config and or project>
```
To ensure all my ESPHome nodes have a consistent hostname I use the following when defining my ESPHome config. You can see from the code block below I prefix all my nodes with ```esph_``` and I include the ```{$device_description}```.
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
Here is an example of a binary_sensor. 
```yaml
binary_sensor:
  - platform: gpio
    pin: 
      number: GPIO4
      mode: INPUT_PULLUP
      inverted: True      
    name: "ESPH ${friendly_name} Bed Left Occupied"
```
This results in all the entities from this node when searching for the entity in Home Assistant with a text search of ```esph_bedroom*```. This also means I don't need to rename the entity or change the display name of each entity when exposed to Home Assistant.

### Splitting up the ESPHome configuration
A pattern I have seen used by a few others in the community which makes an ESPHome config much more readable is to split out common components to their own ```.yaml``` files. 

In this repo you will see I have a [.common](/.common) directory where there is a number of ```.yaml``` files that define these common components. Alongside these there is a file called [common.yaml](.common/common.yaml) which references these components through a number of ```<<: !include``` references. This means I can remove a huge amount of code in my config by removing the common components and in turn make my config much more readable. 

Common components included:
1. API
1. Captive portal
1. Logger
1. OTA
1. Time
1. Web Server
1. Wifi

And also include the following packages which define the following entities:
1. Status sensor
1. Restart switch
1. Uptime sensor
1. Version sensor
1. Wifi sensor

By including this file ```<<: !include .common/common.yaml``` in my configs I no longer need to define each of the components referenced in this file. This also means I can change the configuration of these common components in one place rather than editing each device config.

### Template and Reference configs
I have a number of template configs which I copy to get started and flash a node really quickly. I also save examples of configs that are really great inspirations and points of reference. Each of these should include a credit to the source at the top of the config.

| Config | Description |
|--------|-------------|
| generic_esp32.yaml | Base config for most ESP32 nodes which includes the above includes to pull through the common components I want with each of my nodes. |
| m5stickc.yaml | Template config for M5 Stick C devices. This includes all the components that are available on this board type. |

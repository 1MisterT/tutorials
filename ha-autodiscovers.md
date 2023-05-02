# HA Autodiscovers   
Verschiedene Autodiscovery Strings und Umsetzung in Arduino   
 --- 
## Dokumentation:   
[MQTT](https://www.home-assistant.io/integrations/mqtt/#mqtt-discovery)    
   
## Autodiscorvery Arduino   
### **Sensor:**    
**Discovery Topic:**    
  `homeassistant/sensor/entity\_name/config`    
**Discovery String:**    
```
{
  "name": "Temperature",
  "stat_t": "state/topic",
  "unique_id": "temperature",
  "unit_of_measurement": "°C"
}
```
###    
### Light:   
**Discovery Topic:**    
  `homeassistant/light/entity\_name/config`    
**Discovery String:**    
```
{
  "name": "Light",
  "cmd_t": "command/topic",
  "stat_t": "state/topic",
  "brightness_command_topic": "command/topic/brightness", (optional)
  "unique_id": "light",
  "payload_on": "on",
  "payload_off": "off"
}
```
   
### Binary Sensor:   
**Discovery Topic:**    
  `homeassistant/binary\_sensor/entity\_name/config`    
**Discovery String:**    
```
{
  "name": "Radar",
  "device_class": "motion",
  "stat_t": "state/topic",
  "unique_id": "radar",
  "payload_on": "1.00",
  "payload_off": "0.00",
}
```
   
### Switch:   
**Discovery Topic:**    
  `homeassistant/switch/entity\_name/config`    
**Discovery String:**    
```
{
  "name": "Outlet",
  "device_class": "switch",
  "cmd_t": "command/topic",
  "stat_t": "state/topic",
  "unique_id": "outlet",
  "payload_off": 0,
  "payload_on": 1
}
```
   

# Irrigation-with-display
Garden irrigation based on Sonoff 4ch, with built-in Nextion display

![Irrigation](/images/irrigation.jpg "Sonoff 4ch with Nextion display")

## Getting Started

### Architecture

Using Sonoff 4ch Pro to control four 24V AC valves (Gardena). The control of the solenoid valves is simple - open the valve by connecting 24V to it, it will close automatically when disconnected. I am using the original Gardena 24V AC adapter - one of its wires wire is connected to the (1-4) Sonoff 4ch Pro C (common) connectors. The NO (Normally Open) connectors (1-4) are then connected to the solenoids. The second AC adapter wire is then connected to the second contact of all 4 solenoids.

The Sonoff relays (galvanically separated from the main device power supply) then control open valves by closing the relay. The valves can be closed by closing the relay. The 4 buttons on Sonoff 4ch are configured in the ESPHome script to toggle the relay. The ESPHome script also has timers, that automatically close the relays after preset time (even if the device closes connectivity to Wifi or Home Assistant, or if you open the relay by pushing the button manually. You can change the time preset for each valve using the ESPHome API.

This is the example of the script to change the preset and open the valve:
```
  - service: esphome.irrigation_set_time_1
    data:
      time: 900
  - service: switch.turn_on
    entity_id: switch.zavlazovani_rele_1

```
This will automatically shutdown the valve after 900 seconds (15 minutes). If you do not set the preset, it will use the default or previously set value.

More advanced scenario is to set the valve based on [Google callendar event](https://www.home-assistant.io/components/calendar.google/). This is the example:
```
  trigger: 
      platform: state
      entity_id: calendar.irrigation1
      to: 'on'
  condition:
      # checking the condition, e.g. rain sensors or pevious events
  action:
  - service: esphome.irrigation_set_time_1
    data_template:
      time: "{{  (as_timestamp(state_attr('calendar.irrigation1','end_time')) - as_timestamp(now())) | int }}"
  - service: switch.turn_on
    entity_id: switch.irrigation_rele_1
```

The Nextion display shows the remaining time. It also has 3 buttons to start automation scrips from teh touch screen. The display is connected via 4 wires: GND, +5V, RX, and TX. So to control the display is extremely easy - just connect these 4 wires to the corresponding ports in the Sonoff 4CH. GND, TX and RX is easy. But you need to find the 5V contact. This can be done in 2 ways - either power the whole Sonoff 4ch Pro by 5V (which is what I have done). Or power it by mains and find a 5V contact on the circuit board.

### Design

The principle is very simple: I used Nextion Editor to create a layout for the display screen. There is a static image background with all graphics that do not change (including text, icons, frames, or color background). On top of the background, I created text objects for all the values that change. When the design is finished, you can upload the layout to the display from the NextionEditor using the serial connection (via USB FTDI board).

The program  is then very simple - it just sets the text value for the individual text objects. For example, for the inside temperature it does this:
```
it.set_component_text_printf("time1","%2d:%02d",id(remaining_time1)/60,id(remaining_time1)%60);
```

ESPHome can read Home Assistant sensors through API, so the ESPHome sensor "rain_today" is linked to Home Assistant sensor sensor.rain_today this way:
```
sensor:
  - platform: homeassistant
    id: rain_today
    entity_id: sensor.rain_today
 ```

It is using the [2.4 inch Nextion display](https://www.banggood.com/Nextion-NX3224T024-2_4-Inch-Man-machine-Interface-HMI-Screen-Kernel-In-English-p-1105052.html?utm_campaign=19381694_november&utm_content=2635&p=@K220219381694201802&cur_warehouse=CN).

### Source files

- **irrigation.yaml** - this is the main file - the actual ESPHome configuration
- **display.HMI** - this is the actual design of the display I use - the file can be edited via the NextionEditor (can be [downloaded](https://nextion.itead.cc/resources/download/nextion-editor/) from ITEAD pages)
- **images (folder)** - I have included the source images for the Nextion display editor. These are not essential for the project, but if you want to create own design, you can start from there.

## Author

* **Václav Chaloupka** - *Initial work* - [bruxy70](https://github.com/bruxy70)

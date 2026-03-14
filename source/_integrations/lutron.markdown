---
title: Lutron
description: Instructions on how to use Lutron devices with Home Assistant.
ha_category:
  - Binary sensor
  - Cover
  - Event
  - Fan
  - Hub
  - Light
  - Scene
  - Select
  - Switch
ha_release: 2026.3
ha_iot_class: Local Polling
ha_codeowners:
  - '@cdheiser'
  - '@wilburCForce'
ha_domain: lutron
ha_platforms:
  - binary_sensor
  - cover
  - event
  - fan
  - light
  - scene
  - select
  - switch
ha_integration_type: hub
ha_config_flow: true
---

[Lutron](https://www.lutron.com/) is an American lighting control company. They have several lines of home automation devices that manage light switches/dimmers, occupancy sensors, HVAC controls, etc. The **Lutron** integration in Home Assistant is responsible for communicating with the main hub for these systems.

Presently, the integration supports communicating with the RadioRA 2 Main Repeater and handles various devices like light switches, dimmers, keypad scenes, fans, and shades.

## Configuration

When configured, the `lutron` integration will automatically discover the rooms and their associated switches/dimmers as configured by the RadioRA 2 software from Lutron. Each room will be treated as a separate group.

To use Lutron RadioRA 2 devices in your installation, you'll need to first create a username/password in your Lutron programming software. Once a telnet username/password has been programmed, you can follow the instructions from the next chapter.

{% include integrations/config_flow.md %}

{% tip %}
It is recommended to assign a static IP address to your main repeater. This ensures that it won't change IP addresses, so you won't have to change the `host` if it reboots and comes up with a different IP address.
{% endtip %}

{% important %}
If you are using RadioRA2 software version 12 or later, the default `lutron` user with password `integration` is not configured by default. To configure a new telnet user, go to **Settings** > **Integration** in your project and add a new telnet login. Once configured, use the transfer tab to push your changes to the RadioRA2 main repeater(s).
{% endimportant %}

## Keypad buttons

Keypad button actions are provided in event entities.

The following event types are available:
- `single_press`: Fired when a button is pressed on most keypads.
- `press`: Fired when a button is pressed on keypads that support release events (like Raise/Lower buttons).
- `release`: Fired when a button is released on keypads that support release events.

Buttons also fire a `lutron_event` on the event bus for backward compatibility. This event includes the following data attributes:
- `id`: The slugified name of the button (for example, `office_pico_on`).
- `action`: The action performed (`single`, `pressed`, or `released`).
- `full_id`: The slugified name of the area and button.
- `uuid`: The Lutron unique identifier for the button.

## Keypad LEDs

Each full-width button on a Lutron SeeTouch, Hybrid SeeTouch, and Tabletop SeeTouch Keypad has an LED that can be controlled by Home Assistant. 

Indicator LEDs for scenes (keypad buttons) are available as **Select** entities. They support the following states:
- **Off**
- **On**
- **Slow Flash**
- **Fast Flash**

{% note %}
Previous versions of this integration used **Switch** entities to control Keypad LEDs. These switch entities are now deprecated and will be removed in a future release. You should update your automations to use the new **Select** entities instead.
{% endnote %}

The Lutron system also controls the LED state independent of Home Assistant, according to the programming of the RadioRA 2 system. This means you can query LED states to determine if a certain scene is active, since the LED will have been illuminated by the RadioRA 2 repeaters. This includes the "phantom" LEDs of Main Repeater Keypad buttons; even though there is no physical button or LED, the RadioRA 2 system tracks the scenes and will "light" the LED that can be queried.

If a button is not programmed to control any lights or other devices in the RadioRA 2 system but is given a name in the programming software, it will be available to fire events in Home Assistant. However, since there is no way to have a scene "active" on a button with no devices associated, the Main Repeater will automatically extinguish the keypad LED a few seconds after the button press. If you wish to have Home Assistant light the keypad LED after a button press, you will need to delay your action to light the LED for several seconds, so it arrives after the Main Repeater has sent the command to turn it off.

## Scene

This integration uses keypad programming to identify scenes. Currently, it works with seeTouch, hybrid seeTouch, main repeater, homeowner, Pico, and seeTouch RF tabletop keypads.

The Lutron scene platform allows you to control scenes programmed into your seeTouch keypads. After setup, scenes will appear in Home Assistant using the area, keypad, and button name.

## Occupancy sensors

Any configured Powr Savr occupancy sensors will be added as occupancy binary sensors. Lutron reports occupancy for an area, rather than reporting individual sensors. Sensitivity and timeouts are controlled on the sensors themselves, not in software.

## Example automations

### Keypad button notification

This example shows how to use the `event` entity for a notification.

```yaml
- alias: "Keypad button pressed notification"
  triggers:
    - trigger: state
      entity_id: event.office_pico_on
  actions:
    - action: notify.telegram
      data:
        message: "The Pico button was just pressed!"
```

### Flash keypad LED

This example shows how to set a keypad LED to slow flash.

```yaml
- alias: "Flash keypad LED when garage door is open"
  triggers:
    - trigger: state
      entity_id: cover.garage_door
      to: "open"
  actions:
    - action: select.select_option
      target:
        entity_id: select.office_keypad_garage_led
      data:
        option: "Slow Flash"
```

# NOAA Tides and Currents Sensor for Home-Assistant

This library is a [fork of the core component](https://www.home-assistant.io/integrations/noaa_tides/) which adds some additional features and migrates the backend from the now-defunct [py_noaa](https://github.com/GClunies/py_noaa) to the superseding [noaa_coops](https://github.com/GClunies/noaa_coops).

![beta_badge](https://img.shields.io/badge/maturity-Beta-yellow.png)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Installation

#### Using HACS (Home Assistant Community Store)

Make sure that [Home Assistant Community Store (HACS)](https://github.com/custom-components/hacs) is setup, then add the "Integration" repository: `jshufro/home_assistant_noaa_tides`

#### Manual

1. Clone the repository.
2. Copy the `custom_components/noaa_tides` directory into `<home assistant directory>/custom_components/`
3. Configure a sensor in configuration.yaml

## Configuration

#### Example

``` yaml
sensor:
  - platform: noaa_tides
    name: tides           # Useful when more than one sensor is used. Otherwise a name is generated.
    type: tides           # This station will be used to measure water level
    station_id: 8512354   # A station ID from https://tidesandcurrents.noaa.gov/
    scan_interval: 360    # polling interval for the NOAA API

  - platform: noaa_tides
    entity_id: sensor.ocean_temp
    name: "Ocean temperature"
    type: temp
    station_id: 8510560   # station used to measure water temperature
    scan_interval: 360

  - platform: noaa_tides
    name: buoy
    type: buoy
    station_id: 44017     # station used for off-shore buoy data (buoy ID from https://www.ndbc.noaa.gov/)
    scan_interval: 360
```

Different stations support different features (products/datums), so use the [station finder](https://tidesandcurrents.noaa.gov/) to select the right station for `tides` vs `temp` types.

For complex [lovelace](https://www.home-assistant.io/lovelace/) widgets, it is suggested to use the [template platform](https://www.home-assistant.io/integrations/template/).

Sample template sensor:

``` yaml
- platform: template
  sensors:
    next_tide:
      friendly_name: Next tide
      entity_id: sensor.tides
      value_template: >-
        {{ state_attr('sensor.tides', 'next_tide_type') }} tide at {{ state_attr('sensor.tides', 'next_tide_time') }}
      icon_template: >-
        {% if is_state_attr('sensor.tides', 'next_tide_type', 'High') %}mdi:waves{% else %}mdi:wave{% endif %}
    last_tide:
      friendly_name: Last tide
      entity_id: sensor.tides
      value_template: >-
        {{ state_attr('sensor.tides', 'last_tide_type') }} tide at {{ state_attr('sensor.tides', 'last_tide_time') }}
      icon_template: >-
        {% if is_state_attr('sensor.tides', 'last_tide_type', 'High') %}mdi:waves{% else %}mdi:wave{% endif %}
    water_level:
      friendly_name: Water level
      entity_id: sensor.internet_time
      value_template: '{{ state_attr(''sensor.tides'', ''tide_factor'') }}'
      unit_of_measurement: '%'
    beach_air_temp:
      friendly_name: Air temperature
      entity_id: sensor.water_temp
      value_template: '{{ state_attr(''sensor.water_temp'', ''air_temperature'') }}'

```

**Note:** The tide curve requires `sensor.internet_time` to be updated correctly. Use the `time_date` sensor platform like this:

``` yaml
  - platform: time_date
    display_options:
      - 'beat'
```

### Lovelace

![Lovelace Configuration](/noaa_tides_lovelace.png)

These cards are created using the [custom mini-graph card](https://github.com/kalkih/mini-graph-card) with the following Lovelace configuration:

```yaml
entities:
  - color: '#02ace5'
    entity: sensor.beach_air_temp
    name: Air
    show_state: true
    state_adaptive_color: true
  - color: darkblue
    entity: sensor.ocean_temp
    name: Water
    show_state: true
    state_adaptive_color: true
hours_to_show: 24
name: Air and Water Temperatures
points_per_hour: 12
show:
  fill: false
type: 'custom:mini-graph-card'
unit: °F
```

```yaml
entities:
  - entity: sensor.next_tide
  - entity: sensor.last_tide
footer:
  entities:
    - sensor.ocean_water_level
  hours_to_show: 12
  icon: 'mdi:swim'
  line_color: darkblue
  lower_bound: 0
  points_per_hour: 60
  show:
    labels: false
  type: 'custom:mini-graph-card'
  upper_bound: 100
title: Tides
type: entities
```

## Support

For setup and other support questions, see the [Home Assistant community discussion for this add-on](https://community.home-assistant.io/t/i-made-an-improved-noaa-tides-sensor-for-my-familys-summer-house/203466).

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)

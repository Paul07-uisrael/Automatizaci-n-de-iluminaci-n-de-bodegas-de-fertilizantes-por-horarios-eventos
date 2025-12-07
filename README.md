# Automatizaci-n-de-iluminaci-n-de-bodegas-de-fertilizantes-por-horarios-eventos
Automatización de iluminación de bodegas de fertilizantes por horarios/eventos

# Input setpoint
input_number:
  setpoint_temp:
    name: Temperatura objetivo
    initial: 20
    min: 5
    max: 35
    step: 0.5
    unit_of_measurement: "°C"
input_boolean:
  thermostat_enabled:
    name: Termostato habilitado
    initial: true
# Climate using generic_thermostat
climate:
  - platform: generic_thermostat
    name: Termostato Flores
    heater: switch.calefactor           
    target_sensor: sensor.temp_salon     
    min_temp: 10
    max_temp: 35
    ac_mode: false                       
    target_temp: 20
    cold_tolerance: 0.3
    hot_tolerance: 0.3
    min_cycle_duration:
      seconds: 30
    away_temp: 16
    precision: 0.1
    keep_alive:
      minutes: 2
    initial_hvac_mode: "heat"
    auxiliary_control: false


# Template sensor - lectura en °C formateada

sensor:
  - platform: template
    sensors:
      temp_salon_display:
        friendly_name: "Temperatura Salón (display)"
        unit_of_measurement: "°C"
        value_template: "{{ states('sensor.temp_salon') | float | round(1) }}"


# Seguridad / automatización básica
automation:
  - id: 'thermostat_disable_on_high_temp'
    alias: "Apagar calefactor si temp > limite seguridad"
    trigger:
      - platform: numeric_state
        entity_id: sensor.temp_salon
        above: 40
    condition: []
    action:
      - service: switch.turn_off
        target:
          entity_id: switch.calefactor
      - service: notify.mobile_app_yourphone
        data:
          title: "Alerta Termostato"
          message: "Temperatura > 40°C, calefactor apagado seguridad."

  - id: 'sync_setpoint_to_climate'
    alias: "Sincronizar setpoint input_number -> climate"
    trigger:
      - platform: state
        entity_id: input_number.setpoint_temp
    action:
      - service: climate.set_temperature
        target:
          entity_id: climate.termometro_flores, climate.termometro_flores  # placeholder
        data:
          temperature: "{{ states('input_number.setpoint_temp') | float }}"

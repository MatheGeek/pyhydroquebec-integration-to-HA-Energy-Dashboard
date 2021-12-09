#############
Integration of pyhydroquebec to the Home Assistant Energy Dashboard
#############


TODO
####

***STILL TESTING***
I need to make sure it will work at  the end of the billing cycle.

Fix the consumption date : As  I use the "sensor.hydroquebec_123456789_yesterday_total_consumption",  the consumption is postponed by one day (consumption shown for Nov 24th  is in fact consumption of Nov 23th)


Installation of pyhydroquebec
############

Please refer to  https://github.com/titilambert/pyhydroquebec  to install it in Docker  or   to  https://github.com/arsenicks/pyhydroquebec-hass-addons  to get a Home Assistant addon



pyhydroquebec installation using docker-compose
######

Docker image list: https://gitlab.com/ttblt-hass/pyhydroquebec/container_registry

::

  services: 
    pyhydroquebec:
        image: registry.gitlab.com/ttblt-hass/pyhydroquebec/mqtt:master
        restart: unless-stopped
        container_name: pyhydroquebec
        network_mode: bridge
        volumes:
        - pyhydroquebec_data:/etc/pyhydroquebec
        environment:
            - MQTT_USERNAME=yourusername
            - MQTT_PASSWORD=yourpassword
            - MQTT_HOST=192.168.X.X
            - MQTT_PORT=1883
            - ROOT_TOPIC=myhome
            - MQTT_NAME=hydro   
            - LOG_LEVEL=INFO

pyhydroquebec configuration file  (pyhydroquebec.yaml)
######

::

    timeout: 30
    frequency: 10800
    accounts:
        - username: hydroquebecaccount@email.com  
          password: yourpassword    
          contracts:
            - id: 123456789

        
Consumption and cost sensors for the Energy Dashboard
######

Once you are able to retrieve pyhydroquebec sensors in Home Assistant,  you need to create 2 sensors that will comply with HA long-term statistic rules : 
https://developers.home-assistant.io/docs/core/entity/sensor/#long-term-statistics

To add in configuration.yaml

::

  template:
    - sensor:
        - name: "Hydro Consumption"
          state: '{{ states.sensor.hydroquebec_123456789_period_total_consumption.state }}'
          unit_of_measurement: "kWh"
          state_class: "total_increasing"
          device_class: "energy"
        
        - name: "Hydro Cost"
          state: '{{ states.sensor.hydroquebec_123456789_period_total_bill.state }}'
          unit_of_measurement: "CAD"
          state_class: "total_increasing"
          device_class: "monetary"

# ha-rapt-package - Package for Home Assistant to integrate KegLand RAPT API
# Home Assistant "Package" for KegLand RAPT

Home Assistant (https://www.home-assistant.io/) Package - More info on Packages here: https://www.home-assistant.io/docs/configuration/packages/

This Package can be included into your configuration.yaml to connect to the KegLand RAPT API. This is a work in progress and for now, will bring one RAPT BrewZilla 4, one RAPT Pill one RAPT Temperature Controller into Home Assistant as Entities. Duplicating and adjusting sensors will allow multiple to be added. Other products can also be added pretty easily. Please feel free to reach out if you need a hand with either of these and I'll try to get back to you ASAP. There is a second package that will allow the data to be forwarded on to BrewFather

# Disclaimers
1. This is an unofficial integration of KegLand RAPT API for Home Assistant, the developer and the contributors are not, in any way, affiliated to KegLand. "RAPT" and "KegLand" are trademarks of Kegland Distribution PTY LTD
2. The add-on for BrewFather is also unofficial, the developer and contributors are not, in any way, affiliated to BrewFather. "BrewFather" is Copyright Warpkode AS
3. This is a rushed, "just get it in there" package. I know it doesn't meet best standards for keeping credentials in the Secrets File and I'm working on that separately
4. As mentioned above, only the RAPT Pill is integrated in a read-only mode so far. RAPT BrewZilla 4, and RAPT Temperature Controller have a "set temperature" HA Service and are are integrated in a read-only mode otherwise. This is a work in progress

# Install manually:
1. Create entries in your secrets.yaml file with the IDs of your RAPT. If you decide to use different secret names, you'll need to adjust the package. If you have more than one of any item, create the appropriate entry, incrementing the id#
 ```
    rapt_api_hydrometer_id1: <enter Hydrometer ID here>
    rapt_api_tempcontroller_id1: <enter Temperature Controller ID here>
```
If using the BrewFather Add-on package, also enter that into the secret (note that this requires the full URL)
```
    brewfather_stream_id: "http://log.brewfather.net/stream?id=<enter Stream ID here>"
```
2. Copy the package_rapt-sample.yaml file found in https://github.com/thewolf-oz/ha-rapt-package to your config folder/directory in Home Assistant. A different folder/directory can be used however the following instructions assume it is directly in the config folder/directory 
  If using the BrewFather Add-on package, do the equivalent for package_brewfather-sample.yaml
3. Rename the file to package_rapt.yaml or to a file name of your liking/standard. The following instructions assume you rename the file to package_rapt.yaml
  If using the BrewFather Add-on package, do the equivalent for package_brewfather-sample.yaml
4. Using the instructions on this FB Post under "API Secrets", generate your own secret: https://www.facebook.com/groups/raptusersgroup/permalink/1087286405153972
    
    Note: you can name the secret anything you want, so I'd suggest referencing Home Assistant if you're using it here
  If using the BrewFather Add-on package, under settings, use the "Generate API-Key" options
5. Replace `<EnterYourEmailHere>` on line 52 with the email address you used to register to the RAPT Portal (not the name of your secret from 4 above)
6. Replace `<EnterYourAPISecretHere>` on line 52 with the secret you generated in the RAPT Portal (from 4 above. The secret, not the name)
7. Comment out the relevant sections for products you don't have
8. Duplicate the relevant sections for products you have more than one of, incrementing the number as appropriate
9. Add the configuration file under the homeassistant\packages section, adding the the tree structure as appropriate
```
    homeassistant:
      packages:
        rapt: !include package_rapt.yaml
```
   If using the BrewFather Add-on package, do the equivalent for package_brewfather.yaml:
```
        brewfather: !include package_brewfather.yaml
```
10. Restart Home Assistant.
11. Wait 10 minutes after restart for the first telemetry to appear. The first attempt will fail as the bearer is not yet ready, this is expected at this stage

# Additional Configuration Outside of Package
I use an Ikea Zigbee Remote to set the Input_Number and Input_DateTime in the package. Once Home Assistant has an update from RAPT, pressing the button will save the current Specific Gravity and Date/Time to calculate the ABV and for use elsewhere
  This is the configuration I have on the button-press
![image](https://user-images.githubusercontent.com/86336633/158142354-a4752f14-40c2-409a-a033-60dba44b1ec3.png)

  This is the graph I use to show the Specific Gravity and ABV over time. It uses both the ABV derived from the Input_Number and the Fermentation Duration derived from the Input_DateTime to identify the span (date/time range) of the graph
![image](https://user-images.githubusercontent.com/86336633/158143224-6dc8636a-a855-4a4a-90e7-c9547d82d879.png)

  This graph uses the config-template-card and the apexcharts-card custom front-end integrations
```
            type: custom:config-template-card
            variables:
              - states['sensor.rapt_pill_fermentation_duration'].state + "h"
            entities:
              - sensor.rapt_pill_gravity
              - sensor.rapt_pill_current_abv
            card:
              type: custom:apexcharts-card
              experimental:
                color_threshold: true
              header:
                show: false
                title: RAPT Pill Gravity and ABV
                show_states: true
                colorize_states: true
                floating: false
              graph_span: ${vars[0]}
              all_series_config:
                show:
                  legend_value: true
                fill_raw: last
              series:
                - entity: sensor.rapt_pill_gravity
                  yaxis_id: gravity
                  float_precision: 4
                  color_threshold:
                    - color: '#00ff00'
                      value: 1.015
                    - color: '#ffcc00'
                      value: 1.03
                    - color: '#ff3300'
                      value: 1.06
                - entity: sensor.rapt_pill_current_abv
                  yaxis_id: abv
                  float_precision: 2
              yaxis:
                - id: gravity
                  decimals: 2
                  min: ~1
                  max: ~1.06
                - id: abv
                  decimals: 1
                  opposite: true
                  min: 0
                  max: ~5
```

# Contribute
Feel free to contribute by opening a PR, issue on this project    

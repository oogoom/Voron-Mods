

### **DISCLAIMER:** This is not a perfect tracking system.  Currently, it only tracks the total duration of a print job (i.e., heatsoak time + printing time).  It does not track extra time for delayed gcode filter shutoff and it does not track total number of days since refreshing the filter media (e.g., 30 days).  I am hoping to add both these things in the future, so check back soon<sup>tm</sup>?

These macros help you track the number of hours your filter media has been used and gives a simple M117 reminder when the tracked hours exceeds a predefined maximum.

### REQUIRED:

[save_variables] config section must be added to your printer.cfg.  Your path must be defined.  Use whatever path it is to your klipper config.  The example below is shown with the default path for Klipper since last year.  Older versions of Klipper will have a different path.  This config section will enable saving variables to a file (e.g., variables.cfg) that will allow variables and their values to be saved across restarts.  The filename can be changed to whatever you choose.

```
#######################################
#  VARIABLE SAVING
#######################################
[save_variables]
filename: ~/printer_data/config/variables.cfg
```

Once [save_variables] has been added to your printer.cfg, you can add these macros as well.  

[FILTER_RESET_CHECK] does error checking and alerting after counter reset.  This is to deal with a quirk where the variable value doesn't seem to update immediately for use until another macro is called.  

[FILTER_RESET] resets the hour counter and is where you can define your desired maximum hours.  It is a float value to reflect print jobs less than 1 hour.  Change the maxhours value to whatever you wish.  This macro should be run every time you change your filter media.  It will call FILTER_ALERT_CHECK to display an M117 status message once complete or an error if something went wrong.

[FILTER_TIME] simply displays the currently tracked hour counter value.  Quick way to see how close you are to reaching your max.

[FILTER_ADD_TIME] does all the tracking.  Takes the total_duration of the recently completed print job and adds it to the value stored for the hourcounter variable.  Note that print jobs that are cancelled or fail and do not produce print_stats values for total_duration are not tracked.  Call this in your [PRINT_END] macro.  Checks to see if hour counter exceeds the max and alerts if it is.

```
[gcode_macro FILTER_ALERT_CHECK]
description: checks to make sure hourcounter is reset
gcode:
    {% set option = params.MODE|default(0)|int %}                         # Get mode
    {% set svv = printer.save_variables.variables %}                      # shortcut for stored variables
    {% if option == 0 %}                                                  # called from FILTER RESET
        {% if svv.hourcounter == 0 %}                                    
              M117 FILTER TIME HAS BEEN RESET TO {svv.hourcounter} HOURS  # if you reset your hour counter, it displays success message
        {% else %} 
              M117 ERROR: UNABLE TO RESET COUNTER                         # displays error message.  If you get this, check your variables.cfg file.
        {% endif %} 
    {% elif option == 1 %}                                                # called from FILTER_ADD_TIME
        {% if svv.hourcounter > svv.maxhours %}                        
            M117 WARNING: FILTER CARBON EXCEEDS {svv.maxhours} HOURS      # if hour counter is greater than your max, displays warning
        {% endif %}
    {% endif %}

[gcode_macro FILTER_RESET]
description: sets max hours and resets hour counter
gcode:
    SAVE_VARIABLE VARIABLE=maxhours VALUE=300.00               # Set maximum number of hours before displaying reminder
    SAVE_VARIABLE VARIABLE=hourcounter VALUE=0                 # Reset print hours counter before displaying reminder
    FILTER_ALERT_CHECK                                         # need to do this because the variable value doesn't update immediately for some reason

[gcode_macro FILTER_TIME]
description: displays current filter hours
gcode:
    {% set svv = printer.save_variables.variables %}           # shortcut for stored variables
    M117 FILTER CARBON IS {svv.hourcounter} HOURS OLD

[gcode_macro FILTER_ADD_TIME]
description: updates the hourcounter variable with the recently completed print.  call this in your print_end.
gcode:
   {% set svv = printer.save_variables.variables %}
   # Convert total_duration to hours
   {% set total_hours = ((printer.print_stats.total_duration/3600)|round(2,'common')) %}
   # Check if there is even a value (for cases where cancelled/failed print doesn't produce one). If not, do nothing.
   {% if total_hours > 0 %}
        SAVE_VARIABLE VARIABLE=hourcounter VALUE={svv.hourcounter + total_hours}
        # Call macro to check if hour counter exceeds max
        FILTER_ALERT_CHECK MODE=1     
   {% endif %}
```

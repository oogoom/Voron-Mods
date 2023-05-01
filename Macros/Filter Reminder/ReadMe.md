

### **DISCLAIMER:** This is not a perfect tracking system.  Currently, it only tracks the total duration of a print job (i.e., heatsoak time + printing time) plus any delayed gcode time for turning off fans.  It does not track total number of days since refreshing the filter media (e.g., 30 days).  I am hoping to add this in the future, so check back soon<sup>tm</sup>?

These macros help you track the number of hours your filter media has been used and gives a simple M117 reminder, console alert, and optionally a notifier message, when the tracked hours exceeds a predefined maximum.

Thanks to PF for the notifier idea!

### REQUIRED:

[save_variables] config section must be added to your printer.cfg.  Your path must be defined.  Use whatever path it is to your klipper config.  The example below is shown with the default path for Klipper since last year.  Older versions of Klipper will have a different path.  This config section will enable saving variables to a file (e.g., variables.cfg) that will allow variables and their values to be saved across restarts.  The filename can be changed to whatever you choose.  The file will be created by Klipper automatically, so creating one is not necessary.

```
#######################################
#  VARIABLE SAVING
#######################################
[save_variables]
filename: ~/printer_data/config/variables.cfg
```

Once [save_variables] has been added to your printer.cfg, you can add these macros as well.  

[FILTER_ALERT_CHECK] does error checking and alerting.  This is to deal with a quirk where the variable value doesn't seem to update immediately for use until another macro is called. Also, includes support for moonraker notifier.  Alerts are displayed in M117 status as well as the console.  Hopefully, you see one of them but feel free to comment out whichever you don't want.

[FILTER_RESET] resets the hour counter and is where you can define your desired maximum hours.  It is a float value to reflect print jobs less than 1 hour.  Change the maxhours value to whatever you wish.  This macro should be run every time you change your filter media.  It will call FILTER_ALERT_CHECK to display an M117 status message once complete or an error if something went wrong.

[FILTER_TIME] simply displays the currently tracked hour counter value.  Quick way to see how close you are to reaching your max.

[FILTER_ADD_TIME] does all the tracking.  Takes the total_duration of the recently completed print job and adds it to the value stored for the hourcounter variable.  Note that print jobs that are cancelled or fail and do not produce print_stats values for total_duration are not tracked.  Call this in your [PRINT_END] macro.  Calls FILTER_ALERT_CHECK to check if hour counter exceeds the max and alerts if it is.  If you use delayed gcode to turn off your filter fans, send the duration value (which is in seconds) when calling this macro in the DELAY parameter.  For example, 

```
# Calls macro with the delayed_gcode duration value of 1800 seconds (i.e., 30 minutes)
FILTER_ADD_TIME DELAY=1800
```
If you don't use delayed gcode, then simply call FILTER_ADD_TIME without the DELAY parameter.

### INSTALLATION
Copy and paste these macros into your printer.cfg or whatever file you keep your macros in.

```
[gcode_macro FILTER_ALERT_CHECK]
description: checks to make sure hourcounter is reset
gcode:
    {% set option = params.MODE|default(0)|int %}
    {% set svv = printer.save_variables.variables %}
    {% if option == 0 %}                                                  # called from FILTER RESET
        {% if svv.hourcounter == 0 %}                                     # Check if counter was reset
              # Display success message to STATUS
              M117 FILTER TIME HAS BEEN RESET TO {svv.hourcounter} HOURS  # if you reset your hour counter, it displays success message
              # Display success message to CONSOLE
              {action_respond_info("FILTER TIME HAS BEEN RESET TO %.2f HOURS."|format(svv.hourcounter))}
        {% else %}                                                        # Reset wasn't successful for some reason
              # Display error message to STATUS
              M117 ERROR: UNABLE TO RESET COUNTER
              # Display error message to CONSOLE
              {action_respond_info("ERROR: UNABLE TO RESET FILTER HOUR COUNTER.  Check your variables.cfg to make sure data is correct.")}
        {% endif %} 
    {% elif option == 1 %}                                                # called from FILTER_ADD_TIME
        {% if svv.hourcounter > svv.maxhours %}                           # Check if max hours is exceeded.        
            # Display exceeded warning to STATUS
            M117 WARNING: FILTER CARBON EXCEEDS {svv.maxhours} HOURS     
            # Display exceeded warning to CONSOLE
            {action_respond_info("WARNING: FILTER CARBON EXCEEDS %.2f HOURS WITH %.2f HOURS."|format(svv.maxhours, svv.hourcounter))}
            #######################################################################################################################################################
            # MOONRAKER NOTIFIER 
            # Requires moonraker notifier bot to be enabled.  See: https://moonraker.readthedocs.io/en/latest/configuration/#notifier
            # The command below calls the moonraker notifier.  Change NOTIF_NAME of the name variable to whatever you call your notifier and uncomment the line.
            ########################################################################################################################################################
            # Sends notification via notifier to your moonraker notification service
            {action_call_remote_method("notify", name="NOTIF_NAME", message="WARNING: FILTER CARBON EXCEEDS %.2f HOURS WITH %.2f HOURS."|format(svv.maxhours, svv.hourcounter))}
        {% endif %}
    {% endif %}

[gcode_macro FILTER_RESET]
description: sets max hours and resets hour counter
gcode:
    SAVE_VARIABLE VARIABLE=maxhours VALUE=300.00               # Set maximum number of hours before displaying reminder. Change it to whatever value you want.
    SAVE_VARIABLE VARIABLE=hourcounter VALUE=0                 # Reset print hours counter before displaying reminder
    FILTER_ALERT_CHECK                                         # need to do this because the variable value doesn't update immediately for some reason

[gcode_macro FILTER_TIME]
description: displays current filter hours
gcode:
    {% set svv = printer.save_variables.variables %}
    M117 FILTER CARBON IS {svv.hourcounter} HOURS OLD
    {action_respond_info("FILTER CARBON IS %.2f HOURS OLD.  YOUR MAX HOURS IS: %.2f"|format(svv.hourcounter, svv.maxhours))}

[gcode_macro FILTER_ADD_TIME]
description: updates the hourcounter variable with the recently completed print.  call this in your print_end.
gcode:
   {% set svv = printer.save_variables.variables %}
   {% set extra_time = params.DELAY|default(0)|int %}              # delayed gcode extra time after a print ends, provided in SECONDS
   # Convert total_duration to hours
   {% set total_hours = ((printer.print_stats.total_duration/3600)|round(2,'common')) %}
   # Convert delayed gcode extra time to hours and add it to total_hours
   {% set total_hours = total_hours + (extra_time/3600)|round(2,'common') %}
   # Check if there is even a value (for cases where cancelled/failed print doesn't produce one). If not, do nothing.
   {% if total_hours > 0 %}
        SAVE_VARIABLE VARIABLE=hourcounter VALUE={svv.hourcounter + total_hours}
        # Call macro to check if hour counter exceeds max
        FILTER_ALERT_CHECK MODE=1     
   {% endif %}
```

# info
- spent the day figuring out amd overclocks this is what i came up with

# unlock gpu settings
- allows overclocking

```
sudo nano /etc/default/grub # edit bootloader
```
```
sudo update-grub && sudo update-grub2 && sudo update-initramfs -u -k all && reboot # update grub and reboot pc
```
- editing /etc/default/grub (change to the text below)
```
GRUB_CMDLINE_LINUX_DEFAULT="amdgpu.ppfeaturemask=0xfffd7fff"
```
- extra info
- amdgpu.ppfeaturemask=0xfffd7fff creates /sys/class/drm/card0/device/pp_od_clk_voltage that you can use for overclocking.
- amdgpu.ppfeaturemask=0xffffffff could also work but has a higher chance to cause issues (mainly RX 470/570)

# setting overclocks and undervolts
## my examples are with vega10 so make sure you read what to do if you are using vega20
## double check card number and hwmon
- example of setting clocks for rx570
```
sudo su # enter root user
echo manual > /sys/class/drm/card1/device/power_dpm_force_performance_level
echo 150000000 > /sys/class/drm/card1/device/hwmon/hwmon1/power1_cap
echo 5 > /sys/class/drm/card1/device/pp_power_profile_mode
echo "s 1 1150 890" > /sys/class/drm/card1/device/pp_od_clk_voltage
echo "m 1 1990 900" > /sys/class/drm/card1/device/pp_od_clk_voltage
echo 'c' > /sys/class/drm/card1/device/pp_od_clk_voltage
echo 1 > /sys/class/drm/card1/device/pp_dpm_sclk
echo 1 > /sys/class/drm/card1/device/pp_dpm_mclk
echo 1 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1_enable
echo 220 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1
exit # exit root user
```

- more info below to help understand the above
```
switch to manual mode                /sys/class/drm/card1/device/power_dpm_force_performance_level
set power level in microWatts        /sys/class/drm/card1/device/hwmon/hwmon1/power1_cap
set to compute mode                  /sys/class/drm/card1/device/pp_power_profile_mode
edit core or memory clock values     /sys/class/drm/card1/device/pp_od_clk_voltage
set core clock speed                 /sys/class/drm/card1/device/pp_dpm_sclk
set memory clock speed               /sys/class/drm/card1/device/pp_dpm_mclk
switch to manual fan mode            /sys/class/drm/card1/device/hwmon/hwmon1/pwm1_enable
set fan speed                        /sys/class/drm/card1/device/hwmon/hwmon1/pwm1
```

```
# set gpu to use manual settings
echo manual > /sys/class/drm/card1/device/power_dpm_force_performance_level

# power level in microWatts
echo 150000000 > /sys/class/drm/card1/device/hwmon/hwmon1/power1_cap

# set to compute mode
echo 5 > /sys/class/drm/card1/device/pp_power_profile_mode

# edit core clock settings
echo "s 1 1150 890" > /sys/class/drm/card1/device/pp_od_clk_voltage

# edit memory clock settings
echo "m 1 1990 900" > /sys/class/drm/card1/device/pp_od_clk_voltage

# c for commit changes that were edited
echo 'c' > /sys/class/drm/card1/device/pp_od_clk_voltage

# use new core clock settings
echo 1 > /sys/class/drm/card1/device/pp_dpm_sclk

# use new memory clock settings
echo 1 > /sys/class/drm/card1/device/pp_dpm_mclk

# turn fan speed to manual
echo 1 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1_enable

# set fan speed
echo 220 > /sys/class/drm/card1/device/hwmon/hwmon1/pwm1
```

- https://www.kernel.org/doc/html/latest/gpu/amdgpu.html#gpu-sysfs-power-state-interfaces
- https://www.kernel.org/doc/html/v5.10/gpu/amdgpu.html
- refer to these kernel docs if needed
- these doc's say the below
```
< For Vega10 and previous ASICs >

To manually adjust these settings

first select manual using power_dpm_force_performance_level

Enter a new value for each level by writing a string that contains “s/m level clock voltage” to the file

E.g., “s 1 500 820” will update sclk level 1 to be 500 MHz at 820 mV;

“m 0 350 810” will update mclk level 0 to be 350 MHz at 810 mV

When you have edited all of the states as needed, write “c” (commit) to the file to commit your changes

If you want to reset to the default power levels, write “r” (reset) to the file to reset them.
```
```
< For Vega20 and newer ASICs >
To manually adjust these settings:

    First select manual using power_dpm_force_performance_level

    For clock frequency setting, enter a new value by writing a string that contains “s/m index clock” to the file.
    
    The index should be 0 if to set minimum clock. And 1 if to set maximum clock.
    
    E.g., “s 0 500” will update minimum sclk to be 500 MHz. “m 1 800” will update maximum mclk to be 800Mhz.

    For sclk voltage curve, enter the new values by writing a string that contains “vc point clock voltage” to the file. The points are indexed by 0, 1 and 2.
    
    E.g., “vc 0 300 600” will update point1 with clock set as 300Mhz and voltage as 600mV.
    
    “vc 2 1000 1000” will update point3 with clock set as 1000Mhz and voltage 1000mV.

    When you have edited all of the states as needed, write “c” (commit) to the file to commit your changes

    If you want to reset to the default power levels, write “r” (reset) to the file to reset them
```

# card number
- in the above examples i was using card1 ```/sys/class/drm/card1/```
- so double check you are trying to apply settings to the correct card
- also hwmon number will change

```
/sys/class/drm/card1/device/hwmon/hwmon1/pwm1
vs
/sys/class/drm/card0/device/hwmon/hwmon0/pwm1
```
- trouble shooting
```
ls /sys/class/drm
```
```
sudo lshw -class display
```

# simple script to set settings
- might be useful, might not (would have to change for multiple gpus)
```
nano oc_script
```
- paste in the script to the oc_script file (right mouse button OR shift + ins)
- edit the variables in the script to correct values (near the top of the script)
- ctrl + s to save
- ctrl + x to exit
```
chmod 777 oc_script # allows script to be executable 
```
```
sudo ./oc_script # start the script
```
- note you can also add ```view``` at the end of ```./oc_script``` to print some info
```
sudo ./oc_script view
```
- next block is the script text to copy
```
#!/bin/bash

########## start variables to edit ##########

    # correct card number try - ls /sys/class/drm/
    card_number=1

    # clock clock values to use
    core_clockMHz=1150
    core_clockmV=890

    # memory clock values to use
    memory_clockMHZ=1990
    memory_clockmV=900

    # powerlevel in watts
    watts=150

    # fan speed (0 - 255)
    fan_speed=220

    # compute mode value
    compute_mode=5
########## end variables to edit ##########

# prints a menu of info if you type view after this script name
if [ "$1" == "view" ]; then
    echo "--- gpu setting mode ---"
    cat /sys/class/drm/card$card_number/device/power_dpm_force_performance_level
    echo "--- power level ---"
    cat /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/power1_cap
    echo "--- fan setting mode ---"
    cat /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/pwm1_enable
    echo "--- fan speed ---"
    cat /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/pwm1
    echo "--- core clock ---"
    cat /sys/class/drm/card$card_number/device/pp_dpm_sclk
    echo "--- memory clock ---"
    cat /sys/class/drm/card$card_number/device/pp_dpm_mclk
    echo "--- power profile ---"
    cat /sys/class/drm/card$card_number/device/pp_power_profile_mode
    exit
fi

# set gpu to use manual settings
echo "---setting gpu settings to manual---"
echo manual > /sys/class/drm/card$card_number/device/power_dpm_force_performance_level
cat /sys/class/drm/card$card_number/device/power_dpm_force_performance_level

# set to default low clocks before editing settings
echo "---prepare to edit settings---"
echo 0 > /sys/class/drm/card$card_number/device/pp_dpm_sclk
echo 0 > /sys/class/drm/card$card_number/device/pp_dpm_mclk

# edit core clock settings
echo "---edit core clock settings---"
echo "s 1 $core_clockMHz $core_clockmV" > /sys/class/drm/card$card_number/device/pp_od_clk_voltage

# edit memory clock settings
echo "---edit memory clock settings---"
echo "m 1 $memory_clockMHZ $memory_clockmV" > /sys/class/drm/card$card_number/device/pp_od_clk_voltage

# c for commit - save changes that were made
echo "---save new core and memory clock settings---"
echo 'c' > /sys/class/drm/card$card_number/device/pp_od_clk_voltage

# use new core clock settings
echo "---apply new core clock---"
echo 1 > /sys/class/drm/card$card_number/device/pp_dpm_sclk
cat /sys/class/drm/card$card_number/device/pp_dpm_sclk

# use new memory clock settings
echo "---apply new memory clock---"
echo 1 > /sys/class/drm/card$card_number/device/pp_dpm_mclk
cat /sys/class/drm/card$card_number/device/pp_dpm_mclk

# power level in microWatts
echo "---set power level---"
let microwatt=watt*1000000
echo $microwatt > /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/power1_cap
cat /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/power1_cap

# turn fan speed to manual
echo "---allow manual control of fans---"
echo 1 > /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/pwm1_enable
cat /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/pwm1_enable

# set fan speed
echo "---set fan speed---"
echo $fan_speed > /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/pwm1
cat /sys/class/drm/card$card_number/device/hwmon/hwmon$card_number/pwm1

# set to compute mode
echo "--- set compute mode ---"
echo $compute_mode > /sys/class/drm/card$card_number/device/pp_power_profile_mode
```

# random
- sometimes i find i have to run oc after i start mining to apply the fan and memory clock?
- idk why, does any one else?

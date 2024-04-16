**Rev 0.1**
| 0.1 | 05/27/2019  |  Systems Infra Team     | Initial version                   |
| 0.2 | 06/06/2019  |  Systems Infra Team     | Incorporated feedback             |
| 0.3 | 10/21/2019  |  Systems Infra Team     | GPIO JSON object support          |
| 0.4 | 10/22/2019  |  Systems Infra Team     | Platform 2.0 API support          |
| 0.5 | 10/31/2019  |  Systems Infra Team     | BMC Support                       |
Platform Driver Development Framework (PDDF) is part of SONiC Platform Development Kit (PDK), which enables rapid development of platform drivers and APIs for SONiC platforms. PDK consists of 
PDE details are covered in another document. This document describes Platform Driver Development Framework (PDDF) which can be used as an alternative to the existing manually-written SONiC platform driver framework. It enables platform vendors to rapidly develop the device specific custom drivers and SONiC user space python device object-classes, using a data-driven architecture, to manage platform devices like Fan, PSUs, LEDs, Optics, System EEPROM, etc., and validate a platform on SONiC. 
This document describes the high level design details of PDDF and its components. The PDDF consists of generic device drivers and user space platform APIs which use the per platform specific data in the JSON descriptor files. This document describes the interaction between all the components and the tools used to support these drivers and platform APIs.  
 - Platform drivers developed using the PDDF framework shall support the current SONiC platform CLIs  
 - PDDF developer guide	shall be provided 
	 
 SONiC PDDF (Platform driver development framework) supports the following HW devices on a given platform: 
 
 - System LED control via CPLD 
 - Temp Sensors 
 
 - PDDF JSON Descriptor files  
		 - Eg., PSU(psu_present), SFP/QSFP(sfp_present, lpmode) - CPLD registers/offsets/ masks, etc.,
For I2C based HW components, PDDF provides generic drivers for the following I2C based devices: FAN/PSU/EEPROM/Optics Transceivers/ System LED/ CPLDs/ CPLDMUX. These drivers in kernel space, rely on the per-platform data in JSON descriptor files to expose data via SysFS interface. They provide a generic interface to get/set of these attributes. There are two types of data, a driver works on:
For BMC based HW components such as FAN/PSU/TEMP sensors, PDDF does not need to provide generic drivers to manage these components. The management is done by BMCs which are located on IPMI-compliant hardware. IPMI is an open-standard hardware management interface to monitor and manage HW components. IPMItool utility is provided for users to monitor and manage BMC based components.          
	  - sonic_platform_ref (ref code for pddf derived platform APIs)
 - /service/sonic-buildimage/device/common/pddf
	  - plugins (1.0 platform plugins)
	  
For the Runtime environment, PDDF shall provide a init script pddf_util.py. Any per platform init script would be called from this script. This will load the PDDF modules and APIs and will use the per platform JSON descriptor files for initializing the platform service. 
  * Platform dependent interpretations of some of the attribute values 
			{
				"attr_name":"psu_fan_dir",
				"drv_attr_name": "psu_fan_direction"
			},
 - Fan Controller (CPLD or dedicated Controller EM2305) 
 - PSUs (YM2651, Ym2851, etc.,) 
 - Temp Sensors (LM75, LM90, TMP411, etc.,) 
 - Optics (SFP/QSFPs, EEPROM, etc.,) 
 - System EEPROM (at24, etc.,) 
 - MUX  (PCA954x,..) 
 - GPIO  (PCA955x,..) 
Generally a platform consist of fans, PSUs, temperature sensors, CPLDs, optics (SFP, QSFPs etc), eeproms and multiplexing devices. I2C topology refers to the parent-child and other connectivity details of the I2C devices for a platform. The path to reach any device can be discerned using the I2C topology.
*i2c* and *topo_info* are used for creating the I2C client.
		"channel": [ 
> **cpld_sel**: Value to be written in the **cpld_offset** to select a particular channel. 
Example of a CPLDMUX JSON object 
Some platforms require initialization settings to be performed with respect to the GPIOs. The **ports** list in the JSON object takes care of these initialization after the I2C device creation takes place. Following are the names of 
	    "interface": [  
  * Create the SysFS data attributes 
  * Get/Set  attribute's value from/to HW 
  * Create the SysFS attributes 
Network switches have a variety of LED lights, system LEDs, Fan Tray LEDs, and port LEDs, used to act as indicators of switch status and network port status.  The system LEDs are used to indicate the status of power and the system. The fan tray LEDs indicate each fan status. The port LEDs are used to indicate the state of the links such as link up, Tx/RX activity and speed. The Port LEDs are in general managed by the LED controller provided by switch vendors. The scope of this LED section  is for system LEDs and fan tray LED.
##### 3.4.5.1 LED Driver Design    
| FANTRAY\<x\>_LED         | Status LED for individual fan. X is an integer starting with 1 Example: FANTRAY1_LED, FANTRAY2_LED | 
    "PLATFORM" :  { "num_psu_led":"1",  "num_fantray_led" : "4"}  
    "PSU1_LED" :  { "dev_info": { "device_type":"LED", "device_name":"PSU_LED"},    
                       "dev_attr": { "index":"0"},   
                       "i2c": {   
                                [  
                                  {"attr_name":"on",  "bits" : "6:5", "color" : "Green", "value" : "0x1", "swpld_addr" : "0x60", "swpld_addr_offset" : "0x66"},  
                                  {"attr_name":"faulty",  "bits" : "6:5", "color" : "Amber", "value" : "0x2", "swpld_addr" : "0x60", "swpld_addr_offset" : "0x66"},  
                                  {"attr_name":"off",  "bits" : "6:5", "color" : "Off", "value" : "0x3", "swpld_addr" : "0x60", "swpld_addr_offset" : "0x66"}  
##### 3.4.6.1 Driver Design  
The Linux driver supports LM75/LM90 compatible temperature sensors.  It is used to support communication through the I2C bus and interfaces with the hardware monitoring sub-system. A SysFS interface is added to let the user provides the temperature sensors information to the kernel to instantiate I2C devices.   
| TEMP\<x\>                     | Temperature sensor. x is an integer starting with 1         |  
    Samples:
    "PLATFORM" :  { "num_temp_sensors":"3"}  
                 "i2c": { 
                             "topo_info": { "parent_bus":"0x21", "dev_addr":"0x48", "dev_type":"lm75"},
                             "attr_list":
                       		 [
                                		{ "attr_name": "temp1_high_threshold", "drv_attr_name":"temp1_max"},
                                		{ "attr_name": "temp1_input"}
                             ]
                        }
                }
##### 3.4.6.3 Thermal Object Class Design  
#### 3.4.8 System Status Registers
##### 3.4.8.1 Driver Design
##### 3.4.8.2 JSON Design
This SYSTATUS JSON object can be used to get miscellaneous info from various CPLDs/devices based on the platform. Currently there is no generic class/plugin defined for system status registers. 
#### 3.4.9 Optics Component
##### 3.4.9.1	Driver design
Transceiver devices (SFP, QSFP etc.) expose mainly two kinds of access/control methods.  
EEPROM bin        
##### 3.4.9.2 JSON Design
#### 3.4.10 lm-sensors Tools
##### 3.4.10.1 sensors.conf  
/etc/sensors.conf is a user customized configuration file for libsensors. It describes how libsensors, and so all programs using it, should translate the raw readings from the kernel modules to real-world values. A user can configure each chip, feature and sub-feature that makes sense for his/her system.     
         chip "lm75-i2c-3-49"  
     admin@sonic:~$ sensors  
##### 3.4.10.2 fancontrol:
fancontrol is a shell script for use with lm_sensors. It reads its configuration from a file, /etc/fancontrol, then calculates fan speeds from temperatures and sets the corresponding PWM outputs to the computed values.  
    Example of configuration file   
### 3.5 PDDF BMC Component Design
This section covers the JSON design for BMC based hardware components. PDDF utilizes ipmitool to monitor components.   
 - Fan Controller 
<Device Name> : {
If this field exists, the device name is displayed using this field. Otherwise, the device_name content is shown.   
> **bmc_cmd**: 
There are two types of cmds: raw ipmi request and non raw ipmi request. The list of available ipmitool commands can be found by 
> **raw**: 
> **field_name**: 
This is the first field of an ipmitool command output. Each vendor has different naming conventions. Please check vendor specific documents. 
For a non-raw ipmi request, this field is used to select a specific field position.  
> **type**: 
This field indicates the output presentation. It is a pre-defined list, "ascii", "raw", and "mask". 
  >- "mask": the output needs to be masked with the value of "mask" field. 
> **mask**: 
This is a hex number to mask the output 
    The output of PSU1_POWER_IN is "ok" because the field_index is 3. 
      37 30 30 2d 30 31 33 36 38 34 2d 30 31 30 30 
      The field of type is "ascii", so the output is converted from hex to string. The ascii format is 700-013684-0100. 
    
       The field of type is "mask". For example, bit value of 1:absent and bit value of 0:present. PSU1 is present.    
#### 3.5.1 PSU JSON 
    
#### 3.5.2 FAN JSON 
 >- fan\<index\>_present: present status of FAN  
 >- fan\<index\>_input: rpm speed of front FAN 
 >- fan\<index\>_pwm: puls-width modulation of FAN 
 >- fan\<index\>_direction: direction of FAN 
    
#### 3.5.3 TEMP Sensors JSON 
 >- temp1_input: current temperature reading from theraml 
    
           
#### 3.5.4 IPMITOOL OUTPUTS
   
  
### 3.6 PDDF Platform APIs Design
#### 3.6.1 PSU Class
#### 3.6.2 FAN Class
#### 3.6.3 LED Class
There is no generic LED API class defined in PDDF. LED APIs related to a component has been made part of thats component's platform API class. System LED APIs are made part of PddfChassis class.  
```    
#### 3.6.4 System EEPROM Class
#### 3.5.5 Optics Class
SONiC provides various platform related CLIs to manage various platform devices. Some of the existing CLIs are: 
   device_name: configured in the JSON configuration file such as LOC_LED and DIAG_LED 
   color: STATUS_LED_COLOR_GREEN, STATUS_LED_COLOR_RED, STATUS_LED_COLOR_OFF
        STATUS_LED_COLOR_GREEN maps to LED at normal state
        STATUS_LED_COLOR_RED maps to LED at faulty state
        STATUS_LED_COLOR_OFF maps to LED at off state
# pddf_ledutil setstatusled LOC_LED STATUS_LED_COLOR_GREEN
Blue
```    
# 
# pddf_thermalutil numthermals 
```    
> NOTE: This utility is not yet migrated to platform 2.0 APIs. There should only be one coloumn mentioning speed of a fan as the rear fans are considered as separate fans in platform 2.0 APIs. 

## 7 Warm Boot Support
## 8 Scalability
## 9 Unit Test
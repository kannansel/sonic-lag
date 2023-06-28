**Rev 0.7**
		 * [FPGAI2C Component](#fpgai2c-component)
	     * [FPGAPCIe Component](#pddf-fpgapcie-component)
 * [S3IP Standard Support](#s3ip-standard-support)
	  * [S3IP PDDF Requirements](#s3ip-pddf-requirements)
	  * [Implementation Details](#implementation-details)
		 * [PDDF and S3IP SysFS](#pddf-and-s3ip-sysfs)
		 * [S3IP SysFS Creation and Mapping](#s3ip-sysfs-creation-and-mapping)
		 * [Adding S3IP Support for a Platform](#adding-s3ip-support-for-a-platform)
| 0.1 | 05/27/2019  |  Fuzail Khan, Precy Lee     | Initial version                   |
| 0.2 | 06/06/2019  |  Fuzail Khan, Precy Lee     | Incorporated feedback             |
| 0.3 | 10/21/2019  |  Fuzail Khan, Precy Lee     | GPIO JSON object support          |
| 0.4 | 10/22/2019  |  Fuzail Khan, Precy Lee     | Platform 2.0 API support          |
| 0.5 | 10/31/2019  |  Fuzail Khan, Precy Lee     | BMC Support                       |
| 0.6 | 10/01/2020  |  Fuzail Khan, Precy Lee     | FPGAI2C component support         |
| 0.7 | 01/05/2023  |  Fuzail Khan, Precy Lee     | FPGAPCIe component support        |
| 0.8 | 03/17/2023  |  Fuzail Khan, Precy Lee     | S3IP SysFS support        |
Platform Driver Development Framework (PDDF) is part of SONiC Platform Development Kit (PDK), which enables rapid development of platform drivers and APIs for SONiC platforms. PDK consists of
PDE details are covered in another document. This document describes Platform Driver Development Framework (PDDF) which can be used as an alternative to the existing manually-written SONiC platform driver framework. It enables platform vendors to rapidly develop the device specific custom drivers and SONiC user space python device object-classes, using a data-driven architecture, to manage platform devices like Fan, PSUs, LEDs, Optics, System EEPROM, etc., and validate a platform on SONiC.
This document describes the high level design details of PDDF and its components. The PDDF consists of generic device drivers and user space platform APIs which use the per platform specific data in the JSON descriptor files. This document describes the interaction between all the components and the tools used to support these drivers and platform APIs.
	 - FPGA
 - Platform drivers developed using the PDDF framework shall support the current SONiC platform CLIs
 - PDDF developer guide	shall be provided
	
 SONiC PDDF (Platform driver development framework) supports the following HW devices on a given platform:

 - FPGA
 - System LED control via CPLD or FPGA
 - Temp Sensors

 - PDDF JSON Descriptor files
		 - e.g. PSU(psu_present), SFP/QSFP(sfp_present, lpmode) via CPLD/FPGA registers/offsets/masks, etc.
For I2C based HW components, PDDF provides generic drivers for the following I2C based devices: FAN/PSU/EEPROM/Optics Transceivers/ System LED/ CPLDs/ CPLDMUX FPGA. These drivers in kernel space, rely on the per-platform data in JSON descriptor files to expose data via SysFS interface. They provide a generic interface to get/set of these attributes. There are two types of data, a driver works on:
For BMC based HW components such as FAN/PSU/TEMP sensors, PDDF does not need to provide generic drivers to manage these components. The management is done by BMCs which are located on IPMI-compliant hardware. IPMI is an open-standard hardware management interface to monitor and manage HW components. IPMItool utility is provided for users to monitor and manage BMC based components.
	  - sonic_platform_ref (reference code for pddf derived platform APIs)
For the Runtime environment, PDDF shall provide a init script pddf_util.py. Any per platform init script would be called from this script. This will load the PDDF modules and APIs and will use the per platform JSON descriptor files for initializing the platform service.
  * Platform dependent interpretations of some of the attribute values
			{ "attr_name":"psu_fan_dir", "drv_attr_name": "psu_fan_direction"},
 - Fan Controller (CPLD or dedicated Controller EM2305)
 - PSUs (YM2651, Ym2851, etc.,)
 - Temp Sensors (LM75, LM90, TMP411, etc.,)
 - Optics (SFP/QSFPs, EEPROM, etc.,)
 - System EEPROM (at24, etc.,)
 - FPGAs
 - MUX  (PCA954x,..)
 - GPIO  (PCA955x,..)
Generally a platform consist of fans, PSUs, temperature sensors, CPLDs, FPGAs, optics (SFP, QSFPs etc), eeproms and multiplexing devices. I2C topology refers to the parent-child and other connectivity details of the I2C devices for a platform. The path to reach any device can be discerned using the I2C topology.
*i2c* and *topo_info* are used for creating the I2C client or a platform device.
		"channel": [
> **cpld_sel**: Value to be written in the **cpld_offset** to select a particular channel.
Example of a CPLDMUX JSON object
Some platforms require initialization settings to be performed with respect to the GPIOs. The **ports** list in the JSON object takes care of these initialization after the I2C device creation takes place. Following are the names of
	    "interface": [
  * Create the SysFS data attributes
  * Get/Set  attribute's value from/to HW
  * Create the SysFS attributes
Network switches have a variety of LED lights, system LEDs, Fan Tray LEDs, and port LEDs, used to act as indicators of switch status and network port status.  The system LEDs are used to indicate the status of power and the system. The fan tray LEDs indicate each fan-tray status. The port LEDs are used to indicate the state of the links such as link up, Tx/RX activity and speed. The Port LEDs are in general managed by the LED controller provided by switch vendors. The scope of this LED section is for system LEDs and fan-tray LED.
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
                     "attr_list": [
                         { "attr_name": "temp1_high_threshold", "drv_attr_name":"temp1_max"},
                         { "attr_name": "temp1_input"}
                     ]
                 }
               }
##### 3.4.6.3 Thermal Object Class Design
#### 3.4.8 FPGAI2C Component
FPGAs on the board can be used in two different ways. Their driver implementations, **access-data** attributes and usage differ in each case. This section deals with FPGA component which is similar to a CPLD in terms of its function. It is connected to an I2C bus with a specific I2C device address. Data is read or written from different offsets on the FPGA using simple smbus_read/smbus_write APIs. Such FPGA is termed as FPGAI2C in PDDF.
PDDF has a FPGAI2C module and a FPGAI2C driver module. FPGAI2C module takes care of access-data attributes to transfer the platform and device specific information to kernel space using SysFS interface. FPGAI2C driver module generates the SysFS interface for the device-data attributes. It also provides the read/write functionality for the FPGA.
##### 3.4.8.1 FPGAI2C JSON Design
FPGAI2C JSON object is simple where I2C topolcy is defined under **i2c** keyword. There are no device-data SysFS attributes as the information being read from FPGA is not fixed. Here is a typical FPGAI2C JSON object.
```
"FPGAI2C1":
{
    "dev_info": { "device_type":"FPGA_I2C", "device_name":"FPGAI2C1", "device_parent":"SMBUS0"},
    "i2c":
    {
        "topo_info": { "parent_bus":"0x0", "dev_addr":"0x5E", "dev_type":"i2c_fpga"},
        "dev_attr": { }
    }
},
```

#### 3.4.9 System Status Registers

##### 3.4.9.1 Driver Design
##### 3.4.9.2 JSON Design
This SYSTATUS JSON object can be used to get miscellaneous info from various CPLDs/devices based on the platform. Currently there is no generic class/plugin defined for system status registers.
#### 3.4.10 Optics Component
##### 3.4.10.1	Driver design
Transceiver devices (SFP, QSFP etc.) expose mainly two kinds of access/control methods.
EEPROM bin
##### 3.4.10.2 JSON Design
#### 3.4.11 lm-sensors Tools
##### 3.4.11.1 sensors.conf
/etc/sensors.conf is a user customized configuration file for libsensors. It describes how libsensors, and so all programs using it, should translate the raw readings from the kernel modules to real-world values. A user can configure each chip, feature and sub-feature that makes sense for his/her system.
         chip "lm75-i2c-3-49"
     admin@sonic:~$ sensors
##### 3.4.11.2 fancontrol:
fancontrol is a shell script for use with lm_sensors. It reads its configuration from a file, /etc/fancontrol, then calculates fan speeds from temperatures and sets the corresponding PWM outputs to the computed values.
    Example of configuration file
#### 3.4.12 PFGAPCIe Component

![FPGAPCIe Diagram](../../images/platform/fpgapcie_i2c_diagram.png "FPGAPCIe I2C topology diagram")

FPGA can be programmed as a I2C master controller. Some platforms use a FPGAPCIe card to control I2C devices and  the communication with the CPU is by PCIe interface. PDDF supports a FPGAPCIe card by providing the following modules:

* FPGAPCIe Data Module:
    - Mange access data defined in JSON via SysFS interface
    - Populate data and trigger FPGAPCIe instantiation
* FPGAPCIe Driver Module:
    - PCIe device instantiation
    - Logical I2C bus instantiation
* I2C Algorithm Module:
    - Device specific I2C communication protocol
    - Register algorithm to I2C adaptor.

PDDF PFGAPCIe common driver supports only the FPGAPCIe model where FPGA is acting as I2C master controller. Any other kind of use case of a PCIe
connected FPGA, vendors need to provide the FPGA driver.

##### 3.4.12.1 FPGAPCIe JSON Design

FPGAPCIe JSON object follows PDDF I2C topology JSON object design concept. FPGAPCIE object is under i2c keyword becuase it is programmed as I2c buses to control I2C client devices.


```
    "SYSTEM":
    {
        "dev_info": {"device_type":"CPU", "device_name":"ROOT_COMPLEX", "device_parent":null},
        "i2c":
        {
            "CONTROLLERS":
            [
                { "dev_name":"i2c-0", "dev":"SMBUS0" },
                { "dev_name":"pcie-0", "dev":"PCIE0" }
            ]
        }
    },

    "PCIE0"
    {
        "dev_info": {"device_type": "PCIE", "device_name": "PCIE0", "device_parent": "SYSTEM"},
        "i2c":
        {
            "DEVICES":
            [
               {"dev": "FPGAPCIE0"}
            ]
        }
    },

   "FPGAPCIE0":
   {
       "dev_info": {"device_type": "FPGAPCIE", "device_name": "FPGAPCIE0", "device_parent": "PCIE0"},
       "i2c":
       {
          "dev_attr": { "vendor_id":"0x10EE", "device_id": "0x7021", "virt_bus": "0x64", "data_base_offset":"0x0", "data_size":"0x60", "i2c_ch_base_offset":"0x600", "i2c_ch_size":"0x10",  "virt_i2c_ch":"8"},
          "channel":
          [
              { "chn":"3", "dev":"MUX1" }
          ]
       }
   },

```

![I2C Memory Mapping Diagram](../../images/platform/pcie_i2c_memory_mapping.png "FPGAPCIe I2C Memory Mapping")


Description of the fields inside *dev_attr*

> **vendor_id**: The 16-bit register specifies the PCI vendor.

> **device_id**: The 16-bit register selected by the PCI vendor.

> **virt_bus**: This is an information used internally to denote the base address for the channels of the mux. So if the virt_bus is 0x64 for a pca9548 then channel-buses are addressed as (0x64+0), (0x64+1), (0x64+2) .... , (0x64+7).

> **data_base_offset**: PCIe user-defined memory mapping offset address from PCIE BAR 0

> **data_size**: User-defined memory allocation size

> **i2c_ch_base_offset**: I2C channel 1 offset address from PCIE BAR 0

> **i2c_ch_size**: Memory offset for each I2C channel.

> **virt_i2c_ch**: The total numbers of logical I2C channels


### 3.6 PDDF BMC Component Design

This section covers the JSON design for BMC based hardware components. PDDF utilizes ipmitool to monitor components.
 - Fan Controller
\<Device Name\> : {
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
#### 3.6.1 PSU JSON

#### 3.6.2 FAN JSON
 >- fan\<index\>_present: present status of FAN
 >- fan\<index\>_input: rpm speed of front FAN
 >- fan\<index\>_pwm: puls-width modulation of FAN
 >- fan\<index\>_direction: direction of FAN

#### 3.6.3 TEMP Sensors JSON
 >- temp1_input: current temperature reading from theraml


#### 3.6.4 IPMITOOL OUTPUTS


### 3.7 PDDF Platform APIs Design
#### 3.7.1 PSU Class
#### 3.7.2 FAN Class
#### 3.7.3 LED Class
There is no generic LED API class defined in PDDF. LED APIs related to a component has been made part of thats component's platform API class. System LED APIs are made part of PddfChassis class.
```
#### 3.7.4 System EEPROM Class
#### 3.7.5 Optics Class
SONiC provides various platform related CLIs to manage various platform devices. Some of the existing CLIs are:
   device_name: configured in the JSON configuration file such as LOC_LED and DIAG_LED
   color: green, red, off
        green maps to LED at normal state
        red maps to LED at faulty state
        off maps to LED at off state
# pddf_ledutil setstatusled LOC_LED green
green
```
#
# pddf_thermalutil numthermals
```
## 7 S3IP Standard Support

S3IP sysfs specification defines a unified interface to access peripheral hardware on devices from different vendors, making it easier for SONiC to support different devices and platforms. The S3IP standard support is now available with PDDF. If the user wants, there is a provision to enable/create S3IP sysfs standards.

### 7.1 S3IP PDDF Requirements

- S3IP sysfs should be generated and could be removed on requirement
- Though S3IP can be clubbed with PDDF, PDDF should be independent of the S3IP
- If any attribute which cannot be read should have a value of 'NA' i.e. tools should not fail due to non existance of the attribute
- S3IP sysfs should be able to work with the existing PDDF common driver sysfs
- PDDF common driver attributes should be expanded, if required, to cover the left out attributes from S3IP specifications

### 7.2 Implementation Details

The S3IP specifications and framework are defined [here](https://github.com/sonic-net/SONiC/pull/1068). Both vendors and users are required to follow the S3IP spec. The platform vendors need to provide the implementation of the set/get attribute functions for the platforms which use S3IP sysfs framework. The attributes for each component are defined in the specificaitons. This effort is to combine the S3IP spec and PDDF framework. In other words, the platform which are using PDDF would be S3IP compliant too after this support is added.

#### 7.2.1 PDDF and S3IP SysFS

PDDF implements common kernel drivers for various components. These common drivers exposes a fixed set of sysfs attributes as per the HW support and current SONiC API requirements. Complying to S3IP spec requires the mapping of S3IP component attributes to PDDF exposed sysfs attributes and might even require adding new attributes to PDDF common driver code. Hence, S3IP spec sysfs attributes are divided into the following categories.

 - Platform Info Attributes: This includes the fixed information pertaining to the platform in entirity or any component. There is no need of reading this information from the component in run time. Further, these values will not change in the course of System running the SONiC image. Below are few examples of static info attributes.

     - /sys_switch/temp_sensor/number, /sys_switch/vol_sensor/number, /sys_switch/curr_sensor/number etc.

     - /sys_switch/cpld/cpld[n]/alias, /sys_switch/temp_sensor/temp[n]/alias, /sys_switch/temp_sensor/temp[n]/type etc.

   S3IP file system can be created and the information can be directly written from the PDDF JSON files to them. Since this is static information, there is no need of repeatedly updating it.

 - Component Attributes: These are the attributes are to be read from various HW components. These could be dynamically changing information like PSU voltage and temperature, or fixed like PSU serial number or FAN model etc.

     - Some such S3IP sysfs would match the sysfs exposed by the PDDF frameowrk and hence a proper mapping with softlink creation would suffice.

     - Some S3IP sysfs would not match directly with PDDF exposed sysfs. For such attributes, either PDDF common dirvers can be enhanced to provide the exact match or some other method can be used.

     - There are some S3IP attributes which can't and won't be mapped to common PDDF driver attributes because PDDF common code doesn't support these devices e.g. volt_sensors, curr_sensors etc. There are no PDDF common drivers for such devices. However, ODMs might extend the PDDF framework by providing the custom drivers. In such cases, ODMs need to take care of mapping the S3IP attributes to the attributes exposed by the custom drivers.

     - We faced a practical issue for some S3IP attributes e.g. Fan status. S3IP description says

     |Sysfs path|Permissions|Data type|Description|
     |-|-|-|-|
     |/sys_switch/fan/fan[n]/status |RO| enum| Fan states are defined as follows:<br>0: not present<br>1: present and normal<br>2: present and abnormal

     - This is a combination of 'presence' and 'running_status' informations of a fan unit. In SONiC we can handle this in the platform APIs but S3IP compels to performs this processing inside the kernel modules. Hence if ODM extends the PDDF driver and provide the kernel implementation of such sysfs, we can create the mapping. Otherwise we will map it to 'NA'.

#### 7.2.2 S3IP SysFS Creation and Mapping

![S3IP Support in PDDF](../../images/platform/s3ip_pddf.jpg "S3IP Support in PDDF")


If the S3IP sysfs is required on a PDDF platform, it can be represented using the field "enable_s3ip" in the PDDF JSON file. If this field is not mentioned or has a value "no", then the S3IP sysfs creation is disabled for that platform. The support for S3IP is controlled by a service "pddf-s3ip-init.service". This service is run at the end of PDDF platform initialization service. It is an standalone service which needs to have an 'after' dependency on PDDF platform init service.
```
    "PLATFORM":
    {
        "num_psus":2,
        "num_fantrays":4,
        "num_fans_pertray":2,
        "num_ports":64,
        "num_temps": 8,
        "enable_s3ip": "yes",
        "pddf_dev_types":
        {
            "description":"AS7816-64X - Below is the list of supported PDDF device types (chip names) for various components. If any component uses some other driver, we will create the client using 'echo <de
            "CPLD":
            [
                "i2c_cpld"
            ],
            "PSU":
            [
                "psu_eeprom",
                "psu_pmbus"
            ],
            "FAN":
            [
                "fan_ctrl",
                "fan_eeprom"
            ],
            "PORT_MODULE":
...
...
```

This pddf-s3ip service would create the sysfs as per the standards. It will also take care of linking the appropriate PDDF sysfs with the corrosponding S3IP sysfs.

In case the platform does not support some attributes present in the S3IP spec, 'NA' will be written to the attribute file so that the application does not fail.

Once this is done, users can run their S3IP compliant applicaitons and test scripts on the platform.

#### 7.2.3 Adding S3IP Support for a Platform

For adding support for S3IP on a platform which is already using PDDF for bringup, here are the steps.

- Add "enable_s3ip": "yes" into the pddf-device.json file for that platform
```
         "num_fans_pertray":1,
         "num_ports":54,
         "num_temps": 3,
+        "enable_s3ip": "yes",
         "pddf_dev_types":
         {
```

- Create a softlink for the 'pddf-s3ip-init.service' inside service folder for that platform.

```
diff --git a/platform/broadcom/sonic-platform-modules-<odm>/<platform>/service/pddf-s3ip-init.service b/platform/broadcom/sonic-platform-modules-<odm>/<platform>/service/pddf-s3ip-init.service
new file mode 120000
index 000000000..f1f7fe768
--- /dev/null
+++ b/platform/broadcom/sonic-platform-modules-<odm>/<platform>/service/pddf-s3ip-init.service
@@ -0,0 +1 @@
+../../../../pddf/i2c/service/pddf-s3ip-init.service
\ No newline at end of file


# ls -l platform/broadcom/sonic-platform-modules-<odm>/<platform>/service/pddf*
total 3
lrwxrwxrwx 1 fk410167 nwsoftusers  55 May  8 02:02 pddf-platform-init.service -> ../../../../pddf/i2c/service/pddf-platform-init.service
lrwxrwxrwx 1 fk410167 nwsoftusers  55 May  8 02:02 pddf-s3ip-init.service -> ../../../../pddf/i2c/service/pddf-s3ip-init.service
#

```

Build the platform and sonic-device-data debian packages and load the build on the respective platform.

## 8 Warm Boot Support
## 9 Scalability
## 10 Unit Test
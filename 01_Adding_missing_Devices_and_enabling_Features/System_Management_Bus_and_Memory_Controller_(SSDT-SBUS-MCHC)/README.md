# Fixing AppleSMBus (`SSDT-SBUS-MCHC`)

## Description
What is AppleSMBus? Well, it mainly handles the System Management Bus, which has various functions, such as:

* **AppleSMBusController**: Aids with correct Temperature, Fan, Voltage, ICH, and other readings.  
* **AppleSMBusPCI**: Same idea as AppleSMBusController except for low bandwidth PCI devices.
* **Memory Reporting**: Aids in proper memory reporting and can aid in getting better memory-related kernel panic details.

Other things the System Management Bus does can be found in the [**SMBus WIKI**](https://en.wikipedia.org/wiki/System_Management_Bus).

In order for the SMBus to work properly under macOS, Device `SMBUS` (respectively `SBUS`) has to be present in the IO Registry. Additionally, the Memory Host Controller Device `MCHC` which is serviceable in macOS also needs to be present to wire it to the power management of the PCI bus. So we need to verify the presence/absence of both devices to decide which SSDT(s) needs to be added.

## Instructions

### 1. Evaluating if you need this Fix
- Run Terminal and enter:
	```shell
	kextstat | grep -E "AppleSMBusController|AppleSMBusPCI"
	```
- If the Terminal output contains the following 2 drivers, your SMBus is working correctly:
	![sbus_present](https://user-images.githubusercontent.com/76865553/140615883-3c8af435-b09a-4a3e-9746-28f8a05c9e37.png)
	If non or only one of the drivers is listed, you need a fix.
- Next, run **IORegistryExplorer**
- Search for `MCHC`. If present, it looks like this:</br>![MCHC](https://user-images.githubusercontent.com/76865553/189326100-0ee38b2b-942e-4379-bbba-c92cceb75ba4.png)
- If it's not present, you need a fix. Take a mental note.
- Next, search for **SMBU** or **SBUS** respectively. If present, it looks like this:</br>![SMBU](https://user-images.githubusercontent.com/76865553/189326159-96b10b62-4d89-45c5-99b5-d975f51a6463.png)
- In this case you don't need **SSDT-SMBU** nor **SSDT-SBUS**.
- If it's not present, continue with Step 2.

### 2. Finding the PCI name and path of your system's SMBus

In **DSDT**, search for:

- `0x001F0003` (Broadwell and older) or `0x001F0004` (Skylake and newer) 
- Find device name and location of the SMBus Device. It will either be called `SBUS` or `SMBU`. In this example, it's called `SMBU` and is located under `_SB/PCI0/`:</br>![sbusmchc](https://user-images.githubusercontent.com/76865553/177932530-f2190e85-17f2-4d15-9326-c37cd4c410e3.png)

### 3. Picking the correct SSDT[^1]
Depending on the search results, add the following SSDT to your ACPI Folder and `config.plist`:

- If `SMBU`/`SBUS` is present, but `MCHC` is missing: add ***SSDT-MCHC.aml***
- If `MCHC` is present (highly unlikely) but `SMBU`/`SBUS` is missing, add either:
	- ***SSDT-SBUS.aml*** (if the Device name is `SBUS`) or
	- ***SSDT-SMBU.aml*** (if the device name is `SMBU`)
- If neither `SMBU`/`SBUS` nor `MCHC` are present or if you are in doubt: add ***SSDT-SBUS-MCHC.aml*** (adjust `SMBU`/`SBUS` path accordingly)
- Save and reboot.

### 4. Verify that it's working

Run the GREP command again:
```shell
kextstat | grep -E "AppleSMBusController|AppleSMBusPCI"
```

If the Terminal output contains the following 2 drivers, your SMBus is working correctly:

![sbus_present](https://user-images.githubusercontent.com/76865553/140615883-3c8af435-b09a-4a3e-9746-28f8a05c9e37.png)

[^1]: Additional information about `AppleSMBus` as well as the `GREP` command for testing  were taken from Dortania's Post-Install Guide, since the original Guide by DalianSky was lacking in this regard. The SSDT sample included in the OpenCore package combines `SSDT-SBUS/SMBUS` and `SSDT-MCHC` into one file (`SSDT-SBUS-MCHC.aml`), so I suggest you use this instead.

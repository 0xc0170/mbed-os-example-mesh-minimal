# Example mesh application for Mbed OS

With this application, you can use the [mesh networking API](https://os.mbed.com/docs/latest/reference/mesh-api.html) that [Mbed OS](https://github.com/ARMmbed/mbed-os) provides.

The application demonstrates a light control application, where devices can control the LED status of all devices in the network.
The application can be built for the unsecure 6LoWPAN-ND or Thread network.

See the [6LoWPAN overview](https://os.mbed.com/docs/latest/reference/mesh-tech.html) for the definition of star and mesh networks. These same principles apply also to Thread protocol.

## Setup

### Download the application

```
mbed import mbed-os-example-mesh-minimal
cd mbed-os-example-mesh-minimal
```

### Change the channel settings (optional)

See the file `mbed_app.json` for an example of defining an IEEE 802.15.4 channel to use.

### Selecting optimal Nanostack configuration

To optimize the flash usage, select a proper configuration for Nanostack. The configuration depends mostly on the preferred use case.

Select the protocol the network is based on:

- 6LoWPAN-ND.
- Thread.

Select the device role:

- Mesh network. A router. (default)
- Star network. Nonrouting device. Also known as a host or sleepy host.

Modify your `mbed_app.json` file to see which Nanostack and [Mbed Mesh API](https://os.mbed.com/docs/latest/reference/mesh-api.html) configuration to use.

Example configuration files are provide under `configs/` directory. You may override the `mbed_app.json` with either of these.

|configuration file|Use for|
|------------------|-------|
|`configs/mesh_6lowpan.json` | 6LoWPAN-ND based mesh network. |
|`configs/mesh_thread.json` | Thread based mesh network. |

An example of the `mbed_app.json` file:

```
...
        "mesh-type":{
            "help": "options are MESH_LOWPAN, MESH_THREAD",
            "value": "MESH_LOWPAN"
        }
    },
    "target_overrides": {
        "*": {
            "target.features_add": ["NANOSTACK", "COMMON_PAL"],
            "nanostack.configuration": "lowpan_router",
            "mbed-mesh-api.6lowpan-nd-device-type": "NET_6LOWPAN_ROUTER",
            "mbed-mesh-api.thread-device-type": "MESH_DEVICE_TYPE_THREAD_ROUTER",
            "mbed-mesh-api.heap-size": 32000,
            "mbed-trace.enable": false
        }
    }
```

The following tables show the values to use in the `mbed_app.json` file for your devices in different networks.

- For a 6LoWPAN-ND based network, use `mesh-type: MESH_LOWPAN`.
- For a Thread-based network, use `mesh-type: MESH_THREAD`.

#### 6LoWPAN-ND

**mesh-type: MESH_LOWPAN**

|Device role|`nanostack.configuration` value|`mbed-mesh-api.6lowpan-nd-device-type`|
|-----------|-------------------------|------------------------------------|
|Mesh router (default) | lowpan_router | NET_6LOWPAN_ROUTER |
|Nonrouting device | lowpan_host | NET_6LOWPAN_HOST |

#### Thread

**mesh-type: MESH_THREAD**

|Device role|`nanostack.configuration` value|`mbed-mesh-api.thread-device-type`|
|-----------|-------------------------|------------------------------------|
|Mesh router (default) | thread_router | MESH_DEVICE_TYPE_THREAD_ROUTER |
|Nonrouting device | thread_end_device | MESH_DEVICE_TYPE_THREAD_SLEEPY_END_DEVICE |

##### Thread commissioning

By default, the Thread application uses the static network link configuration defined in the [mesh API configuration file](https://github.com/ARMmbed/mbed-os/blob/master/features/nanostack/mbed-mesh-api/mbed_lib.json).
If you want to commission a Thread device, see [how to commission a Thread device in practice](https://os.mbed.com/docs/latest/reference/mesh-tech.html#thread-commissioning).

The Thread stack learns the network settings from the commissioning process and saves them to RAM memory. Therefore, the learned network settings are lost when you restart the device next time. To prevent re-commissioning, you can save the Thread configuration settings to an SD card as follows (only in `K64F`):

- Change `storage-device` to `MESH_SD_CARD` in the `./configs/mesh_thread.json` file.
- Enable commissioning as descibed in the referred instructions.
- Compile and program the application.
- Insert an erased or FAT-formatted SD card to the `K64F` memory card slot. The application will initialize the SD card with the appropriate file system on first use.
- Commission the device to the Thread network.
- When you restart the device next time, the device reads the Thread configuration settings from the SD card and tries to attach to an existing network.

### Requirements for hardware

The networking stack used in this example requires TLS functionality to be enabled on Mbed TLS.
On devices where hardware entropy is not present, TLS is disabled by default. This results in compile time failures or linking failures.

To learn why entropy is required, read the [TLS Porting guide](https://os.mbed.com/docs/latest/reference/mbed-tls-entropy.html).

See [Notes on different hardware](https://github.com/ARMmbed/mbed-os-example-mesh-minimal/blob/master/Hardware.md) for known combinations of development boards and RF shields that have been tested.

You also need to check how LEDs and buttons are configured for your hardware, and update .json accordingly.

### Changing the radio driver

To run a 6LoWPAN-ND network, you need a working RF driver for Nanostack. This example uses the Atmel AT86RF233 by default.

To change the RF driver modify the `mbed_app.json` file. For example,

```json
    "radio-type":{
        "help": "options are ATMEL, MCR20, NCS36510, KW24D",
        "value": "ATMEL"
    },
```

### Compile the application

```
mbed compile -m K64F -t GCC_ARM
```

A binary is generated in the end of the build process.

### Connect the RF shield to the board

We are using the Atmel AT86RF233, which you can [purchase](https://firefly-iot.com/product/firefly-arduino-shield-2-4ghz/). Place the shield on top of your board, and power it up.

### Program the target

Drag and drop the binary to the target to program the application.

### Update the firmware of the border router

This example supports the following border router:

- [Nanostack-border-router](https://github.com/ARMmbed/nanostack-border-router).

The border router supports static and dynamic backhaul configuration. The static configuration is good for testing, but the dynamic one works if your network infrastructure is supplying an IPv6 address. Make sure that you use the appropiate mode.

Remember to connect the Ethernet cable between the border router and your home/office router. Then power on the board.

## Testing

By default the application is built for the LED control demo, in which the device sends a multicast message to all devices in the network when the button is pressed. All devices that receive the multicast message will change the LED status (red LED on/off) to the state defined in the message. Note, that the Thread devices can form a network without the existance of the border router. The following applies only to the case when the border router is set-up.

As soon as both the border router and the target are up and running you can verify the correct behaviour. Open a serial console and see the IP address obtained by the device.

<span class="notes">**Note:** This application uses the baud rate of 115200.</span>

```
connected. IP = 2001:db8:a0b:12f0::1
```

You can use this IP address to `ping` from your PC and verify that the connection is working correctly.


## Memory optimizations

On some limited platforms, for example NCS36510 or KW24D, building this application might run out of RAM or ROM.
In those cases, you might try following these instructions to optimize the memory usage.

### Mbed TLS configuration

The custom Mbed TLS configuration can be set by adding `"macros": ["MBEDTLS_USER_CONFIG_FILE=\"mbedtls_config.h\""]` to the `.json` file. The [example Mbed TLS config](https://github.com/ARMmbed/mbed-os-example-mesh-minimal/blob/master/mbedtls_config.h) minimizes the RAM and ROM usage of the application. The configuration works on K64F, but it is not guaranteed to work on every Mbed Enabled platform.

This configuration file saves you 8.7 kB of RAM but uses 6.8 kB of more flash.

### Disabling the LED control example

You can disable the LED control example by specifying `enable-led-control-example": false` in the `mbed_app.json`

This saves you about 2.5 kB of flash.

### Change network stack's event loop stack size

Nanostack's internal event-loop is shared with Mbed Client and therefore requires lots of stack to complete the security hanshakes using TLS protocols.
In case client functionality is not used, you can define the following to use 2kB of stack

`"nanostack-hal.event_loop_thread_stack_size": 2048`

This saves you 4kB of RAM.

### Change Nanostack's heap size

Nanostack uses internal heap, which you can configure in the .json. A thread end device with comissioning enabled requires at least 15kB to run.

In `mbed_app.json`, you find the following line:

```
"mbed-mesh-api.heap-size": 15000,
```

For 6LoWPAN, you can try 12kB. For the smallest memory usage, configure the node to be in nonrouting mode. See [module-configuration](https://github.com/ARMmbed/mbed-os/tree/master/features/nanostack/FEATURE_NANOSTACK/mbed-mesh-api#module-configuration) for more detail.

### Move Nanostack's heap inside the system heap

Nanostack's internal heap can be moved within the system heap.
This helps on devices that have split RAM and compiler fails to fit statically allocated symbols into one section, for example, the NXP KW24D device.

Mesh API has the [use-malloc-for-heap](https://github.com/ARMmbed/mbed-os/blob/master/features/nanostack/FEATURE_NANOSTACK/mbed-mesh-api/README.md#configurable-parameters-in-section-mbed-mesh-api) option to help this.

Add the following line to `mbed_app.json` to test:

```
"mbed-mesh-api.use-malloc-for-heap": true,
```

### Use release profile

For devices with small memory, we recommend using release profiles for building and exporting. Please see the documentation about [Build profiles](https://os.mbed.com/docs/latest/tools/build-profiles.html).

Examples:

```
$ mbed export -m KW24D -i make_iar --profile release
OR
$ mbed compile -m KW24D -t IAR --profile release
```

## Troubleshooting

If you have problems, you can review the [documentation](https://os.mbed.com/docs/latest/tutorials/debugging.html) for suggestions on what could be wrong and how to fix it.

## Known issues

1. https://github.com/ARMmbed/mbed-os-example-mesh-minimal/issues/188

   The mesh interface function `mesh.get_mac_address()` has inadvertedly been changed in Mbed OS 5.9 to return the actual MAC address and not the EUI64 address. In order to print the actual EUI64 address for Thread commissioning, use the following code snippet after `printf("connected. IP = %s\n", mesh.get_ip_address());`:
   ```
   uint8_t eui64[8] = {0,0,0,0,0,0,0,0};
   mesh.device_eui64_get(eui64);
   printf("EUI64 address = %02x:%02x:%02x:%02x:%02x:%02x:%02x:%02x\n", eui64[0], eui64[1], eui64[2], eui64[3], eui64[4], eui64[5], eui64[6], eui64[7]);
   ```
   The new API for `device_eui64_get` was introduced in PR [7158](https://github.com/ARMmbed/mbed-os/pull/7158) and it will be available in Mbed OS 5.9.2.


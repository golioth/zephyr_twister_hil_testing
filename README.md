# Golioth Hardware-in-the-Loop (HIL) Workflow Demos

This repository is a general demonstration of using GitHub workflows to
automatically test firmware on target devices. Three workflow samples are
implemented:

* Compile Zephyr's `basic/blinky` on the cloud, then flash the binary and run
  the sample on the self-hosted runner.
* Run Zephyr's `hello_world` Twister testing on a self-hosted runner
* Use Twister to run the sample application in this repo on a self-hosted
  runner.
   * The included sample is `hello` from Golioth's Zephyr SDK
   * Twister tests the device's ability to securely connect to the Golioth
     servers.

These demos run on an nRF52840dk with an ESP32 as an AT Modem.

Looking for a deeper explanation? This repository was covered in-depth during
the [Hardware-in-the-Loop testing with Zephyr
RTOS](https://www.youtube.com/watch?v=940O1CUgh4Q) tech talk.

## Background on Golioth HIL testing

At Golioth, we believe Hardware-in-the-Loop (HIL) is an important part of
testing. In short, this means running tests on the actual hardware (not just
building and/or simulating). This goes beyond ensuring the firmware will run,
to include tests that confirm the device connects to a network, authenticates
as expected, communicates correct with a number of different Golioth services,
and is capable of completing an Over-the-Air firmware update.

We use HIL in our continuous integration (CI) testing via GitHub workflows. To
do so, we maintain our own self-hosted runners. These are single-board
computers with our target hardware (like nRF9160dk and ESP32-S3-DevKitC)
available to run tests.

## Workflows

All workflows are designed to run on an nRF52840dk with an ESP32 as an AT Modem.

To test these workflows:

1. Fork this repository
2. Add your own self-hosted runner to your fork
3. Use the Actions tab of GitHub to select and manually trigger the workflow

### blinky.yml

This workflow demonstrates building on the cloud, then
downloading/flashing/running the binary using a self-hosted runner.

While this example doesn't perform any automatic test verification beyond
successfully flashing the device, you can find a testing example in [a Golioth
workflow](https://github.com/golioth/golioth-zephyr-sdk/blob/main/.github/workflows/test_nrf52840dk.yml)
that uses [a custom python
script](https://github.com/golioth/golioth-zephyr-sdk/blob/main/samples/test/verify.py)
to verify the test.

#### Requirements:

* Repository secret `CI_NRF52840DK_SNR` that contains the serial number of your
  device

You can find the serial number using Nordic command line tools:

```console
$ nrfjprog -i
1050266122
```

### twister_hello_world.yml

This workflow demonstrates running the Twister test on Zephyr's `hello_world`
sample using a self-hosted runner.

#### Requirements:

* Self-hosted runner file to define the hardware map

In the self-hosted runner home directory, create a
`hardware-map-webinar-demo.yml` file that contains the following:

```console
- connected: true
  id: "001050266122"
  platform: nrf52840dk_nrf52840
  product: J-Link
  runner: nrfjprog
  serial: /dev/serial/by-id/usb-SEGGER_J-Link_001050266122-if00
```

Here are some commands you can use to acquire the necessary information for the
hardware map:

```console
$ nrfjprog -i
1050266122

$ ls /dev/serial/by-id
usb-SEGGER_J-Link_001050266122-if00  usb-SEGGER_J-Link_001050266122-if02
```

### golioth_hello.yml

This workflow demonstrates how Golioth uses Zephyr and Twister to test device
connections using a self-hosted runner.

This workflow requires a setup file on the self-hosted runner that contains
WiFi and Golioth credentials for the device. It is possible to abstract all of
these values to the cloud (e.g. utilizing GitHub secrets) but we've taken this
approach to keep the workflow simple.

#### Requirements:

* Self-hosted runner file to define the hardware map
* Self-hosted runner file to set environment variables

In the self-hosted runner home directory, create a
`hardware-map-webinar-demo.yml` file that contains the following:

```console
- connected: true
  id: "001050266122"
  platform: nrf52840dk_nrf52840
  product: J-Link
  runner: nrfjprog
  serial: /dev/serial/by-id/usb-SEGGER_J-Link_001050266122-if00
```

Here are some commands you can use to acquire the necessary information for the
hardware map:

```console
$ nrfjprog -i
1050266122

$ ls /dev/serial/by-id
usb-SEGGER_J-Link_001050266122-if00  usb-SEGGER_J-Link_001050266122-if02
```

In the self-hosted runner home directory, create a `env_runner_webinar_demo.sh`
file that contains the following:

```console
export GOLIOTH_SAMPLE_HARDCODED_PSK_ID=your_golioth@psk-id
export GOLIOTH_SAMPLE_HARDCODED_PSK=your_golioth_psk
export GOLIOTH_SAMPLE_WIFI_SSID="Your WiFi SSID"
export GOLIOTH_SAMPLE_WIFI_PSK="Your WiFi Password"
```

Replace the values with a PSK-ID/PSK for your device and WiFi credentials for your local access point.

## Golioth HIL Testing Examples

For more complex examples, please view the workflows used in the Golioth SDKs:

* [Golioth Firmware SDK](https://github.com/golioth/golioth-firmware-sdk)
* [Golioth Zephyr SDK](https://github.com/golioth/golioth-zephyr-sdk)

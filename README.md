# Tracking Satellites and Processing Real-Time Telemetry Transmissions
An introduction to satellite communications.

## Scope

This project is intended to outline how to track satellites, receive their live telemetry, and process that data into human-interpretable weather images.


## Disclaimer
This project utilizes tools which are capable of receiving data on satellite-specific radio frequencies. At no point should any attempt be made to transmit on any radio frequency without proper licensing. To reiterate: this project is limited to RECEIVE-only.


## Resources
### Hardware
* RTL-SDR Blog V3 Dongle + Dipole Antenna
* A device running Linux (Mint)

### Software
* GQRX
* SatDump

### Acknowledgements

This project was inspired and aided by the following resources:

* [Saveitforparts - How To Get Live Satellite Images Directly From Space](https://www.youtube.com/watch?v=icADyjm3PBE)
* [Ken's RTL-SDR for Linux Quick-Start Guide](https://ranous.wordpress.com/rtl-sdr4linux/)
* [N2YO](https://www.n2yo.com)

## Initial Setup

This project was created in Linux Mint, so the instructions will assume you're using a similar Ubuntu-based distro. We'll need a few tools before we can start.

The `rtl-sdr` package can be installed via `apt-get` or through the Synaptic Package Manager GUI:

![RTLSDR Driver Download](https://github.com/user-attachments/assets/0efd7bbe-cb07-4643-aa23-8014ffd3a3a5)

Once the driver and its dependencies are installed, install the following programs:

* [GQRX](https://github.com/gqrx-sdr/gqrx/releases)
* [SatDump](https://www.satdump.org/download/)


## Satellite Tracking with N2YO

An invaluable resource for tracking satellite trajectories is N2YO. This website provides live tracking data for a host of satellites, including the International Space Station. However, we'll be tracking NOAA weather satellites for this project. 

Under _Most Tracked_, you'll find NOAA 19, NOAA 18, and NOAA 15. Let's take a look at NOAA 19:

![NOAA19MainPage](https://github.com/user-attachments/assets/88cc1116-cfec-492c-87bc-895de256d0ec)

This page contains information on the satellite including a live tracking map, launch information, and the downlink frequency. We'll need that later.

Navigating to _10-day Predictions_ yields a chronological list of expected passes based on our current location. This page provides a quick overview of upcoming passes which will allow us to identify the ideal pass to capture. The most important datapoint in this chart is the _El_ field under the _Max Altitude_ section. This provides us with the angle of elevation which is crucial for capturing clean data:

![NOAA19TenDay](https://github.com/user-attachments/assets/b33d98a7-cb17-41b8-acd9-32291ed30664)

The closer the _El_ value is to 90, the more directly overhead the satellite will be at its maximum elevation during the pass. This lowers the chance of obstruction from objects like trees and buildings. Scrolling down the page, we've found and clicked into an ideal pass at 86 degrees:

![NOAA19PassData](https://github.com/user-attachments/assets/91057004-433c-44b2-81f8-0a6ff6df4489)

This page provides an overview of the expected pass with all necessary information for telemetry capture. Now, let's set up our antenna for the pass.

## Capturing Telemetry with GQRX

GQRX is an open-source SDR tool used for capturing radio signals. This tool will allow us to record the encoded data transmitted by the satellite as it passes overhead. This data comes in the form of a rythmic, pulsing audio stream. Put simply, capturing a pass can be accomplished in the following three steps:

1. Set up the antenna and connect it to the device running GQRX.
2. Tune to the satellite's downlink frequency.
3. Record the pass from the given start time until the signal fades away.

Ideally, passes should be captured outside in wide-open areas as high as possible. Signal absorption, reflection, and interference can be mitigated by avoiding trees, buildings, and strong RF devices near your capture area.

Below is a video capture of the GQRX waterfall when tuned to NOAA 19's downlink frequency of approximately 137.100 MHz (be sure to enable audio!):

https://github.com/user-attachments/assets/d6edcdc3-7e05-4eec-9e44-b56e50f7b4f2

As you can hear in the above video snippet, the telemetry is coming through loud and clear as a rhythmic pulsing and clicking noise. This is the encoded data that we need to record and eventually process with SatDump.

Here's a quick breakdown of the UI elements for the above video:

### Downlink Frequency

According to N2YO, the downlink frequency for NOAA 19 is 137.100 MHz. This can be configured at the top-left portion of the GQRX interface by typing in the desired frequency value.

### Telemetry Data

On the right side of the waterfall, you'll notice a distinct, consistently-formed yellow pattern. This is the frequency range we are monitoring with our filter (the grey rectangle with a red center line above the telemetry data). The video snippet above is from when the satellite was directly overhead and providing us with the clearest possible signal. You can expect the signal strength and quality to degrade as the satellite nears the end of its pass.

### Squelch

Adjusting the squelch value on the right side helps filter out white noise when attempting to zero in on our desired radio signal. The default value for GQRX is around -150.0 dB, but we've raised it to -67.0 dB for a more ideal signal-to-noise ratio. 

## Decoding the Telemetry Data into Weather Images

Now that we've recorded the audio of our satellite's telemetry broadcast, we can convert that into a human-interpretable image using tools like SatDump. 

![SatDumpConfig](https://github.com/user-attachments/assets/d4ab182a-b276-40e4-8d77-f3a5cd76a758)

While you can record directly via GQRX in baseband format, I recorded the audio of the pass via OBS in .wav format. I trimmed and resampled the audio to better accommodate SatDump's decoder, and utilized the settings seen above.

Upon clicking _Start_, the decoding process will begin:

![SatDumpDecoding](https://github.com/user-attachments/assets/6100f0b3-076a-4039-8347-6b2e79b23beb)

The images will then be piped to the output folder specified in the configuration menu. Here, you can view the decoded images in a variety of formats:

![APT-A](https://github.com/user-attachments/assets/7162b8dc-f07d-4780-8f9f-7734e3b7670e)

![avhrr_3_rgb_10 8um_Thermal_IR](https://github.com/user-attachments/assets/77d0317d-12d4-4463-ac98-c36faa34c9d1)

![avhrr_3_rgb_MCIR_Rain_(Uncalibrated)](https://github.com/user-attachments/assets/5196362c-bf2a-4864-9bd4-91993defa648)

I initially struggled with implementing a map overlay and that was likely due to not capturing the proper audio channel (the incoming telemetry signal is composed of a handful of separate tiny audio channels emitted concurrently). The channel I likely missed was responsible for timestamping as that is crucial for properly implementing a terrain/border overlay.

## Final Thoughts

While I wouldn't call this a resounding success due to the lack of natural color output and map overlays, I'm satisfied with the results. This was my fifth solid attempt at capturing a quality audio signal from a NOAA 19 pass using a simple dipole antenna. For future projects, I think I'll stick to SatDump for both recording and processing, as SatDump's recording feature is supposed to be more well integrated and intuitive. 

I also plan on building a quadrifilar helix antenna, which is an omnidirectional antenna that doesn't have to be oriented towards the satellite as it makes its pass overhead. This reduces potential for user error (my arm is not a very stable tracking mount!), and should result in clearer captures. But for now, I think this project is a good foundation for moving onto more complex approaches to satellite communications. I hope you enjoyed it!

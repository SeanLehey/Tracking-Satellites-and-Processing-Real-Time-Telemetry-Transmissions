# Tracking Satellites and Processing Real-Time Telemetry Transmissions
An introduction to satellite communications.

## Scope

This project is intended to outline how to track satellites, receive their live telemetry, and process that data into human-interpretable weather images.


## Disclaimer
This project utilizes tools which are capable of receiving data on satellite-specific radio frequencies. At no point should any attempt be made to transmit on any radio frequency without proper licensing. To reiterate: this project is limited to RECEIVE-ONLY.


## Resources
### Hardware
* RTL-SDR Blog V3 Dongle + Dipole Antenna
* A device running Linux (Mint)

### Software
* GQRX
* SatDump

### Acknowledgements
* [Saveitforparts - How To Get Live Satellite Images Directly From Space](https://www.youtube.com/watch?v=icADyjm3PBE)
* [N2YO](https://www.n2yo.com)

## Satellite Tracking with N2YO

An invaluable resource for tracking satellite trajectories is N2YO. This website provides live tracking data for a host of satellites, including the International Space Station. However, we'll be tracking NOAA weather satellites for this project. 

Under _Most Tracked_, you'll find NOAA 19, NOAA 18, and NOAA 15. Let's take a look at NOAA 19:

![NOAA19MainPage](https://github.com/user-attachments/assets/88cc1116-cfec-492c-87bc-895de256d0ec)

This page contains information on the satellite including a live tracking map, launch information, and the downlink frequency. We'll need that later.

Navigating to _10-day Predictions_ yields a chronological list of expected passes based on our current location. This page provides a quick overview of upcoming passes which will allow us to identify the ideal pass to capture. The most important datapoint in this chart is the _El_ field under the _Max Altitude_ section. This provides us with the angle of elevation which is crucial for capturing clean data:

![NOAA19TenDay](https://github.com/user-attachments/assets/b33d98a7-cb17-41b8-acd9-32291ed30664)

The closer the _El_ value is to 90, the more directly overhead the satellite will be at its max elevation during the pass. This lowers the chance of obstruction from objects like trees and buildings. Scrolling down the page, we've found and clicked into an ideal pass at 86 degrees:

![NOAA19PassData](https://github.com/user-attachments/assets/91057004-433c-44b2-81f8-0a6ff6df4489)

This page provides an overview of the expected pass with all necessary information for telemetry capture. Now, let's set up our antenna for the pass.

## Capturing Telemetry with GQRX

GQRX is an open-source SDR tool used for capturing radio signals. This tool will allow us to record the encoded data transmitted by the satellite as it passes overhead. This data comes in the form of a rythmic, pulsing audio stream. Put simply, capturing a pass can be accomplished in the following three steps:

1. Set up the antenna and connect it to the device running GQRX.
2. Tune to the satellite's downlink frequency.
3. Record the pass from the given start time until the signal fades away.

Ideally, passes should be captured outside in wide-open areas as high as possible. Signal absorption, reflection, and interference can be mitigated by avoiding trees, buildings, and strong RF devices near your capture area.


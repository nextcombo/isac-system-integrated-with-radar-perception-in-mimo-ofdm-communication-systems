# ISAC System Integrated with Radar Perception in MIMO-OFDM Communication Systems [with MATLAB Code]

2. Introduction (Introduction) The ISAC system is a key strategy to address the challenges of increasingly crowded spectrum and the growing demand for large bandwidth from radar and communication systems. Within the ISAC framework, radar and communication functions are integrated, sharing the same hardware infrastructure and adopting common emission waveforms. This example will demonstrate how to enhance radar perception capabilities in a typical MIMO-OFDM communication system.
OFDM is a multicarrier signal composed of a set of orthogonal complex exponential signals called subcarriers. Each subcarrier's complex amplitude can carry communication data, achieving high data rate transmission. Moreover, OFDM waveforms have Doppler immunity and do not suffer from distance-Doppler coupling issues. These characteristics make OFDM waveforms attractive for radar applications. Since radar functions are embedded within the communication system, this ISAC approach is defined as "centered around communication."
This example employs a dual-base station topology structure with the aim of achieving dual objectives: the transmitter's primary task is to efficiently transmit data loads through multiple spatial channels to the receiver; simultaneously, the receiver must not only receive this data load but also measure the position of specific target of interest to perceive the surrounding environment.
3. System Parameters (System Parameters) Consider a MIMO-OFDM communication system operating at 6GHz carrier frequency with a bandwidth of 100MHz.
•
The peak transmitter power is set at 1 watt.
•
The receiver noise figure is set at 3 dB, with a reference noise temperature of 290 Kelvin.
•
Both the transmitter and receiver use uniform linear arrays (ULAs), each containing 8 isotropic antennas, with an antenna spacing of half a wavelength.
4. ISAC Scenario (ISAC Scenario)
The transmitter is located at the origin of coordinates, and the normal direction of the transmit array is along the x-axis. The receiver and the transmitter maintain a certain distance, with the array of the receiver oriented along the negative y-axis and both in a stationary state.
•  
The environment between the transmitter and receiver contains scatterers, which can be stationary or moving. The rich scattering environment is key to MIMO communication for spatial multiplexing, while in the perception function, scatterers within the communication channel are considered as radar targets, regardless of their motion state.
•  
Assuming the maximum path length between the transmitter and receiver is 300 meters, and the velocity of the moving scattering body does not exceed 50 meters per second (m/s).
•  
The scene contains 3 moving scatterers (i.e., the focus of perception functions), and all other scatterers are stationary. Through the helper function `helperGenerateStaticScatterers`, positions of stationary scatterers can be generated within a specific focus area.
•  
Model the propagation channel between transmitter and receiver using the Phased.ScatteringMIMOChannel object, and visualize the scattering MIMO channel through the helper function, VisualizeScatteringMIMOChannel.
5. Signaling Scheme (Signaling Scheme) This example also demonstrates how to design the following signalling scheme for joint communication and perception, with the specific process as follows:
1  
Initial Channel Probing: The transmission begins with a preamble sequence, and the receiver obtains an initial channel estimate through the preamble sequence.
2  
Channel coding: The receiver uses initial channel estimation to calculate the required precoding and combining weights, and assumes that there is no delay in feedback and it is completely reliable, returning the precoding weights back to the transmitter.
3  
Data Transmission: The transmitter sends a single frame, which is divided into two subframes: Subframe A only transmits data symbols; Subframe B simultaneously transmits both data symbols and pilots, achieving dual transmission of communication and perception.
4  
Channel Estimation and Precoding Weight Update: After receiving subframe B, the receiver uses pilots to obtain new channel estimations, recalibrates the precoding and combining weights, and then feeds the updated precoding weights back to the transmitter.
5  
Radar Data Cube Construction: Summing the current channel estimates along the transmit antenna dimension and storing it as a separate page in the radar data cube.
The above process repeats from step 3 until the preset number (numFrames) of frames have been transmitted. Subsequently, the estimated target position and relative velocity can be derived from the collected radar data cube.
In addition, the signal scheme also includes the following key design:
•  
The number of OFDM subcarriers is set to 2048, and the subcarrier interval is calculated based on the bandwidth and the number of subcarriers. To avoid Doppler shift caused by the motion of scattering bodies affecting the orthogonality of the subcarriers, the subcarrier interval must be at least 10 times the maximum Doppler frequency shift (in this example, the interval is 48.79 times the maximum Doppler frequency shift, which meets the requirement).
•  
To avoid Intersymbol interference (ISI), a cyclic prefix (CP) is added before the OFDM symbol, based on the maximum attention distance.
•  
Some subcarriers are used as guard bands (the first 9 and the last 8 subcarriers), to prevent spectrum leakage from adjacent frequency systems.
•  
The pilots in subframe B are used for both channel equalization and radar perception: the density of pilots in time-frequency domain must balance the amount of sensed data with the data payload capacity. The maximum interval is determined by the maximum expected delay and the maximum expected Doppler shift (in this example, the maximum time-domain pilot interval Mt=22, and the maximum frequency-domain pilot interval Mf=10).
•  
The frame length is set to the time-domain pilot interval (Mt), and the length of subframe B equals to the number of transmitting antennas (each antenna transmits a pilot in different OFDM symbols to avoid interference). The remaining symbols belong to subframe A.
6. Initial Channel Sounding (Initial Channel Sounding) pilot sequence is used to obtain initial channel estimates and calculate precoding and combining weights required for spatial multiplexing. In this example, the pilot sequence consists of a single OFDM symbol, with different transmit antennas using different subcarriers to transmit the pilot (to avoid interference).
•  
The pilot symbols are generated using the maximum length sequence (MLS), with all transmit antennas using the same pilot value.
•  
The Modulator and demodulator objects of comm.OFDMModulator and comm.OFDMDemodulator are used to implement the modulation and demodulation of pilot signals, respectively.
•  
The transmitter sends the modulated pilot signal, which is then propagated through the channel by the receiver. The receiver adds thermal noise and demodulates the signal. Subsequently, the helper InterpolateChannelMatrix function calculates the channel estimation, and the diagbfweights function computes the precoding and combining weights.
7. Data Frame Transmission (Data Frame Transmission) 7.1 Modem Configuration
Subframe A only transmits data, and modulation and demodulation are implemented through the comm.OFDMModulator and comm.OFDMDemodulator objects.
•  
The pilots in subframe B are transmitted over Mf subcarriers in the frequency domain, and each transmit antenna transmits the pilots (also generated based on MLS sequences) in different OFDM symbols. They are processed through dedicated modulation and demodulation entities.
7.2 Key Parameters and Performance •
Assuming that the radar data of 24 OFDM frames can be coherently processed, the Coherent Processing Interval (CPI) determines the Doppler resolution, and the final velocity resolution is 4.21 m/s.
•  
System adopts 64-Quadrature Amplitude Modulation (64-QAM), each symbol carries 6 bits of data, with a modulation order of 64.
•
Utilize multi-antenna and rich scattering environment for spatial multiplexing, where the transmitter transmits two independent data streams through two spatial channels to the receiver via 2 spatial channels.
7.3 Transmission and reception process performs the following operations on each frame of data:
1
Generate binary data loads for subframes A and B, modulated by 64-QAM symbols.
2
Precode the modulated symbols to generate OFDM symbols.
3
Merge subframes A and B to form an entire OFDM frame and transmit it in real time; update the target position during transmission.
4
The receiver receives the signal, separates subframes A and B, and demodulates them separately; apply combining weights to the demodulated symbols.
5
Demodulate QAM data and calculate the bit error rate (BER) per frame, with a range of 0.0055~0.0132 for 24 frames.
6
Use the pilots from subframe B to update channel estimation and precoding / combining weights, and store the current channel estimate along the transmit antenna dimension, summed up, into the radar data cube.
Finally, draw a 64-QAM constellation diagram for the last frame's received data using the comm.ConstellationDiagram object.
8. Radar Data Processing (Radar Data Processing) estimates the positions of scatterers in the channel under known transmitter and receiver positions. Since the focus is on moving scatterers, static scatterers (with a Doppler shift of 0) can be filtered out in the Doppler domain. The specific steps are as follows:
1
Perform a Fast Fourier Transform (FFT) on the radar data cube along the slow time dimension, zero out the DC component to remove the impact of static scatterers, and then obtain the processed data through Inverse FFT (IFFT).
Utilize the helperPositionHeatmap auxiliary class to draw heat maps, showcasing the positions of moving scatterers within the focus area.
3  
4  
Non-coherent integration of all received antenna signals, plotting distance - Doppler response graphs and marking the double-baseline Doppler shifts and total ranges of interest.
9. Conclusion (Conclusion) This example demonstrates the integration of radar perception capabilities into a generic MIMO-OFDM communication system, transforming it into an ISAC system. The specific process includes: constructing a MIMO-OFDM communication system with a single transmit array and receive array, configuring a scattering channel with stationary and moving scatterers, designing a signal scheme that balances data transmission and target perception (utilizing pilots to achieve dual functions), transmitting multiple OFDM frames and building a radar data cube, and ultimately estimating the positions of moving scatterers and double-baseline Doppler shifts from the data cube. Since radar processing is performed at the receiver end, the radar configuration in this example is set for a double-baseline mode.
10. References (References) 1
Liu, F., Masouros, C., Petropulu, A. P., Griffiths, H., & Hanzo, L. (2020). Joint radar and communication design: Applications, state-of-the-art, and the road ahead. IEEE Transactions on Communications, 68(6), 3834-3862.  
(Co-Design of Radar and Communication: Applications, Research Status, and Future Directions, IEEE Communications Magazine)
2  
Liu, A., Huang, Z., Li, M., Wan, Y., Li, W., Han, T. X., Liu, C., et al. (2022). A survey on fundamental limits of integrated sensing and communication. IEEE Communications Surveys & Tutorials, 24(2), 994-1034.  
(Basic Limits of Integrated Perception and Communication, "IEEE Communications Review and Tutorial")
3  
de Oliveira, L. G., Nuss, B., Alabd, M. B., Diewald, A., Pauli, M., & Zwick, T. (2021). Joint radar-communication systems: Modulation schemes and system design. IEEE Transactions on Microwave Theory and Techniques, 70(3), 1521-1551.  
("Radar - Communication Joint Systems: Modulation Schemes and System Design", "IEEE Microwave Theory and Technology Magazine")
4  
Sit, Y. L., Nuss, B., & Zwick, T. (2017). On mutual interference cancellation in a MIMO OFDM multiuser radar-communication network. IEEE Transactions on Vehicular Technology, 67(4), 3339-3348.  
(IEEE Vehicle Technology Journal) - MIMO-OFDM Multi-User Radar - Interference Mitigation in Communication Networks
End

## Images

![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](01.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](02.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](03.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](04.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](05.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](06.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](07.jpg)
![(ISAC)MIMO-OFDM communication system integrated with radar perception ISAC system [with MATLAB code]](08.jpg)

<!-- PAYMENT_BLOCK -->

## Get this dataset

- Payment link (Stripe): [Buy now](https://buy.stripe.com/3cs8yP7sY87d0vu9AB)
- Payment QR code: ![Payment QR code](pay.png)

<!-- /PAYMENT_BLOCK -->

## What happened in WFSIM?

(This section is mainly copied and paste from `Fax_Tutorial` of [`WFSim`](https://github.com/XENONnT/WFSim))

What happens behind the scenes is that the instructions are first grouped together in chunks. Then we loop over the instructions and the full chunk is returned before starting with the next one.

We use a S1 and S2 class to calculate the arrival times of the photons and the channels which have been hit. Then we'll hand them over to the Pulse class to calculate the currents in the channels. Finally the currents go to a RawData class where we fake the digitizer response.

### S1

For S1s we start with calculating the light yield based on the position of the interaction, and draw the number of photons seen from a poisson distribution.

Second we calculate the arrival times of the photons. This is based on the scintiallation of the xenon atoms. It is dependend on the recombination time, the singlet and triplet fractions.

Finally the channels are calcuated. Based on the pattern map we use a interpolation map to get a probability distribution for channels to be hit for a S1 signal based on the position of the interaction, and then we draw from this distribution for every photon.

### S2

S2s are slightly more complicated. First we need to drift the electrons up, and while doing so we'll lose some of them. To get the photon timings, we first need to get the arrival times of the electrons at the gas interface based on a diffusion model. Then we can calculate the photon timings based on a luminesience model for every individual electron. And for the channels we do the same trick with the interpolating map.

### Pulse

When we have our lists of channels and timing we can generate actual pulses. First we add a pmt transition time. Then we loop over all channels, calculate the double pe emission probabilities, and add a current in the pmt channel based on the arrival time. This is all stored in a big dictionary. Afterwards this is passed to our fake digitizers which then returns you with your very own pretty data


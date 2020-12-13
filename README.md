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

---

# What did I do to generate the data?

- Problem: seems that the instruction is never working?

  - How is it called?

  - Let's change the chunk size stuff, and see if we got different size fo simulated data.

    ```python
    st.set_config(dict(nchunk=2, event_rate=5, chunk_size=10, z = -5, n_electron = 5)) 
    # Generate events at -5 cm,
    # Only 5 electrons generated
    # nchunk changed from 1
    ```

    ```python
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option n_mveto_pmts not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option n_nveto_pmts not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option gain_model_nv not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option gain_model_mv not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option channel_map not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option z not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:589: UserWarning: Option n_electron not taken by any registered plugin
      plugins = self._get_plugins(targets, run_id)
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option n_mveto_pmts not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option n_nveto_pmts not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option gain_model_nv not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option gain_model_mv not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option channel_map not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option z not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    /opt/XENONnT/anaconda/envs/XENONnT_development/lib/python3.6/site-packages/strax/context.py:1259: UserWarning: Option n_electron not taken by any registered plugin
      p = self._get_plugins((target,), run_id)[target]
    Simulating Raw Records:  10%|▉         | 19/193 [00:20<02:50,  1.02it/s]
    ```

    Yes it is called. But how about the others (`z`, `n_electron`)?

  - `strax_interface.py`:

    ```python
        def setup(self):
            c = self.config
            c.update(get_resource(c['fax_config'], fmt='json'))
    				(...)
            else:
                self.instructions = rand_instructions(c)
    ```

    


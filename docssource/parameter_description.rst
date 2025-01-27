Parameter Description
=====================

This section describes the different **parameters** that can be set for running the simulator.
All parameters for e.g. the transmitter, receiver, simulation etc. can be changed in the **_settings** directory.

Every simulation must have the following settings files.

+------------------+------------------------------------------+
| File name        |   Description                            |
+==================+==========================================+
| general          | general simulation parameters            |
+------------------+------------------------------------------+
| scenario         | Set transceivers, modulation, channel... |
+------------------+------------------------------------------+
| rf_chain         | RF chain impairments parameters          |
+------------------+------------------------------------------+
| coding/          | Parameters related to coding             |
+------------------+------------------------------------------+

Depending on the desired transmission technologies, technology-specific files must be defined.
Note that the names can be changed, such that different modems can have different parameters.

The current technologies are currently available:

+------------------+------------------------------------------+
| File name        |   Description                            |
+==================+==========================================+
| chirp_fsk        | modulation parameter of chirp-fsk        |
+------------------+------------------------------------------+
| ofdm             | OFDM specific parameters                 |
+------------------+------------------------------------------+
| psk_qam          | PSK/QAM specific modulation parameters   |
+------------------+------------------------------------------+

The other files are optional, depending on the desired simulation.

+------------------+------------------------------------------+
| File name        |   Description                            |
+==================+==========================================+
| quadriga         | required for simulation with Quadriga    |
+------------------+------------------------------------------+
| rf_chain         | RF chain impairments parameters          |
+------------------+------------------------------------------+
| coding/          | Parameters related to coding             |
+------------------+------------------------------------------+

We will now describe the settings files more in detail.
All parameters are in SI units, unless otherwise stated.

======================
settings_chirp_fsk.ini
======================

**[Modulation]**: The modulator generates a sequence of frequency-modulated chirps with no time overlap.

The parameters apply to each chirp:

- ``chirp_duration``:  defines time duration in seconds of chirp
- ``chirp_bandwidth``: defines the bandwidth of a chirp in Hz

Parameters for an M-FSK chirpmodulation with ``M`` being the modulation order.

- ``modulation_order``: must be of power of two
- ``freq_difference``: distance in Hz between neighbouring modulation symbols
- ``oversampling_factor``: in relation to chirp bandwidth

**[Frame]**: Specifications for transmission frame.

- ``number_pilot_chirps``: number of pilot chirps
- ``number_data_chirps``:  number of data chirps
- ``guard_interval``: guard interval between frames in s

====================
settings_general.ini
====================

**[RandomNumber]**: Actual random number generator seeds will be obtained from a list of good seeds in "RandomNumberGeneratorSeeds.mat".
If index is negative, then a random seed is used, based on the computer clock

- ``seed``: Seed to be used
- ``seed_file``: file to pick seed from

**[Drops]**: The simulator will perform a number of simulation drops of ``duration`` seconds.
Up to ``max_num_drops`` drops will be performed, but simulation may be stopped if a certain confidence interval is
reached. However, at least ``min_num_drops`` drops will be performed.

- ``duration``: each drop will last at least as long as one communication frame for all transmitting modems, in seconds.
- ``min_num_drops``: minimum number of simulation drops
- ``max_num_drops``: maximum number of drops to simulate
- ``confidence_level``: desired confidence interval in [0,1]
- ``confidence_margin``: 

    Desired relative confidence margin, i.e., if confidence interval is (lower, upper),
    then simulation is stopped when ``(upper-lower) < confidence_margin``.
    The confidence margin is calculated using the Bayesian approach from ``scipy.stats.bayes_mvs``.
    If ``confidence_margin == 0``, then ``max_num_drops`` drops are always performed.

- ``confidence_metric``: target metric (BER or BLER), i.e., confidence value will be calculated for this metric for stopping simulation.

**[NoiseLoop]**: The simulator can run for different noise levels, as specified here.

- ``snr_type``: signal-to-noise-ratio: allowed values: "Eb/N0(dB)", "Es/N0(dB)", "CUSTOM"
- ``snr_vector``: 

    Simulator will run for all of the SNR values specified in this vector (in numpy format).
    E.g., ``snr_vector = [0,2,4,6,8]``  or ``snr_vector = np.arange(0,10,2)``.
    obs: SNR is relative to transmitted power (receiver SNR may be larger if channel is not normalized).
    If ``snr_type`` is ``CUSTOM``, the values of the ``snr_vector`` are interpreted as actual noise variance.

**[Statistics]**: Parameters related to the generated Statistics.

- ``verbose``: if true, print results for each drop
- ``plot``: if true, generate plots at end of simulation
- ``calc_theory``: if true, calculate theoretical BER/BLER for given configuration (if possible)

- ``calc_spectrum_tx``: perform spectral analysis for transmitted signal using Welch's method
- ``calc_stft_tx``: STFT is calculated for first frame of transmitted signal.
- ``calc_spectrum_rx``: 
- ``calc_stft_rx``: 
- ``spectrum_fft_size``: FFT size for Welch's method

=================
settings_ofdm.ini
=================

**[Modulation]**: The main OFDM parameters are defined here, i.e., subcarrier spacing in Hz and FFT size.
Subcarriers are occupied in the centre of the spectrum, except the DC subcarrier, the remaining subcarriers are left
empty.

CP for each OFDM symbol is defined in terms of its length relative to the OFDM symbol, i.e., the number of
samples in the CP is ``cp_ratio * fft_size``. Several different CPs can be defined,
separated by comma, and their usage will be defined in the frame specifications.
Precoding can either be ``none`` or ``DFT`` for DFT-spread OFDM (SC-FDMA).

- ``subcarrier_spacing``:
- ``fft_size``:
- ``number_occupied_subcarriers``:
- ``cp_ratio``:
- ``precoding``: 

- ``modulation_order``: for the ofdm symbol resources. currently, M = 2, 4 (PSK), 16, 64, 256 (QAM) are supported

- ``oversampling_factor``: 

    the sampling rate will be ``fft_size * subcarrier_spacing * oversampling_rate``.
    upsampling is performed using a polyphase filter approach

- ``dc_suppresion``: Define if DC subcarrier is to be suppressed or not.

**[MIMO]**:
In case of multiple antennas, the specifications for the MIMO processing are given here

Currently, only open-loop MIMO is supported-
The following schemes are supported: ``SM``, ``SFBC``, ``MRC``, ``SC`` and ``NONE``

**Receive diversity schemes**:
``MRC`` (maximum ration combining) and ``SC`` (selection combining) can be applied for 1 tx antenna and any number of rx antennas.

**Transmit diversity schemes**:
``SFBC`` (space-frequency block codes), following 3GPP specifications (TS 36.211, Sec, 6.3.3.3) for 2 or 4 tx antennas.

**Spatial Multiplexing schemes**:
``SM`` / ``SM-ZF``: spatial multiplexing with linear zero-forcing receiver.
``SM-MMSE``: spatial multiplexing with linear MMSE receiver.

- ``mimo_scheme``: defines the scheme to be employed.

**[Receiver]**: 
channel estimation algorithm, the following values are supported:

  - ``ideal``: perfect channel knowledge at the beginning of each OFDM symbol
  - ``ideal_preamble``:
  - ``ideal_midamble``:
  -  ``ideal_postamble``: perfect channel knowledge at the beginning / middle / end of each frame

- ``channel_estimation``:
- ``equalization``: ZF for zero-forcing or MMSE


**[Frame]**:
Specifies the frame structure in time and frequency domain.
The ``ofdm_symbol_resources`` parameter defines the resource types of the ofdm symbols itself. Multiple ofdm symbols can be defined.
They are separated by commas. **Maximum number of ofdm symbols is 10**. These resources are afterwards mapped to the subcarriers.
``r`` denotes a reference symbol, ``d`` a data symbol, and ``n`` a null carrier. A prepended number indicates the number of repetitions, i.e.
``5rd`` means that 5 reference symbols will be created, followed by one data symbol. Pattern enclosed by brackets are called a *block*,
which will be repeated given the number in front of the opening bracket, i.e. ``100(r11d)`` means that the *block*
``(r11d)``, consisting of the sequence "1 reference symbol, 11 data symbols" will be repeated ``100`` times.

Cyclic prefixes are described in ``cp_length``. Multiple CPs are to be separated with a comma. Each CP is connected to a ofdm symbol, whose resources are defined in ``ofdm_symbol_resources``. 
**Therefore, the number of defined cp_length must match the number of defined ofdm_symbol_resources.** You can choose from multiple
cps. Simply define your desired ``cp_ratio``s. The number after ``c`` indicates the cyclic prefix ratio to pick. It is 1-index based.

The composition of a frame is defined in ``frame_structure``. ``g`` denotes a guard interval. ``ofdm_symbols`` are separated by commas.
Numbers indicate the resource 1-based index of ``ofdm_symbol_resources`` to pick from, i.e. ``1`` picks the first ofdm symbol resource mapping defined
in ``ofdm_symbol_resources``. ``12`` indicates to pick the first ofdm symbol resource and afterwards append the second ofdm symbol resource.
Do not forget that cyclic prefixes are appendedn in time domain according to ``cp_length``!

In accordance with ``ofdm_symbol_resources``, ofdm symbol resources enclosed by brackets are called blocks. A number in front of the
opening bracket indicates the number of repetitions. For instance, ``7(12)`` basically means the following frame structure:
``12 12 12 12 12 12 12``. Note: Spaces were inserted for better readability. Don't forget the cp prefixes that are automatically inserted.

.. note::
   Only one link-direction can be simulated at a time. Internally, there is no difference between UL and DL.
   Therefore, it is up to the user to define the frames accordingly.

- ``frame_guard_interval``: Inserted between two succeeding ofdm symbols. Will be filled up with zeros. Factor of ofdm_symbol length.

====================
settings_psk_qam.ini
====================
**[Modulation]**: Everything related to modulation.

- ``modulation_order``: 

    - PSK: 2, 4, 8
    - QAM: 16, 64, 256
    - PAM: 2, 4, 8, 16

- ``is_complex``: False for PAM, PAM/QAM True
- ``symbol_rate``: in bauds
- ``filter_type``: allowed values: ``root_raised_cosine``, ``raised_cosine``, ``rectangular``, ``FMCW``, ``none``.

    If filter is ``root_raised_cosine`` or ``raised_cosine``: Roll-off factor between 0 and 1, and filter length (in symbols) must be defined.
    Be careful with (root)-raised-cosine filters,as the non-ISI is guaranteed only for the continuous-time
    filter, and long filter length may be required.
    Default values are ``roll-off factor = 0.3, filter length = 16, oversampling factor = 2``.

    if filter is ``FMCW`` (chirp): bandwith and chirp duration must be given.

    if filter is ``rectangular``: rectangular pulses with a given width are considered.

    .. note::
       The oversampling factor may have different meanings:
    
       For FMCW modulation, ``sampling_rate = oversampling_factor * chirp_bandwidth``.
       For other modulation schemes, ``sampling_rate = oversampling_factor * symbol_rate``.

Parameters for RRC:

- ``filter_length``:
- ``roll_off_factor``:

- ``chirp_bandwidth``:
- ``chirp_duration``:

    parameters for FMCW filter (can be given as function of the symbol rate).
    a negative bandwidth means a down chirp.
    please notice that a guard interval of at least ``chirp_duration - 1/symbol_rate`` should be included to guarantee that
    the whole chirps are processed at receiver.

- ``pulse_width``: parameters for rectangular filter, pulse width relative to symbol interval, between 0 and 1
- ``oversampling_factor``: parameters for all filters (number of samples per symbol)

**[Receiver]**: Specifications for receiver implementation.

- ``equalizer``: currently only relevant for inter-chirp interference equalization of FMCW pulses, channel is not equalized),

  - ``NONE``
  - ``ZF``: zero forcing

**[Frame]**: The specifications for the transmission frame.
unmodulated pilot symbols can be added either at the beginning of a frame (preamble) or at its end (postamble)
if filter is "FMCW", then pilots are spaced by 'chirp_duration', i.e., they are non-overlapping
otherwise, then pilots are spaced by the symbol interval
preamble, postamble and guard intervals are optional.

- ``number_preamble_symbols``:
- ``number_data_symbols``:
- ``number_postamble_symbols``:

- ``pilot_symbol_rate``: 

    transmission rate of pilot symbols, can be given as function of symbol_rate or chirp_duration (if defined)
    default value is "symbol_rate". Can also be ``1/chirp_duration``.

- ``guard_interval``: guard interval between frames in s

=====================
settings_quadriga.ini
=====================
**[General]**:

- ``quadriga_executor``: Determines with which application to execute quadriga (either matlab or octave)

**[Scenario]**:

- ``scenario_label``: Scenario to pick from, check quadriga documentation
- ``antenna_kind``: antenna to use for simulation, check quadriga documentation

=====================
settings_scenario.ini
=====================
This file specifies the simulator scenario, i.e., the transmitter, receivers and channel models between them.
The simulation may consist of several transmit and receive modems.
All transmitters must be specified in sections 'TxModem_i', with i the transmit modem index, starting with i=1.
All receivers must be specified in sections 'RxModem_j', with i the receive modem index, starting with j=1.
Between every pair of receiver and transmitter modem, a channel model must be specified, in 'Channel_i_to_j'.

**[TxModem_i]**:

- ``technology_param_file``: file containing technology parameters. Allowed values:
    - settings_psk_qam.ini
    - settings_ofdm.ini
    - settings_chirp_fsk.

- ``encoder_param_file``: file containing coding related parameters. Allowed values:
    - NONE
    - files in *coding/*  directory

- ``rf_chain_param_file``: file containing coding related parameters. Allowed values:
    - NONE
    - any file with RF chain parameters (see example in `settings_rf_chain.ini`_)

3D-position (m) and velocity (m/s) (optional - only relevant for position-dependent channel models):

    - ``position``:
    - ``velocity``:

- ``carrier_frequency``: in Hz
- ``tx_power_db``: optional, only for interference scenarios.

- ``number_of_antennas``:
- ``antenna_spacing``: ratio between distance d and half of wavelength (uniform linear array is considered)

- ``track_angle``:
- ``track_length``:

  Quadriga associates a **track** to each receiver defined in the scenario. The track is a route along which the receiver
  can move. The ``track_angle`` defines the orientation angle of the tracking considering a normal 2D XY
  plane with counter-clockwise rotation. The ``track_length`` is the length of the track itself in meter.
  This track defined is a linear track, i.e. no curves or circles.

- ``device_type``: can be either BASE_SSTATION or UE (required for MIMO channel model, see below)

**[RxModem_i]**:

- ``tx_modem``: index of corresponding transmission modem. Technology will be the same as in the transmission modem

The following parameters are as in **[TxModem_i]**:

- ``position``
- ``velocity``
- ``number_of_antennas``
- ``antenna_spacing``
- ``track_angle``
- ``track_length``
- ``device_type``

**[Channel_<txModemId>_<rxModemId>]**:

First index is the number of the Transmitting modem, second index is the number of the receiving modem.
Parameters are defined for each tx-rx-pair.

- ``multipath_model``: parameter is optional, defaults to ``NONE``

  Supported_values: "None", "STOCHASTIC", "COST259", "EXPONENTIAL", "5G_TDL", "QUADRIGA".

  - ``Stochastic``: a stochastic channel model follows an arbitrary power delay profile, defined in the parameters
  - ``Cost-259``: measurement-based power delay profile defined in COST 259, according to a given scenario
  - ``Exponential``: stochastic model with an exponential-decay power delay profile
  - ``Quadriga``: QuaDRiGa channel model
  - ``5G_TDL``: 5G PHY channel model, TDL implementation

- ``attenuation_db``: channel attenuation, a +3dB attenuation means that signal will have half of its power

The following parameters apply for selection of **stochastic** multipath model:

the power delay profile is given in terms of path delays, relative power (in dB), K-factor of the Rice distribution
(in dB) and the Doppler frequency. For Rayleigh fading, choose K-factor equal to "-inf", for LOS, K-Factor is "inf".
The parameters corresponding to power delay profile (``delays, power_delay_profile_db, k_rice_db``) must be vectors of the
same size.

The LOS components can have a different Doppler frequency, which is scaled by ``los_doppler_factor`` (default = 1.0).
The power delay profile will be normalized to unit power, only relative values are important.
velocity (in m/s) is a scalar.
These parameters are optional, and are needed only for an stochastic channel model
 
Example, for single-path Rayleigh fading (default)
  
  >>> delays = 0
  power_delay_profile_db = 0
  k_rice_db = -inf
 
Example, for a 3-path model with Ricean fading at first path

  >>> delays = 0, 1.0e-6, 3e-6
  power_delay_profile_db = 0, -3, -6
  k_rice_db = 3, -inf, -inf

Therefore, in total we have:

- ``delays``:
- ``power_delay_profile_db``:
- ``k_rice_db``:
- ``los_doppler_factor``:

Parameters for **Cost 259** channel model.
channel can be one of the following types: ``typical_urban``, ``rural_area``, ``hilly_terrain``
the channel delay profile is specified in 3GPP TR25.943 v12.0.0 Rel. 12
This parameter is optional, and is needed only for a COST-259 channel model (default = ``typical_urban``).

- ``cost_type``:

Parameters of **EXPONENTIAL** channel model with Rayleigh-fade paths.
These parameters are optional, and are mandatory only for an exponential channel model,
The channel profile will consist of L+1 paths with delays multiples of dt (tap_interval).
The average power of each path is given by ``g[k] = exp(-alpha * k)``, with alpha the decay exponent, obtained from the
desired rms delay.
``alpha`` is obtained considering a delay profile with infinite paths, whose rms delay is given as
``rms_delay = exp(-alpha/2) / (1-exp(-alpha))``.h
The power delay profile is truncated,such that 99,999% of the channel power is considered.

- ``tap_interval``:
- ``rms_delay``:

Parameters of **QUADRIGA** channel model.
QuaDRiGa channel model was developed by Fraunhofer HHI, more details in https://quadriga-channel-model.de/
Quadriga scenario is defined in **settings_quadriga.ini**.

Parameters of **5G TDL** channel model
Tdl a-e is implemented, cf. ETSI TR 138 901 V14.0.0 ch. 7.7.2 and 7.7.3.
For MIMO simulations, correlation matrices can be defined according to 3gpp TS36.101, Annex B.2.3.
The delays are normalized. If you want to scale them according to rms, specify ``rms_delay``.

- ``tdl_type``: type, can be A, B, C, D, E
- ``rms_delay``:

For MIMO cases, correlation between different antenna channels can be set,
see 3GPP TS36.101, Annex B.2.3.
Correlation can be  LOW, MEDIUM, MEDIUM_A, HIGH or CUSTOM.

.. note::
   Correlation matrices, except CUSTOM, are  defined only for 2 or 4 antennas
   if number of antennas > 4 and correlation is NOT custom, an exception will be raised.

- ``rx_correlation``: correlation for receiver
- ``tx_correlation``: correlation for transmitter

If correlation is set to "CUSTOM", then correlation between adjacent antennas can be set in ``custom_correlation``.
The factor is to be between 0 and 1.

- ``rx_custom_correlation``:
- ``tx_custom_correlation``:

=================
Coding Parameters
=================

**LDPC Encoder**: They are defined by the three values:

- ``encoded_bits_n``:
- ``data_bits_k``:
- ``block_size``: 

Block_size is the approximate size of a block, i.e. the length of a code word. choosing ``n``, ``k``, and ``block_size``
determines the actual block_size.  The same applies to the data_bit_word.

Let ``code_rate = encoded_bits_n / data_bits_k``. 
We support the following ``code_rate``s: 1/3, 1/2, 2/3, 3/4, 4/5, 5/6.

We support the following ``block_size``s: 256, 512, 1024, 2048, 4096, 8192.

We want to emphasize that your own LDPC codes can be defined. For this, the absolute path to ``custom_ldpc_codes`` of the parent directory need to be passed.
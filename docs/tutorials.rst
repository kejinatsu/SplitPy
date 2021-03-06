Tutorials
=========

Creating the ``StDb`` Database
++++++++++++++++++++++++++++++

All the scripts provided require a ``StDb`` database containing station
information and metadata. Let's first create this database for station
LOBS3 and send the prompt to a logfile

.. code-block::

    $ query_fdsn_stdb.py -N TA -C HH? -S TGTN TGTN > logfile

To check the station info for TGTN, use the program ``ls_stdb.py``:

.. code-block::

    $ ls_stdb.py TGTN.pkl
    Listing Station Pickle: TGTN.pkl
    NY.TGTN
    --------------------------------------------------------------------------
    1) NY.TGTN
         Station: NY TGTN 
          Alternate Networks: None
          Channel: HH ;  Location: --
          Lon, Lat, Elev:  61.52670, -128.27269,   0.000
          StartTime: 2013-07-01 00:00:00
          EndTime:   2020-05-20 13:34:38
          Status:    partial
          Polarity: 1
          Azimuth Correction: 0.000000

Automated analysis
++++++++++++++++++

There are two modes for producing shear-wave splitting estimates: Automated or Manual. In
the automated mode, the code uses default uniform settings for all available seismograms
to produce splitting estimates. In the manual mode, the code will search for available 
(pre-processed) data on disk and use a Graphical-User Interface (GUI) to refine the analysis
window based on new picks. If the automated estimates are not available, they will first be 
determined before refining the window. 

Downloading data
----------------

Simply run :ref:`splitauto` with ``TGTN.pkl`` to download all available
seismic data suitable for shear-wave splitting analysis.

.. code-block::

   $ split_calc_auto.py --keys=NY.TGTN --local-data=/mnt/datadisk/DaySac/ --start=2020-01-01 --end=2020-05-20 TGTN.pkl

This uses all default settings for window lengths, magnitude criteria, etc. 
In this example, data will be used from IRIS as well as any local data 
on disk (defined with the ``--local-data`` flag - this needs to be customized for
your own file structure). If no data exists on disk, then 
the program will search on the specific data sever (through ``obspy`` clients). In this
example, only events that occurred between January 1, 2020 and May 20, 2020 will 
be considered. Based on the criteria specified (see :ref:`splitauto`), seismograms will be 
downloaded where the minimum SNR threshold is exceeded. All data will be saved in separate 
time-key folders to ``PATH/NY.TGTN/YYYYMMDD_HRMNSC/*``. 

Downloading and Processing
--------------------------

You can run :ref:`splitauto` to automatically estimate the shear-wave splitting 
parameters by specifying the argument ``-C`` or ``--calc``. Choosing ``-V`` 
or ``--verbose`` will display
the results to the terminal as the script proceeds. If you wish to visualize the results
for each event, you can further select ``-P`` or ``--plot-diagnostic``. This will pop a
summary Figure (i.e., ``Figure 1``) of the splitting results for this particular event.

As an example of a Good, non-null estimate, type the following line in the terminal
(note the argument ``-O`` to overwrite existing results, and no key is specified since
there is only one key in the database):

.. code-block::

    $ split_calc_auto.py --start=2020-03-18 --end=2020-03-19 -V -C -P -O TGTN.pkl

This will produce, in the terminal:

.. code-block::

    |==================================================|
    |                       TGTN                       |
    |==================================================|
    |  Station: NY.TGTN                                |
    |      Channel: HH; Locations: --                  |
    |      Lon: -128.27; Lat:  61.53                   |
    |      Start time: 2013-07-01 00:00:00             |
    |      End time:   2020-05-20 13:34:38             |
    |--------------------------------------------------|
    | Searching Possible events:                       |
    |   Start: 2020-03-18 00:00:00                     |
    |   End:   2020-03-19 00:00:00                     |
    |   Mag:   >{0:3.1f} 6.0                           |
    | ...                                              |
    |  Found     2 possible events                     |
    |==================================================|
     
    |**************************************************|
    * #1 (2/2):  20200318_031345 NY.TGTN
    *   Phase: SKS
    *   Origin Time: 2020-03-18 03:13:45
    *   Lat: -13.14;        Lon:  167.03
    *   Dep: 176.00 km;     Mag: 6.1
    *   Dist: 10000.87 km;   Epi dist:  89.94 deg
    *   Baz:  241.82 deg;   Az:  25.63 deg
    * Requesting Waveforms: 
    *    Startime: 2020-03-18 03:34:38
    *    Endtime:  2020-03-18 03:38:38
    *     TGTN.HH - ZNE:
    *          HH[ZNE].-- - Checking Network
    *              - ZNE Data Downloaded
    * Start times are not all close to true start: 
    *   HHE 2020-03-18T03:34:38.110000Z 2020-03-18T03:38:39.100000Z
    *   HHN 2020-03-18T03:34:38.110000Z 2020-03-18T03:38:39.100000Z
    *   HHZ 2020-03-18T03:34:38.110000Z 2020-03-18T03:38:39.100000Z
    *   True start: 2020-03-18T03:34:38.107273Z
    * -> Shifting traces to true start
    * Waveforms Retrieved...
    * SNRQ: 12.51340359244245
    * SNRT: 8.8889144288134
    * --> Calculating Rotation-Correlation (RC) Splitting
    * --> Calculating Silver-Chan (SC) Splitting
    *     Null Classification: 
    *       SNR T Pass: 8.89 > 1.00
    *       dPhi Pass:  3.00 outside 22. < X < 68.
    *     Quality Estimate: Non-Null -- Good
    *         rho: 1.00; dphi: 3.00
    *           Good: 0.8 < rho < 1.1  &&  dphi < 8
    *           Fair: 0.7 < rho < 1.2  &&  dphi < 15
    *           Poor: rho < 0.7 | rho > 1.3 &&  dphi > 15
     ======= Meta data ========

     SNR (dB):            13
     Station:             TGTN
     Time:                2020-03-18T03:13:45.742000Z
     Event depth (km):    0
     Magnitude (Mw):      6.1
     Longitude (deg):     167.03
     Latitude (deg):      -13.14
     GAC (deg):           89.94
     Backazimuth deg):    241.82
     Incidence (deg):      10.17
     SNR - Q:      12.51
     SNR - T:      8.89

     ======= Best-fit splitting results ========

     Best fit values: RC method
     Phi = -75 degrees +/-  7
     dt = 1.3 seconds +/- 0.1

     Best fit values: SC method
     Phi = -78 degrees +/-  5
     dt = 1.3 seconds +/- 0.2

     ======= Nulls and quality ========

     Is Null?      False
     Quality:      Good

``Figure 1`` summarizes the results of the splitting calculation. The top left "Q,T" 
frame shows the un-corrected radial (Q) and tangential (T) components within the 
time window. The second row of panels correspond to the 'Rotation-Correlation' 
results, and the third row of panels is for the 'Silver-Chan' results. In each 
case, the first column shows the corrected Q and T fast and slow components, the 
second column the corrected Q and T components, the third column the before and after 
particle motion, and the fourth column the map of the error surfaces. A text box 
prints out the summary of the results, including whether or not the estimate is a 
Null, and the quality of the estimate ('good', 'fair', 'poor').

.. figure:: ../splitpy/examples/figures/Figure_1.png
    :align: center

Re-Processing
-------------

It is also possible to re-calculate the estimates for different parameters using the 
argument ``--recalc``, which will be applied uniformly to all available data. 
In this case the data will not be re-downloaded and the data files will simply be updated 
in place. Plotting can also be done as in the previous example. For example, let's 
change the frequency settings and re-calculate the previous example:

.. code-block::

    $ split_calc_auto.py --start=2020-03-18 --end=2020-03-19 --fmin=0.05 --fmax=1. -V -R -P -O TGTN.pkl

This will produce, in the terminal:

.. code-block::

    |==================================================|
    |                       TGTN                       |
    |==================================================|
    |  Station: NY.TGTN                                |
    |      Channel: HH; Locations: --                  |
    |      Lon: -128.27; Lat:  61.53                   |
    |      Start time: 2013-07-01 00:00:00             |
    |      End time:   2020-05-20 13:34:38             |
    |--------------------------------------------------|
    | Searching Possible events:                       |
    |   Start: 2020-03-18 00:00:00                     |
    |   End:   2020-03-19 00:00:00                     |
    |   Mag:   >{0:3.1f} 6.0                           |
    | ...                                              |
    |  Found     2 possible events                     |
    |==================================================|
     
    |**************************************************|
    * #1 (2/2):  20200318_031345 NY.TGTN
    *   Phase: SKS
    *   Origin Time: 2020-03-18 03:13:45
    *   Lat: -13.14;        Lon:  167.03
    *   Dep: 176.00 km;     Mag: 6.1
    *   Dist: 10000.87 km;   Epi dist:  89.94 deg
    *   Baz:  241.82 deg;   Az:  25.63 deg
    * SNRQ: 13.03806173520674
    * SNRT: 8.36765404740968
    * --> Calculating Rotation-Correlation (RC) Splitting
    * --> Calculating Silver-Chan (SC) Splitting
    *     Null Classification: 
    *       SNR T Pass: 8.37 > 1.00
    *       dPhi Pass:  2.00 outside 22. < X < 68.
    *     Quality Estimate: Non-Null -- Good
    *         rho: 1.00; dphi: 2.00
    *           Good: 0.8 < rho < 1.1  &&  dphi < 8
    *           Fair: 0.7 < rho < 1.2  &&  dphi < 15
    *           Poor: rho < 0.7 | rho > 1.3 &&  dphi > 15
     ======= Meta data ========

     SNR (dB):            13
     Station:             TGTN
     Time:                2020-03-18T03:13:45.742000Z
     Event depth (km):    0
     Magnitude (Mw):      6.1
     Longitude (deg):     167.03
     Latitude (deg):      -13.14
     GAC (deg):           89.94
     Backazimuth deg):    241.82
     Incidence (deg):      10.17
     SNR - Q:      13.04
     SNR - T:      8.37

     ======= Best-fit splitting results ========

     Best fit values: RC method
     Phi = -76 degrees +/-  6
     dt = 1.3 seconds +/- 0.1

     Best fit values: SC method
     Phi = -78 degrees +/-  4
     dt = 1.3 seconds +/- 0.1

     ======= Nulls and quality ========

     Is Null?      False
     Quality:      Good

.. figure:: ../splitpy/examples/figures/Figure_1b.png
    :align: center

Manual analysis
+++++++++++++++

In the manual mode, the script :ref:`splitmanual` will use the available 
data and/or estimates and
use a Graphical User Interface (GUI) to refine the picking window. The script will 
search for data and splitting estimates in the folder structure. If the estimates
are not available (i.e., not previously calculated in :ref:`splitauto`), the script
will calculate them automatically. 

Re-picking
----------

After loading/processing the automated results, the script will produce two Figures.
``Figure 1`` shows the three 
rotated component waveforms (LQT), along with lines representing the SKS, SKKS, 
S, PKS and ScS arrivals from model ``iasp91``. Red vertical lines denote the analysis 
window. This figure is interactive and the picks in red can be refined by clicking
at the two x-positions of the new analysis window.

From the previous example, examining and possibly refining the results for only 
one day of data:

.. code-block::

    $ split_calc_manual.py --start=2020-03-18 --end=2020-03-19 TGTN.pkl

.. figure:: ../splitpy/examples/figures/Figure_2.png
   :align: center

The diagnostic (summary) figure (``Figure 2``) will also open, showing the results 
from the most recent automated estimate (i.e., can be from a re-calculated estimate, 
see :ref:`splitauto`). A message box will pop up asking whether to Re-pick the 
window in ``Figure 1``. This is done to refine the signal window in 
which the measurements are made in order to eliminate possibly contaminating phases
and improve the measurements. If the ``-V`` or ``--verbose`` argument has been 
selected, the terminal will show a summary of the processing, as in previous examples.

Once ``No`` is selected for the picking/re-picking of the window, a second box 
will pop up asking whether to keep the estimates. Click ``Yes`` to save the results, 
or ``No`` to discard the measurement.

Station average
+++++++++++++++

Plotting and subsequent processing of splitting results is carried out using 
:ref:`splitaverage`, where options are present to control selection of nulls 
and quality settings, as well as which methods are used. All available data are
processed. By default, the script will search for the ``manual`` results. The user
can specify to use the ``auto`` results with the argument ``-A`` or ``--auto``. The final 
average splits are then saved in a text file for future use.

For example, after running the refined processing for 4 years of data for station
TGTN (i.e., typing ``split_calc_auto.py --start=2016-01-01 -V -C TGTN.pkl``, which will 
take a long time to run and process all the data), we can visualize the results
by typing in a terminal:

.. code-block::

    $ split_average.py --show-fig -V TGTN.pkl
    ---------------------------
    Selection Criteria 
     Null Value: 
        Non Nulls: True
        Nulls:     False
     Quality Value: 
        Goods:  True
        Fairs:  True
        Poors:  False
    ---------------------------
      Processing 141 Events...
          20160413_135517 Good Non-Null -> Retained
          20160527_040843 Good Non-Null -> Retained
          20160607_191515 Fair Non-Null -> Retained
          20160619_094723 Fair Non-Null -> Retained
          20160713_121112 Fair Non-Null -> Retained
          20160924_212841 Good Non-Null -> Retained
          20170819_020052 Good Non-Null -> Retained
          20170926_042000 Good Non-Null -> Retained
          20180728_170723 Good Non-Null -> Retained
          20180819_042858 Good Non-Null -> Retained
          20180906_154914 Fair Non-Null -> Retained
          20180910_041902 Fair Non-Null -> Retained
          20180910_193137 Good Non-Null -> Retained
          20180916_211148 Good Non-Null -> Retained
          20181016_002812 Fair Non-Null -> Retained
          20190531_101232 Poor Non-Null -> Skipped
          20190616_051716 Good Non-Null -> Retained
          20190707_150840 Good Non-Null -> Retained
          20190731_150233 Good Non-Null -> Retained
          20190929_020251 Fair Non-Null -> Retained
          20191031_011119 Good Non-Null -> Retained
          20200318_031345 Fair Non-Null -> Retained

    *** Station Average from 21 measurements ***
       Loc: -128.2727, 61.5267
       PHI:  -4.111 d +- 2.891
       DT:    0.729 s +- 0.052
       Saved to: PLOTS/NY.TGTN_RC-SC_Nons_G-F_results.dat

.. figure:: ../splitpy/examples/figures/Figure_3.png
   :align: center

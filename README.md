# Optical link data format
This repository contains the data exchange format for optical fiber link clock comparisons.

This document as been prepared for the European Partnership on Metrology Project [TOCK](https://www.ptb.de/epm2022/tock/home).

This data format was developed for the European Metrology Programme for Innovation and Research (EMPIR) Projects [OFTEN](https://www.euramet.org/research-innovation/search-research-projects/details/?tx_eurametctcp_project[project]=1409) and [ROCIT](http://empir.npl.co.uk/rocit/).  The format implements the universal formalism presented by Lodewyck et al. (2020) and handles both microwave clocks, optical clocks and designed oscillators (DOs) such as ultrastable cavities and signals disseminated by optical fiber links.

Clock comparison is understood to mean measuring the ratio of the frequencies of two clocks. Time transfer and time comparison are beyond the scope of this format.


## Procedure for clock comparisons
Participating laboratories designate a number of oscillators that form the user-level nodes of the network.
User-level links relate the frequencies of pairs of these designated oscillators.
When labs compare clocks, they use an agreed time window, e.g. 1 second, during which each measures their clock against a local designated oscillator. User-level link data is shared in a central repository and can be accessed by other users. Combining the retrieved and measured data is used to relate the two local designated oscillators across the agreed time window, potentially through a series of intermediate designated oscillators.
This allows the frequency ratio of the clocks to be determined.
The frequency fluctuations of intermediate designated oscillators should cancel out in the ratio of clock frequencies.


## Data exchange format for fiber link comparisons

The data exchange is based on the publication of a comparator output for each pair of adjacent oscillators.
The comparator output $\Delta_{A\rightarrow B}$ resulting from the comparison of oscillators A and B is defined as a scaled
measured transfer beat:

$$\Delta_{A\rightarrow B} = \frac{\hat{f}^T_{A\rightarrow B}}{s_B}  = \frac{\hat{\nu}_B -\rho^0 _{B,A} \hat{\nu}_A}{s_B} $$

where the nominal frequency ratio $\rho^0_{B,A}$ and the scaling factor $s_B$ are numerical constants, freely and independently chosen by the operator of each comparator. The frequency reference against which the transfer beat is measured is also freely chosen by the operator.


> **_NOTE:_**  The relation between the comparator output and the transfer beat is exposed in a different way than in Lodewyck et al. (2020). The aim of the presentation chosen here is to seamlessly publish in a unified way comparator outputs and transfer beats.

> **_NOTE:_**  In general $\Delta_{A\rightarrow B} \neq - \Delta_{B\rightarrow A}$.

### Definition of the exchange format
The data format is defined by the following set of requirements:

1. A file, or a set of files, located in the data’s main directory report on the chosen constants and related information. This file is written in YAML, and has the extension .yml. A single file for all institutes may be used, or different files  for each institute, the latter being equivalent to the former after concatenating all the YAML files. The files contain entries specifying the constant used to produce the comparator output $\Delta_{A\rightarrow B}$, using the following format:

	| | | |
	|---|---|---|
	|`- name`| comparator name, in the form `INSTITUTEB_OSCB-INSTITUTEA_OSCA` | String |
	|`numrhoBA`| numerator of the nominal frequency ratio $\rho^0_{B,A}$ | Arbitrary precision floating point |
	|`denrhoBA`| denominator of the nominal frequency ratio $\rho^0_{B,A}$ | Arbitrary precision floating point |
	|`sB` |  scaling factor | Double precision floating point |
	|`nu0A` | nominal value $\hat{\nu}^0_A$ for the frequency of A (optional) | Arbitrary precision floating point |
	|`grsA` | Gravitational redshift correction for the oscillator A, in relative units  (optional)| Float  |
	|`uA_sys`| fractional uncertainty of the oscillator A (optional) | Float |
	|`nu0B` | nominal value $\hat{\nu}^0_B$ for the frequency of B (optional) | Arbitrary precision floating point |
	|`grsB` | Gravitational redshift correction for the oscillator B, in relative units  (optional)| Float  |
	|`uB_sys`| fractional uncertainty of the oscillator B (optional) | Float |
	|`interval`|  Duration of each measurement, in seconds (optional) | Float |
	|`lag`| Fractional timetag location wrt the interval (1=end of the interval) (optional) | Float |
	|`weighting`| Weighting function of the frequency counter (optional) | "lambda" or "pi" |
	|`ref_osc`| Local reference oscillator (optional) | String |
	

	> **_NOTE:_**  The comparator output does not depend on the `nu0A` or `nu0B` values. They are only used for data processing.

	> **_NOTE:_** `nu0A` or `nu0B` values should be given for accurate oscillators (discrepancy between its actual frequency and its nominal frequency must be $\lessapprox 10^{-13}$), such as masers and optical clocks.

	> **_NOTE:_**  For input data, it is reccomended to have A as the accurate oscillator for which to specify `nu0A`, `grsA` and `uA_sys`. `nu0B`, `grsB` and `uB_sys` are there for simmetry.

	> **_NOTE:_**  This specification does not allow to change the parameters such as the nominal frequency ratio in the course of a campaign. This is deemed sufficient for most cases. 

2. The comparator output $\Delta_{A\rightarrow B}$ is stored in data file or files located in a folder called `INSTITUTEB_OSCB-INSTITUTEA_OSCA`. Any file name is acceptable, provided the lexicographic order of the filename matches the chronological order of the data. The data files contain:

	- An optional header, whose lines must start with `#`. The content of the header is chosen by the person who produces 	that data, and is purely informative, i.e. not necessary to process the data.

	- Column 1: date and time in Modified Julian Date (MJD)

	- Column 2: comparator output $\Delta_{A\rightarrow B}$

	- Column 3: validity flag (0 = invalid, 1 = valid but experimental, 2 = valid)
	
	- Column 4: time-varying systematic uncertainty (optional, for accurate clocks only)

	- Column >4: custom information. Not used in automatic data analysis scripts.
	
	


## Examples of comparator outputs

With the examples listed in the table below, we show that the file format defined in section 1.1 can ac-
commodate with many different physical representations of the data, including the transfer beats that are
currently shared, but also pure optical to optical frequency ratios. These representations are selected by
choosing appropriate NFRs, scaling factor, and reference oscillators.


| $\rho^0_{B,A}$                | $s_B$           | Reference | $\Delta_{A\rightarrow B}$                 | Physical interpretation of the comparator                                             |
|-------------------------------|-----------------|-----------|-------------------------------------------|---------------------------------------------------------------------------------------|
| $1/1$                         | +1              | local RF  | $\nu_B - \nu_A$                           | Beatnote                                                                              |
| $1/1$                         | -1              | local RF  | $\nu_A - \nu_B$                           | Beatnote with opposite sign                                                           |
| $1/1$                         | $\hat{\nu}^0_B$ | local RF  | $(\nu_B - \nu_A)/\hat{\nu}^0_B$           | Beatnote in relative units, with respect to  $\hat{\nu}^0_B=\hat{\nu}^0_A$            |
| $m_B/m_A$                     | +1              | local RF  | $\hat \nu_B - \frac{m_B}{m_A}\hat \nu_A$  | Transfer beat with prefered sign convention (A = clock; B = DO)                       |
| $m_B/m_A$                     | -1              | local RF  | $\frac{m_B}{m_A}\hat \nu_A - \hat \nu_B$  | Transfer beat with opposite sign (A = clock; B = DO)                                  |
| $\hat{\nu}^0_B/\hat{\nu}^0_A$ | +1              | A         | $\hat \nu_B - \hat{\nu}^0_B$              | Frequency of B, referenced to A (e.g., A is a maser or optical clock)                 |
| $\hat{\nu}^0_B/\hat{\nu}^0_A$ | $\hat{\nu}^0_B$ | A         | $\tilde{\rho}_{B,A}$                      | Frequency of B, referenced to A (e.g., A is a maser or optical clock), relative units |
| $\hat{\nu}^0_B/\hat{\nu}^0_A$ | $\hat{\nu}^0_B$ | local RF  | $\tilde{\rho}_{B,x} - \tilde{\rho} _{A,x}$| Difference of reduced frequency ratios, using an external reference x                 |
| $1/1$                         | $\hat{\nu}^0_B$ | A         | $\tilde{\rho}_{B,A}$                      | Relative systematic correction ($B$ = corrected clock, $A$ = uncorrected clock)       |

> **_NOTE:_** The reference oscillator must be accurate, in the sense that the discrepancy between its actual frequency and its nominal frequency must be $\lessapprox 10^{-13}$

## Examples
See the Example folder for fictitious data used to test the data format before the March 2023 campaign of the project ROCIT.

## Funding
![badge](./Acknowledgement%20badge.png)

## References
[Lodewyck et al.,  "Universal formalism for data sharing and processing in clock comparison networks", Phys. Rev. Research  2, 043269 (2020).](https://link.aps.org/doi/10.1103/PhysRevResearch.2.043269)

[https://github.com/INRIM/tintervals](https://github.com/INRIM/tintervals)

[TOCK](https://www.ptb.de/epm2022/tock/home)

[ROCIT](http://empir.npl.co.uk/rocit/)

[OFTEN](https://www.euramet.org/research-innovation/search-research-projects/details/?tx_eurametctcp_project[project]=1409)

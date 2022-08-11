# Audiomath

Convert between stuff! Plug one number in and get the others out

- Frequency in Hertz
- Pitch/Note with fine pitch
- Period of a single cycle in milliseconds
- Wavelength of a cycle in meters
- Digital sample count of a single cycle at the given sample rate (44.1kHz)

## Precision Disclaimer

Due to the imprecise nature of the underlying system (JavaScript) all results are approximations, but you should be warned about sample count in specific.

Converting _to_ samples is ok, but converting _from_ samples is noticebly imprecise (A4 ends up with a 3Hz difference from standard). This is due to the fact that digital audio is discrete, whereas analog is not, which means that, depending on your sample rate, you may have many frequencies corresponding to the same sample count.

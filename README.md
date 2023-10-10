# Audiomath

Convert between stuff! Plug one number in and get the others out

- Frequency in Hertz
- Pitch with fine pitch
- Period of a single cycle in milliseconds
- Wavelength of a cycle in meters
- Digital sample count of a single cycle
- **NEW!** Supports different sample rates

## Precision Disclaimer

Due to the difference in nature between physical measurements (frequency, wavelength, period and pitch) and digital samples, conversions should take two considerations into account:

1. If your **input** is digital sample count, treat the results as **truncated**. Since there's only a single frequency with a single cycle that perfectly matches an integer sample count, that will be your output frequency.

1. If your **output** is digital sample count, treat the result as **rounded up to the nearest integer**. This is needed to account for the fact that not all single wave cycles fit perfectly inside an integer ammount of digital samples, so it always rounds up to the nearest count that fits.
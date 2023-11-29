# UW-Madison HSRL L1B scanning data format

## The geophysical variables and ancillary data

The ground-based University of Wisconsin (UW)-Madison scanning High Spectral Resolution Lidar (HSRL) *L1B* (similar to a NASA CALIOP L1B product) scanning product consists of the following geophysical wavelength calibrated measurements and ancillary data:

| `netcdf4` variable name                     | Units                               |
| ------------------------------------------- | ------------------------------------|
| `particulate_backscatter_532nm`             | $\mathrm{m}^{-1}\,\mathrm{sr}^{-1}$ |
| `particulate_linear_depolarization_532nm`   | unitless                            |
| `particulate_optical_depth_532nm`           | unitless                            |
| `particulate_molecular_optical_depth_532nm` | unitless                            |
| `attenuated_color_ratio_1064nm_532nm`       | unitless                            |
| `temperature`                               | K                                   |
| `pressure`                                  | hPa                                 |
| `dewpoint`                                  | K                                   |
| `telescope_angle`                           | degrees                             |
| `effective_resolution`                      | milliseconds                        |

The calibrated 1064nm attenuated backscatter [$\mathrm{m}^{-1}\,\mathrm{sr}^{-1}$] will be added in the future, along with uncertainties (i.e. standard deviation) of each geophysical variable. The HSRL telescope angle is relative to zenith. The effective resolution is a nuanced variable and requires an explanation of how the HSRL operates as discussed in a subsequent section.

The geophysical variables have been masked for low signal-to-noise ratio, and any "clear-air" areas have been masked out where the backscatter is less than than $10^{-8} [\mathrm{m}^{-1}\,\mathrm{sr}^{-1}]$.

## The three telescope angle configurations

The UW HSRL produces the calibrated measurements in three telescope angle configurations, namely: *vertical stare, horizontal stare and scanning*. The following table shows an example of the telescope angle configuration of 08/18/2023.

| Time UTC | Duration [hour] | Configuration |
| -------- | --------------- | ------------- |
| 00:00    | 1               | Horizontal    |
| 01:00    | 1               | Vertical      |
| 02:00    | 1               | Horizontal    |
| 03:00    | 3               | Scanning      |
| 06:00    | 3               | Vertical      |
| 09:00    | 3               | Scanning      |
| 12:00    | 3               | Vertical      |
| 14:00    | 3               | Scanning      |
| 18:00    | 3               | Vertical      |
| 21:00    | 3               | Scanning      |

A stare is defined as the HSRL telescope being fixed at a specific angle for more than 30 seconds. Here are the telescope angle intervals of each of these configuration.

| Configuration    | Angle $\theta$ interval |
| -----------------| ----------------------- |
| Vertical stare   | $\theta\approx$-15°     |
| Horizontal stare | $89°\leq\theta< 91°$ |
| Scanning         | $70°\leq\theta\leq 90°$ |

The geophysical variables and ancillary data for each of these telescope angle configurations are available in their own `netcdf4` groups. Furthermore, the variables and ancillary data have different dimensions depending on the telescope angle configuration.

## Variable dimensions

| Configuration    | Primary dimensions                | Secondary dimensions                             |
| ---------------- | --------------------------------- | ------------------------------------------------ |
| Vertical stare   | altitude [$\mathrm{m}$] and time  | above ground level (AGL) altitude [$\mathrm{m}$] |
| Horizontal stare | range [$\mathrm{m}$] and time     | altitude, AGL altitude, altitude_time            |
| Scanning         | scan_time, angle [degrees], range | altitude, AGL altitude, distance [$\mathrm{m}$]  |

The range dimension is defined as the axis over which the HSRL laser pulses traverse. Dimension `scan_time` is the start of the scan. The secondary dimensions are defined relative to the primary dimensions, and for the horizontal stare and scanning data these dimensions are two dimensional. For example, `altitude_time` is defined over the range and time dimensions.

## Effective resolution

When the HSRL performs a vertical or horizontal stare the nominal time resolution is fixed for those intervals. Meaning, the effective resolutions of these telescope angle configurations are fixed. During the scanning, however, the HSRL actually does a combination of horizontal scans, vertical stares and sometimes horizontal stares. An example sequence of these telescope scan configuration is:
1. Scan from 70° down to 90°,
2. scan from 90° up to 70°,
3. stare vertically for $\approx$ 30 seconds,
4. and repeat the sequence.

For scan data, that are not aggregated over the `scan_time` dimension, an indvidual scan (i.e., for a fixed `scan_time`) is either a downward ($70° \to 90°$) or upward scan ($90° \to 70°$). The intermediate vertical stare measurements are *not* included in the scan data, **instead** are included in the vertical stare data. Since the time aggregation resolution can differ between the intermediate and normal vertical stare profiles, the effective resolution of the vertical profiles in the vertical stare data can have different time resolutions. Hence, if your analyses is senstive to the temporal resolution of the geophysical variables you should consult the `effective_resolution` ancillary variable.

## The L1B file name convention

The L1B file name consists of the following parts:
```
<instrument name>_<start date>_<end date>_<time resolution>s_<altitude resolution>m_<angle resolution>deg_<number of scan aggregated>sca_L1B<_tag>.nc
```
For example the file name
```
bagohsrl_20230901T000000_20230902T000000_30.0s_30.0m_1.0deg_1sca_L1B.nc
```
specific UW HSRL instrument is designated as the `bagohsrl`. The file consists of measurements of the 1st of September where the nominal time and altitude resolutions are 30 seconds and 30 meters, and the scanning data has a (downsampled) angular resolution of 1 degree. The number of scans that were aggregated is 1, implying no scans were aggregated. Hence, each scan is either a scan sweep from 70° down to 90° or 90° up to 70°.

## Command Data Language (CDL) example of the netcdf4
```
{

// global attributes:
		:documentation = "https://ssec.wisc.edu/hsrl" ;

group: vertical_stare {
  dimensions:
  	time = 288 ;
  	altitude = 667 ;
  variables:
  	double temperature(time, altitude) ;
  		temperature:_FillValue = NaN ;
  		temperature:units = "K" ;
  		temperature:source = "NOAA raob" ;
  		temperature:long_name = "temperature" ;
  		temperature:coordinates = "agl_altitude" ;
  	double pressure(time, altitude) ;
  		pressure:_FillValue = NaN ;
  		pressure:units = "hPa" ;
  		pressure:source = "NOAA raob" ;
  		pressure:long_name = "pressure" ;
  		pressure:coordinates = "agl_altitude" ;
  	double dewpoint(time, altitude) ;
  		dewpoint:_FillValue = NaN ;
  		dewpoint:units = "K" ;
  		dewpoint:source = "NOAA raob" ;
  		dewpoint:long_name = "dewpoint" ;
  		dewpoint:coordinates = "agl_altitude" ;
  	double particulate_linear_depolarization_532nm(time, altitude) ;
  		particulate_linear_depolarization_532nm:_FillValue = NaN ;
  		particulate_linear_depolarization_532nm:long_name = "particulate linear depolarization 532nm" ;
  		particulate_linear_depolarization_532nm:valid_min = 0. ;
  		particulate_linear_depolarization_532nm:valid_max = 0.6 ;
  		particulate_linear_depolarization_532nm:coordinates = "agl_altitude" ;
  	double particulate_optical_depth_532nm(time, altitude) ;
  		particulate_optical_depth_532nm:_FillValue = NaN ;
  		particulate_optical_depth_532nm:long_name = "particulate optical depth 532nm" ;
  		particulate_optical_depth_532nm:valid_min = 0. ;
  		particulate_optical_depth_532nm:coordinates = "agl_altitude" ;
  	double particulate_molecular_optical_depth_532nm(time, altitude) ;
  		particulate_molecular_optical_depth_532nm:_FillValue = NaN ;
  		particulate_molecular_optical_depth_532nm:long_name = "particulate and molecular optical depth 532nm" ;
  		particulate_molecular_optical_depth_532nm:valid_min = 0. ;
  		particulate_molecular_optical_depth_532nm:coordinates = "agl_altitude" ;
  	double particulate_backscatter_532nm(time, altitude) ;
  		particulate_backscatter_532nm:_FillValue = NaN ;
  		particulate_backscatter_532nm:long_name = "particulate backscatter 532nm" ;
  		particulate_backscatter_532nm:valid_min = 1.e-09 ;
  		particulate_backscatter_532nm:units = "1/m 1/sr" ;
  		particulate_backscatter_532nm:coordinates = "agl_altitude" ;
  	double attenuated_color_ratio_1064nm_532nm(time, altitude) ;
  		attenuated_color_ratio_1064nm_532nm:_FillValue = NaN ;
  		attenuated_color_ratio_1064nm_532nm:long_name = "attenuated color ratio 1064nm/532nm" ;
  		attenuated_color_ratio_1064nm_532nm:valid_min = 0. ;
  		attenuated_color_ratio_1064nm_532nm:valid_max = 1. ;
  		attenuated_color_ratio_1064nm_532nm:coordinates = "agl_altitude" ;
  	int64 effective_resolution(time) ;
  		effective_resolution:long_name = "effective_resolution" ;
  		effective_resolution:description = "The effective horizontal temporal resolution of each profile." ;
  		effective_resolution:units = "milliseconds" ;
  	double telescope_angle(time) ;
  		telescope_angle:_FillValue = NaN ;
  		telescope_angle:long_name = "telescope angle" ;
  		telescope_angle:units = "degree" ;
  		telescope_angle:description = "The optical telescope scan angle" ;
  		telescope_angle:dpl_py_binding = "rs_raw.scanning_telescope_rotation_angle" ;
  	int64 time(time) ;
  		time:units = "minutes since 2023-08-30 00:00:00" ;
  		time:calendar = "proleptic_gregorian" ;
  	double altitude(altitude) ;
  		altitude:_FillValue = NaN ;
  		altitude:long_name = "mean sea level altitude" ;
  		altitude:units = "m" ;
  	double agl_altitude(altitude) ;
  		agl_altitude:_FillValue = NaN ;
  } // group vertical_stare

group: horizontal_stare {
  dimensions:
  	time = 288 ;
  	range = 667 ;
  variables:
  	double temperature(time, range) ;
  		temperature:_FillValue = NaN ;
  		temperature:units = "K" ;
  		temperature:source = "NOAA raob" ;
  		temperature:long_name = "temperature" ;
  		temperature:coordinates = "agl_altitude altitude altitude_time" ;
  	double pressure(time, range) ;
  		pressure:_FillValue = NaN ;
  		pressure:units = "hPa" ;
  		pressure:source = "NOAA raob" ;
  		pressure:long_name = "pressure" ;
  		pressure:coordinates = "agl_altitude altitude altitude_time" ;
  	double dewpoint(time, range) ;
  		dewpoint:_FillValue = NaN ;
  		dewpoint:units = "K" ;
  		dewpoint:source = "NOAA raob" ;
  		dewpoint:long_name = "dewpoint" ;
  		dewpoint:coordinates = "agl_altitude altitude altitude_time" ;
  	double particulate_linear_depolarization_532nm(time, range) ;
  		particulate_linear_depolarization_532nm:_FillValue = NaN ;
  		particulate_linear_depolarization_532nm:long_name = "particulate linear depolarization 532nm" ;
  		particulate_linear_depolarization_532nm:valid_min = 0. ;
  		particulate_linear_depolarization_532nm:valid_max = 0.6 ;
  		particulate_linear_depolarization_532nm:coordinates = "agl_altitude altitude altitude_time" ;
  	double particulate_optical_depth_532nm(time, range) ;
  		particulate_optical_depth_532nm:_FillValue = NaN ;
  		particulate_optical_depth_532nm:long_name = "particulate optical depth 532nm" ;
  		particulate_optical_depth_532nm:valid_min = 0. ;
  		particulate_optical_depth_532nm:coordinates = "agl_altitude altitude altitude_time" ;
  	double particulate_molecular_optical_depth_532nm(time, range) ;
  		particulate_molecular_optical_depth_532nm:_FillValue = NaN ;
  		particulate_molecular_optical_depth_532nm:long_name = "particulate and molecular optical depth 532nm" ;
  		particulate_molecular_optical_depth_532nm:valid_min = 0. ;
  		particulate_molecular_optical_depth_532nm:coordinates = "agl_altitude altitude altitude_time" ;
  	double particulate_backscatter_532nm(time, range) ;
  		particulate_backscatter_532nm:valid_min = 1.e-09 ;
  		particulate_backscatter_532nm:units = "1/m 1/sr" ;
  		particulate_backscatter_532nm:coordinates = "agl_altitude altitude altitude_time" ;
  		particulate_backscatter_532nm:_FillValue = NaN ;
  		particulate_backscatter_532nm:long_name = "particulate backscatter 532nm" ;
  	double attenuated_color_ratio_1064nm_532nm(time, range) ;
  		attenuated_color_ratio_1064nm_532nm:_FillValue = NaN ;
  		attenuated_color_ratio_1064nm_532nm:long_name = "attenuated color ratio 1064nm/532nm" ;
  		attenuated_color_ratio_1064nm_532nm:valid_min = 0. ;
  		attenuated_color_ratio_1064nm_532nm:valid_max = 1. ;
  		attenuated_color_ratio_1064nm_532nm:coordinates = "agl_altitude altitude altitude_time" ;
  	int64 effective_resolution(time) ;
  		effective_resolution:long_name = "effective_resolution" ;
  		effective_resolution:description = "The effective horizontal temporal resolution of each profile." ;
  		effective_resolution:units = "milliseconds" ;
  	double telescope_angle(time) ;
  		telescope_angle:_FillValue = NaN ;
  		telescope_angle:long_name = "telescope angle" ;
  		telescope_angle:units = "degree" ;
  		telescope_angle:description = "The optical telescope scan angle" ;
  		telescope_angle:dpl_py_binding = "rs_raw.scanning_telescope_rotation_angle" ;
  	int64 time(time) ;
  		time:units = "minutes since 2023-08-30 00:00:00" ;
  		time:calendar = "proleptic_gregorian" ;
  	float range(range) ;
  		range:long_name = "range" ;
  		range:dpl_py_binding = "rs_raw.range" ;
  		range:_FillValue = NaNf ;
  		range:units = "m" ;
  		range:description = "range from instrument along optical path" ;
  	double altitude(range, time) ;
  		altitude:_FillValue = NaN ;
  		altitude:long_name = "mean sea level altitude" ;
  		altitude:units = "m" ;
  	double agl_altitude(range, time) ;
  		agl_altitude:_FillValue = NaN ;
  		agl_altitude:long_name = "above ground level altitude" ;
  		agl_altitude:units = "m" ;
  	int64 altitude_time(range, time) ;
  		altitude_time:units = "minutes since 2023-08-30 00:00:00" ;
  		altitude_time:calendar = "proleptic_gregorian" ;
  } // group horizontal_stare

group: scanning {
  dimensions:
  	scan_time = 344 ;
  	angle = 21 ;
  	range = 667 ;
  	time = 172800 ;
  variables:
  	int64 scan_time(scan_time) ;
  		scan_time:units = "nanoseconds since 2023-08-30 03:03:05.885381120" ;
  		scan_time:calendar = "proleptic_gregorian" ;
  	double angle(angle) ;
  		angle:_FillValue = NaN ;
  		angle:long_name = "telescope zenith angle rotation bin centers" ;
  		angle:description = "Only positive and negative scans are angle binned, the rest are fill values." ;
  	float range(range) ;
  		range:long_name = "range" ;
  		range:dpl_py_binding = "rs_raw.range" ;
  		range:_FillValue = NaNf ;
  		range:units = "m" ;
  		range:description = "range from instrument along optical path" ;
  	double distance(angle, range) ;
  		distance:_FillValue = NaN ;
  		distance:long_name = "distance" ;
  		distance:units = "m" ;
  	double altitude(angle, range) ;
  		altitude:units = "m" ;
  		altitude:_FillValue = NaN ;
  		altitude:long_name = "mean sea level altitude" ;
  	double agl_altitude(angle, range) ;
  		agl_altitude:units = "m" ;
  		agl_altitude:_FillValue = NaN ;
  		agl_altitude:long_name = "above ground level altitude" ;
  	double temperature(scan_time, angle, range) ;
  		temperature:_FillValue = NaN ;
  		temperature:units = "K" ;
  		temperature:source = "NOAA raob" ;
  		temperature:long_name = "temperature" ;
  		temperature:coordinates = "agl_altitude altitude distance" ;
  	double pressure(scan_time, angle, range) ;
  		pressure:_FillValue = NaN ;
  		pressure:units = "hPa" ;
  		pressure:source = "NOAA raob" ;
  		pressure:long_name = "pressure" ;
  		pressure:coordinates = "agl_altitude altitude distance" ;
  	double dewpoint(scan_time, angle, range) ;
  		dewpoint:_FillValue = NaN ;
  		dewpoint:units = "K" ;
  		dewpoint:source = "NOAA raob" ;
  		dewpoint:long_name = "dewpoint" ;
  		dewpoint:coordinates = "agl_altitude altitude distance" ;
  	double particulate_linear_depolarization_532nm(scan_time, angle, range) ;
  		particulate_linear_depolarization_532nm:_FillValue = NaN ;
  		particulate_linear_depolarization_532nm:long_name = "particulate linear depolarization 532nm" ;
  		particulate_linear_depolarization_532nm:valid_min = 0. ;
  		particulate_linear_depolarization_532nm:valid_max = 0.6 ;
  		particulate_linear_depolarization_532nm:coordinates = "agl_altitude altitude distance" ;
  	double particulate_optical_depth_532nm(scan_time, angle, range) ;
  		particulate_optical_depth_532nm:_FillValue = NaN ;
  		particulate_optical_depth_532nm:long_name = "particulate optical depth 532nm" ;
  		particulate_optical_depth_532nm:valid_min = 0. ;
  		particulate_optical_depth_532nm:coordinates = "agl_altitude altitude distance" ;
  	double particulate_molecular_optical_depth_532nm(scan_time, angle, range) ;
  		particulate_molecular_optical_depth_532nm:_FillValue = NaN ;
  		particulate_molecular_optical_depth_532nm:long_name = "particulate and molecular optical depth 532nm" ;
  		particulate_molecular_optical_depth_532nm:valid_min = 0. ;
  		particulate_molecular_optical_depth_532nm:coordinates = "agl_altitude altitude distance" ;
  	double particulate_backscatter_532nm(scan_time, angle, range) ;
  		particulate_backscatter_532nm:_FillValue = NaN ;
  		particulate_backscatter_532nm:long_name = "particulate backscatter 532nm" ;
  		particulate_backscatter_532nm:valid_min = 1.e-09 ;
  		particulate_backscatter_532nm:units = "1/m 1/sr" ;
  		particulate_backscatter_532nm:coordinates = "agl_altitude altitude distance" ;
  	int64 effective_resolution(scan_time, angle) ;
  		effective_resolution:long_name = "effective_resolution" ;
  		effective_resolution:description = "The effective horizontal temporal resolution of each profile." ;
  		effective_resolution:units = "milliseconds" ;
  	double attenuated_color_ratio_1064nm_532nm(scan_time, angle, range) ;
  		attenuated_color_ratio_1064nm_532nm:_FillValue = NaN ;
  		attenuated_color_ratio_1064nm_532nm:long_name = "attenuated color ratio 1064nm/532nm" ;
  		attenuated_color_ratio_1064nm_532nm:valid_min = 0. ;
  		attenuated_color_ratio_1064nm_532nm:valid_max = 1. ;
  		attenuated_color_ratio_1064nm_532nm:coordinates = "agl_altitude altitude distance" ;
  	int64 time(time) ;
  		time:units = "milliseconds since 2023-08-30 00:00:00" ;
  		time:calendar = "proleptic_gregorian" ;
  	double telescope_angle(time) ;
  		telescope_angle:_FillValue = NaN ;
  		telescope_angle:long_name = "telescope angle" ;
  		telescope_angle:units = "degree" ;
  		telescope_angle:description = "The optical telescope scan angle" ;
  		telescope_angle:dpl_py_binding = "rs_raw.scanning_telescope_rotation_angle" ;
  		telescope_angle:coordinates = "raw_time" ;
  } // group scanning
}
```
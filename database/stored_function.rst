.. _stored_function:

================
Stored Functions
================
For making user's search more convenient, the stored functions (in other relational database, may be called
Stored Procedure) are prepared for use. Additional to standard aggregate and other types of functions which
are `originally implemented in PostgreSQL server <http://www.postgresql.org/docs/9.3/static/functions-aggregate.html>`_,
some user defined functions are available on NAOJ HSC database server for HSC SSP data.
More functions are planned to be added in near future, for example, coordinate string converter to degrees
(hh:mm:ss.s,+/-dd:mm:ss.s -> degrees), and magnitude to/from flux density converters.

Aggregate Functions
^^^^^^^^^^^^^^^^^^^
.. list-table:: **User Defined Functions(Aggregate)**

   * - **Functions**
     - **Argument Type(s)**
     - **Return Type**
     - **Description**

   * - qmedian(expression)
     - int, bigint, numeric, double precision
     - same as argument data type
     - the median of all input values

   * - quantile(expression)
     - int, bigint, numeric, double precision
     - same as argument data type
     - the quantile of all input values

   * - weighted_mean(values, weight_values)
     - set of double precision
     - double precision
     - the weighted mean of all input values

   * - mean
     - double precision
     - same as argument data type
     - the mean of all input values

   * - variance
     - double precision
     - same as argument data type
     - the (unbiased) variance of all input values

   * - stddev
     - double precision
     - same as argument data type
     - sqrt of the unbiased variance

   * - skewness
     - double precision
     - same as argument data type
     - the skewness of all input values: κ\ :sub:`3`\ /κ\ :sub:`2`\ :sup:`3/2` where κ\ :sub:`i` denotes the unbiased i-th cumulant

   * - kurtosis
     - double precision
     - same as argument data type
     - the kurtosis of all input values: κ\ :sub:`4`\ /κ\ :sub:`2`\ :sup:`2` where κ\ :sub:`i` denotes the unbiased i-th cumulant

example of qmedian::

      -- calculate median of seeing values in frame table of UDEEP CCD data

      SELECT qmedian(seeing) from ssp_s14a0_udeep_20140523a.frame;

example of quantile::

      -- calculate 30% quantile of seeing values in frame table of UDEEP CCD data

      SELECT quantile(seeing, 0.3) from ssp_s14a0_udeep_20140523a.frame;

      -- calculate quantile (30, 50 and 70%) of seeing values in frame table of UDEEP CCD data

      SELECT quantile(seeing, array[0.3, 0.5, 0.7]) from ssp_s14a0_udeep_20140523a.frame;

example of weighted_mean::

      -- calculate weighted_mean of Sinc magnitudes with weight by error of Sinc magnitudes (mag_sinc_err)^{-2}
      -- Caution!! only double precision input is allowed currently and cast to numeric is essential

      SELECT weighted_mean(mag_sinc, (1.0/mag_sinc_err)*(1.0/mag_sinc_err))
        FROM ssp_s14a0_udeep_20140523a.frame_forcephoto__deepcoadd__iselect
       WHERE tract=0 and id=207876417126408 and mag_sinc_err > 0.0;

example of mean, variance & stddev::

      -- calculate the mean of mag_sinc_err that are valid,
      -- the variance of mag_gaussian that are valid,
      -- and the standard deviation of flux_cmodel that are not NaN
      -- in the UDEEP CCD frame table

      SELECT mean(mag_sinc_err, '>', 0), variance(mag_gaussian, '<', 99.99), stddev(flux_cmodel, '==', flux_cmodel)
        FROM ssp_s14a0_udeep_20140523a.frame_forcephoto__deepcoadd__iselect
        WHERE tract=0;

example of skewness & kurtosis::

      -- calculate the skewness of mag_sinc that are valid,
      -- and the kurtosis of mag_gaussian that are valid

      SELECT skewness(mag_sinc, '<', 99.99), kurtosis(mag_gaussian, '<', 99,99),
        FROM ssp_s14a0_udeep_20140523a.frame_forcephoto__deepcoadd__iselect
        WHERE tract=0;

Function for spatial searches
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. list-table:: **User Defined Functions(Spatial Search)**

   * - **Functions**
     - **Description**

   * - f_getobj_circle(ra, dec, radius, table_name)
     - get object in circle defined by radius from the central coordinate(ra, dec) on the specified table

   * - f_getobj_rectangle(ra, dec, delta_ra, delta_dec, table_name)
     - get object in rectangle defined by delta_ra and delta_dec centered at the coordinate(ra, dec) on the specified table


example of f_getobj_circle(ra, dec, radius, table_name)::

      -- get objects' tract, patch, pointing, id, ra2000, decl2000, cx, cy, cz, xxyyzz, distance
      -- within 2 arcsec radius centered at (RA,DEC)=(150.403189,1.485288) in frame_forcelist of UDEEP CCD
      -- RA and DEC are in degrees.

      SELECT * from f_getobj_circle(150.403189, 1.485288, 2.0, 'ssp_s14a0_udeep_20140523a.frame_forcelist__deepcoadd__iselect');

      -- get object's id, ra, dec, sinc magnitudes of g,r,i,z,y bands and distance from the central coordinates specified.
      -- the cone search is for objects within 3 arcsec from the coordinate (RA,DEC)=(150.403189,1.485288).
      -- Joining the query result of cone search with photoobj_mosaic table.
      -- distance in arcsec

      SELECT pm.id, pm.ra2000, pm.decl2000, pm.gmag_sinc, pm.rmag_sinc, pm.imag_sinc, pm.zmag_sinc, pm.ymag_sinc, obj.distance
      FROM f_getobj_circle(150.93, 1.93, 3.0, 'ssp_s14a0_udeep_20140523a.photoobj_mosaic__deepcoadd__iselect') obj,
           ssp_s14a0_udeep_20140523a.photoobj_mosaic__deepcoadd__iselect pm
      WHERE obj.id=pm.id and obj.tract=pm.tract and obj.patch=pm.patch and obj.pointing = pm.pointing
      ORDER by obj.distance;

example of f_getobj_rectangle(ra, dec, delta_ra, delta_dec, table_name)::

      -- get objects' tract, patch, pointing, id, ra2000, decl2000, cx, cy, cz, xxyyzz, distance
      -- within 2 x 2 arcsec centered at (RA,DEC)=(150.403189,1.485288) in frame_forcelist of UDEEP CCD
      -- RA and DEC are in degrees.

      SELECT * from f_getobj_rectangle(150.403189, 1.485288, 2.0, 2.0, 'ssp_s14a0_udeep_20140523a.frame_forcelist__deepcoadd__iselect');


Functions for utils
^^^^^^^^^^^^^^^^^^^
Some utility functions for handling HSC information are prepared. They are (visit, ccd) <-> FrameId conversion etc.

.. list-table:: **User Defined Functions(Utils)**

   * - **Functions**
     - **Argument Type(s)**
     - **Return Type**
     - **Description**

   * - frameid2visitccd
     - text
     - set of integer
     - transform of FrameId to (visit, ccd)

   * - visitccd2frameid
     - set of integer
     - text
     - transform of (visit, ccd) to FrameId

   * - hms2deg
     - text (hh:mm:ss.sss)
     - double precision
     - transform RA in hh:mm:ss.sss to degree unit

   * - deg2hms
     - double precision
     - text (hh:mm:ss.sss)
     - transform RA in degree to hh:mm:ss.sss

   * - dms2deg
     - text (+/-dd:mm:ss.ss)
     - double precision
     - transform DEC in +/-dd:mm:ss.ss to degree unit

   * - equ2gal
     - set of double precision (ra, dec) J2000
     - set of double precision (gallon, gallat)
     - transform equatrial coordinates in degree to galactic coordinates in degree (based on SLALIB 2.5-4)

   * - gal2equ
     - set of double precision (gallon, gallat)
     - set of double precision (ra, dec) J2000
     - transform galactic coordinates in degree to equatrial coordinates in degree (based on SLALIB 2.5-4)

   * - date2mjd
     - text (date string: YYYY-MM-DD)
     - integer (mjd in integer)
     - transform date-obs to MJD

   * - datetime2mjd
     - text (datetime string: YYYY-MM-DDThh:mm:ss.sss)
     - double precision (mjd)
     - transform date-obs + UT to MJD

   * - datetime2mjd
     - set of text (date string: YYYY-MM-DD, time string hh:mm:ss.sss)
     - double precision (mjd)
     - transform date-obs + UT to MJD

   * - mjd2date
     - integer (MJD in integer)
     - text (date string: YYYY-MM-DD)
     - transform MJD to date string

   * - mjd2datetime
     - double precision (MJD)
     - text (datetime string: YYYY-MM-DDThh:mm:ss.sss)
     - transform MJD to string DATE-OBJ + UT

   * - mjd2datetime2
     - double precision (MJD)
     - set of text (date string: YYYY-MM-DD, time string: hh:mm:ss.sss)
     - transform MJD to string DATE-OBJ + UT

   * - mag2flux
     - double precision (AB magnitude)
     - double precision (flux in erg/s/cm^2/Hz)
     - transform AB magnitude to flux

   * - mag2fluxJy
     - double precision (AB magnitude)
     - double precision (flux in Jansky)
     - transform AB magnitude to flux in Jy

   * - flux2mag
     - double precision (flux in erg/s/cm^2/Hz)
     - double precision (AB magnitude)
     - transform flux to AB magnitude

   * - fluxJy2mag
     - double precision (flux in Jansky)
     - double precision (AB magnitude)
     - transform flux in Jy to AB magnitude

   * - flux_cgs2Jy
     - double precision (flux in erg/s/cm^2/Hz)
     - double precision (flux in Jansky)
     - transform flux in erg/s/cm^2/Hz to flux in Jy

   * - flux_Jy2cgs
     - double precision (flux in Jansky)
     - double precision (flux in erg/s/cm^2/Hz)
     - transform flux in Jy to flux in erg/s/cm^2/Hz

example of frameid2visitccd and  visitccd2frameid::

      SELECT frameid2visitccd('HSCA00000301');
      return (2,27)

      SELECT visitccd2frameid(2, 27);
      return 'HSCA00000301'

example of hms2deg and dms2deg::

      SELECT hms2deg('12:12:12.345');
        return 183.0514375

      SELECT dms2deg('-01:00:12.00');
        return -1.00333333333333

example of deg2hms and deg2dms::

      SELECT deg2hms(183.051416666667);
        return 12:12:12.34

      SELECT deg2dms(83.0514375);
        return +83:03:05.18

example for getting coordinates with hh:mm:ss.sss and +/-dd:mm:ss.ss rather than degree::

      SELECT deg2hms(ra2000) as ra, deg2dms(decl2000) as dec
      FROM ssp_s14a0_udeep_20140523a.photoobj_mosaic__deepcoadd__iselect
      LIMIT 10;

example for getting objects' id, coordinates in degree and hms/dms formats which are within 20 arcsec from the center at
(RA, DEC) = (10:03:45.000, +02:00:00.00) in photoobj_mosaic table of UDEEP survey. ::

      SELECT id, ra2000, decl2000, deg2hms(ra2000) as ra, deg2dms(decl2000) as dec
      FROM f_getobj_circle(hms2deg('10:03:45.000'), dms2deg('+02:00:00.00'), 20.0, 'ssp_s14a0_udeep_20140523a.photoobj_mosaic__deepcoadd__iselect');

example of equ2gal and gal2equ::

      SELECT equ2gal(120.0, 30.0);

      SELECT gal2equ(230.0, 20.0);

      -- get the objects' id, ra, dec and galactic coordinates which are within 20 arcsec from the center at
      -- (RA, DEC) = (10:03:45.000, +02:00:00.00) in photoobj_mosaic table of UDEEP survey.

      SELECT pm.id, pm.ra2000, pm.decl2000, e2g.l as gallon, e2g.b as gallat
      FROM
         f_getobj_circle(hms2deg('10:03:45.000'), dms2deg('+02:00:00.00'), 20.0, 'ssp_s14a0_udeep_20140523a.photoobj_mosaic__deepcoadd__iselect') pm,
         equ2gal(pm.ra2000, pm.decl2000) e2g
      ;


example of date2mjd and mjd2date::

      SELECT date2mjd('2014-07-17');

      SELECT mjd2date(56855);

example of datetime2mjd and mjd2datetime, mjd2datetime2::

      SELECT datetime2mjd('2014-07-17T12:12:12.000');

      SELECT datetime2mjd('2014-07-17', '12:12:12.000');

      SELECT mjd2datetime(56855.5084722222);

      SELECT mjd2datetime2(56855.5084722222);

example of mag2flux and flux2mag::

      -- Compare flux/magnitude in DB with those computed by the conversion functions
      SELECT iflux_sinc, mag2flux(imag_sinc), imag_sinc, flux2mag(iflux_sinc)
      FROM ssp_s14a0_wide_20140523a.photoobj_mosaic__deepcoadd__iselect
      LIMIT 10;

example of flux_Jy2cgs and fluxJy2mag::

      -- Select those objects that are brighter than 1e-6 Jy
      SELECT iflux_sinc
      FROM ssp_s14a0_wide_20140523a.photoobj_mosaic__deepcoadd__iselect
      WHERE iflux_sinc > flux_Jy2cgs(1e-6)
      LIMIT 10;

      -- or,
      SELECT imag_sinc
      FROM ssp_s14a0_wide_20140523a.photoobj_mosaic__deepcoadd__iselect
      WHERE imag_sinc < fluxJy2mag(1e-6)
      LIMIT 10;

Functions for using WCS
^^^^^^^^^^^^^^^^^^^^^^^^
Conversion of sky to pixel coordinate and vice versa is available by using database only.
Currently these functions use the table 'wcs', which is based on wcs*.fits coming from mosaicking,
therefore, applicable only for CCD sources with mosaic-calibrated.

.. list-table:: **User Defined Functions(WCS related)**

   * - **Functions**
     - **Argument Type(s)**
     - **Return Type**
     - **Description**

   * - sky2pix
     - set of double precision, text and integer (ra, dec, schema, tract, frame-id) [ra and dec in degree]
     - set of double precision (x, y)
     - convert sky to pixel coordinate on spicified image data

   * - pix2sky
     - set of double precision, text and integer (x, y, schema, tract, frame-id)
     - set of double precision (ra, dec) in degree
     - convert pixel to sky coordinate on spicified image data

   * - shape_sky2pix
     - set of double precision, text and integer (shape_array, ra, dec, schema, tract, frame-id) [shape_array (I_xx, I_yy, I_xy), ra and dec in degree]
     - array of double precision (Is_xx, Is_yy, Is_xy) in pixel^2
     - convert shape params (in arcsec^2) to pixel-coord based one

   * - shape_pix2sky
     - set of double precision, text and integer (shape_array, x, y, schema, tract, frame-id)[shape_array (Is_xx, Is_yy, Is_xy)]
     - array of double precision (I_xx, I_yy, I_xy) in arcsec^2
     - convert shape params (in pixel^2) to sky-coord based one

   * - shape_err_sky2pix
     - set of double precision, text and integer (err_array, ra, dec, schema, tract, frame-id) [err_array is covariance of (I_xx, I_yy, I_xy) arranged as (xx-xx, xx-yy, yy-yy, xx-xy, yy-xy, xy-xy), ra and dec in degree]
     - array of double precision (xx-xx, xx-yy, yy-yy, xx-xy, yy-xy, xy-xy) in pixel^4 that is the covariance of (Is_xx, Is_yy, Is_xy)
     - convert the covariance of shape params (in arcsec^4) to pixel-coord based one

   * - shape_err_pix2sky
     - set of double precision, text and integer (err_array, x, y, schema, tract, frame-id)[err_array is covariance of (Is_xx, Is_yy, Is_xy) arranged as (xx-xx, xx-yy, yy-yy, xx-xy, yy-xy, xy-xy)]
     - array of double precision (xx-xx, xx-yy, yy-yy, xx-xy, yy-xy, xy-xy) in arcsec^4 that is the covariance of (I_xx, I_yy, I_xy)
     - convert the covariance of shape params (in pixel^4) to sky-coord based one

   * - f_enum_frames_containing
     - set of double precision and text (ra2000, decl2000, schema) [ra,decl in degree]
     - set of text, integer and double precision (frame_id, tract, x, y) [x,y in pixel coord]
     - get all frames' id in which really contain the specified coordinate in them

   * - f_enum_mosaics_containing
     - set of double precision and text (ra2000, decl2000, schema) [ra,decl in degree]
     - set of text, integer and double precision (frame_id, tract, x, y) [x,y in pixel coord]
     - get all coadds' id in which really contain the specified coordinate in them

example of sky2pix and pix2sky::

      -- get (x, y) coordinate of (RA,DEC)=(150.5 deg, 1.5 deg)
      -- on image data with tract is 0 and frame_id 'HSCA00188753'

      SELECT sky2pix(150.5, 1.5,'ssp_s14a0_udeep_20140523a', 0, 'HSCA00188753');

      -- get (ra, dec) coordinate of (x,y)=(1750.325,359.630)
      -- on image data with tract is 0 and frame_id 'HSCA00188753'

      SELECT pix2sky(1750.325,359.630,'ssp_s14a0_udeep_20140523a', 0, 'HSCA00188753');


example of shape_sky2pix and shape_pix2sky::

      SELECT shape_pix2sky(shape_sdss, centroid_sdss_x, centroid_sdss_y, 'ssp_s14a0_wide_20140523a', tract, frame_id)
      FROM ssp_s14a0_wide_20140523a.frame_forcelist__deepcoadd__iselect
      limit 10;

      SELECT shape_sdss, shape_sky2pix(shape_pix2sky(shape_sdss, centroid_sdss_x, centroid_sdss_y, 'ssp_s14a0_wide_20140523a', tract, frame_id),  ra2000, decl2000, 'ssp_s14a0_wide_20140523a', tract, frame_id)
      FROM ssp_s14a0_wide_20140523a.frame_forcelist__deepcoadd__iselect
      limit 10;

example of f_enum_frames_containing::

      --- get frame_ids (CCD's id) which really include the point with coord of (RA,DEC)=(150.0,2.0)
      --- in UDEEP data
      select f_enum_frames_containing(150.0, 2.0, 'ssp_s14a0_udeep_20140523a')

      select frame_id, tract, x, y from f_enum_frames_containing(150.0, 2.0, 'ssp_s14a0_udeep_20140523a');

      ---
      --- The procedure is doint the following processes.
      ---
      --- 1. First we obtain the healpix index corresponding to (ra, dec).
      --- 2. We then look into "frame_hpx11" table to obtain a list of frames
      ---    that possess the healpix.
      --- 3. Then we join "frame" table and "wcs" table with the condition
      ---    that the "frame_id" is contained in the list we've got in (2).
      --- 4. Finally, we select from the joined table those records
      ---    whose WCSs transform (ra, dec) to pixel coord within themselves.

example of f_enum_mosaics_containing::

      --- get coadds' ids (tract, patch, filter) which really include the point with coord of (RA,DEC)=(150.0,2.0)
      --- in UDEEP data

      select f_enum_mosaics_containing(150.0, 2.0, 'ssp_s14a0_udeep_20140523a');

      select tract, patch, filter01, x, y from f_enum_mosaics_containing(150.0, 2.0, 'ssp_s14a0_udeep_20140523a');


Setting Stored Functions in your own Database
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If you want to run the stored functions in your own database servers, you should
do install and set-up the functions.
All functions described in this document available in the latest **hscDb** package
(version later than 2014.07.04), under 'python/hsc/hscDb/pgfunctions' directories.

For C and C++ functions, you should run Makefile first, then do make install as
root user, then run 'create extension [function_name]' in psql command. ::

     # For example on qmedian

     % cd pgfunctions/c/qmedian
     % make
     % su <-- switch to root user
     % make install

     % /usr/local/pgsql/bin/psql -U hscana -d dr_early -h your_db_host

     psql> create extension qmedian;

Please see README file in each package directory.

For PL/pgSQL functions, you should run all SQL scripts under the plpgsql directory.::

     % /usr/local/pgsql/bin/psql -U hscana -d dr_early -h your_db_host -f f_arcsec2radian.sql
     % /usr/local/pgsql/bin/psql -U hscana -d dr_early -h your_db_host -f ......
     % ..........................

**Caution**

As the stored functions are set up to each database instance, you should run 'create extension'
or 'create function' command when you newly create the database instance with 'create database'
or createdb command.

You can see the set-up functions by using '**\\df**' command on psql prompt.


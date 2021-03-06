## Integer Bulkdata Storage Details

The EXODUS database can store integer bulk data, entity map data, and
mesh entity (block/set) ids in either 32-bit or 64-bit integer format. The data
considered "bulk data" are:

 * element, face, and edge connectivity lists,
 * element, face, edge, and node set entity lists,

The entity map data is any data stored in one of the 'map' objects on
the exodus file.  This includes:

 * id maps
 * number maps
 * order maps
 * processor node maps
 * processor element maps.

A mesh entity id is the id of any block (element block, edge block,
...); set (node set, face set, ...), coordinate frame, and
communication map.


When an EXODUS file is created via the `ex_create()` function, the
'mode' argument provides the method for specifying how integer data
will be passed as arguments to the API functions and also how the
integer data will be stored on the database. The `ex_open()` function
provides the same method for specifying how integer data will be
passed as arguments to the API functions.

The method uses the 'mode' argument to the `ex_open()` and
`ex_create()` functions. The mode is a 32-bit integer in which certain
bits are turned on by or'ing certain predefined constants.
 
```
exoid = ex_create( EX_TEST_FILENAME,
		   EX_CLOBBER|EX_MAPS_INT64_DB|EX_MAPS_INT64_API,
		   &appWordSize, &diskWordSize );
```

The constants related to the integer size (32-bit or 64-bit) specification are:

 Constant           | Description
 -------------------|--------------
`EX_MAPS_INT64_DB`  | entity map data
`EX_IDS_INT64_DB`   | mesh entity ids
`EX_BULK_INT64_DB`  | bulk data
`EX_ALL_INT64_DB`   | (the above 3 or'd together)
`EX_MAPS_INT64_API` | entity map data
`EX_IDS_INT64_API`  | mesh entity ids
`EX_BULK_INT64_API` | bulk data
`EX_ALL_INT64_API`  | (the above 3 or'd together)

The constants that end with `_DB` specify that that particular integer
data is stored on the database as 64-bit integers; the constants that
end with `_API` specify that that particular integer data is passed
to/from API functions as 64-bit integers.

If the range of the data being transmitted is larger than the
permitted integer range (for example, if the data is stored on the
database truly requires 64-bit ints and the application specifies
passing data as 32-bit ints), the api function will return an error.

The three types of integer data whose storage can be specified are:

* maps (`EX_MAPS_INT64_`),
* "bulk data" including connectivity lists and entity lists
  (`EX_BULK_INT64_`), and
* entity ids which are the ids of element, face, edge, and node
  sets and blocks; and map ids (`EX_IDS_INT64_`)

The function `ex_int64_status(exoid)` is used to determine the integer
storage types being used for the EXODUS database 'exoid'. The function
returns an integer which can be and'ed with the above flags to
determine either the storage type or function parameter type. 

For example, if 

```
	(EX_MAPS_INT64_DB & ex_int64_status(exoid))
```
is true, then map data is being stored as 64-bit integers for that database.

It is not possible to determine the integer data size on a database
without opening the database via an `ex_open()` call. However, the
integer size specification for API functions can be changed at any
time via the `ex_set_int64_status(exoid, mode)` function. The mode is
one or more of `EX_MAPS_INT64_API`, `EX_IDS_INT64_API`, or
`EX_BULK_INT64_API`, or'd together.  Any exodus function calls after
that point will use the specified integer size. Note that a call to
`ex_set_int64_status(exoid, mode)` overrides any previous setting for
the integer sizes used in the API.  The `ex_create()` function is the
only way to specify the size of the integers that will be written to a database.


## IOSS: IO System libraries

The IO System (Ioss) is also 64-bit integer capable.  When Ioss opens
and reads the intial metadata from an exodus file, it will determine
the size of integers stored on the database. If either 'maps' or 'bulk
data' are stored as 64-bit integers, then Ioss will automatically
specify that all integers will be transferred as 64-bit integers.
Entity (block/set) ids are always treated as 64-bit integers in Ioss. 

The setting can be queried via the *Ioss::DatabaseIO*
`int_byte_size_api()` member function call which will return either 4 or
8. The setting can be modified via the `set_int_byte_size_api(mode)`
function call where mode is one of:

  * `Ioss::USE_INT64_API` or
  * `Ioss::USE_INT32_API`.

The integer byte size setting should only be changed prior to assiging
the database to an *Ioss::Region* since fields are created using the
setting when the meta data of the database is read.  An *Ioss::Field*
that stores 64-bit integers will have a type of `Ioss::Field::INT64`
and a field that stores 32-bit integers will have a type of
`Ioss::Field::INTEGER` (`Ioss::Field::INT32` can also be used).

The Ioss io_shell application will maintain the integer size of the
input database to the output database by default.  If the `--64`
option is specified, then the output database will use 64-bit integers
for all quantities.


## SEACAS Applications

Many of the SEACAS applications have been modified to be 64-bit
capable. They include:

  * conjoin
  * ejoin
  * epu
  * exodiff
  * nem_spread
  * nem_slice -- Note that only the linear, random, or scattered
                 decomposition options are allowed for 64-bit integer
		 meshes. Other options are being investigated for the
		 slicing of large meshes.

  * algebra, blot, exotxt, gen3d, genshell, gjoin, grepos, explore,
    mapvar, numbers, txtexo
   
## AVAILABILITY:

The 64-bit integer capability is currently available in the following
locations:

 * SEACAS repository https://github.com/gsjaardema/seacas.git
 * Sierra repository (Internal)
 * Trilinos repository https://github.com/trilinos/Trilinos

## Other Details:

* Exodus requires netcdf-4.1.2 or later; netcdf-4.3.3.1 or later is recommended

* The Nemesis library has been absorbed into the exodus library.
  * All api functions that were `ne_*` are now `ex_*`
  * A wrapper exists in the old nemesis library that will forward all
     `ne_*` calls to the corresponding `ex_*` call to maintain backward 
     compatability with unconverted applications. 
  * In other words, we recommend using the exodus library for nemesis
     routines, but the nemesis library can be used if you want.

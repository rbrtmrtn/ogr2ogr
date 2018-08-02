[![Build Status](https://jenkins.adc4gis.com/buildStatus/icon?job=ogr2ogr)](https://jenkins.adc4gis.com/job/ogr2ogr/) [![NPM](https://img.shields.io/npm/v/ogr2ogr.svg)](https://npmjs.com/package/ogr2ogr) ![NPM Downloads](https://img.shields.io/npm/dt/ogr2ogr.svg) [![code-style](https://img.shields.io/badge/code%20style-adc-brightgreen.svg?style=flat)](https://github.com/applieddataconsultants/eslint-config-adc)

ogr2ogr enables spatial file conversion and reprojection of spatial data through the use of ogr2ogr (gdal) tool

# Requirements

ogr2ogr requires the command line tool _ogr2ogr_ - [gdal install page](http://trac.osgeo.org/gdal/wiki/DownloadingGdalBinaries). It is recommended to use the latest version.

# Installation

```
npm install ogr2ogr
```

# Usage

ogr2ogr takes either a path, a stream, or a GeoJSON object. The result of the transformation can be consumed via callback or stream:

```javascript
var ogr2ogr = require('ogr2ogr')
var ogr = ogr2ogr('/path/to/spatial/file')

ogr.exec(function (er, data) {
  if (er) console.error(er)
  console.log(data)
})

var ogr2 = ogr2ogr('/path/to/another/spatial/file')
ogr2.stream().pipe(writeStream)
```

See `/examples` for usage examples and `/test/api.js`.

# Formats

The goal is for ogr2ogr to support most (if not all) formats your underlying ogr2ogr supports. You can see the progress of that in `/tests/drivers.js`.

It also will:

1. Extract zip files for formats that are typically bundled (i.e. shapefiles, kmz, s57, vrt, etc)
2. Will extract geometry from CSVs when a common geometry field can be determined.
3. Cleans up after its messes.
4. Bundles multi-file conversions as a zip
5. Support GeoJSON and GeoRSS urls as path inputs
6. Support raw GeoJSON objects as input

# Options

ogr2ogr takes chainable modifier functions:

```javascript
var shapefile = ogr2ogr('/path/to/spatial/file.geojson')
                    .format('ESRI Shapefile')
                    .skipfailures()
                    .stream()
shapefile.pipe(fs.createWriteStream('/shapefile.zip'))
```

Available options include:

- `.project(dest, src)` - reproject data (defaults to: "ESPG:4326")
- `.format(fmt)` - set output format (defaults to: "GeoJSON")
- `.timeout(ms)` - milliseconds before ogr2ogr is killed (defaults to: 15000)
- `.skipfailures()` - skip failures (continue after failure, skipping failed feature -- by default failures are not skipped)
- `.options(arr)` - array of custom org2ogr arguments (e.g. `['-fieldmap', '2,-1,4']`)
- `.destination(str)` - ogr2ogr destination (directly tell ogr2ogr where the output should go, useful for writing to databases)
- `.onStderr(callback)` - execute a callback function whose parameter is the debug output of ogr2ogr to stderr

## Example of onStderr usage

If you want to debug what is the ogr2ogr binary doing internally, you can attach a callback to the output, 
provided you have passed the option [CPL_DEBUG](https://trac.osgeo.org/gdal/wiki/ConfigOptions#CPL_DEBUG)

```javascript
var shapefile = ogr2ogr('/path/to/spatial/file.geojson')
                    .format('ESRI Shapefile')
                    .skipfailures()
                    .options(["--config", "CPL_DEBUG", "ON"])
			        .onStderr(function(data) {
			            console.log(data);
			        })
                    .stream()
shapefile.pipe(fs.createWriteStream('/shapefile.zip'))
```

You will see in the console someting in the likes of

```sh
GDAL: GDALOpen(/tmp/ogr_542cb61092c/sample.shp, this=0x15ca370) succeeds as ESRI Shapefile.

GDAL: GDALDriver::Create(PGDUMP,/vsistdout/,0,0,0,Unknown,(nil))
PGDump: LaunderName('ID') -> 'id'
PGDump: LaunderName('FIPSSTCO') -> 'fipsstco'
PGDump: LaunderName('STATE') -> 'state'
PGDump: LaunderName('COUNTY') -> 'county'
GDALVectorTranslate: 1 features written in layer 'sample'
Shape: 1 features read on layer 'sample'.
GDAL: GDALClose(/tmp/ogr_542cb61092c/sample.shp, this=0x15ca370)
GDAL: GDALClose(/vsistdout/, this=0x157ded0)
```

This can be useful when something goes wrong and the error provided by this library doesn't provide enough information.

## Conversion of `shp` files

It is trivial to handle the conversion of ESRI Shapefiles when they are packed in a zipfile that contains (at least) the `shp` and `shx` files.
This library is also capable of converting uncompresses ESRI Shapefiles if you use the `shp` file as the input file 
**and the shx file is in the same folder**.

However, it is also possible to convert single `shp` files that lack an `shx` file by forcing its creation 
using ogr2ogr option [SHAPE_RESTORE_SHX](https://trac.osgeo.org/gdal/wiki/ConfigOptions#SHAPE_RESTORE_SHX) provided you have installed
GDAL/OGR version 2.1.0 or newer.

```javascript
var geojson = ogr2ogr('/path/to/spatial/lonely.shp')
    .options(["--config", "SHAPE_RESTORE_SHX", "TRUE"])
    .stream()

geojson.pipe(fs.createWriteStream('/lonely.json'))

```

**Caveat**: ogr2ogr will do its best to infer the corresponding `shx`. However, there's no guarantee it will success.




# License

(The MIT License)

Copyright (c) 2017 Marc Harter <wavded@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

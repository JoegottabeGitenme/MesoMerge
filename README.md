# MesoMerge

workflow to ingest and expose merged GOES imagery

## The problem
Currently satellite mosaics are created by stitching together images in the database, which in this case would lead
to significant bloat. Users typically are not able to view floater sections alongside the rest of the fullscanned images
, so weather context is lost when transitioning between products. A user certainly could add both the 'fullscan' 
and 'floater' sections on the same map using WMS, but there may be muddling in the floater section due to the layers
stacking on top of eachother
## The solution
Create a middleware layer which will detect tile bounds, and stitch together appropriate tiles, creating a cohesive 
mosaic as requests are served. A new WMS layer will be exposed alongside the rest of the WMS server's offerings.

## Some definitions
GOES - a series of satellites in space, taking pictures of the earth
WMS - web service to fetch images, in this case tiles to be displayed on a slippy tile map
fullscan - the fullscan image of the earth taken from a GOES satellite, typically a full scan every 15 minutes
mesoscale/floater - smaller subsection of the earth taken by the GOES satellites, typically ~1min increments

##How it works
When a wms tile request is made, the middleware will compare the requested tiles bounds to the bounds of the configured
mesoscale 'floater' sections, if the floater section is on top of the fullscan image, we stitch the floater on
top of the old data for that tile. This should create a complete layer where the floater image is displayed on top
of the rest of the data.

Bounding boxes for the floater sections will be calculated separately, and can be optionally requested through 
a WFS (or similar) service for viewing on the front end

Time steps exposed to the front end will be at the most granular resolution (typically ~1min for floater sections)
and the background fullscan images will be retrieved from the next timestep down. This will end up creating a bit of
a time-lagged effect between the fullscan and floater products, and is expected in this case, at most we should only
see a ~15 minute discrepancy

Data will be ingested using a small utility, fed into our WMS server, where our middleware will intercept requests and
modify as necessary

For each tile request that is made, the bounding area will be checked against an internal structure to detect where a 
particular tile will need to be manipulated. Manipulation will simply be stitching the most recent on top of the older
image (in most cases the floater will overwrite the fullscan, but there may be edge cases, and this could eventually
be extended to support multiple floaters, or even multiple satellites)

Floater sections may move over time (hurricane tracking) so bboxes will have a time dimension

The capabilities document will be manipulated on the fly ?

The middleware will pass through all 'non-merged' WMS requests, so it should stay decoupled from the front/back ends


## TODO
find data (ftp, s3, etc. Need fullscan and 1 floater to start)

spin up geoserver (or other WMS)

ingest data to WMS (need fullscan and 1 floater to start)

create WMS endpoint

create middleware passthrough

calculate floater bbox for each timestep

middleware catches 'mosaic' request

create front end viewer/container (standard WMS offering plus 'mosaic' product hardcoded)

update capabilities document to include mosaic product

containerize appropriate pieces

end product is one middleware service
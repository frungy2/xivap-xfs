# x-IvAp for X-flight server
A fork of the x-ivap client (https://sourceforge.net/projects/x-ivap/). Intended for usage with X-FlightServer.net only.

# Working with the repo
As the libxplanemp is stored as a submodule, it is vital to run `git clone --recursive` and/or `git submodule update` when getting the latest code

## FMOD
Due to the license of FMOD, no headers/libraries are included in this repo for FMOD. You will need to grab these from somewhere else.

## LIBPNG/ZLIB
It is assumed that libpng and zlib are available and already installed in the system

# Known issues
* As the build is currently configured, only X-Plane 11.10+ is supported
* Mac builds are likely going to fail


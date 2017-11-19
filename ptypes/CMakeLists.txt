# -*- cmake -*-
# written by Darren Ha(darren.ha@samsung.com)

# replace PTypes_Lib_ST with binary name. e.g. mbase or FBase
SET (this_target ptypes)
#PROJECT(${this_target} LANGUAGES C CXX)

## section: include directory

INCLUDE_DIRECTORIES(
  include
  )

## section: source files
# Add your source files here (one file per line), please SORT in alphabetical order for future maintenance
SET (${this_target}_SOURCE_FILES
	src/pasync.cxx
	src/patomic.cxx
	src/pcomponent.cxx
	src/pcset.cxx
	src/pcsetdbg.cxx
	src/pexcept.cxx
	src/pfatal.cxx
	src/pfdxstm.cxx
	src/pinfile.cxx
	src/pinfilter.cxx
	src/pinmem.cxx
	src/pinstm.cxx
	src/pintee.cxx
	src/piobase.cxx
	src/pipbase.cxx
	src/pipmsg.cxx
	src/pipmsgsv.cxx
	src/pipstm.cxx
	src/pipstmsv.cxx
	src/pipsvbase.cxx
	src/pmd5.cxx
	src/pmem.cxx
	src/pmsgq.cxx
	src/pmtxtable.cxx
	src/pnpipe.cxx
	src/pnpserver.cxx
	src/pobjlist.cxx
	src/poutfile.cxx
	src/poutfilter.cxx
	src/poutmem.cxx
	src/poutstm.cxx
	src/ppipe.cxx
	src/ppodlist.cxx
	src/pputf.cxx
	src/prwlock.cxx
	src/psemaphore.cxx
	src/pstdio.cxx
	src/pstrcase.cxx
	src/pstrconv.cxx
	src/pstring.cxx
	src/pstrlist.cxx
	src/pstrmanip.cxx
	src/pstrtoi.cxx
	src/pstrutils.cxx
	src/ptextmap.cxx
	src/pthread.cxx
	src/ptime.cxx
	src/ptimedsem.cxx
	src/ptrigger.cxx
	src/ptypes_test.cxx
	src/punit.cxx
	src/punknown.cxx
	src/pvariant.cxx
	src/pversion.cxx
	)

## section: header files
# Add your header files here(one file per line), please SORT in alphabetical order for future maintenance!
SET(${this_target}_HEADER_FILES
	include/pasync.h
	include/pinet.h
	include/pport.h
	include/pstreams.h
	include/ptime.h
	include/ptypes.h
	)

SET_SOURCE_FILES_PROPERTIES(${this_target}_HEADER_FILES
                            PROPERTIES HEADER_FILE_ONLY TRUE)
LIST(APPEND ${this_target}_SOURCE_FILES ${${this_target}_HEADER_FILES})

## section: add definitions
# 	add prefix -D. example> -DSHP
#  - DO NOT add  the following definitions(already defined in ${OSP_DEFINITIONS}:
# 	-DSHP, -DWIN32, -D_WINDOWS, -D_DEBUG, -D_USRDLL, -D_CRT_SECURE_NO_DEPRECATE
ADD_DEFINITIONS(
	-D_CRT_SECURE_NO_WARNINGS -DPTYPES_ST
	)

## section: add target
ADD_LIBRARY (${this_target} OBJECT ${${this_target}_SOURCE_FILES} )
set_property(TARGET ${this_target} PROPERTY POSITION_INDEPENDENT_CODE ON)

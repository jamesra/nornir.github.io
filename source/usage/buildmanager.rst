=======================
Using the build manager
=======================

Importing data
--------------

The first command for any Nornir volume:

``nornir-build -volume <volumepath> -input <inputpath>``

For example `nornir-build -volume Y:/Volumes/RC2 -input Y:/Captures/RC2/` will instruct Nornir to check the <i>Y:/Captures/RC2/</i> directory for sections and import any that it can find into the volume stored in <i>Y:/Volumes/RC2</i>

The import command extends volumes non-destructively provided different sections are not assigned the same number. Import can be run repeatedly as now sections become available.

Organizing data for import
==========================

Nornir needs the data to be laid out in a predictable fashion.  For random reasons <b>no spaces are allowed in nornir folder names!</b>. 

The convention nornir uses is for a directory to exist for the volume with a numbered subdirectory for each mosaic.  The directory name should contain the section number, and optionally an underscore followed by the name of the section.  

``<experimentfolder>/<section#>_[sectionname]``

Sometimes it is just useful to organize multiple mosaics into a single volume even though they aren't a 3D volume.  In this case just invent different section numbers for each capture. For example lets say I capture three mosaics in SerialEM (which produces an .idoc meta-data file) for Bob's eggplant and he had some ID #'s he wants to make sure get carried along with the data.
 
* Bob

  * 0001_EggID4567

    * capture.idoc
    * 001.tif
    * 002.tif
    * ...
    * 1245.tif

  * 0002_EggID7645

    * ...

The result will two sections named "EggID4567" and "EggID7645" respectively. 

Notes
=====

Often we want to carry more data forward.  The existing importers extract a lot of scope settings from the meta-data files. If you want to record additional data, for example the box Bob's samples are stored in, this can be put into a notes.txt file and saved in the 0001_EggID4567.  The file will copied along with the section.

* Bob

  * 0001_EggID4567

    * capture.idoc
    * ...
    * Notes.txt

  * 0002_EggID7645

Extending the importer
======================

Nornir has an .xml file mapping extensions to importers.  It examines each directory for extensions and if the extensions are found it will attempt to load the python module specified in the xml file and pass it the directory containing the files with matching extensions. 


Pipelines
---------

Nornir defines different pipelines that manipulate the volume.  Pipelines are defined in a pipeline.xml file and can be edited by advanced users.

Builds tend to have many steps in common.  The Marc lab builds TEM images with this pipeline:

1. Prune
#. AdjustContrast
#. Mosaic
#. Assemble
#. MosaicReport
#. CreateBlobFilter
#. AlignSections -Downsample 32
#. RefineSectionAlignment -Downsample 32
#. RefineSectionAlignment -Downsample 32 16
#. SliceToVolume
#. ScaleVolumeTransform 16 -> 1 

Sample batch files can be found in the scripts directory, which is copied to the python scripts directory at setup time.  TEMPrepare, TEMBuildMosaic, and TEMAlign show the commands used to build the RC2 TEM volume at the Marc lab.  
    
.. automodule:: nornir_buildmanager 
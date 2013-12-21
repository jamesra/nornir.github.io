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

Nornir defines different pipelines that manipulate the volume.  Pipelines are defined in a pipeline.xml file and can be edited by advanced users.  See the [pipeline page](wiki/pipelines) for full details. 

Builds tend to have many steps in common.  Below is an outline for the Marc lab process.

1. Remove blank tiles
#. Adjust the contrast
#. Align the tiles to build a section mosaic
#. Construct transforms aligning sections to adjacent sections
#. Add transforms to place sections into the volume

A sample implementation can be found in the scripts directory, which is copied to the python scripts directory at setup time.  TEMPrepare, TEMBuildMosaic, and TEMAlign show the commands used to build the RC2 TEM volume at the Marc lab.  

The existing pipelines are likely to be refactored to be more suitable for general use soon.   However they are outlined below.  Running ``nornir_build`` with no arguments will provide a list of all pipelines and their parameters.

1. TEMPrepare: (due for refactoring into prune and histogram pipelines) 
     Calculates a feature score for each tile in a filter.  Tiles falling below a threshold are removed from the mosaic.  Histograms are calculated for the filter.

#. AdjustContrast:
    Creates a new filter based on an existing filter. 

#. Mosaic:
    Aligns tiles into mosaics.  Assembles mosaic images of a requested size.

#. Assemble:
    Creates assembled images for a filter.  Copies assembled images to an arbitrary output directory.

#. EmailReport:
    Create a webpage summarizing key attributes of sections in a volume and send the link in E-mail.

#. EmailStosReport:
    Create a webpage summarizing key attributes of slice-to-volume registrations and send the link in E-mail.

#. CreateBlobFilter:
    SliceToSlice alignment is sometimes aided by running a form of edge detection on the data.  This creates a "blob" filter for in input filter which can then be passed to slice-to-slice alignment pipelines

#. AlignSections:
    Aligns mosaics to immediate neighbors using rotation and translation

#. RefineSectionAlignment:
    Deforms sections aligned with rotation and translation by aligning small local regions within the images.  Produces much more precise alignments.

#. SliceToVolume:
    Takes a set of slice-to-slice transforms and adds them together to produce slice-to-volume transforms

#. ScaleVolumeTransforms:
    Mosaics are often too large to align at full resolution.  This pipeline scales the resulting transforms.  For example we may align using downsampled by 16 images and then scale the transforms by a factor of 16 to work with the full-resolution images again.

#. VolumeImage:
    Produces an image file for a section after being warped into the volume.

#. MosaicToVolume:
    Passes the mosaic transforms defining how tiles are placed in mosaics through the transform mapping those mosaics into the volume.  The result is mosaic transforms that map tiles directly into the volume.

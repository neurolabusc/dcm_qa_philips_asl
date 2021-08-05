## About

`dcm_qa_philips_asl` is a simple DICOM to NIfTI validator script and dataset. This repository is similar to [dcm_qa](https://github.com/neurolabusc/dcm_qa), but includes data from Philips MRI scanners. Specifically, this dataset uses [Arterial Spin Labeling (ASL)](https://crnl.readthedocs.io/asl/index.html). For ASL data from a Siemens scanner see [dcm_qa_asl](https://github.com/neurolabusc/dcm_qa_asl).

## Details

Data provided by [Paul S. Morgan at the University of Nottigham](https://www.nottingham.ac.uk/research/beacons-of-excellence/precision-imaging/our-experts/paul-morgan/index.aspx). Data was saved as both classic (where each slice is a separate file) as well as enhanced DICOM (where an entire series of images is saved as a single file) format. Each ASL series is saved as both a volume that shows the average difference between the label and control images (e.g. series 201) and the raw data for that series (e.g. series 202).

* Common Parameters 
  * Manufacturer: Philips
  * Model: Ingenia Elition X
  * Implementation Version: Philips MR 57.0
  * Software Versions: 5.7.1.2

* `20x_pCASL`
  * Technique: 2D Pseudo-Continuous Arterial Spin Labeling (pCASL)
  * Volumes: 60 (30 repeats: half label, half control) 
  * Post Label Delay (ms):	1800

* `30x_ASL_MultiPhase`
  * Technique: STAR Pulsed Arterial Spin Labeling (PASL)
  * Volumes: 480 (8 phases, 30 repeats: half label, half control) 
  
* `40x_3D_pCASL_6mm`
  * Technique: 3D Pseudo-Continuous Arterial Spin Labeling (pCASL)
  * Volumes: 16 (8 repeats: half label, half control)
  * Post Label Delay (ms):	2000

## Missing Information

The [BIDS BEP005](https://bids.neuroimaging.io/get_involved) provides recommended and required sequence parameters for analysis of Arterial Spin Labeling (ASL). The Philips DICOM headers do not provide many of these values. Therefore, one should carefully document sequence settings when setting up Philips ASL scans.

## Unexpected Information

There are a number of quirks in the data. These features do not seem to fit a simple interpretation of the DICOM standard. Therefore, to allow automatic processing of these datasets, dcm2niix must make a few assumptions. This may make dcm2niix somewhat fragile for this modality.

The enhanced DICOMs list the difference images (series 201, 301, 401) as having a `REAL` [Complex Image Component (0008,9208)](https://dicom.innolitics.com/ciods/enhanced-mr-image/enhanced-mr-image/00089208) but the classic scans do not.

Philips ASL scans seem to suggest a [cardiac gated trigger](https://github.com/rordenlab/dcm2niix/issues/408). For example, consider the enhanced DICOM for series 301 and 302:

```
(0018,9037) CS [PROSPECTIVE]  #  12, 1 CardiacSynchTechnique
(2001,1010) CS [TRIGGERED]    #  10, 1 Unknown Tag & Data
(0018,1060) DS [300]          #   4, 1 TriggerTime
(0018,9085) CS [VCG]          #   4, 1 CardiacSignalSource
```
To deal with this, dcm2niix will ignore the reported trigger time if the [Acquisition Contrast (0008,9209)](http://dicomlookup.com/lookup.asp?sw=Tnumber&q=(0008,9209)) tag reports `PERFUSION`.


## Ordering of Volumes
 
 dcm2niix will convert raw Philips ASL data as a single 4D NIfTI image. The 3D volumes are concatenated based on the factors of (a) repeat number, (b) phase number, and (c) whether the volume was a control or included a label. To analyze the data (e.g. with [FSL's BASIL](https://users.fmrib.ox.ac.uk/~chappell/asl_primer/ex1GUI/index.html), you will need to specify the order of these factors. 

For enhanced DICOMs, this order is explicitly set by the [Dimension Index Values (0020,9157)](http://dicomlookup.com/lookup.asp?sw=Tnumber&q=(0020,9157)) tag. Therefore, for enhanced data you may want to check the DICOM header to determine the order.
 
The situation is tricky for classic DICOM. Philips often assigns [instance number (0020,0013)](https://dicom.innolitics.com/ciods/ct-image/general-image/00200013) randomly, with no set relationship to these factors. dcm2niix will always stack classic DICOM volumes in the same order: (a) repeat number, (b) phase number, (c) control or label. This order was chosen so that the classic DICOMs match the enhanced DICOMs for the provided datasets.

Here is a concrete example. consider an ASL sequence with three repeats and two phases. The order of the twelve volumes will be:

| Volume | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|--------|---|---|---|---|---|---|---|---|---|----|----|----|
|Repeat  | 1 | 2 | 3 | 1 | 2 | 3 | 1 | 2 | 3 | 1  | 2  | 3  |
|Phase   | 1 | 1 | 1 | 2 | 2 | 2 | 1 | 1 | 1 | 2  | 2  | 2  |
|Label   | C | C | C | C | C | C | L | L | L | L  | L  | L  |


## License

The code and images are covered by the [2-clause BSD license](https://opensource.org/licenses/BSD-2-Clause).

## Versions

* 5-August-2021 Initial public release


## Running

Assuming that the executable dcm2niix is in your path, you should be able to simply run the script `batch.sh` from the terminal.

If you have problems you can edit the first few lines of the `batch.sh` script so that `basedir` reports the explicit location of the `dcm_qa` folder (by default this is assumed to be the folder containing the script) on your computer and `exenam` reports the explicit location of dcm2niix (by default it is assumed to be in your path). Also, make sure the script is executable (`chmod +x batch.sh`). Then run the script.

## Useful Links

 - dcm2niix's handling of these files is described [here](https://github.com/rordenlab/dcm2niix/tree/master/Philips).


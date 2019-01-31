Data Processing
===============

Resting-state fMRI
------------------

Task-related fMRI
-----------------
FS-FAST
~~~~~~~
This section describes how to perform preprocessing and analysis of task-related fMRI data using FreeSurfer's `FSFAST<https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastTutorialV6.0>`_.



PET
---
Overview
~~~~~~~~
Preprocessing and analysis of PET data are done on the MGH cluster as well as on ``Aether``, a shared Windows workstation at the Hooker group. If you don't have access to ``Aether`` yet, ask Baileigh Hightower if she can grant you one. She would need your MGH ID.

Once ``Aether`` access is granted, the machine can be accessed through *Microsoft Remote Desktop*. Click on the icon '+' to add a new Desktop instance. Enter ``170.20.77.30`` in the PC Name field and use your MGH credentials (e.g., ``Partners/yk879``) to log in. Note that, if you're not on the Partners network, you would need to use VPN (details not covered in this page).

Initial Preprocessing
~~~~~~~~~~~~~~~~~~~~~
The first step in preprocessing of PET data involves attenuation correction and data segmentation based on the desired time frames. This is done by Grae Arabasz at Bay 7 console. We usually ask him to take care of this preprocessing, and once done, to send the data over to the Scratch folder on ``Aether`` (this can be found on ``D:``). It is important to make sure that for each subject we have the following sets of DICOM image files:

- 40_60_PRR_AC_IMAGES_40005
- 50_70_PRR_AC_IMAGES_40006
- DYNAMIC_PRR_AC_IMAGES_40001
- MEMPRAGE
- pseudo_muMAP_subjectID
- UMAP

Note that the first two images might have slightly different file names. The above example illustrates PiB data, which are extracted for three different time windows: *40-60* min post-TOI, *50-70* min post-TOI, and the whole acquisition duration). Each set of images is further segmented into chunks of 5 min frames. For T807, the windows of interest are: *75-105* min, *80-100* min, and the whole duration.

Once we confirm these data on the Scratch folder, copy them over to the Dickerson folder on Jacob's cluster space. If you don't see this directory mapped onto ``Aether``, you can do so by navigating to Windows Start > Computer > Map network drive, then type the following path into the Folder field (it doesn't matter which letter you pick in the Drive field)::

  \\sisyphus.nmr.mgh.harvard.edu\hookerlab\collaborators\Dickerson

Within this folder, navigate to ``ARBA`` where data for each individual subject are stored.

Data Quality Control (QC) on Aether
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Once on ``Aether``, launch MATLAB then ``cd`` into the subject's folder (create one if it doesn't exist). Within the subject's folder, create the following subfolders: MR, MR_PET, PET.

MR
  - Move the MEMPRAGE, pseudo_muMAP, and UMAP folders here.
PET
  - Move the remaining folders (e.g., *PRR_AC_IMAGES) here.

From within MATLAB, ``cd`` into the MEMPRAGE folder, then run ``piano_mMR``. Once the GUI comes up, click Dicom to Nifti. When an SPM file selector pops up, select the first .dcm or .ima file within the folder. The conversion process begins and finishes in about a minute. Once done, confirm that a new file called MPRAGE_spm has been generated within the MR_PET folder, then *rename this file to something different* (e.g., MPRAGE_spm2). This is because, as you will see in the next step, we need to do another DICOM-NIfTI conversion to generate an attenuation map, which would overwrite the MPRAGE we just created unless we rename the latter. This is rather an annoying issue, but since I don't really want to rewrite Jacob's code, we will stick to this method for now...

Following the same procedure, convert the DICOM files within the pseudo_muMAP folder. This generates another NIfTI image called MPRAGE_spm. *Rename this file as att_map, then undo renaming of the MPRAGE file*. As a result, you should now have two files within the MR_PET folder - att_map.nii (~5.8MB) and MPRAGE_spm.nii (~23MB).

Again following the same procedure, convert the DICOM files within each of the subfolders you just copied into the PET folder. Output of this conversion will always be called "PET_4D", so make sure that you rename the file after each conversion so as not to overwrite each. For instance, you could rename the file as "PET_4D_40-60min" if you're converting the DICOM files extracted from the 40-60 min time window.

Start with QC on the MPRAGE. Open and scroll through the MPRAGE_spm.nii file on each plane in MRICron to ensure that signal-to-noise ratio (SNR) is acceptable, and to check for any artifacts present in the data.

- It is OK for the ear(s) to be clipped a bit, but make sure that we have a full coverage of the nose (make a note if otherwise)
- In the case of huge subject motion, you might see blurs or ripples within the image. Make a note if these characteristics are present.

To overlay the att_map onto the MPRAGE_spm.nii, navigate to Overlay > Add. Scroll through the image to ensure that the attenuation is good, that the noise doesn’t affect the attenuation; also check for any artifacts, signal loss, etc.

- If the attenuation map appears completely opaque, toggle opacity by navigating to Overlay > Transparency on background, then select the desired level (e.g., 60%)
- A good attenuation map shows nice alignment with the MPRAGE, covering almost entirely the whole brain and nothing more. An example is shown on the left below. A bad attenuation map might show insufficient coverage and/or coverage of extracranial space. An example of this is shown on the right below.

.. image:: /_static/att_map_good.jpg
   :scale: 50 %

.. image:: /_static/att_map_bad.jpg
   :scale: 55 %

Overlay each of the PET recon images onto the MPRAGE_spm.nii. Once overlaid, change the color scheme to ``spectrum`` so that the recon image is displayed as a heat map. Set the lower threshold to 0 and the upper threshold to an arbitrary value, such that there is a nice gradient of intensity within the brain (see below for an example).

.. image:: /_static/recon_example.png
   :scale: 50 %

Inspect one frame at a time for each image for proper registration with MPRAGE, appropriate tracer uptake, if the brain is cut in the PET FOV, for artifacts, motion, etc.

- By clicking a "cone hat" icon in MRICron, you can toggle the visibility of overlay images. This is useful for examining the extent to which subject motion is present (i.e., misalignment with the MPRAGE)

**TO-DO**: Clarify issues regarding pseudo_AC with Baileigh

Further Preprocessing
~~~~~~~~~~~~~~~~~~~~~
Preprocessing steps following QC involve:

- Frame by frame realignment for motion correction
- Coregistration of PET and MR images with spmregister
- Brainmasking with a FreeSurfer mask and skullstripping using aparc/aseg
- Spatial normalization to the template space via FLIRT and FNIRT
- Mean intensity normalization (whole brain normalization)
- Smoothing with FWHM 8mm

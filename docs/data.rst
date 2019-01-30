Data Processing
===============
MRI
---

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
- A good attenuation map shows nice alignment with the MPRAGE, covering the whole brain and nothing more (see below for an example)

.. image:: /_static/att_map_good.jpg
   :scale: 50 %
Good (showing good alignment with MRPAGE)

.. image:: /_static/att_map_bad.jpg
   :scale: 55 %
Bad (low SNR around the chin/mouth)


Open the MPRAGE_spm_normalized .nii file to check that the subject’s nose and/or back of the skull aren’t cut in the FOV (this is a narrower FOV that is similar to the real PET FOV, so if the subject is cut here then the MPRAGE will be cut in the PET recon FOV)
Overlay the PET_4D_5minframes .nii and the PET_4D_60-90frame .nii onto the MPRAGE_spm .nii, respectively
If there is more than one frame, view and QC each frame (i.e., the 5minframes .nii file will contain 6-7 frames)
Change the intensity levels to 0 and ~1 (you can play with this to get the appropriate viewing level), and change the viewing color scheme to Spectrum
View each frame for proper registration of PET onto MR, appropriate tracer uptake, if the brain is cut in the PET FOV, for artifacts, motion, etc


Further Preprocessing
~~~~~~~~~~~~~~~~~~~~~

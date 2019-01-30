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
The first step in preprocessing of PET data involves attenuation correction and data segmentation based on the desired time frames. This is done by Grae Arabasz at Bay 7 console. We usually ask him to take care of this preprocessing, and once done, to send the data over to a ``scratch`` folder on ``Aether`` (this can be found on ``D:``). It is important to make sure that for each subject we have the following sets of DICOM image files:

- 40_60_PRR_AC_IMAGES_40005
- 50_70_PRR_AC_IMAGES_40006
- DYNAMIC_PRR_AC_IMAGES_40001
- MEMPRAGE
- pseudo_muMAP_subjectID
- UMAP

Note that the first two images might have slightly different file names. The above example illustrates PiB data, which are extracted for three different time windows: *40-60* min post-TOI, *50-70* min post-TOI, and the whole acquisition duration). Each set of images is further segmented into chunks of 5 min frames. For T807, the windows of interest are: *75-105* min, *80-100* min, and the whole duration.

Once we confirm these data on the ``scratch`` folder, copy them over to the Dickerson folder on Jacob's cluster space. If you don't see this directory mapped onto ``Aether``, you can do so by navigating to Windows Start > Computer > Map network drive, then type the following path into the Folder field (it doesn't matter which letter you pick in the Drive field)::

  \\sisyphus.nmr.mgh.harvard.edu\hookerlab\collaborators\Dickerson

Within this folder, navigate to ``ARBA`` where data for each individual subject are stored.

Data Quality Control (QC) on Aether
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Once on ``Aether``, launch MATLAB then ``cd`` into the subject's folder (create one if it doesn't exist). Within the subject's folder, create the following subfolders: ``MR``, ``MR_PET``, ``PET``.

MR
  - Move the MEMPRAGE, pseudo_muMAP, and UMAP folders here.
PET
  - Move the remaining folders (e.g., *PRR_AC_IMAGES) here.

From within MATLAB, ``cd`` into the MEMPRAGE folder, then run ``piano_mMR``. Once the GUI comes up, click Dicom to Nifti. When an SPM file selector pops up, select the first .dcm or .ima file within the folder. The conversion process begins and finishes in about a minute. Once done, confirm that a new file called MPRAGE_spm has been generated within the MR_PET folder, then *rename this file to something different* (e.g., MPRAGE_spm2). This is because, as you will see in the next step, we need to do another DICOM-NIfTI conversion to generate an attenuation map, which would overwrite the MPRAGE we just created unless we rename the latter. This is rather an annoying issue, but since I don't really want to rewrite Jacob's code, we will stick to this method for now... 




Further Preprocessing
~~~~~~~~~~~~~~~~~~~~~

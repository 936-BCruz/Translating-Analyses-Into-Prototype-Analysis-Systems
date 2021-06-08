# Producing the NanoAOD to Analyze  

I made a Docker container built from the *cmsopendata/cmssw_5_3_32* image with a mounted volume to move the produced NanoAOD to a local folder. I ran the following command
```bash
docker run -it --name ihepproject --volume C:\Users\bocr9\shared-folder\:/home/cmsusr/shared-folder cmsopendata/cmssw_5_3_32 /bin/bash
```
* The *--name* flag allows me to restart the container with the given name whenever I need to, by running the command 
```bash
docker start -i ihepproject
```
* The *--volume* flag mounts your local folder (on the left of the colon; notice I have a Windows operating system path) to a folder on the container (on the right; with a Linux path).
* *cmsopendata/cmssw_5_3_32* is the image.
* */bin/bash* adds a bash shell to the container. 

## Download what is needed

I downloaded the [CMS Open Data Higgs Analsys Example](https://github.com/cms-opendata-analyses/HiggsExample20112012) and 
the [AOD to NanoAOD Outreach Tool](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool) Github repositories, and
the [2011 Simulated Higgs to 4 Leptons](http://opendata.cern.ch/record/1507) index file to the Docker container.

Once you are in the container, you should be located at the *~/CMSSW_5_3_32/src* directory. In it, download the repositories by running...
```bash
git clone https://github.com/cms-opendata-analyses/HiggsExample20112012.git
```
then, first creating a directory named *workspace*, where the Outreach Tool repository will be downloaded to further seperate it from the other repository...
```bash
mkdir workspace
cd workspace
git clone https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool.git
```
Afterwards, head back to *~/CMSSW_5_3_32/src* to compile the codes... 
```bash
cd ../
scram b
```

Download the index file. I first made some directories for it too,
```bash
mkdir samples/AODSIM/AODSIM_2011/
cd samples/AODSIM/AODSIM_2011
wget http://opendata.cern.ch/record/1507/files/CMS_MonteCarlo2011_Summer11LegDR_SMHiggsToZZTo4L_M-125_7TeV-powheg15-JHUgenV3-pythia6_AODSIM_PU_S13_START53_LV6-v1_20000_file_index.txt
```

## Modify the python configuration file

I used the Outreach Tool's [simulation config file](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool/blob/master/configs/simulation_cfg.py).
Head to file's directory, in my case, from *CMSSW_5_3_32/src*
```bash
cd workspace/AOD2NanoAOD/configs
```
and modifying it with a text editor (I used *vi*) to take the downloaded index file as input
```python
files = FileUtils.loadListFromFile("/home/cmsusr/CMSSW_5_3_32/src/samples/AODSIM/AODSIM_2011/CMS_MonteCarlo2011_Summer11LegDR_SMHiggsToZZTo4L_M-125_7TeV-powheg15-JHUgenV3-pythia6_AODSIM_PU_S13_START53_LV6-v1_20000_file_index.txt")
```

In my case, I wanted to compare the orignal histograms produced from the original code to my translated code's histograms, to be sure the translation was working.
This may not be necessary for you, but I modified the fileservice section of the config file to also include the [HiggsExample20112012 EDAnalyzer](https://github.com/cms-opendata-analyses/HiggsExample20112012/blob/master/HiggsDemoAnalyzer/src/HiggsDemoAnalyzerGit.cc)
```python
# Register fileservice for output file
process.aod2nanoaod = cms.EDAnalyzer("AOD2NanoAOD", isData = cms.bool(False))
process.giteda = cms.EDAnalyzer("HiggsDemoAnalyzerGit")
process.TFileService = cms.Service(
    "TFileService", fileName=cms.string("2011MCNTuples.root"))
    
process.p = cms.Path(process.aod2nanoaod*process.giteda)
```
## Run the python configuration file

Initialize the CMS environment then, use cmsRun on the config file.
```bash
cmsenv
cmsRun simulation_cfg.py
```
## Move the NanoAOD to the mounted volume

Move the produced NanoAOD to your local computer via the mounted volume, to then open the file in JupyterLab for analysis.

```bash
mv 2011MCNTuples.root /home/cmsusr/shared-folder/
```

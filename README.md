# Translating-Analyses-Into-Prototype-Analysis-Systems
IRIS-HEP Fellowship project (January-June 2021), mentored by [Jim Pivarski](https://github.com/jpivarski). Read more about the project [here](https://iris-hep.org/fellows/BrianCruz.html). 


The translated analysis was the Higgs to 4 leptons analysis from CMS Open Data. The original code can be seen [here](https://github.com/cms-opendata-analyses/HiggsExample20112012/blob/master/HiggsDemoAnalyzer/src/HiggsDemoAnalyzerGit.cc) and my translation can be seen [here](https://github.com/936-BCruz/Translating-Analyses-Into-Prototype-Analysis-Systems/blob/main/Higgs%20to%204%20Leptons%20Analysis.ipynb).

You can see my project's final presentation:
* [PDF presentation](https://github.com/936-BCruz/Translating-Analyses-Into-Prototype-Analysis-Systems/blob/main/Brian_Cruz%2C%20Translating%20Analyses%20Into%20Prototype%20Analysis%20Systems.pdf)
* [IRIS-HEP Youtube video](https://www.youtube.com/watch?v=G49KILQUXjs)


## Producing the NanoAOD for Analysis

I used a Docker container built from the CMSSW_5_3_32 image to produced the NanoAOD for the [2011 simulated-Higgs-to-4-leptons sample](http://opendata.cern.ch/record/1507) 
from CERN's Open Data Portal. 
On my *Docker Container* directory, I guide you through this process [here](https://github.com/936-BCruz/Translating-Analyses-Into-Prototype-Analysis-Systems/blob/main/Docker%20Container/Producing%20the%20NanoAOD.md). 
You can additionally find a spreadsheet with an approximation for the total NanoAOD disksize from all the 21 Higgs analysis samples in the 
[Open Data Portal](http://opendata.cern.ch/record/5500). Download it [here](https://github.com/936-BCruz/Translating-Analyses-Into-Prototype-Analysis-Systems/blob/main/Docker%20Container/Disksize%20for%20Higgs%20Analysis%20samples%20NanoAODs.xlsx).

The approximation was made with the hope I could had scaled-up the NanoAOD production and stored the full-educational NanoAODs in the Open Data Portal with their respective samples.


## Scale-up Attempt using a Kubernetes Cluster in Google Cloud

I followed some instructions by Asdrubal Cruz's [CMS Parallel Job](https://github.com/asdru30/CMSParallelJobKubernetes) directory and 
a CMS Open Data [Kubernetes workshop](https://cms-opendata-workshop.github.io/workshop-lesson-kubernetes/). 
I guide you through my process [here](https://github.com/936-BCruz/Translating-Analyses-Into-Prototype-Analysis-Systems/blob/main/Kubernetes%20Cluster%20in%20Google%20Cloud/Running%20Parallel%20Workflows.md).


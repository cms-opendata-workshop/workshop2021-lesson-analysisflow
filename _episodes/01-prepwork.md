---
title: "Prep-work: Warming up for the Higgs to Ditau analysis"
teaching: 0
exercises: 0
questions:
- "What will we do during this lesson?"
- "What should I do before the lesson starts tomorrow?"
objectives:
- "Learn a little about the analysis example we will try to reproduce during the Analysis flow lessons tomorrow"
- "Prepare the newer-version ROOT container (or environment), which will speed things up"
keypoints:
- "The Higgs to tautau analysis is good example of analysis that can be done with CMS open data in a relatively easy and simplified way."
- "A container (or environment) with a newer version of ROOT could really speed things up."
- "See you tomorrow!  "
---

## Overview and some requested reading

Tomorrow, we will try to use most of what you have been learning and reproduce an analysis which has been already replicated using open data in a simplified way.  You will find that you have been already thinking about it in previous lessons. Please, read [the very short description](http://opendata.web.cern.ch/record/12350) of this analysis.  You will notice, at the bottom of that page, that there is already some code written to perform this study.  However, it uses mostly a `ROOT` class called [RDataFrame](https://root.cern/doc/master/classROOT_1_1RDataFrame.html), which is a great tool to perform *columnar* analysis, but it is maybe not very well suited for a first approach.  We invite you to got through this code quickly and try to identify the different selection cuts that are applied. We will be re-implementing these selection criteria using an *event loop* analysis (which is more intuitive) rather than the columnar approach.

## The lesson's activities

As with the othe lessons, tomorrow's lesson on analysis flow will have essentially three parts.  In the first one we will go together through a demonstration.  It is not mandatory to follow or copy every single step, but you are invited to do so if you wish.  For the second part, the hands-on one, you will work on your own with instructions provided in this lesson.  Instructors and facilitators will be there answer questions.  Lastly, you will be presented with a challenge that you may go on overtime to be able to complete (but do not worry it is not that difficult).  They day after tomorrow we will present a proposed solution to the challenge and have some time for questions and discussions.


## Copying the needed files locally

For our hands-on activity tomorrow, if you have some space, you can download the needed files from [here](https://cernbox.cern.ch/index.php/s/yzj0Qopaxtek5FJ). We very much recommend this as we need them for the final part of our analysis.  Copy them over to your container or VM. Working with these files locally will really speed up the analysis and so you will not need to work with an updated `ROOT` version (see below).  However, if you decide, for some reason, to connect remotely through `xrootd`, a newer version of `ROOT` may be needed.

Alternatively, you can copy them using the `xrdcp` command like:

~~~
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/Run2012B_TauPlusX.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/Run2012C_TauPlusX.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/DYJetsToLL.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/TTbar.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/W1JetsToLNu.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/W2JetsToLNu.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/W3JetsToLNu.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/GluGluToHToTauTau.root .
xrdcp root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/VBF_HToTauTau.root .
~~~
{: .language-bash}

## About the version of `ROOT` for our analysis

Our lesson exercises will work just fine in our Docker or VM default environments. However, note that the version of `ROOT` that comes with CMSSW in those environments is a rather old version, 5.32.  Therefore, upgrading our `ROOT` environment to a newer version will really speed things up if connecting remotely.  This is due, as far as we know, to the fact that `xrootd`, the protocol that we use to get access to ROOT files at CERN, has been drastically improved in later versions of the program.

Alternatively (and optionally), if you have some disk space of a few GBs, you could copy the root files we will be needing locally.  This is shown in the last section of this episode.  This action will also speed things up considerably even if you choose not to upgrade your root environment (**we recommend this latter approach, i.e., work with your current ROOT environment but with the locally-downloaded root files**).

## Getting a newer version of `ROOT`

If you decide to work with an upgraded version of `ROOT`.  You can follow these instructions.

If you are working with Docker, what we will do is to create a new container based on a recent `ROOT` container image.  You can follow the [same instructions](https://cms-opendata-workshop.github.io/workshop2021-lesson-docker/03-docker-for-cms-opendata/index.html#download-the-docker-image-for-cms-open-data-and-start-a-container) you used for creating your open data container, but instead of using the `cmsopendata/cmssw_5_3_32:latest` (or `cmsopendata/cmssw_5_3_32_vnc:latest`) image you will use a new stand-alone `ROOT` image.  The `ROOT` container image can be found in the [Docker hub](https://hub.docker.com/r/rootproject/root).  So, for example, for Linux, creating such a container would require a line like:

~~~
docker run -it --name myroot --net=host --env="DISPLAY" -v $HOME/.Xauthority:/home/cmsusr/.Xauthority:rw   rootproject/root:latest /bin/bash
~~~
{: .language-bash}

Follow the corresponding instructions for your own operating system.

In the virtual machine, you could also install a newer stack of `ROOT`.  You can get `ROOT` via CVMFS through the LCG releases. All information about the releases and contained packages can be found at [http://lcginfo.cern.ch](http://lcginfo.cern.ch). The following setup line works in the SLC6 shell and the “Outer shell” (SLC7) of the open data VM (we recommend using the Outer Shell).

~~~
source /cvmfs/sft.cern.ch/lcg/views/LCG_97/x86_64-slc6-gcc8-opt/setup.sh
~~~
{: .language-bash}

You can check the root version with:

~~~
root --version
~~~
{: .language-bash}



{% include links.md %}

---
title: "Demo: Skimming the datasets"
teaching: 0
exercises: 45
questions:
- "blah"
- "blah?"
objectives:
- ""
- ""
keypoints:
- ""
- ""
---

## Overview

As you read yesterday, we will try to re-implement the Higgs to ditau analysis that has been already implemented but in a different fashion.  The original open data analysis is based on two Github repositories to perform the analysis.  The first one is the [https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool] repository.  There, the important piece of code is the [AOD2NanoAOD.cc](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool/blob/master/src/AOD2NanoAOD.cc) code.  This is an analyzer, very similar to the analyzers you saw already in the [POET](https://github.com/cms-legacydata-analyses/PhysObjectExtractorTool) tool.  The main difference is that our POET is capable of extracting the required information in a very modular way.  The second part of this analysis relies on the [HiggsTauTauNanoAODOutreachAnalysis](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis) repository.  The code in this repository is already independent of CMSSW but relies on `ROOT` libraries, some of which are rather complex for a first approach, like the `RDataFrame` class.  In summary, one has to compile and run the `skim.cxx` C++ code, then the `histograms.py` python code, and, as a last step, execute the `plot.py` python script.  I.e., this analysis is done in 4 steps.

We will simplify this analysis by performing it in just 3 steps:

* the first one to skim our data out of CMSSW infrastructure
* the second one to analyze the data and produce `ROOT` histograms
* and the final one to make nice plots.

Later you can refer back to the original open data analysis and compare it to our more orthodox approach.  We hope that adopting this more traditional method, the physics and logic will be clearer.

This particular episode is a demonstration.  You are welcome to follow, but if at some point you get behind the hands-on work, do not worry, you will not be needing to having to completed this section in order to continue with the hands-on activity.  Of course, later you can refer back to this lesson and/or its video recording.

## Datasets and trigger selection

We will concentrate on studying the decays of a Higgs boson into two tau leptons in the final state of a muon lepton and a hadronically decayed tau lepton.  Since we need to have tau leptons, we choose these two datasets from 2012 (we are going to use only 2012 datasets for simplicity):

* the [Run2012B_TauPlusX](http://opendata.cern.ch/record/6024) dataset and
* the [Run2012C_TauPlusX](http://opendata.web.cern.ch/record/6050) dataset.

Note the sizes of these datasets.  It TBs of data.  No way we can easily download or work locally with them, we would need to *skim* them.

For our background processes we will chose the following MC simulations:

* the [DYJetsToLL](http://opendata.cern.ch/record/7730) dataset, which can contribute with real taus and/or misidentified leptons,
* the [TTbar](http://opendata.cern.ch/record/9518) dataset,
* and the set of [W1JetsToLNu](http://opendata.cern.ch/record/9863), [W2JetsToLNu](http://opendata.web.cern.ch/record/9864), and [W3JetsToLNu](http://opendata.web.cern.ch/record/9865) to complete the W+jets contribution.

As it is commonly done, we would also need to have an idea of how our signal (the process we are trying to find/discover) look like.  Thus, for the Higgs production we will consider the following MC simulations:

* the [GluGluToHToTauTau](http://opendata.cern.ch/record/8001) dataset and
* the [VBF_HToTauTau](http://opendata.cern.ch/record/9743) dataset.

Note also that all these datasets are in the TB range.

Now, for the trigger, we will use the `HLT_IsoMu17_eta2p1_LooseIsoPFTau20` as it is [done](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/13b5b3a850b020a19b209d7c8e2df7343cfc5ca0/skim.cxx#L74) in the original open data analysis.  Likewise threr, we will assume that this trigger is *unprescaled*.  Now you know, however, that one should really check that.

## Luminosity calculation

One of the key aspects in any particle physics analysis is the estimation of the integrated luminosity of a given collisions dataset.  From the [original analysis](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/13b5b3a850b020a19b209d7c8e2df7343cfc5ca0/skim.cxx#L53) we can see that the full integrated luminosities are:

<a href="https://www.codecogs.com/eqnedit.php?latex=4.412\&space;{\rm&space;fb^{-1}}\&space;{\rm&space;or\&space;}&space;4412\&space;{\rm&space;pb^{-1}\&space;for\&space;the\&space;Run2012B\_TauPlusX\&space;dataset}\\&space;7.055\&space;{\rm&space;fb^{-1}}\&space;{\rm&space;or\&space;}&space;7055\&space;{\rm&space;pb^{-1}\&space;for\&space;the\&space;Run2012C\_TauPlusX\&space;dataset}\\" target="_blank"><img src="https://latex.codecogs.com/gif.latex?4.412\&space;{\rm&space;fb^{-1}}\&space;{\rm&space;or\&space;}&space;4412\&space;{\rm&space;pb^{-1}\&space;for\&space;the\&space;Run2012B\_TauPlusX\&space;dataset}\\&space;7.055\&space;{\rm&space;fb^{-1}}\&space;{\rm&space;or\&space;}&space;7055\&space;{\rm&space;pb^{-1}\&space;for\&space;the\&space;Run2012C\_TauPlusX\&space;dataset}\\" title="4.412\ {\rm fb^{-1}}\ {\rm or\ } 4412\ {\rm pb^{-1}\ for\ the\ Run2012B\_TauPlusX\ dataset}\\ 7.055\ {\rm fb^{-1}}\ {\rm or\ } 7055\ {\rm pb^{-1}\ for\ the\ Run2012C\_TauPlusX\ dataset}\\" /></a>

for a total of approximately 11.5 inverse femtobarns between the two collisions datasets. **But, how can we figure this out?**

Fortunately we have a wonderful tool for this, called `brilcalc`.  Here we show a simple example, but more details can be found in [this CODP guide](http://opendata.cern.ch/docs/cms-guide-luminosity-calculation).

### Installing brilcalc in your container

You can install the `brilcalc` tool in the same cmsopendata container that you have been using so far or even your own host machine.  We will show the example using the container.

* Start your container or create a new one if you wish.  We will create a fresh new one (with our own setting; feel free to change them) that we will use for the rest of this lesson (although you can recycle the one you have been using):

~~~
docker run --name analysisflow -it --net=host --env="DISPLAY" --volume="$HOME/.Xauthority:/root/.Xauthority:rw" --volume="/home/ecarrera/.ssh:/home/cmsusr/.ssh" -v "/home/ecarrera/playground:/playground" cmsopendata/cmssw_5_3_32:latest /bin/bash
~~~
{: .language-bash}

* In order not to pollute the `CMSSW/src` are, we will change to the home directory:

~~~
cd ~
~~~
{: .language-bash}

* Fetch the installer for the brilcalc conda environment:

~~~
wget https://cern.ch/cmslumisw/installers/linux-64/Brilconda-3.0.0-Linux-x86_64.sh
~~~
{: .language-bash}

* Run the installer:

~~~
bash Brilconda-1.1.7-Linux-x86_64.sh -b -p <localbrilcondabase>
~~~
{: .language-bash}
where we substitute `<localbrilcondabase>` with `brilconda`. This is the directory in which the brilcalc tools will be installed.

* Once installed successfully add the tools to your `PATH` with the following command:

~~~
export PATH=$HOME/.local/bin:$HOME/brilconda/bin:$PATH
~~~
{: .language-bash}

* Then install brilcalc with this command:

~~~
pip install brilws
~~~
{: .language-bash}

* Finally check by running the following:

~~~
brilcalc --version
~~~
{: .language-bash}
which should output 3.6.6.

### Lumi calculation example

* Now, as an example, let's calculate the luminosity for the [Run2012B_TauPlusX](http://opendata.cern.ch/record/6024) dataset.  Let's download the list of validated runs from this site:

~~~
wget "http://opendata.cern.ch/record/1002/files/Cert_190456-208686_8TeV_22Jan2013ReReco_Collisions12_JSON.txt"
~~~
{: .language-bash}

Note that the run range for the `Run2012B_TauPlusX` dataset is from run `193833` to `196531`.

* Obtain the integrated luminosity:

~~~
brilcalc lumi -c web --type pxl -u /pb --begin 193833 --end 196531 -i Cert_190456-208686_8TeV_22Jan2013ReReco_Collisions12_JSON.txt > lumi.log &
~~~
{: .language-bash}

~~~
#Summary:
+-------+------+-------+-------+-------------------+------------------+
| nfill | nrun | nls   | ncms  | totdelivered(/pb) | totrecorded(/pb) |
+-------+------+-------+-------+-------------------+------------------+
| 56    | 146  | 50895 | 50895 | 4493.290785044    | 4411.703590626   |
+-------+------+-------+-------+-------------------+------------------+
~~~
{: .output}

which is exactly the [number quoted](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/13b5b3a850b020a19b209d7c8e2df7343cfc5ca0/skim.cxx#L53) in the original analysis.

> Note: You may notice at the end of the output luminosity sections that are listed in the json quality file but do not have any luminosity values corresponding to them. These correspond to sections that are left-overs at the end of a run, which where still tagged as STABLE RUN, but actually did not provide any luminosity. These are safe to ignore as they do not contain any events.
{: .testimonial}

## Simulated dataset normalization




{% include links.md %}

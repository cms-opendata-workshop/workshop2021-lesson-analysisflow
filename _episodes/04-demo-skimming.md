---
title: "Demo: Skimming"
teaching: 20
exercises: 0
questions:
- "Do we skim our data before perfoming the analysis study?"
- "How can I skim datasets efficiently with CMSSW"
objectives:
- "Justify the need for large scale skimming"
- "Lear how to best skim a dataset using CMSSW EDFilters"
keypoints:
- "One of the earlier steps in a particle physics analysis with CMS open data from Run 1 would be to skim the datasets down to a manageable size"
- "CMSSW offers a simple an efficient way of doing so."
- "Skimming usually requires decent computing resources and/or time."
---

> ## Disclaimer
>
> The first two episodes of this lesson are a demonstration.  You are welcome to follow along, but if at some point you get behind typing, do not worry, you will not be needing this section in order to continue with the hands-on activity after the break.  Of course, later you can refer back to these episodes and/or its video recording.
{: .callout}

## Introduction
So, we are now ready to start out analysis example.  We will go from very early datasets (in `AOD` format) to nice plots.
As you saw, the sizes of the datasets we are interested in are massive.  Obviously, we do not need all the information that is stored in all of those `AOD` files.  We should find a way then to cherry-pick only the fruit we need, i.e., those objects that we are interested in.  For our Higgs to ditau analysis, it will be essentially muons, taus and some missing energy (we do have neutrinos in our decays).

## Physics for POETs

You have been exploring ways (implemented in code) of accessing the information we need from physics objects.  In particular, you have worked already with the [`POET` repository](https://github.com/cms-legacydata-analyses/PhysObjectExtractorTool).  For this demonstration we will use our fresh container (used in the last episode) and check out this repository.  You can recycle the `POET` you already have in your container if you want to explore along with us:

```bash
git clone git://github.com/cms-legacydata-analyses/PhysObjectExtractorTool.git
```

As the instructions recommend, we need to get into the `PhysObjectExtractor` package, and compile:

```bash
cd PhysObjectExtractorTool/PhysObjectExtractor
scram b
```

Let's take a quick look at the configuration file, and in the process we will be identifying some of the modules that will be important for us:

```bash
less python/poet_cfg.py
```

A few things we can see:

* The number of events is just `200`
* The source file feeding the PoolSource module is that: just one file.  We would really need to run over the whole dataset (and not only one dataset but several.)
* There is some utility, called `FileUtils` that could be useful to load not only one but several root files through index lists, like the ones found in each dataset's pages, [e.g](http://opendata.cern.ch/record/6024).
* For collisions data, there is also a module which checks whether the data was quality-blessed through the `json` file runs and lumi sections listing.
* Aside from all the modules that configure the different analyzers, there are a couple of modules labeled as `Filters`, which are commented out.  We will use them in a moment, but first, let's make just one simple change and run.

We will activate the `FileUtils` module, replacing the *one-file* approach that comes as default. So, after the change to config file, the corresponding section will look like:

~~~
#---- Define the test source files to be read using the xrootd protocol (root://), or local files (file:)
#---- Several files can be comma-separated
#---- A local file, for testing, can be downloaded using, e.g., the cern open data client (https://cernopendata-client.readthedocs.io/en/latest/):
#---- python cernopendata-client download-files --recid 6004 --filter-range 1-1
#---- For running over larger number of files, comment out this section and use/uncomment the FileUtils infrastructure below
#if isData:
#       sourceFile='root://eospublic.cern.ch//eos/opendata/cms/Run2012B/DoubleMuParked/AOD/22Jan2013-v1/10000/1EC938EF-ABEC-E211-94E0-90E6BA442F24.root'
#else:
#       sourceFile='root://eospublic.cern.ch//eos/opendata/cms/MonteCarlo2012/Summer12_DR53X/TTbar_8TeV-Madspin_aMCatNLO-herwig/AODSIM/PU_S10_START53_V19-v2/00000/000A$
#process.source = cms.Source("PoolSource",
#    fileNames = cms.untracked.vstring(
#        #'file:/playground/1EC938EF-ABEC-E211-94E0-90E6BA442F24.root'
#       sourceFile
#    )
#)

#---- Alternatively, to run on larger scale, one could use index files as obtained from the Cern Open Data Portal
#---- and pass them into the PoolSource.  The example is for 2012 data
files = FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_10000_file_index.txt")
files.extend(FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_20000_file_index.txt"))
files.extend(FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_20001_file_index.txt"))
files.extend(FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_20002_file_index.txt"))
files.extend(FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_210000_file_index.txt"))
files.extend(FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_30000_file_index.txt"))
files.extend(FileUtils.loadListFromFile("data/CMS_Run2012B_DoubleMuParked_AOD_22Jan2013-v1_310000_file_index.txt"))
process.source = cms.Source(
    "PoolSource", fileNames=cms.untracked.vstring(*files))
~~~
{: .language-python}

We will also activate the reading of local DB files so the process goes faster:

~~~
#---- These two lines are needed if you require access to the conditions database. E.g., to get jet energy corrections, trigger prescales, etc.
process.load('Configuration.StandardSequences.FrontierConditions_GlobalTag_cff')
process.load('Configuration.StandardSequences.Services_cff')
#---- Uncomment and arrange a line like this if you are getting access to the conditions database through CVMFS snapshot files (requires installing CVMFS client)
#process.GlobalTag.connect = cms.string('sqlite_file:/cvmfs/cms-opendata-conddb.cern.ch/FT53_V21A_AN6_FULL.db')
#---- The global tag must correspond to the needed epoch (comment out if no conditions needed)
#if isData: process.GlobalTag.globaltag = 'FT53_V21A_AN6::All'
#else: process.GlobalTag.globaltag = "START53_V27::All"
#---- If the container has local DB files available, uncomment lines like the ones below
#---- instead of the corresponding lines above
if isData: process.GlobalTag.connect = cms.string('sqlite_file:/opt/cms-opendata-conddb/FT53_V21A_AN6_FULL_data_stripped.db')
else:  process.GlobalTag.connect = cms.string('sqlite_file:/opt/cms-opendata-conddb/START53_V27_MC_stripped.db')
if isData: process.GlobalTag.globaltag = 'FT53_V21A_AN6_FULL::All'
else: process.GlobalTag.globaltag = "START53_V27::All"
~~~
{: .language-python}

> The index files are in the `data` directory.  Also, in that directory, you will find other index files and data quality json files.  Also, note that the cfg file is not perfect: the `isData` flag does not control the execution over collisions or simulation index files any more, so you have to be careful about it.
{: .testimonial}

Now, let's run:

```bash
cmsRun python/poet_cfg.py True True > full.log 2>&1 &
```

While this completes, let us remember what this code is doing; essentially, executing all routines necessary to get information about the different physics objects, e.g.:

~~~
#---- Finally run everything!
#---- Separation by * implies that processing order is important.
#---- separation by + implies that any order will work
#---- One can put in or take out the needed processes
if doPat:
        process.p = cms.Path(process.patDefaultSequence+process.myevents+process.myelectrons+process.mymuons+process.myphotons+process.myjets+process.mymets+process.my...
else:
     	if isData: process.p = cms.Path(process.myevents+process.myelectrons+process.mymuons+process.myphotons+process.myjets+process.mymets+process.mytaus+process.myt...
        else: process.p = cms.Path(process.selectedHadronsAndPartons * process.jetFlavourInfosAK5PFJets * process.myevents+process.myelectrons+process.mymuons+process..


~~~
{: .language-python}

The output we get weights `5.2MB`:

```bash
ls -ltrh
```

~~~
-rw-r--r-- 1 cmsusr cmsusr 5.2M Jul 19 04:29 myoutput.root
~~~
{: .output}

## To Filter or not to filter

>Let's do a very crude estimate. If you remember, this file had [12279](https://cms-opendata-workshop.github.io/workshop2021-lesson-cmssw/02-installation/index.html#finding-the-eventsize-of-a-root-edm-file) events.  Since we processed `200`, running over the whole file would result in a `myoutput.root` file of `5.2*12279/200` ~`320MB`.  Accoring to [this dataset's record](https://opendata.cern.ch/record/6004), there are `2279` files.  It means that the final processing of this dataset will be `~730GB`.  Since we have to process several datasets, this is just very difficult.  
{: .testimonial}

So we need some sort of filtering.  For this workshop, we need to either access the output files remotely, via `xrootd`, or download them locally.  Being able to do so, depends on the final sizes.  We could probably store larger files somewhere in the cloud, but the access will also be more difficult, slower and costly.  For this reason, we will filter *hard* on our dataset.

We call this act of filtering the data, **skimming**.  `CMSSW` hast specific tools, called `EDFilters` to do this job.  They could look very much like an `EDAnalyzer`, but they filter events rather than analyze them.



Take a look at the `src` area of the `PhysObjectExtractor` package:

```bash
ls -1 src
```
~~~
ElectronAnalyzer.cc
EventAnalyzer.cc
GenParticleAnalyzer.cc
JetAnalyzer.cc
MetAnalyzer.cc
MuonAnalyzer.cc
PatJetAnalyzer.cc
PhotonAnalyzer.cc
SimpleMuTauFilter.cc
TauAnalyzer.cc
TrackAnalyzer.cc
TriggObjectAnalyzer.cc
TriggerAnalyzer.cc
VertexAnalyzer.cc
~~~
{: .output}

Notice the one `filter` example in the crowd: the **SimpleMuTauFilter.cc**.  Of course, it was one designed with the workshop in mind.  Let's [take a look at it](https://github.com/cms-legacydata-analyses/PhysObjectExtractorTool/blob/master/PhysObjectExtractor/src/SimpleMuTauFilter.cc).  You will see that it filters in the `pT` and `eta` variable for muons and taus and some other qualities.  In the original analysis of Higgs to ditau, this corresponds to the [first few selection cuts](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/skim.cxx#:~:text=auto%20df2%20%3D%20MinimalSelection,df5%20%3D%20FilterGoodEvents(df4)%3B) (except for the trigger requirement).

There is another filter, which we could in principle use to filter on the trigger.  However, we will keep the trigger information for later processing, just to show its usage.  Now, let us activate the `SimpleMuTauFilter`in the configuration file.  We will have to uncomment these line:

~~~
#---- Example of a CMSSW filter that can be used to cut on a given set of triggers
#---- This filter, however, does know about prescales
#---- A previous trigger study would be needed to cut hard on a given trigger or set of triggers
#---- The filter can be added to the path below if needed but is not applied by default
#process.load("HLTrigger.HLTfilters.hltHighLevel_cfi")
#process.hltHighLevel.HLTPaths = cms.vstring('HLT_IsoMu17_eta2p1_LooseIsoPFTau20_v*')




#---- Example of a very basic home-made filter to select only events of interest
#---- The filter can be added to the running path below if needed but is not applied by default
process.mutaufilter = cms.EDFilter('SimpleMuTauFilter',
                                  InputCollectionMuons = cms.InputTag("muons"),
                                   InputCollectionTaus = cms.InputTag("hpsPFTauProducer"),
                                   mu_minpt = cms.double(17),
                                   mu_etacut = cms.double(2.1),
                                   tau_minpt = cms.double(20),
                                   tau_etacut = cms.double(2.3)
                                   )

~~~
{: .language-python}

Finally, let's put it to work in the final `CMSSW` path that will be executed:

~~~
#---- Finally run everything!
#---- Separation by * implies that processing order is important.
#---- separation by + implies that any order will work
#---- One can put in or take out the needed processes
if doPat:
	process.p = cms.Path(process.mutaufilter+process.patDefaultSequence+process.myevents+process.myelectrons+process.mymuons+process.myphotons+process.myjets+process.mymets+process.mytaus+process.mytrigEvent+process.mypvertex+process.mytracks+process.mygenparticle+process.mytriggers)
else:
	if isData: process.p = cms.Path(process.myevents+process.myelectrons+process.mymuons+process.myphotons+process.myjets+process.mymets+process.mytaus+process.mytrigEvent+process.mypvertex+process.mytracks+process.mygenparticle+process.mytriggers)
	else: process.p = cms.Path(process.selectedHadronsAndPartons * process.jetFlavourInfosAK5PFJets * process.myevents+process.myelectrons+process.mymuons+process.myphotons+process.myjets+process.mymets+process.mytaus+process.mytrigEvent+process.mypvertex+process.mytracks+process.mygenparticle+process.mytriggers)
~~~
{: .language-python}

Let's run again to see what happens to the file size:

```bash
python python/poet_cfg.py #always good to check for syntax
cmsRun python/poet_cfg.py True True > filter.log 2>&1 &
```

The output we get weights just `22K`:

~~~
-rw-r--r-- 1 cmsusr cmsusr  22K Jul 19 06:01 myoutput.root
~~~
{: .output}

This is just `~0.4%` of the original size (ok, maybe in this particular case, the root file could actually be empty, but the reduction is indeed of that order), which means that the full dataset could be skimmed down to `~3GB`, a much more manageable size.

Based on these simple checks, you could also estimate the time it would take to run over all the datasets we need using a single computer.  To be efficient, you will need a computer cluster, but we will leave that for the *Cloud Computing* lesson.  Fortunately, we have prepared these skims already at CERN, using CERN/CMS computing infrastructure.  The files you will find at [this place](https://cernbox.cern.ch/index.php/s/yzj0Qopaxtek5FJ) (you probably already downloaded them during the prep-work) were obtained in essentially the same way, except that we dropped out objects like jets, photons, and trigEvent, which we won't be needing for this simple exercise.

> Note that, at this point, you could decide to use an alternative way storing the output.  You could, for instance, decide to just dump a simple csv file with the information you need, or use another commercial library to change to a different output format.  We will continue, however, using the `ROOT` format, just because this is what we are familiar with and it will allow us to get some nice plots.
{: .testimonial}

This completes the first step of the analysis chain.

{% include links.md %}

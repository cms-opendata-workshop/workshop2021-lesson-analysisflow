---
title: "Offline Challenge: Data-driven QCD estimation"
teaching: 0
exercises: 60
questions:
- "Is it possible to estimate the QCD contribution from data?"
objectives:
- "Implement a simple data-driven method for QCD estimation"
keypoints:
- "The QCD multijet background can be estimated using a data-driven approach"
---

## Data-driven QCD estimation

[These comments](https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis/blob/master/plot.py#:~:text=The%20major%20part,a%20scale%20factor.) in the original `plot.py` script, assert the possibility of estimating the `QCD` contribution in our analysis.

To give you a head-start, [this version](https://raw.githubusercontent.com/cms-opendata-workshop/workshop2021-payload-analysisflow/master/plot.py) of the `plot.py` will work with the output of the final version of our `EventLoopAnalysis.cxx` code.  Your task consists of modifying the latter code so we can complete the final plots by adding the QCD background.

Final plots should look like:

![](../fig/npv_final.png)
![](../fig/eta_2_final.png)
![](../fig/m_vis_final.png)

{% include links.md %}

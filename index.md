# Characterizing Memory Requirements for Large-Scale LSSTCam Productions at FrDF

```{abstract}

This technical note analyses memory‑usage patterns of `pipetask` during large‑scale LSSTCam production campaigns at CC‑IN2P3 (FrDF). We compare three campaigns (DM‑53368, DM‑53719, DM‑53877) that employed three different software‑stack releases: `w_2025_48`, `v30.0.0rc2` and `v30.0.0 (rc3)`. The results indicate that, for the majority of tasks, a memory budget of **6 GB per core** is sufficient, with only a small fraction of quanta exceeding this threshold. Consequently, reducing the per‑core RAM from the current 10 GB to 6 GB appears feasible and could generate significant cost savings.

```

# Characterizing Memory Requirements for Large-Scale LSSTCam Productions at FrDF

## Introduction

The purpose of this technical note is to study the memory usage patterns of `pipetask` to assess future RAM requirements for worker nodes. Currently, worker nodes are provisioned with 10 GB of memory per core. Analyzing metrics from recent large-scale tests at CC-IN2P3 can provide valuable insight into whether reducing the memory per core to 6 GB would be both feasible and cost-effective.

We analyse memory‑usage metrics collected by the Butler for four large‑scale test campaigns at the French Data Facility (FrDF - CC‑IN2P3):

```{rst-class} technote-wide-content
```
| Campaign | Software stack | Visits | Fields |
|----------|----------------|--------|--------|
| DM‑53368 | `w_2025_48`    | 4 721  | WIDE, DDF, COSMOS, M49 |
| DM‑53719 | `v30.0.0rc2`   | 1 061  | COSMOS, EDFS |
| DM‑53877 | `v30.0.0 (rc3)`| 1 067  | COSMOS, EDFS |
| DM‑54249 | `v30.0.4`      | 1 067  | COSMOS, EDFS |

*DM‑53368* is a large‑scale test that suffered hardware failures, resulting in incomplete Stage 4 metrics. *DM‑53719* and *DM‑53877* are pilot runs for the upcoming DP2 production at USDF.
*DM‑53877* encountered an issue with a pipetask and was terminated at step 4a.  

All the campaigns described in the table above have been executed at CC-IN2P3. For this analysis, we will focus on the last campaign executed with `v30.0.4`, but we will provide a comparison of each campaign in the annexe.

The fields have been selected in accordance with the cCampaign Management (CM) team’s recommendations to cover regions that could pose challenges for data processing, or at least present more significant issues compared to a wide field.
Details on the fields are available in offfcial LSST documentation, {cite}`RTN-011`.  

The campaigns have been submitted from the CC-IN2P3 using BPS and Panda, with data from the `dp2_prep` butler as input. The input collection was created using the provided CM scripts and selecting the appropriate visits and exposures via ConsDB. 

The scripts to execute the campaigns have been generated from templates using a set of scripts available on [CC_IN2P3 Gitlab](https://gitlab.in2p3.fr/rubin-lsst/cm_scripts_generator). For each stage, the metrics are extracted directly from the butler using scripts available in [GitHub](https://github.com/jchiang87/batch_usage_utils) modified for our scope. And then, the analysis of these metrics is performed stage by stage using Jupyter notebooks.

Our goal is to determine whether a **6 GB / core** threshold would be sufficient.

## Analysis and Results

We analyze each campaign stage-by-stage.

To demonstrate the improvements introduced in version 30, we present a comparative analysis of stage 1 for of all the stacks  in the following figure. However, we will now focus exclusively on data from the `DM-54249` campaign based on the `v30.0.4` release and we included the comparison between stacks in the annexes. 

```{figure} ./images/MaxRSSTask_all_stacks_stage1.png
:figclass: technote-wide-content


The box plot compares the maximum RSS of the Stage 1 pipetask across four software stack releases. The orange and red dashed lines represent the 4GB and 6GB memory thresholds, respectively. The chart clearly demonstrates a significant improvement in memory usage from the older “weekly” stack to the v30 version. 
```

A few words about the nomenclature: in the following pages and charts, we refer to each pipetask execution interchangeably as 'quanta' or 'runs'.

### Stage 1 

The analysis of stage 1 reveals that, when excluding `analyzeSingleVisitStarAssociation` and certain outliers for `associatesIsolateleStar`, all stage 1 pipetasks are well below the 6GB per core threshold, as shown in the figure below.

```{figure} ./images/MaxRSS_box_all_stacks_stage1.png
:figclass: technote-wide-content

Box plot showing maximum RSS for Stage 1 pipetasks in v30.0.4 release. 
Orange and red dashed lines indicate 4GB and 6GB memory thresholds. 
Most tasks require <6GB RSS, except analyzeSingleVisitStarAssociation.
```

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage1.png
:figclass: technote-wide-content

The plot displays number of runs (quanta) per Stage 1 pipetask that exceed the 6 GB memory threshold (dashed red line). Each diamond marks the frequency (or number of quanta) at which a given task reaches a specific memory level. 
```

The detailed view of RSS usage is summarized in the following table. 

```{rst-class} technote-wide-content
```
|                                                    |   max_rss_mean |   max_rss_median |   max_rss_min |   max_rss_max |   max_rss_95th_percentile | 
|:---------------------------------------------------|---------------:|-----------------:|--------------:|--------------:|--------------------------:|
| analyzeSingleVisitStarAssociation                  |       7.602    |         4.24253  |      0.562172 |     23.7612   |                 23.6281   | 
| makeAnalysisSingleVisitStarAssociationMetricTable  |       0.508282 |         0.508282 |      0.508282 |      0.508282 |                  0.508282 | 
| makeInitialVisitTable                              |       0.559509 |         0.559509 |      0.559509 |      0.559509 |                  0.559509 | 
| consolidateSingleVisitStar                         |       0.741334 |         0.737282 |      0.689316 |      0.907547 |                  0.832429 | 
| standardizeSingleVisitStar                         |       3.01989  |         3.00393  |      2.95538  |      5.47653  |                  3.06758  | 
| consolidateVisitSummary                            |       0.741334 |         0.737282 |      0.689316 |      0.907547 |                  0.832429 | 
| makeInitialVisitDetectorTable                      |       0.864044 |         0.864044 |      0.864044 |      0.864044 |                  0.864044 | 
| isr                                                |       2.99997  |         2.99674  |      1.41269  |      3.13416  |                  3.0571   | 
| makeAnalysisSingleVisitStarAssociationWholeSkyPlot |       2.26377  |         2.26377  |      2.26377  |      2.26377  |                  2.26377  | 
| associateIsolatedStar                              |       2.93676  |         2.98122  |      0.609653 |     11.0854   |                  6.13378  | 
| calibrateImage                                     |       3.01213  |         3.00333  |      2.95538  |      5.47653  |                  3.06223  | 

The next two charts illustrate the distribution of RSS in 1 GB increments for all quanta processed during Stage 1. A total of 574 472 quanta were processed; only 40 (≈ 0.007 %) exceeded the 6 GB threshold.


```{figure} ./images/MaxRSS_distro_per_task_v30_stage1.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 1 run, with histogram bars grouped in 1 GB bins. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```

The majority of these exceeding quanta are attributed to the `analyzeSingleVisitStarAssociation` pipetask as visible in the next chart using logaritmic scale to improve the visualization of the outlier .

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage1.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 1 run, with histogram bars grouped in 1 GB bins and the count on a logarithmic scale to improve the visualization. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```

The next two charts, showing integrated wall time per RSS range, i.e. the total walltime summed for all tasks whose peak RSS falls within each defined memory‑usage interval, useful to highlights which memory regimes dominate the overall runtime. 
It further confirms that 6 GB per core is generally sufficient for almost all Stage 1 pipet tasks. In fact, the tasks that require more than 6 GB of RSS correspond to approximately 3.3 hours of wall time, compared to the 10,000 hours of total wall time for the entire stage 1.


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage1.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 1 processes.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of 3 hours of walltime compared to the 10,000 hours of total walltime.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage1.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 1 processes depassing 6GB. 
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of 3 hours of walltime compared to the 10,000 hours of total walltime.
```

### Stage 2 

Even in Stage 2 the 6 GB threshold remains a useful reference, but two tasks—**gbdesHealpix3AstrometricFit** and **analyzeRecalibratedStarAssociation**—exceed it in terms of peak RSS, and a few other tasks show occasional outliers, as illustrated in the figure below.

```{figure} ./images/MaxRSS_box_all_stacks_stage2.png
:figclass: technote-wide-content

Box plot showing maximum RSS for Stage 2 pipetasks in v30.0.4 release. 
Orange and red dashed lines indicate 4GB and 6GB memory thresholds. 
Most tasks require <6GB RSS but `analyzeRecalibratedStarAssociation` and `gbdesHealpix3AstrometricFit`.
```

How we can see in the above chart, differents tasks are using more than 6GB of RSS. The following charts visualize all the quanta using more than 6GB of memory grouped by pipetasks. Most of these cases involve the `analyzeRecalibratedStarAssociation` and `gaussianProcessesTurbulenceFit` pipetask.

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage2.png
:figclass: technote-wide-content

The plot displays number of runs (quanta) per Stage 2 pipetask that exceed the 6 GB memory threshold (dashed red line). Each diamond marks the frequency (or number of quanta) at which a given task reaches a specific memory level. 
```

More details are available in the next table. 
```{rst-class} technote-wide-content
```
|                                                     |      max_rss_mean |   max_rss_median |   max_rss_min |   max_rss_max |   max_rss_95th_percentile | 
|:----------------------------------------------------|------------------:|-----------------:|--------------:|--------------:|--------------------------:|
| recalibrateSingleVisitStar                          |          0.941043 |         0.942085 |      0.815155 |      0.946217 |                  0.944084 | 
| makeVisitTable                                      |          0.746078 |         0.746078 |      0.746078 |      0.746078 |                  0.746078 | 
| gaussianProcessesTurbulenceFit                      |          6.12501  |         6.2077   |      1.07148  |      6.54644  |                  6.51897  | 
| fgcmFitCycle                                        |         34.4069   |        34.4069   |     34.4069   |     34.4069   |                 34.4069   | 
| fitStellarMotion                                    |          0.713758 |         0.656345 |      0.502529 |      1.43505  |                  1.09982  | 
| makeAnalysisRecalibratedStarAssociationWholeSkyPlot |          2.2629   |         2.2629   |      2.2629   |      2.2629   |                  2.2629   | 
| analyzeRecalibratedStarAssociation                  |          6.52842  |         3.60212  |      0.486923 |     19.7006   |                 19.4489   | 
| consolidateRefitPsfModelDetector                    |          0.880231 |         0.873501 |      0.864693 |      0.955421 |                  0.914266 | 
| standardizeRecalibratedStar                         |          0.941053 |         0.942085 |      0.815155 |      0.946217 |                  0.944084 | 
| refitPsfModelDetector                               |          3.41789  |         3.56515  |      0.96085  |      4.92125  |                  4.36204  | 
| consolidateRecalibratedStar                         |          0.874706 |         0.864555 |      0.837677 |      1.01746  |                  0.950039 | 
| analyzeRecalibratedStarAstrometricRefMatch          |          0.842907 |         0.842958 |      0.835007 |      0.846905 |                  0.845247 | 
| fgcmOutputProducts                                  |          8.3409   |         8.3409   |      8.3409   |      8.3409   |                  8.3409   | 
| gbdesHealpix3AstrometricFit                         |          4.73779  |         3.74013  |      0.684643 |     12.0505   |                 11.3513   | 
| makeAnalysisRecalibratedStarAssociationMetricTable  |          0.505554 |         0.505554 |      0.505554 |      0.505554 |                  0.505554 | 
| makeAnalysisRecalibratedStarAstrometricRefMatch     |          0.885111 |         0.883438 |      0.835526 |      0.977863 |                  0.937105 | 
| updateVisitSummary                                  |          1.24466  |         1.2331   |      0.976486 |      1.66946  |                  1.39579  | 
| fgcmBuildFromIsolatedStar                           |         11.2081   |        11.2081   |     11.2081   |     11.2081   |                 11.2081   | 
| makeVisitDetectorTable                              |          1.0936   |         1.0936   |      1.0936   |      1.0936   |                  1.0936   | 



The next two charts show the RSS distribution in 1 GB increments for all quanta processed during Stage 2. A total of 567,886 quanta were processed, with 1916 (around 0.3%) exceeding the 6 GB threshold. 

```{figure} ./images/MaxRSS_distro_per_task_v30_stage2.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 2 run, with histogram bars grouped in 1 GB bins. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage2.png
:figclass: technote-wide-content


Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 2 run, with histogram bars grouped in 1 GB bins and the count on a logarithmic scale to improve the visualization. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```



The next two charts, showing integrated wall time per RSS range, further confirms that 6 GB per core is generally sufficient for almost all Stage 2 pipeline tasks but the `gaussianProcessesTurbulenceFit` pipetask consistently utilizes around 6 GB of memory for extended periods.

If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will mobilize some cores for ~2,100 hours compared (~25%) to ~8,300 hours total walltime. 
```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage2.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 2 processes.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~2,100 hours of walltime compared to the 8,300 hours of total walltime.
```


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage2.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 2 processes depassing 6GB.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~2,100 hours of walltime compared to the 8,300 hours of total walltime.
```

### Stage3


In the chart below, we see that memory usage becomes more critical at this stage: a larger number of quanta exceed the 6 GB limit compared with earlier stages. However, an analysis of the integrated walltime shows that the overall impact is negligible. 

```{figure} ./images/MaxRSS_box_all_stacks_stage3.png
:figclass: technote-wide-content

Box plot showing maximum RSS for Stage 3 pipetasks in v30.0.4 release. 
Orange and red dashed lines indicate 4GB and 6GB memory thresholds. 
Most tasks require < 6 GB RSS, but many quanta of `deblendCoaddFootprints`, `makeDeepCoaddInputSummaryTract`, and `makeHealSparsePropertyMaps` exceed this threshold. 
```

How we can see in the above chart, differents tasks are using more than 6GB of RSS. The following charts visualize all the quanta using more than 6GB of memory grouped by pipetask. We can see that the pipetask `deblendCoaddFootprint` and `makeHealSparsePropertyMaps` are the one exceeding more ofthen the 6GB threshold. 

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage3.png
:figclass: technote-wide-content

The plot displays number of runs (quanta) per Stage 3 pipetask that exceed the 6 GB memory threshold (dashed red line). Each diamond marks the frequency (or number of quanta) at which a given task reaches a specific memory level. 
```

Summary of RSS usage for each pipetask in Stage 3 is reported in the next table. 
```{rst-class} technote-wide-content
```
|                                           |   max_rss_mean |   max_rss_median |   max_rss_min |   max_rss_max |   max_rss_95th_percentile |
|:------------------------------------------|---------------:|-----------------:|--------------:|--------------:|--------------------------:|
| selectTemplateCoaddVisits                 |       0.65371  |         0.652361 |      0.595356 |      0.688534 |                  0.684475 |
| splitPrimaryObject                        |       4.38222  |         1.9906   |      0.63155  |     17.4722   |                 13.1409   |
| fitDeepCoaddPsfGaussians                  |       1.0076   |         1.04602  |      0.621536 |      1.09909  |                  1.08942  |
| metadetectionShear                        |       1.67526  |         1.742    |      0.706963 |      1.84409  |                  1.83381  |
| assembleTemplateCoadd                     |       2.37399  |         2.45605  |      0.598366 |      2.90918  |                  2.73577  |
| consolidateParentTable                    |       0.768712 |         0.679214 |      0.625923 |      1.1706   |                  1.08776  |
| mergeObjectMeasurement                    |       0.6944   |         0.633909 |      0.620609 |      0.863731 |                  0.845725 |
| plotPropertyMapTract                      |       4.69683  |         4.70322  |      4.41187  |      5.04356  |                  4.92766  |
| fitDeblendedObjectsSersic                 |       2.12916  |         1.34765  |      0.822514 |      4.61545  |                  4.37458  |
| refCatObjectTract                         |       0.921133 |         0.888134 |      0.798336 |      1.08419  |                  1.07457  |
| makePsfMatchedWarp                        |       3.39722  |         3.42105  |      0.617081 |      3.8084   |                  3.57501  |
| catalogMatchTract                         |       0.870329 |         0.729294 |      0.649548 |      1.50202  |                  1.33685  |
| plotPropertyMapSurvey                     |       4.1602   |         4.25547  |      3.05684  |      4.82431  |                  4.79948  |
| makeMetricTableObjectParentTableCore      |       0.490383 |         0.490383 |      0.490383 |      0.490383 |                  0.490383 |
| makeBinnedCoaddImage                      |       0.740526 |         0.740738 |      0.706959 |      0.755222 |                  0.751319 |
| selectDeepCoaddVisits                     |       0.653076 |         0.651211 |      0.595356 |      0.688534 |                  0.684441 |
| measureObjectForced                       |       1.2724   |         1.31315  |      0.620621 |      1.68477  |                  1.59369  |
| detectCoaddPeaks                          |       4.04251  |         4.19313  |      0.598366 |      4.70403  |                  4.49203  |
| makeDirectWarp                            |       3.34297  |         3.40371  |      0.617081 |      3.8084   |                  3.44984  |
| consolidateObject                         |       2.04476  |         1.18666  |      0.63308  |      6.5104   |                  5.16442  |
| mergeObjectDetection                      |       0.658362 |         0.651159 |      0.541325 |      0.686981 |                  0.682064 |
| makeWholeTractImage                       |       0.767424 |         0.759033 |      0.675655 |      0.895149 |                  0.874706 |
| makeWholeTractTemplateNImage              |       0.804325 |         0.799746 |      0.695847 |      0.925613 |                  0.922905 |
| photometricRefCatObjectTract              |       0.916859 |         0.882507 |      0.788025 |      1.08813  |                  1.04813  |
| consolidateShearObject                    |       0.899483 |         0.801559 |      0.624893 |      1.45676  |                  1.26816  |
| validateObjectTableCore                   |       0.701319 |         0.640598 |      0.621323 |      0.916172 |                  0.882812 |
| objectParentTableCoreWholeSkyPlot         |       0.586369 |         0.586369 |      0.586369 |      0.586369 |                  0.586369 |
| makeHealSparsePropertyMaps                |       5.72563  |         4.59602  |      1.01052  |     17.6435   |                 14.9179   |
| makeBinnedDeepNImage                      |       0.743317 |         0.744091 |      0.621178 |      0.755222 |                  0.752476 |
| analyzeObjectTableSurveyCore              |       2.06782  |         2.06782  |      2.06782  |      2.06782  |                  2.06782  |
| assembleCellCoadd                         |       4.03475  |         4.18464  |      0.598366 |      4.60523  |                  4.48033  |
| deblendCoaddFootprints                    |       3.7279   |         2.79314  |      1.79509  |      6.90266  |                  6.37775  |
| photometricCatalogMatch                   |       0.864245 |         0.754807 |      0.647858 |      1.44056  |                  1.28093  |
| analyzeObjectTableCore                    |       2.0096   |         1.48076  |      1.04328  |      4.32022  |                  3.62084  |
| analyzeObjectParentTableCore              |       0.661792 |         0.62991  |      0.623722 |      0.764782 |                  0.759708 |
| consolidateHealSparsePropertyMaps         |       1.00861  |         1.01416  |      0.85503  |      1.11505  |                  1.11164  |
| makeMetricTableObjectTableCoreRefCatMatch |       0.488861 |         0.488861 |      0.488861 |      0.488861 |                  0.488861 |
| makeMetricTableObjectTableCore            |       0.494553 |         0.494553 |      0.494553 |      0.494553 |                  0.494553 |
| makeDeepCoaddInputSummary                 |       0.592121 |         0.592121 |      0.592121 |      0.592121 |                  0.592121 |
| computeObjectEpochs                       |       1.76113  |         0.852695 |      0.674358 |      4.414    |                  4.12537  |
| assembleDeepCoadd                         |       2.36339  |         2.44578  |      0.598366 |      2.84563  |                  2.72003  |
| objectTableCoreWholeSkyPlot               |       2.64798  |         2.64798  |      2.64798  |      2.64798  |                  2.64798  |
| objectTableCoreRefCatMatchWholeSkyPlot    |       1.15347  |         1.15347  |      1.15347  |      1.15347  |                  1.15347  |
| makeTemplateCoaddInputSummaryTract        |       1.51558  |         0.98008  |      0.664383 |      6.76672  |                  3.8465   |
| makeDeepCoaddInputSummaryTract            |       2.95651  |         0.954704 |      0.665157 |     17.955    |                 10.5732   |
| makeWholeTractDeepNImage                  |       0.653418 |         0.6502   |      0.621967 |      0.688782 |                  0.687188 |
| rewriteObject                             |       1.74653  |         0.852695 |      0.624527 |      4.414    |                  4.12537  |
| deconvolve                                |       1.53463  |         1.59141  |      0.817303 |      1.63986  |                  1.62381  |
| makeTemplateCoaddInputSummary             |       0.533478 |         0.533478 |      0.533478 |      0.533478 |                  0.533478 |
| standardizeObject                         |       1.83685  |         0.929029 |      0.836823 |      4.414    |                  4.12537  |
| measureObjectUnforced                     |       1.39125  |         1.4261   |      0.775192 |      1.72871  |                  1.64419  |
| makeBinnedTemplateNImage                  |       0.743883 |         0.744183 |      0.709461 |      0.755222 |                  0.752476 |
| fitDeblendedObjectsExp                    |       2.12206  |         1.3449   |      0.821854 |      4.60065  |                  4.3463   |



The next two charts show the RSS distribution in 1 GB increments for all quanta processed during Stage 3. A total of 887,683 quanta were processed, with 637  (around 0.07%) exceeding the 6 GB threshold.  

```{figure} ./images/MaxRSS_distro_per_task_v30_stage3.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 3 run, with histogram bars grouped in 1 GB bins. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage3.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 3 run, with histogram bars grouped in 1 GB bins and the count on a logarithmic scale to improve the visualization. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```


The next two charts, showing integrated wall time per RSS range, further confirms that 6 GB per core is generally sufficient for almost all Stage 3 pipeline tasks but the `gaussianProcessesTurbulenceFit` pipetask consistently utilizes around 6 GB of memory for extended periods.

If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will mobilize some cores for ~124  hours compared (~0.5%) to ~25,973 hours total walltime. 

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage3.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 3 processes.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~124 hours of walltime compared to the 25,973 hours of total walltime.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage3.png
:figclass: technote-wide-content
Integrated walltime per RSS range for Stage 3 processes depassing 6GB.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~124 hours of walltime compared to the 25,973 hours of total walltime.
```


### Stage 4 

For Stage 4 we don't have completed metrics for `w_2025_48` and `v30.0.0` stacks, so our analysis is solely based on `v30.0.0.rc2` stack. 

As shown in the following chart, we have few pipeatsk exceeding systematically the 6GB threshold. 

```{figure} ./images/MaxRSS_box_all_stacks_stage4.png
:figclass: technote-wide-content

Box plot showing maximum RSS for Stage 4 pipetasks in v30.0.4 release. 
Orange and red dashed lines indicate 4GB and 6GB memory thresholds. 
Most tasks require < 6 GB RSS, but some quanta of `associateAnalysisSource`, `skyCorr`, and `analyzeSourceAssoclation` exceed this threshold.
```

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage4.png
:figclass: technote-wide-content

The plot displays number of runs (quanta) per Stage 4 pipetask that exceed the 6 GB memory threshold (dashed red line). Each diamond marks the frequency (or number of quanta) at which a given task reaches a specific memory level. 
```
Summary of RSS usage for each pipetask in Stage 4 is reported in the next table. 
```{rst-class} technote-wide-content
```
|                                           |   max_rss_mean |   max_rss_median |   max_rss_min |   max_rss_max |   max_rss_95th_percentile |
|:------------------------------------------|---------------:|-----------------:|--------------:|--------------:|--------------------------:|
| computeReliability                        |       3.40115  |         3.35599  |      1.24137  |      4.74003  |                  3.89839  |
| standardizeDiaSource                      |       3.4009   |         3.35596  |      1.24137  |      4.74003  |                  3.89839  |
| makeAnalysisSourceAssociationWholeSkyPlot |       2.26207  |         2.26207  |      2.26207  |      2.26207  |                  2.26207  |
| splitPrimarySource                        |       2.05626  |         2.08863  |      0.998497 |      3.54106  |                  3.03666  |
| forcedPhotDiaObjectDetector               |       1.23566  |         1.22809  |      0.952499 |      1.32349  |                  1.29137  |
| standardizeSource                         |       1.82261  |         1.79641  |      1.10159  |      3.47886  |                  2.07528  |
| filterDiaSource                           |       3.40121  |         3.35601  |      1.24137  |      4.74003  |                  3.89839  |
| associateDiaSource                        |       1.08492  |         1.05676  |      0.989223 |      1.40002  |                  1.3506   |
| standardizeDiaObjectForcedSource          |       1.01899  |         0.95816  |      0.921131 |      2.55805  |                  1.46923  |
| consolidateDiaSource                      |       1.20771  |         1.00259  |      0.992447 |      2.75513  |                  2.18333  |
| associateAnalysisSource                   |       8.29251  |         8.62498  |      1.06258  |     20.243    |                 16.9474   |
| analyzeSourceAssociation                  |      17.2298   |        13.8867   |      1.11311  |     52.4146   |                 52.2619   |
| analyzeDiaSourceTableTract                |       1.02561  |         0.999922 |      0.992092 |      1.31514  |                  1.2466   |
| reprocessVisitImage                       |       1.75376  |         1.73119  |      1.13905  |      3.47886  |                  1.99347  |
| makeWholeTractPrettyCoaddImage            |       0.942893 |         0.933582 |      0.925068 |      0.988426 |                  0.960958 |
| skyCorr                                   |      15.5696   |        15.5163   |     15.4282   |     15.6787   |                 15.6703   |
| consolidateVisitDiaSource                 |       1.02144  |         1.02146  |      1.01239  |      1.02584  |                  1.02387  |
| calculateDiaObject                        |       1.08962  |         1.05954  |      0.989223 |      1.43301  |                  1.36909  |
| consolidateDiaObject                      |       1.03114  |         1.00198  |      0.991974 |      1.43875  |                  1.26616  |
| detectAndMeasureDiaSource                 |       3.3683   |         3.31585  |      1.24137  |      4.74003  |                  3.88423  |
| standardizeObjectForcedSource             |       1.29109  |         0.959755 |      0.949585 |      3.6499   |                  3.31977  |
| makeBinnedPrettyCoaddImage                |       0.970211 |         0.952377 |      0.920452 |      1.19641  |                  1.12585  |
| makeBinnedPrettyNImage                    |       0.943863 |         0.95055  |      0.919342 |      0.961758 |                  0.959702 |
| makePrettyDirectWarp                      |       2.65413  |         2.7501   |      0.918617 |      3.33527  |                  2.95333  |
| assemblePrettyCoadd                       |       2.53094  |         2.5476   |      0.926598 |      3.44558  |                  2.87072  |
| makeWholeTractPrettyNImage                |       0.939257 |         0.929531 |      0.925667 |      0.960793 |                  0.95993  |
| splitPrimaryObjectForcedSource            |       1.35045  |         0.959896 |      0.949177 |      4.02045  |                  3.55207  |
| makeAnalysisSourceAssociationMetricTable  |       0.503857 |         0.503857 |      0.503857 |      0.503857 |                  0.503857 |
| filterDiaSourcePostReliability            |       3.40105  |         3.35599  |      1.24137  |      4.74003  |                  3.89839  |
| forcedPhotObjectDetector                  |       1.29355  |         1.28862  |      1.19354  |      1.40014  |                  1.34867  |
| consolidateSource                         |       1.17934  |         1.18887  |      0.996571 |      1.54529  |                  1.39815  |
| subtractImages                            |       3.25719  |         3.19458  |      1.24137  |      4.65683  |                  3.78299  |
| makePrettyPsfMatchedWarp                  |       1.89612  |         1.96549  |      0.918522 |      2.2924   |                  2.28677  |
| buildTemplate                             |       2.73482  |         2.73365  |      1.24137  |      4.21315  |                  3.24956  |
| analyzeRecalibratedStarObjectMatch        |       1.50746  |         1.67629  |      1.06583  |      1.7955   |                  1.73455  |


However, as demonstrated in the next two charts, the number of quanta (quanta) exceeding the thresholds is quite minimal compared to the total: in fact, we had 1137 (0.04%) quanta exceeding the threshold and 3,408,460 quanta processed in total. 

```{figure} ./images/MaxRSS_distro_per_task_v30_stage4.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 4 run, with histogram bars grouped in 1 GB bins. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage4.png
:figclass: technote-wide-content

Distribution of the maximum RSS per pipetask for the DM‑54249 (v30.0.4) Stage 4 run, with histogram bars grouped in 1 GB bins and the count on a logarithmic scale to improve the visualization. The green dashed line indicates the 95th percentile of the RSS values, showing that 95 % of the pipetasks fall at or below this level, while the red dashed line marks the 6 GB memory threshold; tasks that exceed this line may require further optimisation or additional resources. This plot highlights that the majority of pipetasks stay within the prescribed memory threshold, with only a small tail of outliers surpassing the 6 GB boundary.
```

If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will mobilize a few cores for ~2,893 hours  (~4%) compared to ~73,137 hours of total walltime. 


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage4.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 4 processes.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~2,893 hours of walltime (~4%) compared to the 73,137 hours of total walltime.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage4.png
:figclass: technote-wide-content
Integrated walltime per RSS range for Stage 4 processes depassing 6GB.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~2,893 hours of walltime (~4%) compared to the 73,137 hours of total walltime.
```


## 4. Conclusion

The analysis demonstrates that reducing memory allocation to 6 GB per core is feasible for the majority of pipetasks across all stages. Less than 0.3% of quanta exceed the 6 GB threshold, with only a few pipetasks consistently requiring more memory. 
The cumulative plot in the following figures shows the proportion of pipetasks that consume up to a specified memory threshold (Max RSS). This allows us to evaluate how many pipetasks exceed the critical 6 GB threshold and identify the percentile at which most tasks operate below this threshold.

As shown, 99.9% of pipetasks in Stage 1 use less than 6 GB of memory (Figure 6). Only 0.007% exceed this threshold, indicating that reducing memory allocation to 6 GB per core is feasible for the majority of operations. The green dashed line represents the 95th percentile, which is approximately 3 GB.

```{figure} ./images/MaxRSS_CDF_v30_stage1.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 1 quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB threshold. 
```
As shown in the next Figure,  99% of pipetasks in Stage 2 use less than 6 GB of memory (Figure 6). Only 0.007% exceed this threshold, indicating that reducing memory allocation to 6 GB per core is feasible for the majority of operations. The green dashed line represents the 95th percentile, which is approximately 3.8 GB.

```{figure} ./images/MaxRSS_CDF_v30_stage2.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 2 quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB threshold. 
```
When comparing cumulative plots between Stage 1 and Stage 3 (Figures 6 and 22), Stage 3 shows a slightly higher fraction of pipetasks exceeding 6 GB (0.15% vs. 0.007%). However, the majority of tasks still remain below the critical threshold, supporting the feasibility of a 6 GB per core allocation.

```{figure} ./images/MaxRSS_CDF_v30_stage3.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 3  quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB threshold. 
```
Also for the tage 4, cumulative plots confirm that reducing memory allocation to 6 GB per core would suffice for over 99% of pipetasks.

```{figure} ./images/MaxRSS_CDF_v30_stage4.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 4 quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB threshold. 
```

The following table presents the number of quanta exceeding the GB threshold for each campaign, indicating that:

* Stage 1: 6 GB per core is sufficient for 99.994 % of the quanta in terms of the number of jobs, and for 99.7 % of the quanta in terms of walltime.
* Stage 2: 6 GB per core is sufficient for 99.7 % of the quanta in terms of the number of jobs, and for 75 % of the quanta in terms of walltime.
* Stage 3: 6 GB per core is sufficient for 99.85 % of the quanta in terms of the number of jobs, and for 99.85% of the quanta in terms of wall‑time.
* Stage 4: 6 GB per core is sufficient for 99.96 % of the quanta in terms of the number of jobs, and for 96 % of the quanta in terms of wall‑time.

```{rst-class} technote-wide-content
```
| Stage | Quanta | N. quanta > 6GB | % quanta > 6GB | Walltime (h) > 6GB |  % walltime > 6GB | Total Walltime| 
|-------|--------|---------------|--------------|--------------| --------------| --------------|
| Stage 1 | 574,472 | 36 | 0.006 | 3.3  | 0.3 | 10,045 |
| Stage 2 | 567,886 | 1916 | 0.3 | 2077 | 25| 8,267 |
| Stage 3 |1,308,272 | 1,979 | 0.15| 124 | 0.5 |25,973 | 
|Stage 4| 3,408,460 | 1,137 | 0.04 | 2,894 | 4 | 73,137 |


## References

```{bibliography}
```
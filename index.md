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

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage1_log.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 1 processes with y-axes in logaritmic scale.
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

How we can see in the above chart, differents tasks are using more than 6GB of RSS. The following charts visualize all the quanta using more than 6GB of memory grouped by pipetasks. Most of these cases involve the `analyzeRecalibratedStarAssociation` and `gaussianProcessesTurbulenceFit` pipetask.

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage2.png
:figclass: technote-wide-content

Number of runs (quanta) per pipetask that exceed the 6 GB memory threshold. 
```

The next two charts, showing integrated wall time per RSS range, further confirms that 6 GB per core is generally sufficient for almost all Stage 2 pipeline tasks but the `gaussianProcessesTurbulenceFit` pipetask consistently utilizes around 6 GB of memory for extended periods.

If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will mobilize some cores for ~2,100 hours compared (~25%) to ~8,300 hours total walltime. 
```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage2.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 2 processes.
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~2,100 hours of walltime compared to the 8,300 hours of total walltime.
```


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage2_log.png
:figclass: technote-wide-content

Integrated walltime per RSS range for Stage 2 processes with y-axes in logaritmic scale..
Dashed orange and red lines indicate the 4 GB and 6 GB memory thresholds, respectively.
Tasks that exceed 6 GB of RSS account for a total of ~2,100 hours of walltime compared to the 8,300 hours of total walltime.
```

### Stage3

For stage 3, there are significant differences between the software stack `w_2025_48` and `v30`. Consequently, a comparison becomes complicated. From now on, we no longer use the old stack in our analysis.

In the following chart, we observe that memory usage becomes more critical in this stage compared to previous stages and the regression in memory usage between `v30.0.0.rc2` and `v30.0.0` is confirmed. 

```{figure} ./images/MaxRSS_box_v30_stacks_stage3.png
:figclass: technote-wide-content

Box Plot comparing the stage 3 pipetask max RSS. 
The dashed red line show the 6GB threshold. 
```
We again focus on metrics from DM-53877 (`v30.0.0 rc3`). The next two charts show the RSS distribution in 1 GB increments for all quanta processed during Stage 3. A total of 1 308 272 quanta were processed, with 1979 (around 0.15%) exceeding the 6 GB threshold.  

```{figure} ./images/MaxRSS_distro_per_task_v30_stage3.png
:figclass: technote-wide-content

Distribution of maximum RSS per pipetask for the DM‑53877 (`v30.0.0‑rc3`) Stage 3 run.  
  The green dashed line marks the 95th percentile, the red dashed line the 6 GB threshold.  
  Bars are grouped in 1 GB bins.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage3.png
:figclass: technote-wide-content

Distribution of maximum RSS per pipetask for the DM‑53877 (`v30.0.0‑rc3`) Stage 3 run with the count on a logarithmic scale.  
The green dashed line marks the 95th percentile, the red dashed line the 6 GB threshold.  
Bars are grouped in 1 GB bins
```

How we can see in the above chart, differents tasks are using more than 6GB of RSS. The following charts visualize all the quanta using more than 6GB of memory grouped by pipetask. We can see that the pipetask `deblendCoaddFootprint` is the one exceeding more ofthen the 6GB threshold. 

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage3.png
:figclass: technote-wide-content

quanta exceeding 6GB threshold per pipetask. 
```

If we switch to 6GB/core worker nodes, these quanta, as shown in the next plot, will block a few cores for a few hundred hours compared to 40 000 hours of total wall time. 

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage3.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 3 process thresholded to quanta exceeding 6GB of memory. 
The dashed red line indicates the 6GB threshold.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage3.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 3 process. 
The dashed red line indicates the 6GB threshold.
```


### Stage 4 

For Stage 4 we don't have completed metrics for `w_2025_48` and `v30.0.0` stacks, so our analysis is solely based on `v30.0.0.rc2` stack. 

As shown in the following chart, we have few pipeatsk exceeding systematically the 6GB threshold. 

```{figure} ./images/MaxRSS_box_all_stacks_stage4.png
:figclass: technote-wide-content

Box Plot comparing the stage 4 pipetask max RSS. 
The dashed red line show the 6GB threshold. 
```

However, as demonstrated in the next two charts, the number of quanta (quanta) exceeding the thresholds is quite minimal compared to the total: in fact, we had 72 quanta exceeding the threshold and 2 321 953 quanta processed. 

```{figure} ./images/MaxRSS_distro_per_task_v30_stage4.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipetask executed during the DM-53719 stage 4. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB threshold.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage4.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipetask executed during the DM-53719 Stage 4 with the count on a logarithmic scale. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB threshold.
```
If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will block a few cores for ~60 hours compared to ~63 000 hours of total walltime. 

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage4.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 4 process thresholded to quanta exceeding 6GB of memory. 
The dashed red line indicates the 6GB threshold.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage4.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 3 process. 
The dashed red line indicates the 6GB threshold.
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

* Stage 1: 6 GB/core is sufficient for 99.993% of quanta.
* Stage 2: 6 GB/core is sufficient for 99.7% of quanta.
* Stage 3: 6 GB/core is sufficient for 99.85% of quanta.
* Stage 4: 6 GB/core is sufficient for 99.997% of quanta.
 

```{rst-class} technote-wide-content
```
| Stage | Quanta | N. quanta > 6GB | % quanta > 6GB| 
|-------|--------|---------------|--------------|
| Stage 1 | 574472 | 40 | 0.007 |
| Stage 2 | 565778 | 1706 | 0.3 |
| Stage 3 |1308272 | 1979 | 0.15| 
|Stage 4| 2321953| 72 | 0.003 | 


## References

```{bibliography}
```
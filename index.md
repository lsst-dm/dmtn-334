# Characterizing Memory Requirements for Large-Scale LSSTCam Productions at FrDF

```{abstract}

This technical note analyses memory‑usage patterns of `pipetask` during large‑scale LSSTCam production campaigns at CC‑IN2P3 (FrDF). We compare three campaigns (DM‑53368, DM‑53719, DM‑53877) that employed three different software‑stack releases: `w_2025_48`, `v30.0.0rc2` and `v30.0.0 (rc3)`. The results indicate that, for the majority of tasks, a memory budget of **6 GB per core** is sufficient, with only a small fraction of quanta exceeding this limit. Consequently, reducing the per‑core RAM from the current 10 GB to 6 GB appears feasible and could generate significant cost savings.

```

# Characterizing Memory Requirements for Large-Scale LSSTCam Productions at FrDF

## Introduction

The purpose of this technical note is to study the memory usage patterns of `pipetask` to assess future RAM requirements for worker nodes. Currently, worker nodes are provisioned with 10 GB of memory per core. Analyzing metrics from recent large-scale tests at CC-IN2P3 can provide valuable insight into whether reducing the memory per core to 6 GB would be both feasible and cost-effective.

We analyse memory‑usage metrics collected by the Butler for three large‑scale test campaigns at the French Data Facility (FrDF - CC‑IN2P3):

```{rst-class} technote-wide-content
```
| Campaign | Software stack | Visits | Fields |
|----------|----------------|--------|--------|
| DM‑53368 | `w_2025_48`    | 4 721  | WIDE, DDF, COSMOS, M49 |
| DM‑53719 | `v30.0.0rc2`   | 1 061  | COSMOS, EDFS |
| DM‑53877 | `v30.0.0 (rc3)`| 1 067  | COSMOS, EDFS |

*DM‑53368* is a large‑scale test that suffered hardware failures, resulting in incomplete Stage 4 metrics. *DM‑53719* and *DM‑53877* are pilot runs for the upcoming DP2 production at USDF.
*DM‑53877* encountered an issue with a pipetask and was terminated at step 4a.  

Our goal is to determine whether a **6 GB / core** limit would be sufficient for each pipeline stage.

## Comparison and Results

We analyze each campaign stage-by-stage, using DM-53368 primarily to illustrate the improvements introduced in the `v30` releases.

A few words about the nomenclature: in the following pages and charts, we refer to each pipetask execution interchangeably as 'quanta' or 'runs'.

### Stage 1 

The figure below shows the marked reduction in RSS peak  when moving from the `w_2025_48` stack to the `v30` releases. 

All tasks, **except `analyzeSingleVisitStarAssociation`**, stay below the 6 GB threshold when using the `v30` stacks.

A modest increase in RSS is visible between `v30.0.0‑rc2` and `v30.0.0`.

```{figure} ./images/MaxRSS_box_all_stacks_stage1.png
:figclass: technote-wide-content

Box plot comparing the maximum RSS of Stage 1 pipetask across three software stack releases. The red dashed line indicates the 6 GB memory limit.
The dashed red line show the 6GB limit. 
```

Fron now on, analysis focuses on the DM‑53877 campaign, specifically the stack `v30.0.0‑rc3`.

The next two charts illustrate the distribution of RSS in 1 GB increments for all quanta processed during Stage 1. A total of 574 472 quanta were processed; only 40 (≈ 0.007 %) exceeded the 6 GB limit.


```{figure} ./images/MaxRSS_distro_per_task_v30_stage1.png
:figclass: technote-wide-content

Distribution of maximum RSS per pipetask for the DM‑53877 (`v30.0.0‑rc3`) Stage 1 run.  
  The green dashed line marks the 95th percentile, the red dashed line the 6 GB threshold.  
  Bars are grouped in 1 GB bins.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage1.png
:figclass: technote-wide-content

Distribution of maximum RSS per pipetask for the DM‑53877 (`v30.0.0‑rc3`) Stage 1 run with the count on a logarithmic scale.  
The green dashed line marks the 95th percentile, the red dashed line the 6 GB threshold.  
Bars are grouped in 1 GB bins
```
The majority of these exceeding quanta are attributed to the `analyzeSingleVisitStarAssociation` pipetask as shown in the next chart.

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage1.png
:figclass: technote-wide-content

quanta exceeding 6GB limit per pipetask. 
```

The next two charts, showing integrated wall time per RSS range, further confirms that 6 GB per core is generally sufficient for almost all Stage 1 pipeline.


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage1.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for Stage 1 process limited to quanta exceeding 6GB of memory. 
The dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage1.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for Stage 1 process. 
The dashed red line indicates the 6GB limit.
```

### Stage 2 

For Stage 2, the improvements from `w_2025_48` are confirmed, though a few more pipetask now require more than 6 GB of RSS, as shown in the following box plots.

The increase in memory usage between `v30.0.0.rc2` and `v30.0.0` is also confirmed for this stage. 


```{figure} ./images/MaxRSS_box_all_stacks_stage2.png
:figclass: technote-wide-content

Box Plot comparing the Stage 2 pipetask max RSS. All the stack are compared.  
The dashed red line show the 6GB limit. 
```

```{figure} ./images/MaxRSS_box_v30_stacks_stage2.png 
:figclass: technote-wide-content

Box Plot comparing the Stage 2 pipetask max RSS. Only v30 stacks are compared.  
The dashed red line show the 6GB limit. 
```

We again focus on metrics from DM-53877 (`v30.0.0 rc3`). The next two charts show the RSS distribution in 1 GB increments for all quanta processed during Stage 2. A total of 565 778 quanta were processed, with 1706 (around 0.3%) exceeding the 6 GB limit. 

```{figure} ./images/MaxRSS_distro_per_task_v30_stage2.png
:figclass: technote-wide-content

Distribution of maximum RSS per pipetask for the DM‑53877 (`v30.0.0‑rc3`) Stage 2 run.  
  The green dashed line marks the 95th percentile, the red dashed line the 6 GB threshold.  
  Bars are grouped in 1 GB bins.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage2.png
:figclass: technote-wide-content

Distribution of maximum RSS per pipetask for the DM‑53877 (`v30.0.0‑rc3`) Stage 2 run with the count on a logarithmic scale.  
The green dashed line marks the 95th percentile, the red dashed line the 6 GB threshold.  
Bars are grouped in 1 GB bins
```

How we can see in the above chart, differents tasks are using more than 6GB of RSS. The following charts visualize all the quanta using more than 6GB of memory grouped by pipetasks. Most of these cases involve the `analyzeRecalibratedStarAssociation` and `gaussianProcessesTurbulenceFit` pipetask.

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage2.png
:figclass: technote-wide-content

quanta exceeding 6GB limit per pipetask. 
```

The next two charts, showing integrated wall time per RSS range, further confirms that 6 GB per core is generally sufficient for almost all Stage 2 pipeline tasks but the `gaussianProcessesTurbulenceFit` pipetask consistently utilizes around 6 GB of memory for extended periods.

If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will block few cores for ~1900 hours compared to ~8800 hours total walltime. 
```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage2.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for Stage 2 process limited to quanta exceeding 6GB of memory. 
The dashed red line indicates the 6GB limit.
```


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage2.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for Stage 2 process. 
The dashed red line indicates the 6GB limit.
```

### Stage3

For stage 3, there are significant differences between the software stack `w_2025_48` and `v30`. Consequently, a comparison becomes complicated. From now on, we no longer use the old stack in our analysis.

In the following chart, we observe that memory usage becomes more critical in this stage compared to previous stages and the regression in memory usage between `v30.0.0.rc2` and `v30.0.0` is confirmed. 

```{figure} ./images/MaxRSS_box_v30_stacks_stage3.png
:figclass: technote-wide-content

Box Plot comparing the stage 3 pipetask max RSS. 
The dashed red line show the 6GB limit. 
```
We again focus on metrics from DM-53877 (`v30.0.0 rc3`). The next two charts show the RSS distribution in 1 GB increments for all quanta processed during Stage 3. A total of 1 308 272 quanta were processed, with 1979 (around 0.15%) exceeding the 6 GB limit.  

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

How we can see in the above chart, differents tasks are using more than 6GB of RSS. The following charts visualize all the quanta using more than 6GB of memory grouped by pipetask. We can see that the pipetask `deblendCoaddFootprint` is the one exceeding more ofthen the 6GB limit. 

```{figure} ./images/MaxRSS_stripplot_per_task_v30_upper6_stage3.png
:figclass: technote-wide-content

quanta exceeding 6GB limit per pipetask. 
```

If we switch to 6GB/core worker nodes, these quanta, as shown in the next plot, will block a few cores for a few hundred hours compared to 40 000 hours of total wall time. 

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage3.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 3 process limited to quanta exceeding 6GB of memory. 
The dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage3.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 3 process. 
The dashed red line indicates the 6GB limit.
```


### Stage 4 

For Stage 4 we don't have completed metrics for `w_2025_48` and `v30.0.0` stacks, so our analysis is solely based on `v30.0.0.rc2` stack. 

As shown in the following chart, we have few pipeatsk exceeding systematically the 6GB limit. 

```{figure} ./images/MaxRSS_box_all_stacks_stage4.png
:figclass: technote-wide-content

Box Plot comparing the stage 4 pipetask max RSS. 
The dashed red line show the 6GB limit. 
```

However, as demonstrated in the next two charts, the number of quanta (quanta) exceeding the limits is quite minimal compared to the total: in fact, we had 72 quanta exceeding the limit and 2 321 953 quanta processed. 

```{figure} ./images/MaxRSS_distro_per_task_v30_stage4.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipetask executed during the DM-53719 stage 4. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage4.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipetask executed during the DM-53719 Stage 4 with the count on a logarithmic scale. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```
If we switch to 6GB/core workernodes, these quanta, as shown in the next plot, will block a few cores for ~60 hours compared to ~63 000 hours of total walltime. 

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_upper6_stage4.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 4 process limited to quanta exceeding 6GB of memory. 
The dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage4.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 3 process. 
The dashed red line indicates the 6GB limit.
```



## 4. Conclusion

The analysis demonstrates that reducing memory allocation to 6 GB per core is feasible for the majority of pipetasks across all stages. Less than 0.3% of quanta exceed the 6 GB limit, with only a few pipetasks consistently requiring more memory. 
The cumulative plot in the following figures shows the proportion of pipetasks that consume up to a specified memory limit (Max RSS). This allows us to evaluate how many pipetasks exceed the critical 6 GB threshold and identify the percentile at which most tasks operate below this limit.

As shown, 99.9% of pipetasks in Stage 1 use less than 6 GB of memory (Figure 6). Only 0.007% exceed this threshold, indicating that reducing memory allocation to 6 GB per core is feasible for the majority of operations. The green dashed line represents the 95th percentile, which is approximately 3 GB.

```{figure} ./images/MaxRSS_CDF_v30_stage1.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 1 quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB limit. 
```
As shown in the next Figure,  99% of pipetasks in Stage 2 use less than 6 GB of memory (Figure 6). Only 0.007% exceed this threshold, indicating that reducing memory allocation to 6 GB per core is feasible for the majority of operations. The green dashed line represents the 95th percentile, which is approximately 3.8 GB.

```{figure} ./images/MaxRSS_CDF_v30_stage2.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 2 quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB limit. 
```
When comparing cumulative plots between Stage 1 and Stage 3 (Figures 6 and 22), Stage 3 shows a slightly higher fraction of pipetasks exceeding 6 GB (0.15% vs. 0.007%). However, the majority of tasks still remain below the critical threshold, supporting the feasibility of a 6 GB per core allocation.

```{figure} ./images/MaxRSS_CDF_v30_stage3.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 3  quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB limit. 
```
Also for the tage 4, cumulative plots confirm that reducing memory allocation to 6 GB per core would suffice for over 99% of pipetasks.

```{figure} ./images/MaxRSS_CDF_v30_stage4.png 
:figclass: technote-wide-content

Cumulative fraction of Stage 4 quanta per Max RSS. The green dashed line represents the 95% of total quanta, and the dashed red line indicates the 6GB limit. 
```

The following table presents the number of quanta exceeding the GB limit for each campaign, indicating that:

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



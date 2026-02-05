# Characterizing Memory Requirements for Large-Scale LSSTCam Productions at FrDF

```{abstract}

This technical note analyzes memory usage patterns during large-scale LSSTCam production campaigns at CC-IN2P3 (FrDF) to assess future RAM requirements for worker nodes. The analysis compares three campaigns with different software stack releases and suggests that reducing memory per core to 6 GB may be feasible.
```

# Technical Note: Comparison of Three Campaigns with Different Software Stack Releases

## Introduction

The purpose of this technical note is to study the memory usage patterns of `pipetask` to assess future RAM requirements for worker nodes. Currently, worker nodes are provisioned with 10 GB of memory per core. Analyzing metrics from recent large-scale tests at CC-IN2P3 can provide valuable insight into whether reducing the memory per core to 6 GB would be both feasible and cost-effective.

We compare metrics extracted via the Butler for each `pipetask` across three campaigns: DM-53368, DM-53719, and DM-53877. These campaigns, described and summarized in the following section, used three different software stack releases: `w_2025_48` for DM-53368, `v30.0.0rc2` for DM-53719, and `v30.0.0 (rc3)` for DM-53877.

The details of the campaigns are reported in the following table.

```{rst-class} technote-wide-content
```

| CAMPAIGN  | WEEKLY         | N. OF VISIT     | FIELDS                |
|-----------|----------------|-----------------|-----------------------|
| DM-53368  | w_2025_48      | 4721            | WIDE, DDF, COSMOS, M49|
| DM-53719  | v30.0.0rc2     | 1061            | COSMOS, EDFS          |
| DM-53877  | v30.0.0 (rc3)  | 1067            | COSMOS, EDFS          |

- **DM-53368:** A large-scale test covering multiple fields but affected by hardware failures, resulting in incomplete metrics for Stage 4.
- **DM-53719:** A pilot campaign to pre-test the software for the official DP2 production at USDF.
- **DM-53877:** A pilot campaign to pre-test the software for the official DP2 production at USDF.


## Comparison and Results

We analyze each campaign stage-by-stage, using DM-53368 primarily to illustrate the improvements introduced in the `v30` releases.

### Stage 1 

The plot below clearly demonstrates the significant improvement achieved by moving from the `w_2025_48` stack to the `v30` stack. Notably, except for the `analyzeSingleVisitStarAssociation` pipetask, all tasks are now well within the 6 GB limit.


```{figure} ./images/MaxRSS_box_all_stacks_stage1.png
:figclass: technote-wide-content

Box Plot comparing the stage 1 pipetask max RSS. 
The dashed red line show the 6GB limit. 
```

From now on we focus on metrics from DM-53877 campaign (stack `v30.0.0 rc3`).
The next two charts illustrate the distribution of RSS in 1 GB increments for all quanta processed during Stage 1. A total of 574,472 quanta were processed, with only 40 (approximately 0.007%) exceeding the 6 GB memory limit. The majority of these occurrences are attributed to the `analyzeSingleVisitStarAssociation` pipetask.


```{figure} ./images/MaxRSS_distro_per_task_v30_stage1.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipesk executed during the DM-53877 stage 1. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage1.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipesk executed during the DM-53877 stage 1 with the count on a logarithmic scale. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```
The following chart, showing integrated wall time per RSS range, further confirms that, globally, worker nodes with 6 GB/core can easily handle almost all Stage 1 pipelines.

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage1.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 1 process. 
The dashed red line indicates the 6GB limit.
```

As shown in the cumulative plot below, almost 100% of Stage 1 runs required less than 6 GB of RSS.

```{figure} ./images/MaxRSS_CDF_v30_stage1.png 
:figclass: technote-wide-content

Cumulative fraction of runs per Max RSS. The green dashed line represents the 95% of total runs, and the dashed red line indicates the 6GB limit. 
```
### Stage 2 

For Stage 2, the improvements from `w_2025_48` are confirmed, though a few more pipetasks now require more than 6 GB of RSS, as shown in the following box plots.

```{figure} ./images/MaxRSS_box_all_stacks_stage2.png
:figclass: technote-wide-content

Box Plot comparing the stage 2 pipetask max RSS. All the stack are compared.  
The dashed red line show the 6GB limit. 
```

```{figure} ./images/MaxRSS_box_v30_stacks_stage2.png 
:figclass: technote-wide-content

Box Plot comparing the stage 2 pipetask max RSS. Only v30 stacks are compared.  
The dashed red line show the 6GB limit. 
```

We again focus on metrics from DM-53877 (`v30.0.0 rc3`). The next two charts show the RSS distribution in 1 GB increments for all quanta processed during Stage 2. A total of 565,778 quanta were processed, with 1,706 (around 0.3%) exceeding the 6 GB limit. Most of these cases involve the `analyzeRecalibratedStarAssociation` pipetask.

```{figure} ./images/MaxRSS_distro_per_task_v30_stage2.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipesk executed during the DM-53877 stage 2. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage2.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipesk executed during the DM-53877 stage 1 with the count on a logarithmic scale. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```
The next chart, showing integrated wall time per RSS range, further confirms that 6 GB per core is generally sufficient for almost all Stage 2 pipeline tasks. Note that the `gaussianProcessesTurbulenceFit` pipetask consistently utilizes around 6 GB of memory for extended periods.


```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage2.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS for stage 2 process. 
The dashed red line indicates the 6GB limit.
```

Reflecting previous results, nearly all Stage 2 runs required less than 6 GB RSS, as shown below.

```{figure} ./images/MaxRSS_CDF_v30_stage2.png 
:figclass: technote-wide-content

Cumulative fraction of runs per Max RSS. The green dashed line represents the 95% of total runs, and the dashed red line indicates the 6GB limit. 
```

### Stage3


```{figure} ./images/MaxRSS_box_all_stacks_stage3.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_stage3.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage3.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage3.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_CDF_v30_stage3.png 
:figclass: technote-wide-content

My plot.
```

### Stage 4 


```{figure} ./images/MaxRSS_box_all_stacks_stage4.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_stage4.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage4.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage4.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_CDF_v30_stage4.png 
:figclass: technote-wide-content

My plot.
```


## 4. Conclusion


```{rst-class} technote-wide-content
```

| Stage | Quanta | N. runs > 6GB | % runs > 6GB| 
|-------|--------|---------------|--------------|
| Stage 1 | 574472 | 40 | 0.007 |
| Stage 2 | 565778 | 1706 | 0.3 |



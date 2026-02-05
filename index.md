# Characterizing Memory Requirements for Large-Scale LSSTCam Productions at FrDF

```{abstract}

This technical note analyzes memory usage patterns during large-scale LSSTCam production campaigns at CC-IN2P3 (FrDF) to assess future RAM requirements for worker nodes.
The analysis compares three campaigns with different software stack releases and suggests that reducing memory per core to 6GB may be feasible.
```

# Technical Note: Comparison of Three Campaigns with Different Software Stack Releases

## Introduction

The purpose of this technical note is to study the memory usage patterns of pipetask to assess the future RAM requirements for worker nodes. We are currently purchasing worker nodes with 10GB of memory per core. Analyzing the metrics from our recent large-scale test at CC-IN2P3 could provide valuable insights into whether reducing the memory per core to 6GB would be feasible and cost-effective. 

In this technical note, we compare the metrics extracted from the Butler for each pipetask across three campaigns: DM-53368, DM-53719 and DM-53877. These campaigns described and summarized in the following section, used three differents software stack: `w_2025_48` for DM-53368, `v30.0.0rc2` for DM-53719 and `v30.0.0 (rc3)` for DM-53877.  

The details of the executed campaign are reported in the following table. 

```{rst-class} technote-wide-content
```

|CAMPAIGN | WEEKLY | N. OF VISIT| FIELDS | 
|---------|--------|------------|--------| 
|DM-53368 | w_2025_48 | 4721|WIDE,DDF,COSMOS,M49| 
|DM-53719 | v30.0.0.rc2 |1061 |COSMOS,EDFS |
|DM-53877 | v30.0.0 (rc3) |1067 |COSMOS,EDFS | 

*  DM-53368: A large-scale test covering multiple fields, but encountered hardware failures, resulting in incomplete metrics for Stage 4.
* DM-53719: A pilot campaign to test software in advance on official DP2 production executed at USDF.
* DM-53877: A pilot campaign to test software in advance on official DP2 production executed at USDF.


## Comparison and Results

We are going analyze each campaign stage by stage but we use results from DM-53368 only to show the improvement intriduced on v30 versions.  


### Stage 1 

In the provided plot, we can clearly observe the significant improvement introduced between the `w_2025_48` stack and the `v30` stack. Notably, if we exclude the `analyzesingleVisitStarAssociation` pipetask, all other tasks are now well within the 6GB limit. 

```{figure} ./images/MaxRSS_box_all_stacks_stage1.png
:figclass: technote-wide-content

Box Plot comparing the stage 1 pipetask max RSS. 
The dashed red line show the 6GB limit. 
```

From now on we focus on metrics from DM-53877 campaign (stack `v30.0.0 rc3`).
The two following charts illustrate the distribution of RSS in 1 GB increments for all quanta processed during Stage 1 of the DM-53877 campaign.
A total of 574472 quanta were processed, with only 40 (approximately 0.007%) exceeding the 6 GB memory limit. 
The majority of these cases were attributed to the `analyzeSingleVisitStarAssociation` pipetask.

```{figure} ./images/MaxRSS_distro_per_task_v30_stage1.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipesk executed during the DM-53877 stage 1. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage1.png
:figclass: technote-wide-content

The distribution of the RSS for all the pipesk executed during the DM-53877 stage 1 with the count on a logarithmic scale. The green dashed line represents the 95th percentile, and the dashed red line indicates the 6GB limit.
```

The next chart, which displays the integrated waltime per range of RSS, further confirms that globally, 6GB/core workers can easily handle almost all the stage 1 pipelines. 

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage1.png
:figclass: technote-wide-content

Integrated Wall Time per Range of RSS. 
The dashed red line indicates the 6GB limit.
```

Finally, we can observe here all that we had previously mentioned: almost 100% of the stage 1 runs required less than 6GB of RSS.

```{figure} ./images/MaxRSS_CDF_v30_stage1.png 
:figclass: technote-wide-content

Cumulative fraction of runs per Max RSS. The green dashed line represents the 95% of total runs, and the dashed red line indicates the 6GB limit. 
```
### Stage 2 


```{figure} ./images/MaxRSS_box_all_stacks_stage2.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_stage2.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_per_task_v30_log_stage2.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_distro_CumulatedCPU_v30_stage2.png
:figclass: technote-wide-content

My plot.
```

```{figure} ./images/MaxRSS_CDF_v30_stage2.png 
:figclass: technote-wide-content

My plot.
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
>w




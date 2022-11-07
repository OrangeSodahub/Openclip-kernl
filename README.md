## Openclip-CUDAGraph
Tips: There are three steps from the original model to final callable optimized model:
1. Trace the torch model through `torch.fx._symbolic_trace` to generate `torch.fx.GraphModuel` object.
2. Replace some modules in this object to optimize it **(not included in this version)** and output is a `torch.fx.GraphModuel` object.
3. Build static graph model through `torch.cuda.CUDAGraph`.
And also not use `torchdynamo.optimize()`.

### Benchmark

Tested Model (on RTX3080): ViT-L-14::laion2b-s32b-b82k

<p float="left">
    <img src=./assets/1.png width=60%>
</p>


**RES**: Since data needs to be created on the CPU and the moved to the GPU, the value of RES will get a peak when the program starts and then decrease. Here record the stable value after the decrease.

**graph**: `[torch dynamo]/[symbolic trace]` for torch dynamo's time, it includes building graphs and inference while for symbolic trace, building graph's cost is not been counted. It saves the time cost on starting the model multiply times. When N gets larger and batch_size gets smaller, it performs better.

--------------------------------------------------------------------------------------------
times   |shape           | pt (s)   | graph (s)         | speed up | GPU(MB)      | RES(MB)
--------|----------------|----------|-------------------|----------|--------------|---------
|1      |(1, 77)         |0.0068    |1.1747/0.00402     |0.00575   |3105/1643     |2515/3539
|       |(2, 77)         |0.0073    |1.1755/0.00503     |0.00605   |3101/1705     |2511/3534
|       |(4, 77)         |0.0070    |1.1780/0.00642     |0.00583   |3163/1807     |2510/3543
|       |(8, 77)         |0.0071    |1.2095/0.01075     |0.00575   |3443/1991     |2512/3538
|       |(16, 77)        |0.0087    |1.1905/0.01882     |0.00796   |3693/2419     |2511/3548
|       |(1, 3, 224, 224)|0.0134    |3.9472             |0.00339   |              |
|       |(2, 3, 224, 224)|0.0352    |4.0213             |0.00875   |              |
|       |(4, 3, 224, 224)|0.0323    |4.0170             |0.00804   |              |
|       |(8, 3, 224, 224)|0.0416    |4.1213             |0.01009   |              |
|100    |(1, 77)         |0.0645    |1.4387/0.27083     |0.44370   |3105/1643     |2513/3546
|       |(2, 77)         |0.0749    |1.4735/0.36702     |0.44843   |3101/1705     |2510/3547
|       |(4, 77)         |0.0659    |1.5138/0.52818     |0.43349   |3163/1807     |2511/3549
|       |(8, 77)         |0.0672    |1.6625/0.97432            |0.42214   |3443/1991     |2510/3546
|       |(16, 77)        |0.0855    |2.1333/1.82640     |0.37915   |3693/2419     |2511/3551
|       |(1, 3, 224, 224)|0.1328    |3.9772             |0.03339   |              |
|       |(2, 3, 224, 224)|0.1507    |4.0623             |0.03709   |              |
|       |(4, 3, 224, 224)|0.3066    |4.2564             |0.07203   |              |
|       |(8, 3, 224, 224)|0.3949    |4.4650             |0.08844   |              |
|1000   |(1, 77)         |6.3198    |3.7619/2.59695     |1.67443   |3105/1643     |2510/3544
|       |(2, 77)         |6.5773    |4.1487/3.61783     |1.55306   |3101/1705     |2508/3546
|       |(4, 77)         |6.6337    |4.3992/5.28708     |1.47605   |3163/1807     |2510/3539
|       |(8, 77)         |6.7029    |5.7641/9.82082     |1.13833   |3443/1991     |2511/3545
|       |(16, 77)        |8.0548    |7.8367/18.50580    |0.82446   |3693/2419     |2511
|       |(1, 3, 224, 224)|13.0253   |11.9307            |1.09174   |              |
|       |(2, 3, 224, 224)|15.0432   |17.7647            |0.84680   |              |
|       |(4, 3, 224, 224)|-         |-                  |-         |              |
|10000  |(1, 77)         |63.0495   |18.0050/26.30049   |3.50177   |3105/1643     |2510/3546
|       |(2, 77)         |65.4867   |21.7872/36.66389   |3.00574   |3101/1705     |2510/3549
|       |(4, 77)         |64.9916   |24.3028/53.85116   |2.67424   |3163/1887     |2511/3541
|       |(8, 77)         |65.7108   |37.9319/100.29747  |1.73254   |3443/1991     |2511/3540
|       |(16, 77)        |81.8811   |77.0026/187.34221  |1.06335   |3693/2419     |2512
|       |(1, 3, 224, 224)|-         |-                  |-         |         |
|       |(2, 3, 224, 224)|-         |-                  |-         |         |
|       |(4, 3, 224, 224)|-         |-                  |-         |         |

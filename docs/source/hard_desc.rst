Hardware Description
====================


The Betif GPU machine is a high-end computational server engineered for AI workloads, HPC simulations, and data-intensive tasks. This powerhouse integrates cutting-edge CPU, GPU, network, and storage technologies to deliver exceptional performance, scalability, and throughput.


Processor Configuration
-----------------------


* **CPUs** 2 × Intel\ :sup:`®` Xeon\ :sup:`®` Platinum 8462Y+ (Sapphire Rapids)
* **Total Cores/Threads** 64 cores / 128 threads
* **Base Frequency** 2.80 GHz
* **Max Turbo Frequency** 4.10 GHz
* **Cache** 60 MB L3 per CPU
* **TDP** 300 W per CPU
* **Memory Support**
        * Max Capacity: 4 TB per CPU
        * Memory Types: DDR5 (Up to 4800 MT/s @ 1DPC, 4400 MT/s @ 2DPC)
        * Memory Channels: 8 per CPU
        * ECC Support: Yes
* **Launch Date** Q1 2023


Graphics Processing Units
-------------------------


* **GPUs** 2 × NVIDIA H100
* **FP64 Performance** 30 TFLOPS
* **FP32 Performance** 60 TFLOPS
* **Memory** 94 GB HBM3
* **Memory Bandwidth** 3.9 TB/s
* **Decoders** 7 NVDEC + 7 JPEG
* **TDP** 350 W per GPU
* **Interface** PCIe Gen5 (128 GB/s)


Memory
------

* **Installed RAM** 2.0 TB DDR5 ECC


Networking
----------

* **Ethernet** 2 × Intel X710 Dual Port SFP+
   * Link Speed: 10 Gbps per port
   * Protocol: 10 Gigabit Ethernet
* **High-Speed Interconnect** 2 × Mellanox MT2892 (ConnectX-6 Dx)
     * Data Rate: 40 Gbps each

Storage
-------

* **Drives** 4 × Intel D7-P5520 NVMe SSDs
* **Capacity** 15.36 TB each
* **Interface** PCIe 4.0 x4 NVMe
* **Performance**
    * Sequential Read: 7100 MB/s
    * Sequential Write: 3700 MB/s
    * Random 4KB Read: 1,000,000 IOPS
    * Random 4KB Write: 200,000 IOPS
* **Endurance** 28.0 PBW
* **Power** 20 W active / 5 W idle



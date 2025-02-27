<!--
SPDX-FileCopyrightText: Copyright (c) 2023, NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# DOCA Sensitive Information Detection Example
## Obtaining the Morpheus DOCA Container
DOCA Support is in early access and may only be used via the Morpheus DOCA Container found in NGC. Please speak to your NVIDIA Morpheus contact for more information.

The container must be run in privileged mode and mount in hugepages as configured according to the DOCA GPUNetIO documentation.

```
docker run -v /dev/hugepages:/dev/hugepages --privileged --rm -ti --runtime=nvidia --net=host --gpus=all --cap-add=sys_nice ${MORPHEUS_DOCA_IMAGE} bash
```

## Finding the GPU and NIC PCIe Addresses
The DOCA example requires specifying the PCIe Address of both the GPU and NIC explicitly. Determining the correct GPU and NIC PCIe Addresses is non-trivial and requires coordinating with those who have configured the physical hardware and firmware according to the DOCA GPUNetIO documentation, but the following commands can help find a NIC and GPU situation on the same NUMA node.
```
$ lspci -tv | grep -E "NVIDIA|ella|(^\+)|(^\-)"
-+-[0000:ff]-+-00.0  Intel Corporation Device 344c
 |           \-02.0-[ca-cf]----00.0-[cb-cf]--+-00.0-[cc]--+-00.0  Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller
 |                                           |            +-00.1  Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller
 |                                           |            \-00.2  Mellanox Technologies MT42822 BlueField-2 SoC Management Interface
 |                                           \-01.0-[cd-cf]----00.0-[ce-cf]----08.0-[cf]----00.0  NVIDIA Corporation Device 20b9
 |           \-02.0-[b1]--+-00.0  Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller
 |                        +-00.1  Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller
 |                        \-00.2  Mellanox Technologies MT42822 BlueField-2 SoC Management Interface
```
From the result we can assemble the PCIe addresses of the nearest GPU and NIC. But it will be easier to cross-reference them with the explicit PCIe addresses from these commands:
```
$ lspci | grep ella
b1:00.0 Ethernet controller: Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller (rev 01)
b1:00.1 Ethernet controller: Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller (rev 01)
b1:00.2 DMA controller: Mellanox Technologies MT42822 BlueField-2 SoC Management Interface (rev 01)
ca:00.0 PCI bridge: Mellanox Technologies MT42822 Family [BlueField-2 SoC PCIe Bridge] (rev 01)
cb:00.0 PCI bridge: Mellanox Technologies MT42822 Family [BlueField-2 SoC PCIe Bridge] (rev 01)
cb:01.0 PCI bridge: Mellanox Technologies MT42822 Family [BlueField-2 SoC PCIe Bridge] (rev 01)
cc:00.0 Ethernet controller: Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller (rev 01)
cc:00.1 Ethernet controller: Mellanox Technologies MT42822 BlueField-2 integrated ConnectX-6 Dx network controller (rev 01)
cc:00.2 DMA controller: Mellanox Technologies MT42822 BlueField-2 SoC Management Interface (rev 01)
cd:00.0 PCI bridge: Mellanox Technologies MT42822 Family [BlueField-2 SoC PCIe Bridge] (rev 01)
ce:08.0 PCI bridge: Mellanox Technologies MT42822 Family [BlueField-2 SoC PCIe Bridge] (rev 01)
```
```
$ lspci | grep NVIDIA
cf:00.0 3D controller: NVIDIA Corporation Device 20b9 (rev a1)
```
We can see the GPU's PCIe address is `cf:00.0`, and we can infer from the above commands that the nearest ConnectX-6 NIC's PCIe address is `cc:00.*`. In this case, we have port `1` physically connected to the network, so we use PCIe Address `cc:00.1`.

## Running the Example
The DOCA example is similar to the Sensitive Information Detection (SID) example in that it uses the `sid-minibert` model in conjunction with the `TritonInferenceStage` to detect sensitive information. The difference is that the sensitive information we will be detecting is obtained from a live TCP packet stream provided by a `DocaSourceStage`.

Prior to running the example, the `rdma-core` conda package needs to be _removed by force_ from the conda environment, otherwise the environment is incompatible with the DOCA-provided packages.
```
conda remove --force rdma-core
```

To run the example from the Morpheus root directory and capture all TCP network traffic from the given NIC, use the following command and replace the `nic_addr` and `gpu_addr` arguments with your NIC and GPU PCIe addresses.
```
# python examples/doca/run.py --nic_addr cc:00.1 --gpu_addr cf:00.0
```
```
====Registering Pipeline====
====Building Pipeline====
DOCA GPUNetIO rate: 0 pkts [00:00, ? pkt====Building Pipeline Complete!====
Deserialize rate: 0 pkts [00:00, ? pktsStarting! Time: 1689110835.1106102
EAL: Detected CPU lcores: 72, ? pkts/s]
EAL: Detected NUMA nodes: 200, ? pkts/s]
EAL: Detected shared linkage of DPDKs/s]
EAL: Multi-process socket /var/run/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'PA'
EAL: VFIO support initialized
TELEMETRY: No legacy callbacks, legacy socket not created
EAL: Probe PCI driver: mlx5_pci (15b3:a2d6) device: 0000:cc:00.1 (socket 1)
EAL: Probe PCI driver: gpu_cuda (10de:20b9) device: 0000:cf:00.0 (socket 1)
DOCA GPUNetIO rate: 0 pkts [00:03, ? pkts/s]====Registering Pipeline Complete!====
====Starting Pipeline====[00:02, ? pkts/s]
====Pipeline Started====0:02, ? pkts/s]
====Building Segment: linear_segment_0====
Added source: <from-doca-0; DocaSourceStage(nic_pci_address=cc:00.1, gpu_pci_address=cf:00.0)>
  └─> morpheus.MessageMeta
Added stage: <monitor-1; MonitorStage(description=DOCA GPUNetIO rate, smoothing=0.05, unit=pkts, delayed_start=False, determine_count_fn=None, log_level=LogLevels.INFO)>
  └─ morpheus.MessageMeta -> morpheus.MessageMeta
Added stage: <deserialize-2; DeserializeStage(ensure_sliceable_index=True)>
  └─ morpheus.MessageMeta -> morpheus.MultiMessage
Added stage: <monitor-3; MonitorStage(description=Deserialize rate, smoothing=0.05, unit=pkts, delayed_start=False, determine_count_fn=None, log_level=LogLevels.INFO)>
  └─ morpheus.MultiMessage -> morpheus.MultiMessage
Added stage: <preprocess-nlp-4; PreprocessNLPStage(vocab_hash_file=/workspace/models/training-tuning-scripts/sid-models/resources/bert-base-uncased-hash.txt, truncation=True, do_lower_case=True, add_special_tokens=False, stride=-1, column=data)>
  └─ morpheus.MultiMessage -> morpheus.MultiInferenceNLPMessage
Added stage: <monitor-5; MonitorStage(description=Tokenize rate, smoothing=0.05, unit=pkts, delayed_start=False, determine_count_fn=None, log_level=LogLevels.INFO)>
  └─ morpheus.MultiInferenceNLPMessage -> morpheus.MultiInferenceNLPMessage
Added stage: <inference-6; TritonInferenceStage(model_name=sid-minibert-onnx, server_url=localhost:8000, force_convert_inputs=True, use_shared_memory=True)>
  └─ morpheus.MultiInferenceNLPMessage -> morpheus.MultiResponseMessage
Added stage: <monitor-7; MonitorStage(description=Inference rate, smoothing=0.05, unit=pkts, delayed_start=False, determine_count_fn=None, log_level=LogLevels.INFO)>
  └─ morpheus.MultiResponseMessage -> morpheus.MultiResponseMessage
Added stage: <add-class-8; AddClassificationsStage(labels=None, prefix=, probs_type=TypeId.BOOL8, threshold=0.5)>
  └─ morpheus.MultiResponseMessage -> morpheus.MultiResponseMessage
Added stage: <monitor-9; MonitorStage(description=AddClass rate, smoothing=0.05, unit=pkts, delayed_start=False, determine_count_fn=None, log_level=LogLevels.INFO)>
  └─ morpheus.MultiResponseMessage -> morpheus.MultiResponseMessage
====Building Segment Complete!====
Stopping pipeline. Please wait... Press Ctrl+C again to kill.
DOCA GPUNetIO rate: 0 pkts [00:09, ? pkts/s]
Deserialize rate: 0 pkts [00:09, ? pkts/s]
Tokenize rate: 0 pkts [00:09, ? pkts/s] 
Inference rate: 0 pkts [00:09, ? pkts/s]
AddClass rate: 0 pkts [00:09, ? pkts/s]
```
The output can be found in `doca_output.csv`
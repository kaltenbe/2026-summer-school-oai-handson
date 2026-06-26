# 2026-summer-school-oai-handson
Collection of material for the 2026 summer school OAI hands on tutorial

## Prerequisites

For this tutorial you have the possibility to use Eurecom Jupyterhub cluster, which is part of the SLICES Research Infrastructure ([SLICES-RI](https://www.slices-ri.eu/)). This gives you access to some state of the art GPU accelerated compute infrastructure. It can be accessed from [here](https://jupyterhub.slices.cs.eurecom.fr/).

The jupyterhub is available to all SLICES partners using their home institutions SSO. You can check if your host institution is a partner [here](https://www.slices-ri.eu/community/). If yes you can create a SLICES account [here](https://portal.slices-ri.eu/signup). You can skip the step "Create or Join Project". More information [here](https://doc.slices-ri.eu/SupportingServices/getanaccount.html).

Once your SLICES account is created, you can sign in the jupyterhub by selecting "Sign in with Keycloak" and then slicesri. Please double check that this is working and let me know if not. 

If your home institution is not a SLICES partner, don't despair - just get in touch with me directly and I will give you access through the Eurecom gitlab. 

Some documentation of the jupyterhub cluster is [here](https://gitlab.eurecom.fr/jupyterhub-users/jupyterhub). 

If you prefer to use your own computer (laptop or server you have access to) you can do that as well. In this case just double check that your computer fulfills the [system requirements](https://github.com/duranta-project/openairinterface5g/blob/develop/doc/system_requirements.md). 

## General Resources

There is a lot of training material, tutorials and documentation available. A good starting point are the dedicated training [repository](https://gitlab.eurecom.fr/oai/trainings/oai-workshops) containing

1. [Core Network Hands-on Tutorial](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/cn/README.md)
2. [RAN Hands-on Tutorial](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/ran/README.md)
3. [OAM Hands-on Tutorial](https://gitlab.eurecom.fr/oai/trainings/oai-workshops/oam/README.md)

## Summer school tutorial (own deployment)
In this summer school we are going to follow the tutorial [UE positioning in a Digital Twin using Ray-Tracing Channel Emulator with OAI 5G Stack](https://github.com/duranta-project/openairinterface5g/blob/nrppa_mac_and_rrc_procedures/doc/UE_Positioning_in_DigitalTwin_Tutorial.md).

At the time of writing this tutorial and the associated code is in the branch nrppa_mac_and_rrc_procedures. So make sure you check out this branch

```bash
cd openairinterface5g
git checkout nrppa_mac_and_rrc_procedures
```

The branch will soon be merged into develop, after which the tutorial will be available [here](https://github.com/duranta-project/openairinterface5g/blob/develop/doc/UE_Positioning_in_DigitalTwin_Tutorial.md) 

The tutorial explains the whole end-to-end setup. If you are running the tutorial on the EURECOM jupyterhub cluster you need to make some modifications.

## Summer school tutorial (jupyterhub deployment)

First of all log into the Eurecom jupyterhub. 

Click on "Start My Server".

Select "OAI GPU Dev" from the section  "GPU Small (1g.18gb)" and click on "Start" (all the way at the bottom). 

### Compiling

Compile the openairinterface project like described in the tutorial. Additionally you may add `-DENABLE_CHANNEL_SIM_CUDA=ON` to your cmake command to accelerate the channel simulation using GPUs. You also have to set some CUDA compile options. 

```bash
./build_oai -w None --ninja --nrUE --gNB --cmake-opt -DOAI_VRTSIM_TAPS_CLIENT=ON --cmake-opt -DENABLE_CHANNEL_SIM_CUDA=ON  --cmake-opt -DCMAKE_CUDA_ARCHITECTURES=native  --cmake-opt -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc
```

### Configuration

When running on the Eurecom jupyterhub, we do not need to deploy the core network as we will be using the core network already deployed on the Eurecom openshift cluster. So you can skip parts 5.3 and 6.1 of the tutorial. 

For PLMN `00101` the AMF IP address of the core network is `172.21.6.4`. 

Since we are all using the same core network, it would be good to use different `gNB_name` and `nr_cellid`. Please coordinate this with others in the group.

Also you will need to figure out the IP address of your own container, which will be different from others. To do so type

```bash
ip a show dev oai0
```

Put all this information in the config file `targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.positioning.conf`. Here is an example diff

```diff
diff --git a/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.positioning.conf b/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.positioning.conf
index b7cbcddd7d..807a7d13ba 100644
--- a/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.positioning.conf
+++ b/targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.positioning.conf
@@ -1,4 +1,4 @@
-Active_gNBs = ( "gNB-OAI-DT");
+Active_gNBs = ( "gNB-OAI-name");
 # Asn1_verbosity, choice in: none, info, annoying
 Asn1_verbosity = "none";
 gNBs =
@@ -6,11 +6,11 @@ gNBs =
  {
     ////////// Identification parameters:
     gNB_ID    =  0xe00;
-    gNB_name  =  "gNB-OAI-DT";
+    gNB_name  =  "gNB-OAI-name";
     // Tracking area code, 0x0000 and 0xfffe are reserved values
     tracking_area_code  =  1;
     plmn_list = ({ mcc = 001; mnc = 01; mnc_length = 2; snssaiList = ({ sst = 1; }) });
-    nr_cellid = 0xe0001;
+    nr_cellid = 0xe0002;
     ////////// Physical parameters:
     pdsch_AntennaPorts_XP = 2;
     pdsch_AntennaPorts_N1 = 2;
@@ -137,12 +137,12 @@ gNBs =
         SCTP_OUTSTREAMS = 2;
     };
     ////////// AMF parameters:
-    #amf_ip_address = ({ ipv4 = "172.21.6.4"; });
-    amf_ip_address = ({ ipv4 = "192.168.70.132"; });
+    amf_ip_address = ({ ipv4 = "172.21.6.4"; });
+    #amf_ip_address = ({ ipv4 = "192.168.70.132"; });
     NETWORK_INTERFACES :
     {
-        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "192.168.70.129/24";
-        GNB_IPV4_ADDRESS_FOR_NGU                 = "192.168.70.129/24";
+        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "172.21.21.50/24";
+        GNB_IPV4_ADDRESS_FOR_NGU                 = "172.21.21.50//24";
         GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
     };
   }
```

### Running
You can now run the gNB, the UE, and the channel emulator as described in the tutorial. 

*gNB*

```
cd ~/openairinterface5g/cmake_targets/ran_build/build

sudo ./nr-softmodem \
  -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.positioning.conf \
  --gNBs.[0].min_rxtxtime 6 \
  --device.name vrtsim \
  --vrtsim.role server \
  --vrtsim.taps-socket ipc:///tmp/ru_socket_0 \
  --vrtsim.timescale 0.5
```

Note that I changed the `--vrtsim.timescale` parameter to `0.5` due to the GPU acceleration. 

*UE*

Since we are all using the same core network, it is also necessary that every user uses a different IMSI for the UE. For our lab we have assigned the IMSI range `001010000000030` - `001010000000050`. Coordinate with others so that we don't use the same. 

Create a file ~/openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/ue-os-core.conf

```file
uicc0 = {
imsi = "001010000000030";
key = "8BAF473F2F8FD09487CCCBD7097C6862";
opc= "8e27b6af0e692e750f32667a3b14605d";
pdu_sessions = ({ dnn = "internet"; nssai_sst = 1; type="ipv4"});
}
```

```
cd ~/openairinterface5g/cmake_targets/ran_build/build

sudo ./nr-uesoftmodem \
  -C 3619200000 \
  -r 106 \
  --band 78 \
  --numerology 1 \
  --ssb 516 \
  --device.name vrtsim \
  --vrtsim.taps-socket ipc:///tmp/ue_socket_0 \
  -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/ue-os-core.conf \
```

*Channel Emulator*

```
source ~/.venv/bin/activate
cd ~/raytracing-channel-emulator/server
python main.py scenes/EURECOM/example_config.yaml
```

*LMF*

Last but not least to start the positioning request you will need to use the IP address of the LMF running on the openshift cluster instead of the local one. Also you will need to adapt the IMSI in the `InputData.json` to match the one you configured the UE with. 

```
cd ~/openairinterface5g/doc/tutorial_resources/oai-cn5g/positioning
curl --http2-prior-knowledge \
  -H "Content-Type: application/json" \
  -d "@InputData.json" \
  -X POST http://172.21.4.117:30080/nlmf-loc/v1/determine-location
```

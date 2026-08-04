🚀 Mamba Attention Network: Fast Inter Frame CU Partition for VVC Screen Content Coding
Official implementation of the paper "Mamba Attention Network: Fast Inter Frame CU Partition for VVC Screen Content Coding", 
Abstract: This work proposes a novel fast CU partitioning algorithm for VVC Screen Content Coding (SCC). We introduce a Mamba Attention Network that leverages the global modeling capabilities of State Space Models (SSMs) and attention mechanisms to predict CU partitioning, significantly reducing the encoding time of VVC Inter frames for screen content.
Authors: Jiaxin Zeng, Li Yu, Linyun Liu, Jiabao Zhu, Jinhao Kuang, Yucheng Zhong, Licong Ma, Hongbo Li, Tiansong Li

# Prepare
📑 Table of Contents
Prerequisites
Dataset Preparation
Model Training
Integration with VTM (Inference)
Results
Citation

⚙️ Prerequisites
To reproduce the results, please ensure your environment meets the following requirements:
Hardware: NVIDIA GPU (Recommended for training), CPU/GPU (for VTM inference).
OS: Windows 10/11 or Linux (Ubuntu 18.04+).
Python: 3.7 or higher.
Deep Learning Framework: PyTorch (tested on 1.12+).
VVC Reference Software: VTM-23.11.
Install Python Dependencies:
pip install torch torchvision torchaudio
pip install numpy opencv-python scikit-image tqdm einops

# Add other dependencies if necessary, e.g., timm, matplotlib
📂 Dataset Preparation
We utilize standard Screen Content Coding datasets for training and testing.
1. Datasets
The model is trained on SCVD and SCVQE 
Additional Datasets: CTC NCVS 

Train and valid video：https://pan.baidu.com/s/1P4PttrToESmH_LA6RGmcLg?pwd=vtp4 

Train label and cost and Train model： https://pan.baidu.com/s/1tNq-Y-96rRseAiCvFZIEfg?pwd=pwx5

2. Data Generation
Similar to standard VVC learning-based approaches, we need to extract RDcost and partition labels from the VTM encoder.
Modify VTM Source:Typedef.h ->enable_tracing :1

# Example command to generate training data
Extract Features:
Run the modified VTM encoder on the training set to generate raw data.

/EncoderAppStatic -c encoder_randomaccess_vtm.cfg -c classSCC.cfg -i /home/zjx/sequence/SCVD/cf444_1792x1024.yuv -wdt 1792 -hgt 1024 -fr 30 -fs 0 -f 300 -q 22 -b AI_Bin/str_AI_cf444_1792x1024_qp22_nf300.bin > AI_Log/log_AI_cf444_1792x1024_qp22_nf300.txt
mv CUInfoCost.dat AI_Bin/CUInfoCost_cf444_1792x1024_22_nf300.dat

The training labels are stored in the bin file, which has not been uploaded due to its large size. 
They can be obtained by executing the command `DecoderAppStatic -b **.bin --TraceFile="%s" --TraceRule="D_BLOCK_STATISTICS_CODED:poc>=0"`


Use the provided script to convert raw data into the dataset format for the Mamba network.
bash
🎓 Model Training
We implement the Mamba Attention Network using PyTorch.
1. Train the Network
Run the training script to optimize the partitioning prediction model.

⚡ Integration with VTM (Inference)
To accelerate the VVC encoding process, we integrate the trained Mamba model into the VTM encoder.

 Running the Fast Encoder
Use the following command  and set Thr.txt to run the fast encoding with the proposed method.

./EncoderAppStatic -c encoder_intra_vtm.cfg -c classSCC.cfg -i /codec/zjx/test/Web2_1280x720_300_444_8b.yuv -wdt 1280 -hgt 720 -fr 30 -fs 0 -f 300 -q 22 > AI_Log/log_AI_Web2_1280x720_300_444_8b_qp22_nf2.txt


Key Parameters:
-q: Quantization Parameter (22, 27, 32, 37).

# Comparative method:

@article{feng2025partition,
  title={Partition Map-Based Fast Block Partitioning for VVC Inter Coding},
  author={Feng, Xinmin and Li, Zhuoyuan and Li, Li and Liu, Dong and Wu, Feng},
  journal={arXiv preprint arXiv:2504.18398},
  year={2025}
}

./EncoderAppStatic -el /codec16t1/zjx/IPMfastinter/IPM_TMM2025-main/results/PartitionMat/my_test/GlassHalf_3840x2160_24p_10b_444_Luma_QP22_PartitionMat.txt -c encoder_randomaccess_vtm2.cfg -c classSCC.cfg -i /codec/zjx/sequence/GlassHalf_3840x2160_24p_10b_444.yuv -wdt 3840 -hgt 2160 -fr 24 -fs 0 -q 22 -f 300 -ip 32 >AI_Log/log_AI_GlassHalfipm_3840x2160_24p_10b_444_qp22_nf300.txt 

@misc{liu2023,
      title={CNN-based Prediction of Partition Path for VVC Fast Inter Partitioning Using Motion Fields}, 
      author={Yiqun Liu and Marc Riviere and Thomas Guionnet and Aline Roumy and Christine Guillemot},
      year={2023},
      eprint={2310.13838},
      archivePrefix={arXiv},
      primaryClass={cs.MM},
      url={https://arxiv.org/abs/2310.13838}, 
}


 ./EncoderAppStaticmvf -c encoder_randomaccess_vtm.cfg -c classSCC.cfg -i /codec/zjx/sequence/MissionControlClip3_1920x1080_60p_8b444.yuv -wdt 1920 -hgt 1080 -fr 60 -fs 0 -q 22 -f 300  -cnn /home/zjx/LNN/model/AI_OUTTrainoutAllOri/CNN400.json AI_Log/log_AI_MissionControlClip3Mvf_1920x1080_60p_8b444_qp22_nf300.txt &


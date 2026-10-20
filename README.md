# Robosapiens-AKD: INTO-CPS Case Study for Anchor-based Knowledge Distillation

Robosapiens-AKD is an INTO-CPS case study and the reference implementation of the paper *Trustworthy Knowledge Distillation via Anchor-Guided Distribution Learning*. The method transfers knowledge from a teacher network to a smaller student network using a set of anchors, which are representative training samples selected from each class. The student is trained to match the teacher's similarity distributions in the feature space, both among the samples of a mini-batch and between the samples and the anchors. Because the anchor set contains samples of every class, each mini-batch iteration receives information from all classes.

The repository reproduces the CIFAR-100 results reported in the paper.

---

## Relation to INTO-CPS

This repository is maintained as part of the INTO-CPS ecosystem and was developed within the RoboSAPIENS project.

It illustrates how:

* a large model can be compressed into a smaller model that fits the compute limits of an embedded or robotic platform,
* the compressed model can keep the accuracy of the large model,
* model compression can be evaluated reproducibly on a standard benchmark.

---

## Method Overview

1. A teacher network is pretrained on CIFAR-100 (or downloaded, see below).
2. Before student training, one anchor per class is selected. For each class, the teacher features of all training samples are compared with cosine similarity, and the sample with the highest total similarity to the other samples of its class is chosen. For CIFAR-100 this gives 100 anchors.
3. During student training, the loss is the cross-entropy loss with the ground truth labels plus the AKD loss, weighted by `--l_AKD`. The AKD loss has three terms. `L_1` matches the similarity distribution among the mini-batch samples. `L_2` and `L_3` match the similarity distribution between the mini-batch samples and the anchors, in both directions.

`--distill AKD_CRD` adds the Contrastive Representation Distillation (CRD) loss to the AKD loss.

---

## Repository Structure

```
robosapiens-AKD/
├── crd/                        # CRD loss and memory bank (used by AKD_CRD)
├── dataset/
│   └── cifar100.py             # CIFAR-100 data loaders
├── distiller_zoo/              # other distillation losses (KD, FitNet, AT, RKD, ...)
├── helper/
│   ├── akd.py                  # AKD loss and anchor set selection
│   ├── loops.py                # training and validation loops
│   └── util.py
├── models/
│   ├── cifar/                  # teacher and student architectures for CIFAR-100
│   └── imagenet/
├── scripts/
│   ├── fetch_pretrained_teachers_fixed.sh   # download the CIFAR-100 teachers
│   └── ...
├── save/                       # checkpoints and logs are written here
├── train_cifar_teacher.py      # train a teacher
├── train_cifar_student.py      # train a student with AKD
├── requirements.txt
├── LICENSE
├── NOTICE
├── CITATION.cff
├── CONTRIBUTING.md
└── README.md
```

---

## Requirements

* Python 3.8+
* PyTorch and torchvision
* A CUDA GPU (the training scripts call `.cuda()` directly)

---

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## Dataset

CIFAR-100 is downloaded automatically by torchvision into `./data` on first use.

---

## Running the Experiments

1. Download the pretrained CIFAR-100 teachers. The script places six checkpoints in `save/models/<teacher>_vanilla/ckpt_epoch_240.pth` and checks that they load into the model definitions of this repository:

   ```bash
   bash scripts/fetch_pretrained_teachers_fixed.sh
   ```

2. Train a student with AKD. Example for ResNet32x4 (teacher) to ResNet8x4 (student):

   ```bash
   python train_cifar_student.py --path_t ./save/models/resnet32x4_vanilla/ckpt_epoch_240.pth --distill AKD --model_t resnet32x4 --model_s resnet8x4
   ```

Supported teachers (`--model_t`): `wrn_40_2`, `resnet56`, `resnet110`, `resnet32x4`, `vgg13`, `ResNet50`.

Supported students (`--model_s`): `wrn_16_2`, `wrn_40_1`, `resnet20`, `resnet32`, `resnet8x4`, `vgg8`, `MobileNetV2`, `ShuffleV1`, `ShuffleV2`.

Main AKD arguments of `train_cifar_student.py`:

| Argument | Default | Description |
|---|---|---|
| `--distill` | `AKD` | `AKD` or `AKD_CRD` |
| `--l_AKD` | `10.0` | Weight of the AKD loss |
| `--l_1` | `1.0` | Weight of `L_1` |
| `--l_2` | `0.1` | Weight of `L_3`. `L_2` is weighted by `1 - l_2` |
| `--epochs` | `240` | Number of training epochs |
| `--learning_rate` | `0.05` | Initial learning rate |

Student checkpoints are written to `save/student_model/` and TensorBoard logs to `save/student_tensorboards/`.

To train a teacher from scratch:

```bash
python train_cifar_teacher.py --model resnet32x4
```

---

## Reproducibility & Assets

* The teacher checkpoints are downloaded from the release mirror of the mdistiller project, because the original download server of RepDistiller is no longer available.
* The anchor set is selected from the training data at the start of each run.
* Results can differ slightly across hardware and library versions.

---

## Known Limitations

* Only the CIFAR-100 experiments are covered. The `models/imagenet/` definitions are included, but no ImageNet training script is provided.
* A CUDA GPU is required.

---

## License

This project is licensed under the Apache License 2.0. See the `LICENSE` file for details.

The code is based on NORM and RepDistiller, which are licensed under the BSD 2-Clause License. See the `NOTICE` file.

---

## Acknowledgment

This work has received partial funding from the Hellenic Foundation for Research & Innovation (H.F.R.I.) scholarship under grant agreement No 20490 (Deep Learning Methodologies for Trustworthy Intelligent Systems) and the research project "Robotic Safe Adaptation In Unprecedented Situations (RoboSAPIENS)", which is implemented in the framework of Horizon Europe 2021-2027 research and innovation programme under grant agreement No 101133807. This publication reflects the authors' views only. The European Commission are not responsible for any use that may be made of the information it contains.

<p align="center">
<img src="https://robosapiens-eu.tech/wp-content/uploads/elementor/thumbs/europeanlogo-r2pgiiytkuoyehehz8416uvf52f3bxv6hkmztxq1am.jpeg"
     alt="EU Flag"
     width="120" />
</p>
<p align="center">
Learn more about <a href="https://robosapiens-eu.tech/">RoboSAPIENS</a>.
</p>
<p align="center">
<img src="https://robosapiens-eu.tech/wp-content/uploads/elementor/thumbs/full_robot2-r2pgiiysrgwxbxr64iuhw1nmi4k46lxqopg2va2qos.png"
     alt="RoboSAPIENS Logo"
     width="80" />
</p>

---

## Citation

If you use this repository, please cite:

```bibtex
@article{spanos2025akd,
  author    = {Spanos, Dimitrios and Passalis, Nikolaos and Tefas, Anastasios},
  title     = {Trustworthy Knowledge Distillation via Anchor-Guided Distribution Learning},
  journal   = {Knowledge-Based Systems},
  pages     = {114297},
  year      = {2025},
  publisher = {Elsevier}
}
```

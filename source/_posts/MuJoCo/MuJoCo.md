---
title: MuJoCo
date: 2022-05-23 22:29:35
tags: [Reinforcement Learning,Simulation]
categories: Simulation
---

# MuJoCo Instruction

## 1. Install

The installation of `MuJoCo` is simple and it have been archived by official website. There are two steps you need to install `MuJoCo` in Ubuntu operating system which I use usually. You need to choose download two files from [official website](https://roboti.us/download.html), which I have downloaded are `mujoco200_linux` and the License of the it.

After unpack it, drop the license file in the bin folder and navigate to the sample folder in terminal.

```bash
cd mujoco200_linux/sample && make
```

Then the codes will be compiled with makefile and you could navigate to bin folder and run the simulate file with following command.

```bash
./simulate ../model/arm26.xml
```

After these commands, you should see a GUI open up and a two joint arm moving if everything worked fine.

<img src="/image/Mujoco/1.png" alt="1" style="zoom:80%;" />

# RLHF Installation Guidance

## 1 SL-RLHF source code

Code：https://github.com/HuJiabin2018/SelfLearning-RLHF

### 1.1 Install MRN and B-Pref

1. **install mujoco**

```linux
sudo apt update
sudo apt install unzip gcc libosmesa6-dev libgl1-mesa-glx libglfw3 patchelf libegl1 libopengl0
sudo ln -s /usr/lib/x86_64-linux-gnu/libGL.so.1 /usr/lib/x86_64-linux-gnu/libGL.so
wget https://www.roboti.us/download/mujoco200_linux.zip -P /tmp
unzip /tmp/mujoco200_linux.zip -d ~/.mujoco
wget https://www.roboti.us/file/mjkey.txt -P /tmp
mv /tmp/mjkey.txt ~/.mujoco/
```

Related Article：https://zhuanlan.zhihu.com/p/472290066

The pit Opencv version encountered needs to be 3.4：

``` linux
pip install opencv-python==3.4.2.17 -i https://pypi.tuna.tsinghua.edu.cn/simple 
pip install opencv-contrib-python==3.4.2.17 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

2. **Install dependencies**

   1. Installation

      ```linux
      git clone https://github.com/RyanLiu112/MRN.git
      conda env create -f conda_env.yml
      conda activate mrn
      pip install -e .[docs,tests,extra]
      cd custom_dmcontrol
      python setup.py install
      cd ../custom_dmc2gym
      pip install -e .
      cd ..
      pip install git+https://github.com/rlworkgroup/metaworld.git@04be337a12305e393c0caf0cbf5ec7755c7c8feb
      pip install pybullet termcolor
      ```

      

   2. Problem1：github.com/deepmind/dm_control Unable to connect to github (even with a VPN)
   
      conda_env.yml File needs to be installed
      `git+git://github.com/deepmind/dm_control.git@d1fe1d14fb229d3ad784e40d8c54491d30e1f08a`
   
      and
   
      `git+git://github.com/facebookresearch/hydra@0.11_branch`
   
      These two cannot be installed even with a VPN.
   
      Related Article：https://blog.csdn.net/qq_38308388/article/details/134582143
   
      Need to download separately github.com/deepmind/dm_control.git和github.com/facebookresearch/hydra，Note that specific tag and branch。
   
      ```linux
      dm_control: git clone -b python3.6_eol https://github.com/deepmind/dm_control.git
      hydra:git clone -b 0.11_branch https://github.com/facebookresearch/hydra.git
      ```
   
      Last step：
   
      `python setup.py install`
   
   3. Problem 2：with FileNotFoundError: 'stable_baselines/version.txt'
   
      Solution：Manually entering the docker image and creating the `/root/code/stable-baselines/stable_baselines/version.txt` file solved the problem.
   
      
   
   4. Problem 3：无法找到mujoco_linux
   
      Solution: Delete the corresponding file_{}. The reason for this is that different tutorials have different names for mujoco. In spinningup, mujoco deletes '\_linux'
   
      `DEFAULT_HEADERS_DIR = '/home/user/.mujoco/mujoco200_{}/include'.format(
          PLATFORM_SUFFIXES[platform.system()])`
   
      Change to
   
      `DEFAULT_HEADERS_DIR = '/home/user/.mujoco/mujoco200/include'.format(
          PLATFORM_SUFFIXES[platform.system()])`
   
      
   
   5. Problem 4：cd custom_dmcontrol & pip install -e .

      dm_tree Unable to build：
   
       `Building wheels for collected packages: dm-tree, labmaze
        Building wheel for dm-tree (setup.py) ... error`
   
      Solution：
   
      `pip install CMake`
   
      
   
      labmaze Unable to build：
   
      `Building wheel for labmaze (setup.py) ... error`
   
      Solution：
   
      `pip install git+git://github.com/deepmind/labmaze`
   
      However, the above method will still encounter
   
      ```linux
        fatal: unable to connect to github.com:
        github.com[0: 20.205.243.166]: errno=Connection timed out
      
      WARNING: Discarding git+git://github.com/deepmind/labmaze. Command errored out with exit status 128: git clone -q git://github.com/deepmind/labmaze /tmp/pip-req-build-tff7vnr0 Check the logs for full command output.
      ```
   
      Still the original method：
   
      ```linux
      git clone https://github.com/google-deepmind/labmaze.git
      python setup.py install
      ```
   
      However, installing labmaze requires bazel
   
      Tutorial：https://bazel.build/install/ubuntu?hl=zh-cn
   
      Add Bazel distribution URI as package source
       ```linux
      sudo apt install apt-transport-https curl gnupg -y
      curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor >bazel-archive-keyring.gpg
      sudo mv bazel-archive-keyring.gpg /usr/share/keyrings
      echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
       ```
   
      The version that needs to be installed is 5.4.0
      
      ```linux
      sudo apt install bazel-5.4.0
      sudo ln -s /usr/bin/bazel-5.4.0 /usr/bin/bazel
      bazel --version
      bazel 5.4.0
      python setup.py install
      ```
   
      last：
      
      `cd custom_dmcontrol & pip install -e .`
      
   6. mujoco Installation error：mujoco_py/cymj.pyx:67:5: Exception check on 'c_warning_callba 

     Solution： `pip install cython==3.0.0a10`
   
3. Visualize TensorBoard

   https://blog.csdn.net/qq_41656402/article/details/131123121

   `tensorboard --logdir=/home/user/RLHF/MRN/exp/metaworld_door-open-v2/H256_L3_B512_tau0.005/sac_unsup9000_topk5_sac_lr0.0003_temp0.1_seed12345/tb`

   CLick TensorBoard 2.10.1 at http://localhost:6006/

4. To avoid overwriting the original data during retraining

   It can be found in train.yaml under config

   hydra:

     name: ${env}

     run:

   ​    dir: 

   **Add timestamp**`_temp${agent.params.init_temperature}_seed${seed}_${now:%Y-%m-%d_%H-%M-%S}`


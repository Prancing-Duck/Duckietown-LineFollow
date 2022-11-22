## Team members
* @deakg4 Deák Gergely MEVPIT
* @martonoravecz Oravecz Márton Péter J38LZK

Third teammember left the team, due to scheduling issues with his work commitments.

## Milestone 1

We were working based on the e-mail sent by Móna Róbert in regards of the the the deliverables in each step.
Our setup was run a freshly installed Ubuntu 22.04

### 1. Clone the duckietown-gym repository

```
git clone https://github.com/duckietown/gym-duckietown.git
mkdir gym
mv gym-duckietown gym

```

### 2. Setup the environments and install required libraries

There were some issues with our choice of operation system and the environment. In ideal situation we would have been able to follow the [gym-ducketown repository's README.md](https://github.com/duckietown/gym-duckietown/blob/daffy/README.md), but the following issue stopped us.

#### Running gym in a docker
We run docker with following coomands
```
docker pull duckietown/gym-duckietown && \
docker run -it duckietown/gym-duckietown bash
```

Following commands has to be run before manual-control.py
```
apt-get update && apt-get install -y freeglut3-dev fontconfig
Xvfb :0 -screen 0 1024x768x24 -ac +extension GLX +render -noreset &> xvfb.log &
export DISPLAY=:0
```
We could not document results on docker visually, so local run was reuired

#### Running gym locally

##### 0. Installed miniconda3 and navigated to /gym folder

##### 1. Tried to install with following command
```
conda env create -f environment.yml
```
mlk version was not decideable

Log output can be find [here](errors/mlk.log)

##### 2. Modified environment.yml to the following:
```
name: gym-duckietown
channels:
    - pytorch
    - defaults
dependencies:
    - python=3.7
    - pytorch    
    - torchvision
    - pyzmq
    - numpy
    - pyyaml=3.12
    - pip:
        - gym
        - pyglet
        - opencv-python
        - scikit-image
```
And tried to install again with following command
```
conda env create -f environment.yml
```
**Install was sucessful**

##### 3. Activated conda environment with the following command
```
conda activate gym-duckietown
```

##### 4. Pyglet
Tried to run the example with
```
./manual_control.py --env-name Duckietown-udem1-v0
```
Pyglet version was only compatible with python 3.8.
Log output can be find [here](errors/pyglet.log)

So I reinstalled with the last version which did not require python 3.8
```
pip uninstall pyglet
pip install pyglet==1.5.27
pip install -r requirements.txt
```

##### 5. Graphical library issues
Tried to run the example
```
./manual_control.py --env-name Duckietown-udem1-v0
```
`swrast` and `iris` libraries were not found
Log output can be find [here](errors/swrast.log)
Issue was solved with help of (The following StackOverflow isse)[https://stackoverflow.com/questions/71010343/cannot-load-swrast-and-iris-drivers-in-fedora-35]

```
sudo apt install --reinstall libgl1-mesa-dri -y
sudo apt-get update
sudo apt-get install libstdc++6 -y
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
```
Where the last commands path was determined with the followin command:

```
find / -name libstdc++.so.6 2>/dev/null
```

##### 6. PythonPath
Tried to run the example
```
./manual_control.py --env-name Duckietown-udem1-v0
```
ModuleNotFoundError: No module named 'gym_duckietown'

Log output can be find [here](errors/gymDuckietown.log).
```
export PYTHONPATH="${PYTHONPATH}:`pwd`"
cd src/
export PYTHONPATH="${PYTHONPATH}:`pwd`"
```
##### 6. Carnivalmirror
Tried to run the example
```
./manual_control.py --env-name Duckietown-udem1-v0
```
ModuleNotFoundError: No module named 'carnivalmirror'

Log output can be find [here](errors/carnivalmirror.log).

Installed the library

```
pip install carnivalmirror
```

##### 8. randInt rewrite in randomizer.py

Tried to run the example
```
./manual_control.py --env-name Duckietown-udem1-v0
```
AttributeError: 'numpy.random._generator.Generator' object has no attribute 'randint'

Log output can be find [here](errors/randInt1.log).

Issue is caused by numpy renaming the function.

The following line was modified in `src/gym-duckietown/randomization/randomizer.py`
Line 55 from
```
                    setting = rng.randint(low=low, high=high, size=size)
```
to

```
                    setting = rng.integers(low=low, high=high, size=size)
```

##### 9. randInt rewrite in simulator.py
Tried to run the example
```
./manual_control.py --env-name Duckietown-udem1-v0
```
AttributeError: 'numpy.random._generator.Generator' object has no attribute 'randint'

Log output can be find [here](errors/randInt2.log).

Issue is caused by numpy renaming the function.

Modified the following lines of the `src/gym_duckietown/simulator.py`
Line 654 from
```
                obj.visible = self.np_random.randint(0, 2) == 0
```
to

```
                obj.visible = self.np_random.integers(0, 2) == 0
```
and
Line 675 from
```
                tile_idx = self.np_random.randint(0, len(self.drivable_tiles))
```
to

```
                tile_idx = self.np_random.integers(0, len(self.drivable_tiles))
```

##### 10. Started example
```
./manual_control.py --env-name Duckietown-udem1-v0
```
#### 3. Run ./manual_control.py --env-name Duckietown-udem1-v0
Already run in previous sub step (2.10)

#### 4. Create tracks
[Deák Gergely's map](tracks/deak_map.yaml)

[Oravecz Márton Péter's map](tracks/marton_map.yaml)

#### 5. Run agent and save the video

We reworked to the preparation steps to the following:
```
conda env create -f gym/environment.yaml
conda activate gym-duckietown
sudo apt install --reinstall libgl1-mesa-dri -y
sudo apt-get update
sudo apt-get install libstdc++6 -y
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
export PYTHONPATH="${PYTHONPATH}:`pwd`/gym:`pwd`/gym/src"

```
It was done by extending environment.yaml in gym folder


How to start after setup the examples
```
./gym/manual_control.py --env-name Duckietown-<<name>>-<<version>> --map-name <<path_to_map>>
```
If we stand in the root:

Oravecz Márton's map
```
./gym/manual_control.py --env-name Duckietown-marton-v0 --map-name tracks/marton_map
```
[Video on youtube](https://www.youtube.com/watch?v=Sihvt9_jITQ)


Deák Gergely's map
```
./gym/manual_control.py --env-name Duckietown-deak-v0 --map-name tracks/deak_map
```
[Video on youtube](https://youtu.be/J6C1rYYIpqk)

#### 6. Document steps

Steps is documented in this file

#### 7. Work balance

Oravecz Márton Péter mainly worked on solving issues with starting the environment and Deák Gergely mainly worked on experimenting with the environment and exploring map creation.

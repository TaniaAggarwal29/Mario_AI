
![1.png](https://github.com/jugshaurya/Mario_AI/blob/main/1.png)
---
![2.png](https://github.com/jugshaurya/Mario_AI/blob/main/2.png)
---
![3.png](https://github.com/jugshaurya/Mario_AI/blob/main/3.png)
---
![4.png](https://github.com/jugshaurya/Mario_AI/blob/main/4.png)
---
![5.png](https://github.com/jugshaurya/Mario_AI/blob/main/5.png)
---
![6.png](https://github.com/jugshaurya/Mario_AI/blob/main/6.png)
---
![7.png](https://github.com/jugshaurya/Mario_AI/blob/main/7.png)
---
![8.png](https://github.com/jugshaurya/Mario_AI/blob/main/8.png)
---
![9.png](https://github.com/jugshaurya/Mario_AI/blob/main/9.png)
---
![10.png](https://github.com/jugshaurya/Mario_AI/blob/main/10.png)
---
![11.png](https://github.com/jugshaurya/Mario_AI/blob/main/11.png)
---

## 1. Setup Mario

```python
!pip install gym_super_mario_bros==7.3.0 nes_py
```
```python
# Import the game
import gym_super_mario_bros
# Import the Joypad wrapper
from nes_py.wrappers import JoypadSpace
# Import the SIMPLIFIED controls
from gym_super_mario_bros.actions import SIMPLE_MOVEMENT
```
```python
# Setup game
env = gym_super_mario_bros.make('SuperMarioBros-v0')
env = JoypadSpace(env, SIMPLE_MOVEMENT)
```
```python
# Create a flag - restart or not
done = True
# Loop through each frame in the game
for step in range(100000): 
    # Start the game to begin with 
    if done: 
        # Start the game
        env.reset()
    # Do random actions
    state, reward, done, info = env.step(env.action_space.sample())
    # Show the game on the screen
    env.render()
# Close the game
env.close()
```


https://user-images.githubusercontent.com/63960670/200620662-6b5b7c73-56ee-430a-9d06-9f317906f200.mp4


## 2. Preprocess Environment

```python
# Install pytorch
!pip install torch==1.10.1+cu113 torchvision==0.11.2+cu113 torchaudio===0.10.1+cu113 -f https://download.pytorch.org/whl/cu113/torch_stable.html
```
```python
# Install stable baselines for RL stuff
!pip install stable-baselines3[extra]
```
```python
# Import Frame Stacker Wrapper and GrayScaling Wrapper
from gym.wrappers import GrayScaleObservation
# Import Vectorization Wrappers
from stable_baselines3.common.vec_env import VecFrameStack, DummyVecEnv
# Import Matplotlib to show the impact of frame stacking
from matplotlib import pyplot as plt
```
```python
# 1. Create the base environment
env = gym_super_mario_bros.make('SuperMarioBros-v0')

# 2. Simplify the controls 
env = JoypadSpace(env, SIMPLE_MOVEMENT)

# 3. Grayscale
env = GrayScaleObservation(env, keep_dim=True)

# 4. Wrap inside the Dummy Environment
env = DummyVecEnv([lambda: env])

# 5. Stack the frames
env = VecFrameStack(env, 4, channels_order='last')
```
```python
state = env.reset()
```
```python
state, reward, done, info = env.step([5])
```
```python
plt.figure(figsize=(20,16))
for idx in range(state.shape[3]):
    plt.subplot(1,4,idx+1)
    plt.imshow(state[0][:,:,idx])
plt.show()
```


![Capture](https://user-images.githubusercontent.com/63960670/200667215-11e0718c-14f3-4ecd-8b1e-1959856eafe5.PNG)


## 3. Train the RL Model

```python
# Import os for file path management
import os 
# Import PPO for algos
from stable_baselines3 import PPO
# Import Base Callback for saving models
from stable_baselines3.common.callbacks import BaseCallback
```
```python
class TrainAndLoggingCallback(BaseCallback):

    def __init__(self, check_freq, save_path, verbose=1):
        super(TrainAndLoggingCallback, self).__init__(verbose)
        self.check_freq = check_freq
        self.save_path = save_path

    def _init_callback(self):
        if self.save_path is not None:
            os.makedirs(self.save_path, exist_ok=True)

    def _on_step(self):
        if self.n_calls % self.check_freq == 0:
            model_path = os.path.join(self.save_path, 'best_model_{}'.format(self.n_calls))
            self.model.save(model_path)

        return True
```        
```python
CHECKPOINT_DIR = './train/'
LOG_DIR = './logs/'
```
```python
# Setup model saving callback
callback = TrainAndLoggingCallback(check_freq=10000, save_path=CHECKPOINT_DIR)
```
```python
# This is the AI model started
model = PPO('CnnPolicy', env, verbose=1, tensorboard_log=LOG_DIR, learning_rate=0.000001, 
            n_steps=512) 
```
```python
# Train the AI model, this is where the AI model starts to learn
model.learn(total_timesteps=1000000, callback=callback)
```
```python
model.save('thisisatestmodel')
```




## 4. Test it Out

```python
# Load model
model = PPO.load('./train/best_model_1000000')
```
```python
state = env.reset()
```
```python
# Start the game 
state = env.reset()
# Loop through the game
while True: 
    
    action, _ = model.predict(state)
    state, reward, done, info = env.step(action)
    env.render()
```


### After loading model 500,000:

https://user-images.githubusercontent.com/63960670/200736744-dd614e70-893d-4396-9192-80ab206739ea.mp4


### After loading model 100,000:

https://user-images.githubusercontent.com/63960670/200738250-f2da9d5a-3893-41be-887d-cbf64c4e3e5e.mp4

### After loading model 3M:

https://user-images.githubusercontent.com/63960670/200739095-82ddac93-761d-4672-9562-fa5d5e14cc5f.mp4

### After loading model 4M:

https://user-images.githubusercontent.com/63960670/200739766-e3b82b87-7162-4063-ad83-d8af06771ca5.mp4




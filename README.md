[//]: # (Image References)
[simulator]: ./doc/simulator.png
[sim_red_detected]: ./doc/sim_red_detected.png
[sim_green_detected]: ./doc/sim_green_detected.png


This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

# Team Member

Name 				| Udacity Account Email
---------------- | ---------------------
hesham abdelfattah | hesham.abdelfattah81@gmail.com
Naveen Jagadish    |naveen.lsu@gmail.com
Frederic Liu       |liufubo4213@126.com




## Description


### Waypoint Updater

This node publishes a fixed number (200 in this implementation) ahead of the vehcile with the correct target velocities. If an upcoming stop light is detected, the velocity of the waypoints will be adjusted and the car decelerates or accelerates depending on the light state.

The implementation mainly follow the classroom's solion, subscribing to /current_pose , /base_waypoints and /traffic_waypoints, using KDTree to get the closest point, and decelate when close to red light. Finally publishes /final_waypoints.

### Drive-By-Wire Node / Twist Controller

#### Drive-By-Wire Node

This node represents a drive by wire controller. It receives /twist_cmd and current velocity, calculates throttle, brake and steering, and finaly published them to vehicle. 
#### Twist Controller

This controller is responsible for acceleration and steering. The acceleration is calculated via PID controller. Steering is calculated using YawController which simply calculates needed angle to keep needed velocity.

### Traffic Light Detection / Classification

This node is responsible for detecting upcoming traffic lights and classify their states (red, yellow, green).

For the classification model, we take reference of some previous teams of this projects. Finally we choose Tensorflow's [Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection) to train the traffic-light classification model, for its easily training and frendly using.


Training image dataset could be downloaded [here](https://drive.google.com/file/d/0B-Eiyn-CUQtxdUZWMkFfQzdObUE/view?usp=sharing).

The workflow of training the classification model with Object Detection API is as followings:

#### Install dependencies

```
sudo apt-get install protobuf-compiler
sudo pip install pillow
sudo pip install lxml
sudo pip install jupyter
sudo pip install matplotlib
```

#### Protobuf Compilation

```
# From root directory
protoc object_detection/protos/*.proto --python_out=.
```

#### Add Libraries to PYTHONPATH

```
# From root directory
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```

#### Download SSD MobileNet Model

```
# From root directory
curl http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_coco_2017_11_17.tar.gz | tar -xv -C model/ --strip 1
```


### Creating TFRecord files

```
python object_detection/dataset_tools/create_pascal_tf_record.py --data_dir=data/sim_training_data/sim_data_capture --output_path=sim_data.record
```

```
python object_detection/dataset_tools/create_pascal_tf_record.py --data_dir=data/real_training_data/real_data_capture --output_path=real_data.record
```


## Training

### Trainning on Simulator Data

```
python object_detection/train.py --pipeline_config_path=config/ssd_mobilenet_v1_coco_sim.config --train_dir=data/sim_training_data/sim_data_capture
```


### Training on  Real Data

```
python object_detection/train.py --pipeline_config_path=config/ssd_mobilenet_v1_coco_real.config --train_dir=data/real_training_data
```


### Result
Currently in simulator the algorithm could drive car smoothly along the waypoints, stop at stopline when traffic light is red, and could recover from manual mode.



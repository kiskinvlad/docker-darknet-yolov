# Docker Darknet YOLOv3 YOLOv4 Training and Test (With GPU CUDA support on Linux)

### Build it
```
$ docker build -t umka/umka:gpu -f Dockerfile .
```

### Run it
```
$ docker run -p 8080:8080 -p 8090:8090 --gpus all --name umka  --rm -v ${pwd}:/workspace -w /darknet -it umka/umka:gpu /bin/bash
```

### Test it
#### YOLOv4
```
$ ./darknet detector test ./cfg/coco.data ./cfg/yolov4.cfg yolov4.weights data/dog.jpg -dont_show
```
#### YOLOv3
```
$ ./darknet detector test ./cfg/coco.data ./cfg/yolov3.cfg yolov3.weights data/dog.jpg -dont_show
```
## For video testing:
### Static video
```
$ docker run -p 8080:8080 -p 8090:8090 --gpus all --name umka --rm -v ${pwd}:/workspace -w /darknet  -it umka/umka:gpu /bin/bash
$ ./darknet detector demo ./cfg/coco.data ./cfg/yolov4.cfg yolov4.weights /workspace/dataset/data/test/test.mp4 -out_filename /workspace/dataset/data/result/res.avi 
```

### Stream video
```
$ docker run -p 8080:8080 -p 8090:8090 --gpus all --name umka  --rm -v ${pwd}:/workspace -w /darknet -it umka/umka:gpu /bin/bash
$ ./darknet detector demo ./cfg/coco.data ./cfg/yolov4.cfg  yolov4.weights rtsp://172.31.0.1:8080/live -mjpeg_port 8090 -ext_output
```
Note: rtsp server address and port would be other

## For training:
  
### On the host machine
1. Clone the repository 
```
$ git clone https://github.com/kiskinvlad/docker-darknet-yolov.git
```
2. Put all your custom ".jpg" and ".txt" annotations on this folder : `/dataset/data/obj`

3. Generate the **val.txt** and **train.txt** on the **/dataset/data** folder 
Note: to rewrite class id in each image label .txt file use next script:
```python class_id_modifier.py <path_to_folder_with_txt> <class_number>
```
``` 
$ cd dataset
$ python train_test_split.py
OR
$ python3 train_test_split.py
```
4. Please change the **yolo-obj.cfg**, **obj.data** and **yolo-obj.names** file according to your custom class numbers and ***YOLOv3 and YOLOv4***. This repo has been configured of **6 classes** for **YOLOv3**
5. The full **workspace** directory structures should look like below-
```
├── data
│   ├── obj
│   │   ├── 000000000036.jpg
│   │   ├── 000000000036.txt
│   │   ├── 000000581921.jpg
│   │   ├── 000000581921.txt
│   │   └── ................
│   ├── train.txt
│   ├── val.txt
│   └── yolo-obj.names
├── obj.data
├── train_test_split.py
└── yolo-obj.cfg
```
## Run the following command to start the training (Manually)
```
 docker run -v ${pwd}/dataset/data/obj:/darknet/data/obj -v ${pwd}/dataset/data/val.txt:/darknet/data/val.txt -v ${pwd}/dataset/data/train.txt:/darknet/data/train.txt -v ${pwd}/dataset/data/yolo-obj.names:/darknet/data/yolo-obj.names -v ${pwd}/dataset/obj.data:/darknet/obj.data -v ${pwd}/dataset/yolo-obj.cfg:/darknet/yolo-obj.cfg -v ${pwd}/dataset/backup:/darknet/backup -p 8080:8080 -p 8090:8090 --gpus all --name umka --rm -it umka/umka:gpu /bin/bash
  
```  

### Inside the running container (/darknet#):
#### YOLOv3
```
./darknet detector train obj.data yolo-obj.cfg darknet53.conv.74 -map -dont_show
```
#### YOLOv4
```
./darknet detector train obj.data yolo-obj.cfg yolov4.conv.137 -map -dont_show -mjpeg_port 8090 -ext_output
```
## Running the container in detached mode:
#### YOLOv3
```
$ docker run -p 8080:8080 -p 8090:8090 --gpus all --name umka --rm umka/umka:gpu -d\
  -v '${pwd}/dataset/data/obj:/darknet/data/obj' \
  -v '${pwd}/dataset/data/val.txt:/darknet/data/val.txt' \
  -v '${pwd}/dataset/data/train.txt:/darknet/data/train.txt' \
  -v '${pwd}/dataset/data/yolo-obj.names:/darknet/data/yolo-obj.names' \
  -v '${pwd}/dataset/obj.data:/darknet/obj.data' \
  -v '${pwd}/dataset/yolo-obj.cfg:/darknet/yolo-obj.cfg' \
  -v '${pwd}/dataset/backup:/darknet/backup' \
  ./darknet detector train obj.data yolo-obj.cfg darknet53.conv.74 -map -dont_show
```

#### YOLOv4
```
$ docker run -p 8080:8080 -p 8090:8090 --gpus all --name umka --rm umka/umka:gpu -d\
  -v '${pwd}/dataset/data/obj:/darknet/data/obj' \
  -v '${pwd}/dataset/data/val.txt:/darknet/data/val.txt' \
  -v '${pwd}/dataset/data/train.txt:/darknet/data/train.txt' \
  -v '${pwd}/dataset/data/yolo-obj.names:/darknet/data/yolo-obj.names' \
  -v '${pwd}/dataset/obj.data:/darknet/obj.data' \
  -v '${pwd}/dataset/yolo-obj.cfg:/darknet/yolo-obj.cfg' \
  -v '${pwd}/dataset/backup:/darknet/backup' \
  ./darknet detector train obj.data yolo-obj.cfg yolov4.conv.137 -map -dont_show
```

### Run demo with trained weights
```
 docker run -v ${pwd}/dataset/data/obj:/darknet/data/obj -v ${pwd}/dataset/data/val.txt:/darknet/data/val.txt -v ${pwd}/dataset/data/train.txt:/darknet/data/train.txt -v ${pwd}/dataset/data/yolo-obj.names:/darknet/data/yolo-obj.names -v ${pwd}/dataset/obj.data:/darknet/obj.data -v ${pwd}/dataset/yolo-obj.cfg:/darknet/yolo-obj.cfg -v ${pwd}/dataset/backup:/darknet/backup -v ${pwd}:/workspace -p 8080:8080 -p 8090:8090 --gpus all --name umka --rm -it umka/umka:gpu /bin/bash
 ./darknet detector demo obj.data yolo-obj.cfg backup/yolo-obj_1000.weights /workspace/dataset/data/test/test.mp4 -out_filename /workspace/dataset/data/result/res.avi
```

### To attach to the container
```
$docker attach --detach-keys="ctrl-a,x" darknet_training
```
In order to get out of the container without exiting it **press** `CTRL+A` then **press** `X`. It will get the user out of the running container.

### Starting a stopped container
First get the docker container ID/Name
```
$ docker ps -a
```
It will provide the ID/Name of the docker container we are interested. In this example **darknet_training**
Now you can start the stopped ***darknet_training*** container using the following commands-
```
$ docker start umka
$ docker exec -it umka /bin/sh
```

### To delete that container 
```
$ docker rm -f umka
```

## To continue a finished training
We can change the iteration number of the config file located at **/dataset/yolo-obj.cfg** and restart the training from where we stopped or finished training (Suppose, it was **100200**). You need to change the following lines 150200 and steps would be 80% (120160), 90% (135180)

```
max_batches = 150200
policy=steps
steps=120160,135180
````

After that, remove the stopped container named **darknet_training**
```
$ docker rm -f umka
```

Now, to start the training run the following command again. Please change the last iteration weights file accordingly. In this example, **backup/yolo-obj_100000.weights**, since the previous iterations were **100200** and the previous training saved the **yolo-obj_100000.weights** in the backup folder.

```
$ docker run -p 8080:8080 -p 8090:8090 --gpus all --name umka --rm -it umka/umka:gpu -d\
  -v '${pwd}/dataset/data/obj:/darknet/data/obj' \
  -v '${pwd}/dataset/data/val.txt:/darknet/data/val.txt' \
  -v '${pwd}/dataset/data/train.txt:/darknet/data/train.txt' \
  -v '${pwd}/dataset/data/yolo-obj.names:/darknet/data/yolo-obj.names' \
  -v '${pwd}/dataset/obj.data:/darknet/obj.data' \
  -v '${pwd}dataset/yolo-obj.cfg:/darknet/yolo-obj.cfg' \
  -v '${pwd}/dataset/backup:/darknet/backup' \
  ./darknet detector train obj.data yolo-obj.cfg backup/yolo-obj_100000.weights -map
```

## Building on a Xavier

```
$ sudo docker build -f Dockerfile_jetson -t umka/umka:gpu .

$ sudo docker run -it --runtime=nvidia umka/umka:gpu

# run inside the docker container to succefully build darknet with NVIDIA GPU Runtime
$ root@4b397cef37d7:/darknet# make
$ exit

#To get the container ID (then check for the last stopped/exited container)
$ sudo docker ps -a

$ sudo docker commit 4b397cef37d7 umka/umka:gpu
```

## Running on a Xavier
```
$ sudo docker run -it --runtime=nvidia umka/umka:gpu

#inside the container
$ ./darknet detector test ./cfg/coco.data ./cfg/yolov4.cfg yolov4.weights data/dog.jpg -dont_show
```

## Running on a Xavier with GUI

```
$ xhost +

#With GUI X11
$ sudo docker run -it --runtime=nvidia --net host -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix umka/umka:gpu

#inside the container
$ ./darknet detector test ./cfg/coco.data ./cfg/yolov4.cfg yolov4.weights data/dog.jpg

```

## Output

![Alt text](results.jpg?raw=true "After running in docker")

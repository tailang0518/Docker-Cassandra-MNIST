# Introduction 

This project is constructed by two docker Containers: an Application Docker Container and a Database Docker Container.
While the user submits ports in any browser, the Flask Router will link to the HTML5 file. In the HTML5 file, there is a canvas which user can use mouse to draw. The Router then saves the handwritten image and requests the prediction from the MNIST Keras model. After the result returns, the Router forwards the result to HTML5 and show it to user via browser and submits all two data to the Cassandra Database Container through the Docker Network Bridge.

The main job for this poject is to recognize the user's handwritten digit. User can draw the digit in the canvas and press `"predict it"` button to start to predict the image, and it will return the result on the right side of the page. User can also press `"clear it"` button to clear their drawing if he is not satisify with it. Each time, the prediction and date will be recorded in the Cassandra database. 

# DEMO
![](https://github.com/tailang0518/Docker-Cassandra-MNIST/blob/master/docker_MNIST/summary/UI.gif)

## __WITH DOCKER AND CASSANDRA FEATURE__

`how to use docker and cassandra database feature is at the bottom Preparation Part` 

![](https://github.com/tailang0518/Docker-Cassandra-MNIST/blob/master/docker_MNIST/summary/DEMO.GIF)


# Background

This project is deployed by Docker and used Canssandra database to record the predictions of user's handwritten digit and datetime. It used two Docker containers, Cassandra database container and our Application container, and they are connected by the Docker Network Bridge, which allows the communication between them.

It used Keras with backend TensorFlow to construct the MNIST model so that it can classify the user's handwritten digit. This MNIST model gets to 99.33% test accuracy after 4 epochs. *User can run mnist_train.py first to test the accuracy.* 

We used JSON to save the MNIST model. Sicne Keras separates the concerns of saving model architecture and saving model weights. Thus, we saved MNIST model architecture to JSON and model weights to HDF5 format. 

```python
model_json = model.to_json()
with open("model_mnist.json", "w") as json_file:
    json_file.write(model_json)

model.save_weights("model_mnist.h5")
```
Before MNIST model compiling, we need to open JSON file, read it and load the model. 

```python
file = open('model_mnist.json', 'r')
mnist_model = file.read()
file.close()
loaded_model = model_from_json(mnist_model)
loaded_model.load_weights("model_mnist.h5")
```

We use `HTML5` and `CSS` to design the UI of this project. Since it includes a canvas, we also use Javascript to catch the event of mouse on the canvas, so that user can draw on HTML5 canvas using a mouse. 

The Canvas part is the most difficult part in UI design. 

First we create a 280*280 Canvas in the HTML5 file index.html 

```python
<canvas id="canvas" width="280" height="280" style="border:8px solid; float: left; margin: 140px; margin-top:360px;  border-radius: 5px; cursor: crosshair;"></canvas>
```

`index.js` includes all the events of the mouse movements. Including mouse press down, mouse press up, mouse move and mouse out of the canvas. 

We set a `boolean flag` and assert it to be false at first so that we can make sure whether the mouse can be used to draw. 
If mouse press up or out of the canvas, it will lose the ability to draw in the canvas. 

For the event of mouse press down. 

```java
canvas.onmousedown=function(evt){

				var BeginX=evt.clientX-this.offsetLeft;
				var BeginY=evt.clientY-this.offsetTop;
				context.beginPath();
				context.moveTo(BeginX, BeginY);
				flag=true;

		}
 ```
 
 For the event of mouse press up. 
 
 ```java
 canvas.onmouseup=function(){
			flag=false;
		}
 ```
 
 For the event of mouse move. 
 
 ```java
 canvas.onmousemove=function(evt){
			if(flag){
				var endX=evt.clientX-this.offsetLeft;
				var endY=evt.clientY-this.offsetTop;
				context.lineTo(endX, endY);
				context.stroke();
			}
		}
 ```
 
 For the event of mouse out. 
 
 ```java
 canvas.onmouseout=function(){
			flag=false;
		}
 ```

The main UI looks like the following image. 

![](https://github.com/tailang0518/Docker-Cassandra-MNIST/blob/master/docker_MNIST/summary/screenshot2.png)

# Requirements  
- [x] Python (3 or more) 
- [x] Docker
- [x] Cassandra Database

# Preparation 

To run project properly, here are several preparetion that you need to follow. 

__1. construct the Docker Network Bridge so that these to containers can commnuicate with each other.__

```Bash
docker network create mnist_test1
```


__2. pull the Cassandra Database image from Docker Hub.__

```Bash 
docker pull Cassandra
```

__3. construct the Docker container for Cassandra image.__

```Bash 
docker run —-name mnist_cassandra —-net=mnist_test1 —-net-alias=mnist_cassandra -p 9042:9042 -d cassandra:latest 
```
  
__4. build the Application image from the Dockerfile.__

```Bash
docker build -t mnist:latest .
```

__5. construct the Docker container for Application image.__

```Bash
docker run —-name mn_app —-net=mnist_test1 —-net-alias=mn_app -d -p 8000:5000 mnist:latest 
```
__6. find the Appilcation container Ports.__

```Bash
docker ps 
```

__7. submit the Ports (it should be 0.0.0.0:8000) to any web browser.__

__8. call cqlsh shell.__ 

```Bash
docker exec -it mnist_cassandra cqlsh 
```

__9. view the data recorded in Cassandra database.__ 

```Bash
USE mnist_key;
select*from record_data;
```

:blush:

`Just a kindly remainder, we can directly find the project_video and project_report in docker_MNIST/summary folder. `




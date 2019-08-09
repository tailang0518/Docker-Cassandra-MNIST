# MNIST
In this project, the main job is to recognize the handwritten digit by the database which was construced by Keras. 

While the user goes to the port and draw the digit in the canvas,the router then saves the image and requests the prediction from the MNIST Keras model. After the result returns, the Router forwards the result to the website and submits the predict result and datetime to the Cassandra Database Container through the Docker Network Bridge.

Finally, the application currently uses a MNIST model and get to 99.33% test accuracy after 4 epochs.

*The following image is what this project looks like, you can click the image and download the raw file to watch the video.* 


[![](https://github.com/tailang0518/MNIST/blob/master/docker_MNIST/summary/Screen%20Shot%20.png)](https://github.com/tailang0518/MNIST/blob/master/docker_MNIST/summary/mnistProject_1.mp4)




# Preparation

This project used two Docker containers, Cassandra database container and our Application container. 

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




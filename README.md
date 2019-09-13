# DSB Docker Images

A place to maintain Docker images we use across projects


## Pushing to Docker Hub

1. First, make sure you are logged in on Docker Hub from `docker`:

  ```
  docker login
  ```
2. Build the image tagging it with its final name:

  ```
  docker build -t <hub-user>/<repo-name>[:<tag>]
  ```

For example:

  ```
  cd pyspark-image && docker build . -t datasciencebrigade/pyspark-image:latest
  ```

3. Push to the repository:

  ```
  docker push <hub-user>/<repo-name>[:<tag>]
  ```

For example:

  ```
  docker push datasciencebrigade/pyspark-image:latest
  ```  

## More Info

For more information on Docker Hub and image tags, visit https://docs.docker.com/docker-hub/repos/#pushing-a-docker-container-image-to-docker-hub

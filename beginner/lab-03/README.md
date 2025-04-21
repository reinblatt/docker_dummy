# Lab 03: Docker Images

## Objective
In this lab, you will learn how to work with Docker images, including pulling, inspecting, and managing them. You'll also learn about image layers and best practices for image management.

## Prerequisites
- Completed Lab 02 (Your First Container)
- Docker Desktop running on your system
- Basic understanding of container concepts

## Key Concepts
- Image Layers: Docker images are made up of multiple read-only layers
- Image Registry: A storage and distribution system for Docker images
- Image Tag: A label applied to Docker images to identify different versions
- Image Digest: A unique identifier for an image's content

## Exercises

### 1. Image Management Basics
1. List all images on your system:
```powershell
docker images
```

2. Pull a specific version of an image:
```powershell
docker pull nginx:1.21.6
```

3. Tag an existing image:
```powershell
docker tag nginx:1.21.6 my-nginx:v1
```

4. Remove an image:
```powershell
docker rmi my-nginx:v1
```

### 2. Image Inspection
1. View detailed information about an image:
```powershell
docker inspect nginx:latest
```

2. View the history of an image:
```powershell
docker history nginx:latest
```

3. Save an image to a tar file:
```powershell
docker save -o nginx.tar nginx:latest
```

4. Load an image from a tar file:
```powershell
docker load -i nginx.tar
```

### 3. Image Cleanup
1. Remove all unused images:
```powershell
docker image prune -a
```

2. Remove dangling images (untagged images):
```powershell
docker image prune
```

## Challenges

1. Pull three different versions of the Ubuntu image
2. Compare their sizes using `docker images`
3. Create custom tags for each version
4. Export one of the images and then import it with a new name
5. Clean up all images except the latest Ubuntu version

## Best Practices
1. Always use specific version tags instead of 'latest'
2. Regularly clean up unused images to save disk space
3. Use multi-stage builds to reduce image size
4. Keep images minimal by removing unnecessary packages
5. Use .dockerignore to exclude unnecessary files

## Common Commands Reference
```powershell
# List images
docker images

# Pull an image
docker pull <image-name>:<tag>

# Tag an image
docker tag <source-image> <target-image>

# Remove an image
docker rmi <image-name>

# Save an image to file
docker save -o <file-name>.tar <image-name>

# Load an image from file
docker load -i <file-name>.tar

# Clean up unused images
docker image prune [options]
```

## Next Steps
- Once you're comfortable with image management, move on to Lab 04: Dockerfile Basics
- Practice the commands until you can execute them without referring to the documentation

## Additional Resources
- [Docker Images Documentation](https://docs.docker.com/engine/reference/commandline/images/)
- [Docker Hub](https://hub.docker.com/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) 
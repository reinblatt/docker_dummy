# Lab 01: Docker Installation and Setup

## Objective
In this lab, you will learn how to install Docker on your system and verify the installation. This is the first step in your Docker learning journey.

## Prerequisites
- A Windows 10/11 system with WSL 2 enabled
- Administrator access to your system
- At least 4GB of RAM
- 64-bit processor

## Installation Steps

### 1. Install WSL 2 (Windows Subsystem for Linux)
1. Open PowerShell as Administrator and run:
```powershell
wsl --install
```
2. Restart your computer when prompted

### 2. Install Docker Desktop
1. Download Docker Desktop for Windows from [Docker's official website](https://www.docker.com/products/docker-desktop)
2. Run the installer and follow the installation wizard
3. When prompted, make sure to enable WSL 2 integration
4. Start Docker Desktop after installation

### 3. Verify Installation
1. Open PowerShell and run:
```powershell
docker --version
```
2. You should see output similar to:
```
Docker version 24.0.7, build 24.0.7
```

### 4. Test Docker
1. Run the hello-world container:
```powershell
docker run hello-world
```
2. You should see a message confirming that your installation is working correctly

## Common Issues and Solutions

### Issue 1: WSL 2 not installed
- Solution: Follow the WSL 2 installation guide from Microsoft's documentation

### Issue 2: Docker Desktop won't start
- Solution: Check if virtualization is enabled in your BIOS
- Solution: Ensure WSL 2 is properly installed and running

### Issue 3: Permission denied errors
- Solution: Make sure you're running commands in an elevated PowerShell window

## Exercises

1. Install Docker Desktop on your system
2. Verify the installation by running `docker --version`
3. Run the hello-world container
4. Check Docker's system information using `docker info`

## Next Steps
- Once you've completed this lab, you're ready to move on to Lab 02: Your First Container
- Make sure you can run all the verification commands without errors

## Additional Resources
- [Docker Desktop for Windows documentation](https://docs.docker.com/desktop/install/windows-install/)
- [WSL 2 installation guide](https://docs.microsoft.com/en-us/windows/wsl/install)
- [Docker troubleshooting guide](https://docs.docker.com/desktop/troubleshoot/overview/) 
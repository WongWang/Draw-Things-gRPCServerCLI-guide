# Draw Things gRPCServerCLI Implementation Guide

A comprehensive, step-by-step guide for setting up the gRPCServerCLI from the [draw-things-community](https://github.com/drawthingsai/draw-things-community) project, with a focus on Windows implementation via WSL2 and common pitfalls.

## Overview

### What is Draw Things?

Draw Things is a powerful AI image generation application for iPhone and iPad. The [draw-things-community](https://github.com/drawthingsai/draw-things-community) project provides gRPCServerCLI, a high-performance backend server that allows your iOS devices to offload computationally intensive image generation tasks to a computer with a powerful GPU.

### Why Use gRPCServerCLI?

Mobile devices have limited processing power and thermal constraints. By leveraging gRPCServerCLI, you can:

- **Offload Processing**: Let your iPhone/iPad send image generation requests to a desktop/server with a high-performance GPU (NVIDIA, AMD, or even Apple Silicon)
- **Faster Generation**: Utilize the full power of desktop-class GPUs for significantly faster image generation
- **Preserve Battery**: Reduce battery drain on your mobile device by moving heavy computation to a dedicated machine
- **Better Thermal Management**: Avoid thermal throttling on mobile devices during extended generation sessions

### Platform Support

**Official Support** (from draw-things-community):
- macOS (native support with Metal)
- Linux (native support with CUDA/ROCm)

**Community Support** (this guide):
- **Windows** via WSL2 (Windows Subsystem for Linux 2)
- **Windows** via Docker

> ⚠️ **Important**: While official guides from draw-things-community focus on macOS and Linux, this repository provides detailed instructions for running gRPCServerCLI on Windows using WSL2 or Docker, addressing the unique challenges and common mistakes encountered in these environments.

## Why This Guide?

The official draw-things-community repository provides the source code and basic setup instructions for macOS and Linux. However, Windows users face additional complexity when setting up the gRPCServer through WSL2. This guide:

1. **Fills the Windows Gap**: Provides detailed, Windows-specific setup instructions using WSL2 and Docker
2. **Highlights Pitfalls**: Identifies and explains common mistakes and setup issues specific to WSL2 environments
3. **Step-by-Step Approach**: Breaks down the entire process into clear, actionable steps
4. **Troubleshooting Focus**: Includes comprehensive troubleshooting for mistake-prone areas
5. **Community-Driven**: Based on real-world implementation experience and common issues encountered by the community

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Methods](#installation-methods)
  - [Method 1: WSL2 Installation](#method-1-wsl2-installation)
  - [Method 2: Docker Installation](#method-2-docker-installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [WSL2-Specific Warnings and Common Pitfalls](#wsl2-specific-warnings-and-common-pitfalls)
- [Troubleshooting](#troubleshooting)
- [Performance Optimization](#performance-optimization)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

### Hardware Requirements

- **Desktop/Server with GPU**:
  - NVIDIA GPU with CUDA support (recommended for best performance)
  - AMD GPU with ROCm support
  - Or Apple Silicon Mac with Metal support
- **iOS Device**: iPhone or iPad with Draw Things app installed
- **Network**: Both devices must be on the same local network (or accessible via VPN/port forwarding)

### Software Requirements

#### For Windows Users (WSL2 Method)

- Windows 10 version 2004+ or Windows 11
- WSL2 enabled and configured
- Ubuntu 20.04 or later (recommended distribution for WSL2)
- NVIDIA GPU: CUDA drivers installed on Windows host
- At least 10GB free disk space in WSL2

#### For Windows Users (Docker Method)

- Windows 10 version 2004+ or Windows 11
- Docker Desktop with WSL2 backend
- NVIDIA GPU: NVIDIA Container Toolkit
- At least 15GB free disk space

#### For macOS/Linux Users

- Refer to the official [draw-things-community](https://github.com/drawthingsai/draw-things-community) documentation

## Installation Methods

### Method 1: WSL2 Installation

This method provides better performance and more control but requires careful setup to avoid common pitfalls.

#### Step 1: Enable WSL2 and Install Ubuntu

```bash
# Open PowerShell as Administrator
wsl --install -d Ubuntu-22.04

# After installation, restart your computer
# Launch Ubuntu from Start menu and create a user account
```

#### Step 2: Install CUDA Support in WSL2

> ⚠️ **CRITICAL**: Do NOT install NVIDIA drivers inside WSL2. The drivers are provided by the Windows host.

```bash
# Update package lists
sudo apt update && sudo apt upgrade -y

# Install CUDA toolkit (without drivers)
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-3

# Verify CUDA installation
nvidia-smi  # This should show your GPU information
```

> ⚠️ **Common Mistake #1**: Installing NVIDIA drivers inside WSL2 will cause conflicts. WSL2 automatically uses the GPU drivers from Windows.

#### Step 3: Install Build Dependencies

```bash
# Install essential build tools
sudo apt install -y build-essential git cmake pkg-config

# Install Python and pip
sudo apt install -y python3 python3-pip python3-dev

# Install additional dependencies (may vary by project requirements)
sudo apt install -y libhdf5-dev libssl-dev
```

#### Step 4: Clone and Build gRPCServerCLI

```bash
# Clone the draw-things-community repository
git clone https://github.com/drawthingsai/draw-things-community.git
cd draw-things-community

# Follow the build instructions from the official repository
# (Build steps will vary based on the current state of the project)
```

> 💡 **Note**: Refer to the official repository for the latest build instructions, as they may change over time.

#### Step 5: Configure Network Access

> ⚠️ **Common Mistake #2**: WSL2 uses a virtual network adapter. Your iOS device needs to connect to the correct IP address.

```bash
# Get your WSL2 IP address
hostname -I

# This IP is only accessible within your Windows machine
# You need to use your Windows host IP for external access
```

**Port Forwarding Setup** (required for iOS access):

```powershell
# Run in PowerShell as Administrator
# Replace <WSL2_IP> with the IP from hostname -I
# Replace <PORT> with the gRPC server port (e.g., 50051)

netsh interface portproxy add v4tov4 listenport=<PORT> listenaddress=0.0.0.0 connectport=<PORT> connectaddress=<WSL2_IP>

# Example:
# netsh interface portproxy add v4tov4 listenport=50051 listenaddress=0.0.0.0 connectport=50051 connectaddress=172.20.144.1
```

**Check Port Forwarding Rules**:

```powershell
netsh interface portproxy show all
```

**Windows Firewall Configuration**:

```powershell
# Allow inbound connections on the gRPC port
New-NetFirewallRule -DisplayName "Draw Things gRPC" -Direction Inbound -LocalPort <PORT> -Protocol TCP -Action Allow
```

> ⚠️ **Common Mistake #3**: Forgetting to configure Windows Firewall will prevent iOS devices from connecting even with port forwarding set up.

### Method 2: Docker Installation

Docker provides easier setup and isolation but may have slightly lower performance.

#### Step 1: Install Docker Desktop

1. Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop) for Windows
   - Verify system requirements (Windows 10 64-bit: Pro, Enterprise, or Education version 21H2 or higher, or Windows 11)
   - Ensure WSL2 is installed and updated before installing Docker Desktop
2. Enable WSL2 backend in Docker Desktop settings
3. Restart Docker Desktop

#### Step 2: Install NVIDIA Container Toolkit

For NVIDIA GPU support in Docker:

```bash
# In WSL2 Ubuntu terminal
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
```

#### Step 3: Build or Pull Docker Image

```bash
# If a Dockerfile is provided in draw-things-community:
cd draw-things-community
docker build -t draw-things-grpc .

# Run with GPU support
docker run --gpus all -p 50051:50051 draw-things-grpc
```

> 💡 **Note**: Check the official repository for Docker-specific instructions, as they may provide pre-built images or specific Dockerfiles.

## Configuration

### Server Configuration

Create a configuration file for the gRPC server (exact format depends on the implementation):

```yaml
# Example configuration (adjust based on actual gRPCServerCLI requirements)
server:
  host: 0.0.0.0
  port: 50051
  
gpu:
  device: cuda:0  # or rocm:0, metal:0
  
models:
  path: /path/to/models
  cache: /path/to/cache
```

### iOS App Configuration

1. Open Draw Things on your iPhone/iPad
2. Go to Settings/Preferences
3. Find the "Remote Server" or "gRPC Server" option
4. Enter your server details:
   - **Host**: Your Windows PC's local IP address (not WSL2 IP)
   - **Port**: The port you configured (default: 50051)

To find your Windows IP address:

```powershell
# In PowerShell
ipconfig

# Look for "IPv4 Address" under your active network adapter
# Usually something like 192.168.1.xxx
```

## Usage

### Starting the Server

```bash
# In WSL2 or Linux terminal
cd draw-things-community
./gRPCServerCLI --config config.yaml

# Or with command-line arguments
./gRPCServerCLI --host 0.0.0.0 --port 50051 --gpu cuda:0
```

### Connecting from iOS

1. Ensure your iOS device and Windows PC are on the same network
2. Start the gRPC server on your PC
3. Open Draw Things on iOS
4. Configure the server connection (see Configuration section)
5. Test the connection by generating an image

### Stopping the Server

```bash
# Press Ctrl+C in the terminal running the server
# Or use systemd/supervisor for managed service
```

## WSL2-Specific Warnings and Common Pitfalls

### Critical Issues to Avoid

#### 1. **Driver Installation in WSL2**
- ❌ **NEVER** install NVIDIA drivers inside WSL2
- ✅ Install CUDA drivers on Windows host only
- ✅ WSL2 automatically accesses GPU through Windows drivers
- **Symptom**: "CUDA driver version is insufficient" errors
- **Solution**: Remove any NVIDIA drivers installed in WSL2, ensure Windows drivers are up to date

#### 2. **Network Configuration**
- ❌ Don't use WSL2 IP address (172.x.x.x) for iOS connection
- ✅ Use Windows host IP address (192.168.x.x)
- ✅ Set up port forwarding from Windows to WSL2
- **Symptom**: "Connection refused" or "Connection timeout"
- **Solution**: Configure port forwarding and firewall rules (see Installation Method 1, Step 5)

#### 3. **File System Performance**
- ❌ Don't store models in Windows filesystem (`/mnt/c/`)
- ✅ Store models in WSL2 native filesystem (`/home/user/`)
- **Symptom**: Extremely slow model loading times
- **Solution**: Move models to WSL2 filesystem using `cp` or `rsync`

#### 4. **Memory and Resource Limits**
- ❌ Default WSL2 memory limit may be too low
- ✅ Configure `.wslconfig` to allocate sufficient memory
- **Location**: `C:\Users\<YourUsername>\.wslconfig`
- **Example**:
  ```ini
  [wsl2]
  memory=16GB
  processors=8
  swap=8GB
  ```
- **Symptom**: Out-of-memory errors during large model loading
- **Solution**: Create or edit `.wslconfig`, restart WSL2 (`wsl --shutdown`)

#### 5. **Port Forwarding Persistence**
- ❌ Port forwarding rules reset after Windows reboot
- ✅ Create a startup script to re-apply rules
- **Symptom**: Server works after setup but stops working after reboot
- **Solution**: Create a scheduled task to run port forwarding commands at startup

#### 6. **Firewall Interference**
- ❌ Assuming port forwarding is enough
- ✅ Configure both port forwarding AND firewall rules
- **Symptom**: Port forwarding shows correct rules, but connection still fails
- **Solution**: Add Windows Firewall inbound rule for the gRPC port

### WSL2 Best Practices

1. **Keep WSL2 Updated**: Regularly update WSL2 kernel and distribution
   ```bash
   wsl --update
   sudo apt update && sudo apt upgrade
   ```

2. **Monitor Resource Usage**: Use `htop` in WSL2 and Task Manager in Windows
   ```bash
   sudo apt install htop
   htop
   ```

3. **Backup Configuration**: Keep copies of your config files and port forwarding scripts

4. **Use systemd Services**: Configure gRPCServerCLI as a systemd service for auto-start
   ```bash
   # Example systemd service file
   sudo nano /etc/systemd/system/draw-things-grpc.service
   ```

## Troubleshooting

### Connection Issues

**Problem**: iOS app cannot connect to server

**Diagnosis Steps**:
```bash
# 1. Verify server is running
ps aux | grep gRPCServerCLI

# 2. Check if server is listening
netstat -tulpn | grep 50051

# 3. Test if port is accessible (check port connectivity only)
nc -zv localhost 50051
# Or install grpcurl for proper gRPC testing:
# sudo apt install -y grpcurl
# grpcurl -plaintext localhost:50051 list

# 4. Check Windows port forwarding
# (In PowerShell)
netsh interface portproxy show all

# 5. Test connectivity from another device
# (On another computer or phone, use a network tool to test connectivity to Windows IP:port)
```

**Solutions**:
- Verify server is running: `ps aux | grep gRPCServerCLI`
- Check port forwarding rules are active
- Ensure Windows Firewall allows the port
- Confirm iOS device is on the same network
- Try connecting using Windows IP address, not WSL2 IP

### GPU Not Detected

**Problem**: "No CUDA devices found" or similar errors

**Diagnosis Steps**:
```bash
# Check GPU visibility in WSL2
nvidia-smi

# Verify CUDA installation
nvcc --version

# Check CUDA libraries
ldconfig -p | grep cuda
```

**Solutions**:
- Run `nvidia-smi` in WSL2 to verify GPU access
- Update NVIDIA drivers on Windows host
- Reinstall CUDA toolkit (without drivers) in WSL2
- Check that no NVIDIA drivers were installed inside WSL2

### Performance Issues

**Problem**: Slow image generation or model loading

**Common Causes and Solutions**:

1. **Models on Windows filesystem**:
   - Move models to WSL2 filesystem (`/home/user/models/`)
   - Use `rsync` for large transfers

2. **Insufficient memory**:
   - Edit `.wslconfig` to allocate more RAM
   - Restart WSL2: `wsl --shutdown`

3. **CPU bottleneck**:
   - Allocate more processors in `.wslconfig`
   - Check host system resource usage

4. **Network latency**:
   - Use wired connection instead of WiFi
   - Ensure both devices are on same network segment

### Build Errors

**Problem**: Compilation or build failures

**Common Solutions**:
- Install all required dependencies
- Check compiler version compatibility
- Ensure sufficient disk space
- Review official repository's build requirements
- Check for WSL2-specific build flags or requirements

### Port Forwarding Not Working

**Problem**: Port forwarding rules are set but connection still fails

**Solutions**:
1. Restart WSL2: `wsl --shutdown` (in PowerShell)
2. Check that WSL2 IP hasn't changed (it changes on each restart)
3. Update port forwarding rules with new WSL2 IP
4. Verify firewall rule is active:
   ```powershell
   Get-NetFirewallRule -DisplayName "Draw Things gRPC"
   ```

### Server Crashes or Instability

**Problem**: Server crashes during operation

**Diagnosis**:
```bash
# Check system logs
sudo journalctl -xe

# Monitor memory usage
watch -n 1 free -h

# Check for segmentation faults
dmesg | grep gRPCServerCLI
```

**Solutions**:
- Increase memory allocation in `.wslconfig`
- Check for out-of-memory errors
- Verify model files are not corrupted
- Update to latest version of gRPCServerCLI
- Check for conflicts with antivirus software

## Performance Optimization

### GPU Optimization

- Use the latest NVIDIA drivers on Windows
- Enable GPU performance mode (disable power saving)
- Monitor GPU usage: `nvidia-smi -l 1`
- Ensure adequate GPU cooling

### Network Optimization

- Use wired Ethernet connection for both devices when possible
- Configure QoS settings on your router for prioritizing traffic
- Minimize network hops between devices
- Consider 2.5GbE or 10GbE network for high-resolution image generation

### Storage Optimization

- Use NVMe SSD for model storage
- Store models in WSL2 native filesystem
- Enable write-back caching in WSL2
- Pre-load frequently used models

### System Optimization

- Close unnecessary applications on Windows
- Disable Windows Search indexing for WSL2 directories
- Use high-performance power plan
- Monitor system resources during operation

## Additional Resources

- [Official draw-things-community Repository](https://github.com/drawthingsai/draw-things-community)
- [Draw Things App](https://drawthings.ai/)
- [WSL2 Documentation](https://docs.microsoft.com/en-us/windows/wsl/)
- [CUDA on WSL2](https://docs.nvidia.com/cuda/wsl-user-guide/)
- [Docker Desktop Documentation](https://docs.docker.com/desktop/)

## Contributing

This guide is community-driven and welcomes contributions!

### How to Contribute

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Make your changes (fix errors, add sections, improve clarity)
4. Commit your changes (`git commit -am 'Add troubleshooting for X'`)
5. Push to the branch (`git push origin feature/improvement`)
6. Open a Pull Request

### Areas for Contribution

- Additional troubleshooting scenarios
- Performance optimization tips
- Support for other WSL2 distributions
- Docker-specific optimizations
- AMD GPU/ROCm instructions
- Automated setup scripts
- GUI tools for configuration

## Acknowledgments

- **Draw Things Team**: For creating the excellent Draw Things app and open-sourcing the gRPCServerCLI
- **draw-things-community**: For the original implementation and documentation
- **Community Contributors**: For sharing experiences and solutions

## Disclaimer

This is an unofficial community guide and is not affiliated with or endorsed by the Draw Things team. The official source code and documentation are maintained in the [draw-things-community](https://github.com/drawthingsai/draw-things-community) repository.

For official support and the latest updates, please refer to the official repository.

## License

This guide is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.

The draw-things-community project has its own license. Please refer to the official repository for licensing information regarding the gRPCServerCLI software itself.

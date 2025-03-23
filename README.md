# Lilypad GPU Node Setup Guide

A comprehensive guide for setting up a Lilypad node to offer GPU resources to the network.

## Prerequisites

- Ubuntu Linux system (tested on Ubuntu 22.04/24.04)
- NVIDIA GPU with drivers installed
- Internet connection
- Ethereum wallet private key

## Installation Steps

### 1. Install Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y docker-ce
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

**Note**: Log out and back in for the group changes to take effect.

### 2. Install NVIDIA Container Toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker
```

### 3. Download Lilypad Binary

```bash
mkdir -p ~/bin
wget https://github.com/Lilypad-Tech/lilypad/releases/download/v2.15.0/lilypad-linux-amd64-gpu -O ~/bin/lilypad
chmod +x ~/bin/lilypad
echo 'export PATH=$PATH:~/bin' >> ~/.bashrc
source ~/.bashrc
```

### 4. Verify Lilypad Installation

```bash
~/bin/lilypad version
```

### 5. Create Data Directory

```bash
sudo mkdir -p /lilypad/data
```

### 6. Install Bacalhau

Bacalhau is a required component for Lilypad to work properly:

```bash
curl -sL https://get.bacalhau.org/install.sh | bash
```

Verify Bacalhau installation:
```bash
bacalhau version
```

### 7. Prepare Your Ethereum Wallet

You need an Ethereum wallet private key to participate in the Lilypad network. Make sure it starts with "0x".

### 8. Start Your Lilypad Node

```bash
sudo tmux new -s lilypad
~/bin/lilypad resource-provider --lilynext --web3-private-key=YOUR_PRIVATE_KEY_HERE --offer-modules=gpu-cuda --offer-gpu=1000
```

Replace `YOUR_PRIVATE_KEY_HERE` with your actual private key. 

**Tip**: Press Ctrl+B then D to detach from the tmux session while keeping it running.

### 9. Check Node Status

To check your node's status:
```bash
sudo tmux attach -t lilypad
```

You should see logs indicating:
- Connection to the Arbitrum Sepolia network
- Detection of your GPU(s)
- Successful preflight checks
- Registration as a resource provider

## Troubleshooting

### Lilypad command not found
Make sure the binary is in your PATH or use the full path to the binary (~/bin/lilypad).

### GPU not detected
Make sure your NVIDIA drivers are installed and working:
```bash
nvidia-smi
```

### Can't connect to Ethereum network
Make sure your internet connection is working and stable.

### Error about libnvidia-ml.so or CUDA not found
Install the NVIDIA drivers:
```bash
sudo apt install -y nvidia-driver-525
```
(You'll need to reboot after this)

## Managing Your Node

### Stopping Your Node
Attach to the tmux session with `sudo tmux attach -t lilypad` and press Ctrl+C

### Starting Your Node Again 
Run the start command from Step 7

### Auto-starting After Reboot (Optional)

Create a systemd service file:

```bash
sudo nano /etc/systemd/system/lilypad.service
```

Add the following content:

```
[Unit]
Description=Lilypad GPU Node
After=network.target

[Service]
Type=simple
User=root
ExecStart=/home/YOUR_USERNAME/bin/lilypad resource-provider --lilynext --web3-private-key=YOUR_PRIVATE_KEY_HERE --offer-modules=gpu-cuda --offer-gpu=1000
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Replace `YOUR_USERNAME` and `YOUR_PRIVATE_KEY_HERE` with your actual values.

Enable and start the service:

```bash
sudo systemctl enable lilypad.service
sudo systemctl start lilypad.service
```

## Security Considerations

- Store your private key securely
- Consider using a dedicated wallet for your Lilypad node
- Regularly update your system and Docker for security patches

## Resources

- [Lilypad Documentation](https://docs.lilypad.tech/)
- [Lilypad Discord](https://discord.gg/lilypad)
- [GitHub Repository](https://github.com/Lilypad-Tech/lilypad)

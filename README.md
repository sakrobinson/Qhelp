
# Setting Up a Quilibrium Ceremony Client Node

To set up a Quilibrium Ceremony Client node and start acquiring $QUIL, follow these simplified steps. This guide includes session management with `tmux` and comprehensive process management instructions, now ordered to minimize the need for restarts.

## 1. Provision a VPS

- Choose a VPS provider and set up a server running **Ubuntu 22.04**.

- Use SSH to connect to your VPS.

## 2. Install Prerequisite Software

Execute the following commands to install necessary software:

  ```bash
  sudo apt -q update
  sudo apt-get install jq tmux -y  # Includes tmux installation
  sudo apt install git -y
  git --version  # Ensure git version is 2.34.1 or later
  ```

## 3. Install Golang

Download and install Golang with these commands:

  ```bash
  wget https://go.dev/dl/go1.20.14.linux-amd64.tar.gz
  sudo tar -xvf go1.20.14.linux-amd64.tar.gz -C /usr/local
  echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
  echo 'export GOPATH=$HOME/go' >> ~/.bashrc
  echo 'export PATH=$GOPATH/bin:$GOROOT/bin:$PATH' >> ~/.bashrc
  source ~/.bashrc
  go version  # Verify installation with go version go1.20.14 linux/amd64
  ```

## 4. Clone the Ceremony Client Repo

Clone the repository and navigate to the node directory:

```bash
git clone https://github.com/QuilibriumNetwork/ceremonyclient.git
cd ceremonyclient/node
```

## 5. Configure the firewall to allow necessary traffic:

  ```bash
  sudo ufw allow 22 && sudo ufw allow 8336 && sudo ufw allow 8337 && sudo ufw allow 8338 && sudo ufw allow 8317 && sudo ufw allow 8316 && sudo ufw enable
  sudo ufw status  # Confirm firewall settings
  ```

## 6. Increase buffer size for beter network performance and reboot:

  ```bash
  echo -e "\n# Increase buffer sizes for better network performance/nnet.core.rmem_max=60000000\n net.core.wmem_max=600000000" | sudo tee -a /etc/sysctl.conf > /dev/null && sudo sysctl -p
  ```
Now reboot your server:

  ```bash
  sudo reboot
  ```
Wait a couple minutes for your server to reboot and reconnect with SSH

## 7. Start Node in a tmux session

Initiate a tmux session:

```bash
tmux new -s quilnode
```

Start the node within the tmux session:

```bash
GOEXPERIMENT=arenas go run ./...
```

### You are now running a quilibrium node!

## *** some notes on tmux ***

- Detach from the tmux session with `Ctrl+b` then `d` 
- Reattach to tmux session with `tmux a`
- Open a new window within the session with `Ctrl + b` then `c`
- Cycle through open windows with `Ctrl + b` then `p`
- Split window horizontally with `Ctrl + b` then `"`
- Split window vertically with `Ctrl + b` then `%`

# Node Management

## Restarting the node
- ### Auto Restart:

  - Navigate to the node window in tmux
  - Pause the node process with `Ctrl+z`
  - Make the auto script executable:

    ```bash
    chmod +x poor_mans_cd.sh
    ```
  - Start the auto script
    ```bash
    ./poor_mans_cd.sh
    ```
- ### Manual Restart:

  - Navigate to the node window in tmux
  - Pause the node process with `Ctrl+z`
  - Find the PID of the node process. Look for the number for any proces ending in `/b001/exe/node `
    ```bash
    ps -aux | grep go # PID number for the process ending in /b001/exe/node
    ```
  - Use the PID to kill the process:
    ```bash
    kill PID
    ```
  - Recheck the node processes and confirm deletion. If not deleted force kill with:
    ```bash
    kill -9 PID
    ```
  - Now restart the node with:

    ```bash
    GOEXPERIMENT=arenas go run ./...
    ```

  - Restart the node with:

    ```bash
    cd ~/ceremonyclient/node
    GOEXPERIMENT=arenas go run ./...
    ```

## Configure gRPC to enable node query:

- Edit `.config/config.yml`:

  ```bash
  nano ~/ceremonyclient/node/.config/config.yml
  ```
- Scroll to the bottom with the arrow keys to the line, `listenGrpcMultiaddr: ""`, and replace with:
  ```bash
  listenGrpcMultiaddr: /ip4/127.0.0.1/tcp/8337
  ```
- Save the file with `Ctrl+O`, and exit with `Ctrl+X`.
- Restart the node


## Get Node Peer ID and Balance

- Get your Node Peer ID:

  ```bash
  cd ~/ceremonyclient/node
  GOEXPERIMENT=arenas go run ./... -peer-id
  ```
- Get your $QUIL balance (`gRPC` must be configured):

  ```bash
  cd ~/ceremonyclient/node
  GOEXPERIMENT=arenas go run ./... -balance
  ```

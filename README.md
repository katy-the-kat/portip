# Port Forwarding Script

This script is designed to help you forward ports on a VPS or a router/ISP that doesn’t have port forwarding. It uses `ssh` to create reverse tunnels through `serveo.net`.

## Installation

To install the script, use the following commands:

```bash
echo '#!/bin/bash

PORTS_FILE="/ports.info.txt"

add_port() {
    local local_port=$1
    if [[ -z "$local_port" ]]; then
        echo "Please provide a local port to forward."
        exit 1
    fi
    
    local random_port
    random_port=$(shuf -i 1024-65535 -n 1)
    
    ssh -o StrictHostKeyChecking=no -f -N -R ${random_port}:localhost:${local_port} serveo.net > /dev/null &
    ssh_pid=$!
    
    echo "${random_port}:${local_port}" >> $PORTS_FILE
    
    echo "${local_port} is now on 138.68.79.95:${random_port}"
}

remove_port() {
    local local_port=$1
    if [[ -z "$local_port" ]]; then
        echo "Please provide a local port to remove."
        exit 1
    fi
    
    random_port=$(grep ":${local_port}$" $PORTS_FILE | cut -d':' -f1)
    
    if [[ -z "$random_port" ]]; then
        echo "Port ${local_port} not found."
        exit 1
    fi
    
    pkill -f "ssh -o StrictHostKeyChecking=no -f -N -R ${random_port}:localhost:${local_port} serveo.net" > /dev/null
    
    sed -i "/${random_port}:${local_port}/d" $PORTS_FILE > /dev/null
    
    echo "Port ${local_port} has been removed."
}

refresh_ports() {
    if [[ ! -f "$PORTS_FILE" ]]; then
        echo "No ports to refresh."
        exit 1
    fi
    
    while IFS= read -r line; do
        random_port=$(echo $line | cut -d':' -f1)
        local_port=$(echo $line | cut -d':' -f2)
        
        ssh -o StrictHostKeyChecking=no -f -N -R ${random_port}:localhost:${local_port} serveo.net > /dev/null &
    done < $PORTS_FILE
    
    echo "Ports have been successfully restarted."
}

list_ports() {
    if [[ ! -f "$PORTS_FILE" ]]; then
        echo "No ports to list."
        exit 1
    fi
    
    echo "Current port mappings:"
    while IFS= read -r line; do
        random_port=$(echo $line | cut -d':' -f1)
        local_port=$(echo $line | cut -d':' -f2)
        echo "Local port ${local_port} -> Public port ${random_port} (138.68.79.95)"
    done < $PORTS_FILE
}

# Main function to parse the input commands
case "$1" in
    add)
        add_port "$2"
        ;;
    remove)
        remove_port "$2"
        ;;
    refresh)
        refresh_ports
        ;;
    list)
        list_ports
        ;;
    *)
        echo "Usage: $0 {add|remove|refresh|list} [port]"
        exit 1
        ;;
esac
' > /bin/portip

sudo chmod +x /bin/portip
```

## Usage

Once installed, you can use the `portip` command to manage your port forwarding. Here are the available commands:

- **Add a port:**

    ```bash
    portip add [local_port]
    ```

    This will forward the specified local port to a randomly chosen public port on `serveo.net`.

- **Remove a port:**

    ```bash
    portip remove [local_port]
    ```

    This will stop forwarding the specified local port and remove the mapping.

- **Refresh all ports:**

    ```bash
    portip refresh
    ```

    This will re-establish all port mappings listed in `/ports.info.txt`.

- **List all ports:**

    ```bash
    portip list
    ```

    This will display all current port mappings in the format: `Local port [local_port] -> Public port [random_port] (138.68.79.95)`.

## Notes

- Make sure you have `ssh` installed and configured on your system.
- `/ports.info.txt` will be created in the directory where the script is executed, storing the port mappings.


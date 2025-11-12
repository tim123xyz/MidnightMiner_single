## Running as a Systemd Service (Linux Only)

> **⚠️ Linux Only**: The `setup-service.sh` and `miner-cmds.sh` scripts are **Linux-only** and require systemd. They will not work on Windows or macOS. If you're using Windows or macOS, please run the miner manually using `python miner.py` instead.

### Setup

Use the included `setup-service.sh` script to configure and install the service:

1. **Set worker count and install**:
   ```bash
   ./setup-service.sh --workers 4 --install
   ```
   This automatically:
   - Creates a Python virtual environment (`venv/`) in the project directory
   - Installs all required dependencies from `requirements.txt`
   - Creates the service configuration and installs it to systemd

   The service will use the virtual environment's Python interpreter, ensuring isolated dependencies.

2. **Start the service**:
   ```bash
   sudo systemctl start midnight-miner
   ```

3. **Enable auto-start on boot** (optional):
   ```bash
   sudo systemctl enable midnight-miner
   ```

### Updating the Service

After pulling the latest code with `git pull`, update the service configuration:

```bash
# Update service file and reload systemd
# This will also update dependencies in the virtual environment if needed
./setup-service.sh --update
```

To change the number of workers:

```bash
# Update workers to 8 and reload service
./setup-service.sh --workers 8 --update
```

**Note:** After updating, restart the service for changes to take effect:
```bash
sudo systemctl restart midnight-miner
```

**Virtual Environment Management**: The `setup-service.sh` script automatically manages the virtual environment. If you need to manually update dependencies, you can activate the venv and run:
```bash
source venv/bin/activate
pip install -r requirements.txt --upgrade
```

### Service Management

The `setup-service.sh` script provides several management commands:

- **Check status**: `./setup-service.sh --status`
- **View logs**: `./setup-service.sh --logs`
- **Restart service**: `./setup-service.sh --restart`
- **Stop service**: `./setup-service.sh --stop`
- **Start service**: `./setup-service.sh --start`
- **Uninstall service**: `./setup-service.sh --uninstall`

Alternatively, you can use standard systemd commands:
```bash
sudo systemctl status midnight-miner
sudo systemctl restart midnight-miner
sudo journalctl -u midnight-miner -f  # View logs
```

### Automatic Updates

The service is configured to automatically run `git pull` before starting, ensuring you always run the latest code. If the git pull fails (e.g., network issues), the service will still start with the existing code.

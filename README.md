# Midnight Miner

A Python-based mining bot for the Midnight Network's scavenger hunt, allowing users to automatically mine for NIGHT tokens with multiple wallets.

**Supported Platforms:** Windows (x64), Linux (x64, ARM64), macOS (Intel, Apple Silicon)

If you are unfamiliar with python, and/or using windows, check out the [Easy Guide](EasyGuide.md).

## Disclaimer

This is an unofficial tool, and has not been properly tested. Use it at your own risk.

I will be updating and improving this software regularly. Please keep up to date by re-downloading from this repository and copying over your `wallets.json` and `challenges.json` files, or simply by running `git pull`.

## How It Works

The miner operates by performing the following steps:
1.  **Wallet Setup**: It creates or loads Cardano wallets to be used for mining. These wallets are stored in `wallets.json`.
2.  **Registration**: The bot registers the wallets with the Midnight Scavenger Mine API and agrees to the terms and conditions.
3.  **Challenge Loop**: It continuously polls the API for new mining challenges. Challenges are stored in `challenges.json` so they can be resumed later.
4.  **Mining**: When a new challenge is received, the bot uses the provided parameters to build a large proof-of-work table (ROM). It then rapidly searches for a valid nonce that solves the challenge's difficulty requirement. The core hashing logic is performed by a native Rust library for optimal performance.
5.  **Submission**: Once a solution is found, it is submitted to the API to earn NIGHT tokens.
6.  **Worker Rotation**: When a worker completes all available challenges for its wallet, it automatically exits and respawns with a different wallet. New wallets are generated automatically as needed.

## Prerequisites

Before running the miner, ensure you have the following:

1.  **Python 3**: The script is written in Python (version 3.13 or higher required).
2.  **Python venv module**: Usually included with Python 3.3+, but can be installed separately if needed.
3.  **Required Python Libraries**: The following packages are required and will be installed automatically when setting up the virtual environment:
    - `pycardano` - Cardano wallet functionality
    - `wasmtime` - WebAssembly runtime
    - `requests` - HTTP requests
    - `cbor2` - CBOR encoding/decoding
    - `portalocker` - Cross-platform file locking

    These are listed in `requirements.txt` and will be installed automatically when you set up the virtual environment.
4. **Git**: Use Git to download and update your Miner easily.

## Download

Run this command to download MidnightMiner
```
git clone https://github.com/djeanql/MidnightMiner && cd MidnightMiner
```

## Usage

### Manual Setup (Recommended for First-Time Users)

1. **Create a virtual environment**:
   ```bash
   python3 -m venv venv
   ```

2. **Activate the virtual environment**:
   - On Linux/macOS:
     ```bash
     source venv/bin/activate
     ```
   - On Windows:
     ```bash
     venv\Scripts\activate
     ```

3. **Install required dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the miner**:
   -   **Start mining**:
       This command will either load an existing wallet from `wallets.json` or create a new one if it doesn't exist.
       ```bash
       python miner.py
       ```

   -   **Multiple workers**:
       To mine with multiple workers, use:
       ```bash
       python miner.py --workers <number of workers>
       ```
       Each worker uses one CPU core and 1GB of RAM. The miner will automatically create enough wallets for all workers and rotate through them as challenges are completed. Each worker always mines to a unique wallet. Do not run more workers than your system is capable of.

**Note**: When using the systemd service (Linux), the virtual environment is automatically created and managed by the `setup-service.sh` script. You don't need to manually create it.


## Running as a Systemd Service (Linux Only)

> **⚠️ Linux Only**: The `setup-service.sh` and `miner-cmds.sh` scripts are **Linux-only** and require systemd. They will not work on Windows or macOS. If you're using Windows or macOS, please run the miner manually using `python miner.py` instead.

For Linux users, you can run the miner as a systemd service, which allows it to:
- Start automatically on boot
- Restart automatically if it crashes
- Automatically pull the latest code from git before starting
- Run in the background without a terminal

The scripts automatically verify that you're running on Linux before proceeding. If you attempt to run them on Windows or macOS, they will display an error message and exit.

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

## Resubmitting Failed Solutions

If solutions fail to submit due to network issues or API errors, they are automatically saved to `solutions.csv`. To resubmit them:

**If using manual setup**, make sure your virtual environment is activated:
```bash
source venv/bin/activate  # On Linux/macOS
# or
venv\Scripts\activate      # On Windows
```

Then run:
```bash
python resubmit_solutions.py
```

The script automatically removes successfully submitted solutions and keeps any that still failed for retry.
You should run this once a day, as solutions can no longer be submitted after 24 hours.

## ⚠️ Update Regularly

This software will be updated frequently, so it is VERY important you update it to earn the highest rewards.

**For manual runs**: Update by running this command in the MidnightMiner directory:
```
git pull
```

**For systemd service**: The service automatically runs `git pull` before starting, so you'll always have the latest code. However, if you want to update without restarting, you can still run `git pull` manually, then update the service configuration:
```bash
git pull
./setup-service.sh --update
sudo systemctl restart midnight-miner
```

I suggest checking for updates once a day to make sure your miner is up-to-date.


## Developer Donations

This miner includes an **optional 5% donation system** to support ongoing development and maintenance. By default, approximately 1 in 20 (5%) of solved challenges will be mined for the developer's address.

Thank you for considering supporting this project!

To disable donations, add the `--no-donation` flag:
```bash
python miner.py --no-donation
```


## Exporting Wallets

To claim your earned NIGHT tokens (when they are distributed), you will need to import your wallets' signing keys (`.skey` files) into a Cardano wallet like Eternl. The `export_skeys.py` script helps with this process.

1.  **Activate virtual environment** (if using manual setup):
    ```bash
    source venv/bin/activate  # On Linux/macOS
    # or
    venv\Scripts\activate      # On Windows
    ```

2.  **Run the export script**:
    ```bash
    python export_skeys.py
    ```
    This will create a directory named `skeys/` (if it doesn't exist) and export each wallet's signing key from `wallets.json` into a separate `.skey` file.

3.  **Import into Eternl (or other Cardano wallet)**:
    *   Open your Eternl wallet.
    *   Go to `Add Wallet` -> `More` -> `CLI Signing Keys`.
    *   Import the `.skey` files generated in the `skeys/` directory.

## Dashboard

The dashboard displays important information about the status of each worker. Each row represents a worker (not a wallet) - workers automatically rotate through different wallets as they complete challenges.

The `Challenge` column shows which challenge ID the worker is currently solving, or status messages like "Waiting" if all known challenges have been completed. The `Attempts` and `H/s` columns show mining progress and hash rate for each worker.

At the bottom, you'll see totals across all wallets:
- **Total Hash Rate**: Combined hash rate of all workers
- **Total Completed**: Total challenges solved by all wallets. The number in brackets shows how many were solved in this session.
- **Total NIGHT**: Estimated NIGHT token rewards across all wallets (fetched once at startup)

If developer donations are enabled (default), when a worker is mining for the developer, the `Address` field for that worker will temporarily show **"developer (thank you!)"** instead of the wallet address.

```
==============================================================================================================
                                          MIDNIGHT MINER - v0.3
==============================================================================================================
Active Workers: 4 | Last Update: 2025-11-05 14:32:18
==============================================================================================================

ID   Address                                      Challenge                  Attempts      H/s
--------------------------------------------------------------------------------------------------------------
0    addr1vxask5vpp8p4xddsc3qd63luy4ecf...        **D05C02                   470,000       2,040
1    developer (thank you!)                       **D05C19                   471,000       2,035
2    addr1v9hcpxeevkks7g4mvyls029yuvvsm0d...      Building ROM               0             0
3    addr1vx64c8703ketwnjtxkjcqzsktwkcvh...      **D05C20                   154,000       2,028
--------------------------------------------------------------------------------------------------------------

Total Hash Rate:     6,103 H/s
Total Completed:     127 (+15)
Total NIGHT:         45.32
==============================================================================================================

Press Ctrl+C to stop all miners
```

## Platform Support

The miner uses a native Rust library (`ashmaize_py`) for high-performance hashing. Pre-compiled binaries are included for all major platforms:

| Platform | Architecture | Status |
|----------|--------------|--------|
| Windows | x64 | ✓ Supported |
| Linux | x64 | ✓ Supported |
| Linux | ARM64 (Raspberry Pi, etc.) | ✓ Supported |
| macOS | Intel (x64) | ✓ Supported |
| macOS | Apple Silicon (ARM64) | ✓ Supported |

The miner automatically detects your operating system and CPU architecture, loading the appropriate binary from the `libs/` directory.

## Visualise Challenge Data

A script `plot_challenges.py` is included to visualize the number of solved challenges over time.

### Prerequisites

This script requires `matplotlib`. If you're using a virtual environment (recommended), make sure it's activated:

```bash
source venv/bin/activate  # On Linux/macOS
# or
venv\Scripts\activate      # On Windows
```

Then install matplotlib:
```bash
pip install matplotlib
```

### Usage

To generate the plot, run the following command:

```bash
python3 plot_challenges.py
```

The script will read the `challenges.json` file and generate a plot named `solved_challenges_over_time.png`.

> **Note:** The graph uses the challenge's discovery time as an approximation for when solutions were found, as individual solution times are not stored.

## Ashmaize Rust Library Source Code

This miner uses the [Ashmaize](https://github.com/input-output-hk/ce-ashmaize) hashing algorithm, developed by IOHK. Included in this repository are binaries for my python bindings module. You can find the code [here](https://github.com/djeanql/ashmaize-py)

## Stars

If you like MidnightMiner, why not star the repository?

[![Star History Chart](https://api.star-history.com/svg?repos=djeanql/MidnightMiner&type=date&legend=top-left)](https://www.star-history.com/#djeanql/MidnightMiner&type=date&legend=top-left)

[![Star History Chart](https://api.star-history.com/svg?repos=djeanql/MidnightMiner&type=date&legend=top-left)](https://www.star-history.com/#djeanql/MidnightMiner&type=date&legend=top-left)
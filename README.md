# MATRiX Robot MC v0.6.4

MATRiX Robot MC is the external motion-control runtime for GENISOM AI robots and the MATRiX simulator.

- Project: <https://github.com/GENISOM-AI/MATRiX_Robot_MC>
- Runtime packages: <https://github.com/GENISOM-AI/MATRiX_Robot_MC/releases>

> [!WARNING]
> This program can control a physical robot. For the first run, securely lift or support the robot, keep people and obstacles away, and make sure the emergency stop is available.

## Motion-control mode in MATRiX

MATRiX provides built-in and external motion control. Select the mode with `robot.inside_mc` in `Linux/UeSim/Content/model/config/config.json`.

| Configuration | Mode | Start this project |
| --- | --- | --- |
| `"inside_mc": true` | MATRiX built-in motion control | No |
| `"inside_mc": false` | External motion control | Yes |

When external motion control is selected, run MATRiX and this project as separate processes. Do not enable built-in and external motion control at the same time.

## Supported robots

Set the matching `ROBOT_TYPE` in `run_mc.sh` before startup.

| Robot model | `ROBOT_TYPE` |
| --- | --- |
| `zsl-1` | `XG` |
| `zsl-2` | `XG` |
| `zsl-1w` | `XGW` |
| `zsl-2w` | `XGW2` |
| `zsm-1w` | `ZGWS` |
| `zgwsarm` | `ZGWS_ARM` |

`ROBOT_TYPE` is case-sensitive. Use the exact value shown above.

## Getting started

### 1. Download the runtime package

1. Open [GitHub Releases](https://github.com/GENISOM-AI/MATRiX_Robot_MC/releases).
2. Select the required motion-control version.
3. Download the runtime package for Linux x86_64.
4. Extract the package and enter the extracted directory.
5. Make the scripts executable:

```bash
chmod +x install_deps.sh run_mc.sh
```

> [!IMPORTANT]
> Download the runtime asset attached to the release. Do not use GitHub's automatically generated `Source code (zip)` or `Source code (tar.gz)` archives, because they do not contain the complete runtime environment.

### 2. Install the runtime environment

The runtime environment is required only for this external motion controller. Install it from the extracted Robot MC directory; no installation is needed from the parent MATRiX directory.

Requirements:

- Ubuntu 22.04 x86_64;
- internet access;
- `sudo` permission;
- `install_deps.sh` and `deps/` kept together in the extracted directory.

From the extracted Robot MC directory, run:

```bash
chmod +x install_deps.sh
./install_deps.sh
```

The script installs the required system packages and the bundled local packages from `deps/`. Run it once after downloading or updating the runtime package.

After installation, check for missing shared libraries:

```bash
ldd build/export/mc/bin/mc_ctrl | grep "not found" || true
```

No output means that no missing shared libraries were detected. If a library is listed, rerun `install_deps.sh` and review its error output before starting motion control.

### 3. Select the robot model

Open the launcher:

```bash
vim run_mc.sh
```

Find this line:

```bash
export ROBOT_TYPE=XG
```

Replace the value after `=` according to the supported-robots table.

For `zsl-1` or `zsl-2`:

```bash
export ROBOT_TYPE=XG
```

For `zsl-1w`:

```bash
export ROBOT_TYPE=XGW
```

For `zsl-2w`:

```bash
export ROBOT_TYPE=XGW2
```

For `zsm-1w`:

```bash
export ROBOT_TYPE=ZGWS
```

For `zgwsarm`:

```bash
export ROBOT_TYPE=ZGWS_ARM
```

Save the file after making the change.

> [!IMPORTANT]
> An incorrect robot type may load incompatible robot parameters. Verify `ROBOT_TYPE` before every run.

### 4. Configure the robot network

Open the SDK configuration:

```bash
vim config/sdk_config.yaml
```

Set the robot IP address and port. For example:

```yaml
target_ip: "192.168.234.1"
target_port: 43988
```

Synchronize the runtime configuration after editing it:

```bash
cp config/sdk_config.yaml build/export/config/sdk_config.yaml
```

Verify that the robot is reachable:

```bash
ping -c 3 192.168.234.1
```

Replace the example address with the actual robot IP address.

### 5. Start motion control

Before startup, confirm that:

- the physical or simulated robot matches `ROBOT_TYPE`;
- the robot driver and IMU are running;
- the control computer can reach the robot;
- the robot is securely supported or in a safe test area;
- the gamepad and emergency stop are available.

Start the controller:

```bash
./run_mc.sh r
```

For MATRiX external motion control, start the MATRiX simulator first, then run the command above in a second terminal.

The launcher prints the local SDK address selected for the control computer. If automatic address selection is incorrect, specify the IPv4 address of the computer's wired network interface:

```bash
SDK_CLIENT_IP=192.168.234.10 ./run_mc.sh r
```

Replace `192.168.234.10` with an IPv4 address assigned to the control computer. Do not use the robot IP address.

## Basic gamepad controls

| Input | Action |
| --- | --- |
| `LB + RB` | Passive / release torque |
| `LB + X` | Enter ready state |
| `LB + Y` | Stand up |
| `LB + B` | Balance stand |
| Left stick | Move forward, backward, or sideways |
| Right stick left/right | Turn |

To stop, first use `LB + Y` to return to standing or `LB + RB` to enter passive mode. Then press `Ctrl+C` to terminate the program.

## Troubleshooting

### The local SDK address cannot be detected

Confirm that the control computer is connected through a wired network interface, then specify its address manually:

```bash
SDK_CLIENT_IP=<CONTROL_COMPUTER_WIRED_IP> ./run_mc.sh r
```

### The robot cannot be reached

- Check the network cable.
- Verify that `target_ip` is the robot's actual IP address.
- Make sure both copies of `sdk_config.yaml` are synchronized.
- Confirm that the control computer and robot can reach each other.
- Confirm that the robot driver is running.

### The wrong robot model is loaded

Open `run_mc.sh`, verify the spelling and capitalization of `ROBOT_TYPE` against the supported-robots table, and restart the controller.

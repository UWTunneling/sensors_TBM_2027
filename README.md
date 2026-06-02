# sensor_anomaly_detection

This is an [ESP-IDF](https://docs.espressif.com/projects/esp-idf/en/latest/) project targeting the **ESP32-S3**.

## Development Setup (ESP-IDF + VS Code)

The easiest way to build, flash, and monitor is the official **ESP-IDF VS Code extension**, which installs the toolchain for you.

### 1) Install VS Code

Download and install [Visual Studio Code](https://code.visualstudio.com/).

### 2) Install the ESP-IDF extension

1. Open VS Code → **Extensions** panel (`Cmd+Shift+X`).
2. Search for **"ESP-IDF"** (publisher: Espressif Systems) and click **Install**.

### 3) Run the extension's setup wizard

1. Open the Command Palette (`Cmd+Shift+P`) and run **`ESP-IDF: Configure ESP-IDF Extension`**.
2. Choose **Express** install.
3. Pick an ESP-IDF version (this project uses **v6.0.1**) and accept the default install paths.
4. The wizard downloads ESP-IDF, the Xtensa toolchain, and a Python venv. This takes a while — let it finish.

### 4) Open this project and set the target

1. **File → Open Folder** → select the `sensors_TBM_2027` folder.
2. Set the chip target: Command Palette → **`ESP-IDF: Set Espressif Device Target`** → **`esp32s3`**.
3. Select the serial port: **`ESP-IDF: Select Port to Use`** (on macOS it looks like `/dev/cu.usbserial-XXXX`).

### 5) Build, flash, and monitor

Use the icons in the bottom status bar, or the Command Palette:

- **Build** — `ESP-IDF: Build your Project`
- **Flash** — `ESP-IDF: Flash your Project`
- **Monitor** — `ESP-IDF: Monitor your Device` (press `Ctrl+]` to exit)
- **Build + Flash + Monitor (all at once)** — the 🔥 "flame" icon in the status bar

### Command-line alternative

If you prefer the terminal, open the **`ESP-IDF: Open ESP-IDF Terminal`** (it sources the environment for you), then:

```bash
cd sensors_TBM_2027
idf.py set-target esp32s3   # one time
idf.py build flash monitor
```

> If `set-target` reports a target mismatch, an `IDF_TARGET` environment variable is overriding it — run `unset IDF_TARGET` first. Press `Ctrl+]` to exit the monitor.

## Collaboration Workflow

### 1) Clone the repository (one time)

```bash
git clone https://github.com/hemzp/sensor_anomaly_detection.git
cd sensor_anomaly_detection
```

- `git clone ...` downloads the full project and history from GitHub to your computer.
- `cd sensor_anomaly_detection` moves into the project folder so Git commands run in this repo.

### 2) Start work from latest `main`

```bash
git checkout main
git pull origin main
```

- `git checkout main` switches to the main branch.
- `git pull origin main` gets the newest changes from GitHub to avoid starting from old code.

### 3) Create your own feature branch

```bash
git checkout -b feature/<short-name>
```

- Creates and switches to a new branch for your task.
- Keeps `main` clean and makes review easier.

### 4) Save your work in commits

```bash
git add .
git commit -m "Describe what you changed"
```

- `git add .` stages changed files for the next commit.
- `git commit -m ...` saves a snapshot with a message.

### 5) Push your branch to GitHub

```bash
git push -u origin feature/<short-name>
```

- Uploads your branch to GitHub.
- `-u` sets upstream so next time you can use just `git push`.

### 6) Open a Pull Request (PR)

- Open a PR from `feature/<short-name>` into `main`.
- Ask for at least 1 teammate review before merge.
- Merge after approval and passing checks.

### 7) Keep your branch updated before PR merge

```bash
git checkout main
git pull origin main
git checkout feature/<short-name>
git rebase main
```

- Updates your feature branch on top of latest `main`.
- Reduces merge conflicts and keeps history clean.

### Conflict handling

- If Git reports conflicts, open conflicted files and choose/fix final content.
- Then run:

```bash
git add .
git rebase --continue
```

- Repeat until rebase finishes, then push:

```bash
git push --force-with-lease
```

Use `--force-with-lease` only on your own feature branch after rebase, never on `main`.
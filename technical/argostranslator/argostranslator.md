# Offline Argos Translate GUI Setup – Step by Step

This guide walks through setting up Argos Translate GUI to work offline. This was the **only** truly offline Russian / English translator I could find on Windows desktop. The setup process was not obvious.

## Prerequisites

- Windows
- Python (I am not certain a minimal Python env; I have used from [get old verion] up to python 3.13.15)
- Python `virtualenv` pip library (or built-in venv if you wish to use; on Windows virtualenv has historically worked better for me)
- GitBash for Windows (to create virtualenv)
- Windows Command Prompt (CMD) for launching `argos-translate-gui` binary  — **not Git Bash**
- Internet connection only for initial pip installs and downloading language packs

---

## Note about virtual environment use in this guide

This guide's setup uses a virtual environment for installing argostranslate and argos-translate-gui. **This is not required.** I put them in a virtual environment because they have many external dependencies which end up getting installed (50+ perhaps). Feel free to skip the virtual environment steps if you wish to install argostranslate and argos-translate-gui directly

**Regardless of how you install,** you will ultimately need to launch the argos-translate-gui binary from python, and the path to that binary will with be different depending on if you use a virtual environment or not. Step 3 will walk you through finding that path, regardless of which approach you use.

## Setup

### Step 1: Create and activate a virtual environment

**Skip this step if you don't want to use a virtual environment for your argostranslate installs.**

From Gitbash:

```cmd
virtualenv argos_env
source argos_env/Scripts/activate
```

(or if you are using `venv` directly):

```cmd
python -m venv args_env
source argos_env/Scripts/activate
```

**Note**: Consider creating the virtualenv in a dedicated area such as `C:\\Users\\username\\virtualenvs`

### Step 2: Install the correct versions **(online step)**

```cmd
pip install argostranslate==1.10.0
pip install argostranslategui
```

**Note**: You **must** install argostranslate version `1.10.0`; the current version at time of writing (`1.11.0` in May 2026) will not work offline due to a dependency on `stanza` which seems to do some update checks or to download data files.

Regarding argostranslategui, I am currently using 1.6.5, which is the latest. If you experience issues with later versions, consider installing to this as it's a known working point.

### Step 3: Determine path to args-translate-gui binary

Determine path the args-translate-gui binary installed to in the previous step.

- **If you are using a virtual environment**: the binary will be the virtual environment's `Scripts\` directly (e.g. `C:\Users\username\virtualenvs\argos_env\Scripts`)
- **If you are NOT using a virtual environment:** the binary will be in the scripts directory that pip installs to on your machine. To find that directory, run the following in a console:

	`python -c "import sysconfig; print(sysconfig.get_path('scripts'))"`

For example, in my virtual environment install, the binary lives at: ` C:\Users\username\virtualenvs\argos_env\Scripts\argos-translate-gui`

If I were to do a regular pip install, on my machine, the binary would be at: `C:\Users\username\AppData\Local\Programs\Python\Python313\Scripts\argos-translate-gui`

### Step 4: Launch the GUI from Windows CMD (online step)

**Switch to Windows CMD -- this will not work from Gitbash**

1. **If you are using a virtual environment**, activate the virtual environment.

```cmd
C:\Users\username\virtualenvs\argos_env\Scripts\activate
```

2. Call the argos-translate-gui binary **with python** using the path determined in Step 3. You can not call the binary directly. **I don't believe it's sufficient to add the this to system path, as python needs the actual path to the binary to open it.**

```cmd
python C:\your-install-path\argos-translate-gui
```

**Important:** Run this from **CMD**, not Git Bash.

The GUI may take 10–30 seconds to appear on first launch. Be patient.

### Step 5: Download language packs (online step)

While the GUI is open and **you are still online**:

- Click "manage packages" -> "download packages"
- Scroll down and find your desired packages, and click the download button (**make sure you download both directions if you want full translations; e.g. ru→en and en→ru)**)
- It will turn to an hour glass, and a checkmark once download is complete.
- **This takes about 3–5 minutes**

### Step 6: Go offline and test

- Disconnect from the internet
- Close the GUI
- Launch it again:

```cmd
python argos_env\Scripts\argos-translate-gui
```

It should now open and function completely offline. Note: you might see some errors on the console about `stanza` missing required files, but it should not prevent it rom opening.

---

## Notes

| Item | Details |
|------|---------|
| **Working version** | `argostranslate==1.10.0` |
| **Newer versions** | Break offline functionality (Stanza dependency issue) |
| **Terminal** | Must be CMD or PowerShell — Git Bash will fail |
| **First launch** | Slow to appear — wait 10–30 seconds |
| **Language packs** | Download once while online, then work forever offline |

---

## To keep this working long-term

Pin the version in your environment to avoid accidental upgrades:

```cmd
pip freeze | findstr argostranslate
```

If you ever rebuild this environment, repeat the exact steps above. Do not install a newer `argostranslate`.

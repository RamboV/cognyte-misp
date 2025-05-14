# Cognyte Luminar - MISP Connector Setup Guide

This document provides step-by-step instructions to set up and run a Luminar MISP connector. The connector integrates with a MISP instance to fetch feeds (IOCs, Leaked Records, Cyberfeeds) from Luminar Taxii server and push data to MISP using a Python script. Follow these steps to create a Python environment, install dependencies, configure inputs, and schedule the script to run automatically.

# Introduction
Cognyte is a global leader in security analytics software that empowers governments and enterprises with Actionable Intelligence for a safer world. Our open software fuses, analyzes and visualizes disparate data sets at scale to help security organizations find the needles in the haystacks. Over 1,000 government and enterprise customers in more than 100 countries rely on Cognyte’s solutions to accelerate security investigations and connect the dots to successfully identify, neutralize, and prevent threats to national security, business continuity and cyber security. Luminar is an asset-based cybersecurity intelligence platform that empowers enterprise organizations to build and maintain a proactive threat intelligence operation that enables to anticipate and mitigate cyber threats, reduce risk, and enhance security resilience. Luminar enables security teams to define a customized, dynamic monitoring plan to uncover malicious activity in its earliest stages on all layers of the Web. “LUMINAR Threat Intelligence Feed” App allows integration of intelligence-based IOC data , customer-related leaked records and Threat Intelligence Data identified by Luminar

## Prerequisites

Before starting, ensure you have the following:
- A system running a supported Linux distribution (e.g., Ubuntu 20.04 or later) or Windows/macOS with Python installed.
- Python 3.10 or later installed. Verify by running `python3 --version` in your terminal.
- Access to a MISP instance with an API key (available in the MISP web interface under Administration > List Auth Keys).
- The Luminar connector Python script (e.g., `luminar.py`).
- A `requirements.txt` file listing the script’s dependencies.
- A `config.yaml` file template for configuration.
- Administrative or sudo privileges for installing system packages (if required).

## Step 1: Set Up a Python Virtual Environment

A virtual environment isolates the script’s dependencies from other Python projects on your system. Follow these steps to create and activate a virtual environment.

1. **Open a Terminal**
   - On Linux/macOS, open your terminal application.
   - On Windows, use Command Prompt, PowerShell, or Windows Terminal.

2. **Navigate to the Project Directory**
   - The directory includes `lumianr.py`, `requirements.txt`, `confuguration_utils.py` and `config.yaml` files in a project directory (e.g., `misp-luminar`).
   - Navigate to this directory using the `cd` command:
     ```bash
     cd /path/to/misp-luminar
     ```

3. **Create a Virtual Environment**
   - Run the following command to create a virtual environment named `venv`:
     ```bash
     python3 -m venv venv
     ```
   - This creates a `venv` directory in your project folder.

4. **Activate the Virtual Environment**
   - Activate the environment using the appropriate command for your operating system:
     - **Linux/macOS**:
       ```bash
       source venv/bin/activate
       ```
     - **Windows**:
       ```bash
       venv\Scripts\activate
       ```
   - After activation, your terminal prompt should change to indicate the virtual environment (e.g., `(venv)`).

5. **Verify the Environment**
   - Ensure the correct Python version is used by running:
     ```bash
     python --version
     ```
   - Confirm that `pip` points to the virtual environment:
     ```bash
     pip --version
     ```
   - The output should reference the `venv` directory.

## Step 2: Install Dependencies from requirements.txt

The `requirements.txt` file lists the Python libraries required by the connector script (e.g., `pymisp` for MISP API interactions).

1. **Ensure the Virtual Environment is Active**
   - If you don’t see `(venv)` in your terminal prompt, reactivate the environment as described in Step 1.

2. **Install Dependencies**
   - Run the following command to install all dependencies listed in `requirements.txt`:
     ```bash
     pip install -r requirements.txt
     ```
   - This downloads and installs the required packages into the virtual environment.

3. **Verify Installation**
   - Check that the dependencies are installed by running:
     ```bash
     pip list
     ```
   - Look for key packages like `pymisp`. If any are missing, ensure `requirements.txt` is correct and rerun the install command.

4. **Handle System Dependencies (if applicable)**
   - Some Python packages (e.g., `pydeep`) may require system libraries. For example, on Ubuntu, you might need:
     ```bash
     sudo apt-get install libfuzzy-dev
     ```
   - Check the package documentation or error messages during installation for specific requirements.
   - On Windows or macOS, use the appropriate package manager (e.g., Homebrew on macOS).

## Step 3: Configure Inputs in config.yaml

The `config.yaml` file specifies the MISP instance details and other parameters for the connector script. You’ll need to edit this file with your MISP credentials and settings.

1. **Locate the config.yaml File**
   - Ensure `config.yaml` is in your project directory. If it’s a template (e.g., `config.yaml.sample`), make a copy:
     ```bash
     cp config.yaml.sample config.yaml
     ```

2. **Edit config.yaml**
   - Open `config.yaml` in a text editor (e.g., `nano`, `vim`, or VS Code). For example:
     ```bash
     nano config.yaml
     ```
   - A typical `config.yaml` might look like this:
     ```yaml
     Misp_Credentials:
       api_key: 'your-misp-apikey'
       url: 'your-misp-instance'
       verify_ssl: false # recommended to set true for production
       ssl: '/path/to/misp-cert.pem'  # Path to CA bundle or server certificate
     Cognyte_Credentials:
       base_url: "https://www.cyberluminar.com"
       account_id: ''
       client_id: ''
       client_secret: ''
       initial_fetch_date: YYYY-MM-DD
     ```
   - Update the following fields:
     - `api_key`: Your MISP API key, found in the MISP web interface under Administration.
     - `url`: The URL of your MISP instance (e.g., `https://misp.example.com`).
     - `verify_ssl`: Set to `true` for secure connections or `false` for self-signed certificates (not recommended for production).
     - `ssl`: Path to CA bundle or server certificate
     - `account_id`: Luminar Account ID.
     - `client_id`: Luminar Client ID.
     - `client_secret`: Lumianr Client Secret.
     - `initial_fetch_date`: Initial fetch date in format: YYYY-MM-DD

3. **Save and Secure the File**
   - Save your changes and exit the editor.
   - Restrict file permissions to prevent unauthorized access, especially since it contains the API key:
     ```bash
     chmod 600 config.yaml
     ```
     

## Step 4: Run the Script Manually

Before scheduling, test the script to ensure it works correctly.

1. **Ensure the Virtual Environment is Active**
   - Activate the virtual environment if not already active:
     ```bash
     source venv/bin/activate  # Linux/macOS
     venv\Scripts\activate     # Windows
     ```

2. **Execute the Script**
   - Run the script using Python:
     ```bash
     python luminar.py
     ```

3. **Verify Output**
   - Check the terminal for logs or errors.
   - The script produces log files (e.g., `luminar.log` in `misp-luminar` directory), verify its contents.
   - If successful, it interacts with MISP, log in to the MISP web interface to confirm the expected actions (e.g., new events or attributes).

     ![ioc-event](/images/ioc-event.png)
     ![leaked-records-event](/images/leakedrecords-event.png)
     ![cyberfeeds-event](/images/cyberfeeds-event.png)
     ![list-events](/images/list-events.png)

4. **Troubleshoot Errors**
   - **Connection Errors**: Verify the MISP URL and API key in `config.yaml`.
   - **Missing Dependencies**: Reinstall dependencies or check for system libraries.
   - **Permission Issues**: Ensure the script has access to `config.yaml`.

## Step 5: Schedule the Script Using a Scheduler

To run the script automatically at regular intervals, use a scheduler like `cron` on Linux/macOS or Task Scheduler on Windows.

### Option 1: Schedule with Cron (Linux/macOS)

1. **Create a Shell Script Wrapper**
   - Create a file named `run_misp_connector.sh` in your project directory:
     ```bash
     nano run_misp_connector.sh
     ```
   - Add the following content:
     ```bash
     #!/bin/bash
     source /path/to/misp-connector/venv/bin/activate
     python /path/to/misp-connector/luminar.py
     deactivate
     ```
   - Replace `/path/to/misp-connector` with the absolute path to your project directory.
   - Save and make the script executable:
     ```bash
     chmod +x run_misp_connector.sh
     ```

2. **Test the Shell Script**
   - Run the shell script to ensure it works:
     ```bash
     ./run_misp_connector.sh
     ```
   - Verify the same output as in Step 4.

3. **Open the Crontab Editor**
   - Edit the cron jobs for the current user:
     ```bash
     crontab -e
     ```

4. **Add a Cron Job**
   - Add a line to run the script at your desired interval. For example, to run once every 24 hours:
     ```bash
     0 0 * * * /path/to/misp-connector/run_misp_connector.sh >> /path/to/misp-connector/misp_connector.log 2>&1
     ```
   - This runs the script daily at 00:00 (midnight). and logs output to `misp_connector.log`.
   - Adjust the schedule as needed (e.g., `0 */6 * * *` for every 6 hours).
   - Save and exit the editor.

5. **Verify the Cron Job**
   - Wait for the scheduled time and check `misp_connector.log` for output:
     ```bash
     cat misp_connector.log
     ```
   - Ensure the script ran successfully and produced the expected results in MISP or the output directory.

### Option 2: Schedule with Task Scheduler (Windows)

1. **Create a Batch File Wrapper**
   - Create a file named `run_misp_connector.bat` in your project directory:
     ```bash
     notepad run_misp_connector.bat
     ```
   - Add the following content:
     ```bat
     @echo off
     call C:\path\to\misp-connector\venv\Scripts\activate.bat
     python C:\path\to\misp-connector\luminar.py
     call deactivate
     ```
   - Replace `C:\path\to\misp-connector` with the absolute path to your project directory.
   - Save the file.

2. **Test the Batch File**
   - Double-click `run_misp_connector.bat` or run it from Command Prompt:
     ```bash
     run_misp_connector.bat
     ```
   - Verify the same output as in Step 4.

3. **Open Task Scheduler**
   - Press `Win + R`, type `taskschd.msc`, and press Enter.

4. **Create a New Task**
   - Click “Create Task” in the Actions pane.
   - General tab:
     - Name: `MISP Connector`
     - Check “Run whether user is logged on or not.”
   - Triggers tab:
     - Click “New,” select “On a schedule.”
     - Set to “Daily” and configure the time (e.g., once every 24 hours).
   - Actions tab:
     - Click “New,” select “Start a program.”
     - Program/script: Enter the full path to `run_misp_connector.bat` (e.g., `C:\path\to\misp-connector\run_misp_connector.bat`).
   - Conditions/Settings tab:
     - Adjust as needed (e.g., restart if the task fails).
   - Click “OK” and enter your user credentials when prompted.

5. **Test the Scheduled Task**
   - Right-click the task and select “Run” to test it.
   - Check the script’s output (e.g., in `luminar.log` or MISP).
   - Wait for the scheduled time to ensure it runs automatically.

## Troubleshooting

- **Virtual Environment Issues**:
  - Ensure Python 3.10+ is installed and `venv` is created correctly.
  - Recreate the environment if corrupted: `rm -rf venv && python3 -m venv venv`.
- **Dependency Errors**:
  - Update `pip`: `pip install -U pip`.
  - Check for missing system libraries and install them.
- **Configuration Errors**:
  - Verify the MISP URL and API key.
  - Set `verify_ssl: false` temporarily for testing (not for production).
- **Scheduling Issues**:
  - Check cron logs: `grep CRON /var/log/syslog` (Linux).
  - Ensure the shell/batch script uses absolute paths.
  - Verify file permissions for the script and `config.yaml`.
- **Script Errors**:
  - Run with `python luminar.py --log-level DEBUG` for detailed logs.
  - Consult the script’s documentation or contact the developer.

## Additional Resources

- **Cron Syntax**: Use [crontab.guru](https://crontab.guru/) to create cron schedules.
- **Python Virtual Environments**: [https://docs.python.org/3/tutorial/venv.html](https://docs.python.org/3/tutorial/venv.html)

## Notes

- Always secure `config.yaml` and API keys to prevent unauthorized access.
- Regularly update dependencies: `pip install -r requirements.txt --upgrade`.
- Test the script manually after any configuration changes.
- For production, consider logging rotation and monitoring to manage `luminar.log`.

If you encounter issues, consult the MISP community or Cognyte Luminar for support.

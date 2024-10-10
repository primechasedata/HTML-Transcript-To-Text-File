# Transcript Extraction and Formatting Script

This guide will help you set up and use a Python script that extracts transcripts from HTML files, formats them using OpenAI’s GPT-4 model, and saves the results as text files. This guide is designed for users with little to no coding experience.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Setup Guide](#setup-guide)
  1. [Install Python](#install-python)
  2. [Obtain an OpenAI API Key](#obtain-an-openai-api-key)
  3. [Set Up the Project Directory](#set-up-the-project-directory)
  4. [Create a Virtual Environment](#create-a-virtual-environment)
  5. [Activate the Virtual Environment](#activate-the-virtual-environment)
  6. [Install Required Python Packages](#install-required-python-packages)
  7. [Securely Store Your OpenAI API Key](#securely-store-your-openai-api-key)
- [Using the Script](#using-the-script)
  1. [Place Your HTML Files](#place-your-html-files)
  2. [Run the Script](#run-the-script)
  3. [View the Output](#view-the-output)
- [Troubleshooting](#troubleshooting)
- [Additional Notes](#additional-notes)

## Overview

This script automates the process of:

1. Scanning a directory for HTML files containing transcripts.
2. Extracting the transcript text from each HTML file.
3. Formatting the transcript using OpenAI’s GPT-4 model.
4. Saving the formatted transcript as a text file.
5. Renaming the processed HTML files to prevent reprocessing.

## Prerequisites

Before you begin, ensure you have the following:

- Python 3.7 or later installed on your computer.
- An OpenAI API key. You can obtain one by creating an account on the OpenAI website.
- Basic familiarity with using the command line (Terminal on macOS/Linux or Command Prompt on Windows).

## Setup Guide

1. **Install Python**

   If you don’t have Python installed:

   - **Windows:** Download and install Python from the official website.
   - **macOS/Linux:** Python 3 is usually pre-installed. Verify by running the following in Terminal:

     ```bash
     python3 --version
     ```

2. **Obtain an OpenAI API Key**

   1. Sign up or log in to your account at OpenAI.
   2. Navigate to the API Keys section.
   3. Create a new secret key.
   4. **Important:** Keep this key secure and do not share it publicly.

3. **Set Up the Project Directory**

   1. Choose a location on your computer for the project.
   2. Create a new folder (e.g., `TranscriptProject`).
   3. Inside this folder, create two subfolders:
      - `html` – to store your HTML files.
      - `txt` – where the processed transcripts will be saved.

   Your folder structure should look like this:

   ```
   TranscriptProject/
   ├── html/
   ├── txt/
   ```

4. **Create a Virtual Environment**

   A virtual environment isolates the project’s dependencies.

   1. Open your command line interface (CLI).
   2. Navigate to your project directory:

      ```bash
      cd /path/to/TranscriptProject
      ```

   3. Create a virtual environment named `venv`:

      ```bash
      python3 -m venv venv
      ```

      - **Windows:** Use `python` instead of `python3` if necessary.

5. **Activate the Virtual Environment**

   - **macOS/Linux:**

     ```bash
     source venv/bin/activate
     ```

   - **Windows (Command Prompt):**

     ```bash
     venv\Scripts\activate
     ```

   - **Windows (PowerShell):**

     ```bash
     venv\Scripts\Activate.ps1
     ```

   After activation, your command prompt should show `(venv)` at the beginning.

6. **Install Required Python Packages**

   With the virtual environment activated, install the necessary packages:

   ```bash
   pip install openai beautifulsoup4 lxml tiktoken nltk
   ```

7. **Securely Store Your OpenAI API Key**

   It’s essential to keep your API key secure.

   1. Set the API Key as an Environment Variable:
      - **macOS/Linux:**

        ```bash
        export OPENAI_API_KEY='your-api-key-here'
        ```

      - **Windows (Command Prompt):**

        ```bash
        set OPENAI_API_KEY=your-api-key-here
        ```

      - **Windows (PowerShell):**

        ```bash
        $env:OPENAI_API_KEY="your-api-key-here"
        ```

      Replace `'your-api-key-here'` with your actual API key.

   2. Ensure the Environment Variable Is Set:
      - **macOS/Linux:**

        ```bash
        echo $OPENAI_API_KEY
        ```

      - **Windows (Command Prompt or PowerShell):**

        ```bash
        echo %OPENAI_API_KEY%
        ```

      The command should display your API key (or part of it).

## Using the Script

1. **Place Your HTML Files**

   Copy all your HTML files containing transcripts into the `html` folder inside your project directory.

2. **Run the Script**

   Create a new file named `extract_transcript.py` in your project directory with the provided script (which should be already present in this repository).

   Run the script:

   ```bash
   python3 extract_transcript.py
   ```

3. **View the Output**

   - **Processed Transcripts:** Check the `txt` folder for the formatted transcript files.
   - **Renamed HTML Files:** The original HTML files are renamed with a `processed_` prefix to prevent reprocessing.

## Troubleshooting

- **OpenAI API Key Error:**
  - If you see an error about the API key, ensure you’ve set the `OPENAI_API_KEY` environment variable correctly.
- **Missing Packages:**
  - If you encounter an error about a missing package, make sure you’ve installed all required packages in the virtual environment.
- **SSL Warnings on macOS:**
  - You might see warnings related to SSL. These can often be ignored, but if they cause issues, consider downgrading the `urllib3` package:

    ```bash
    pip uninstall urllib3
    pip install 'urllib3<2'
    ```

- **Script Does Not Find Transcripts:**
  - Ensure that your HTML files contain transcripts and that the script’s extraction logic matches the HTML structure of your files.

## Additional Notes

- **Keep Your API Key Secure:**
  - Never share your OpenAI API key publicly.
- **Virtual Environment Activation:**
  - Remember to activate your virtual environment each time you start a new terminal session before running the script.
- **Extending the Script:**
  - The script is designed to be extensible. If your HTML files have a different structure, you may need to adjust the `extract_transcript_text` function.
- **Processing Large Transcripts:**
  - For very large transcripts, the script may need adjustments to handle token limits. This can involve splitting the transcript into smaller chunks.
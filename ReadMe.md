Transcript Extraction and Formatting Script

This guide will help you set up and use a Python script that extracts transcripts from HTML files, formats them using OpenAI’s GPT-4o model, and saves the results as text files. This guide is designed for users with little to no coding experience.

Table of Contents

	•	Overview
	•	Prerequisites
	•	Setup Guide
	•	1. Install Python
	•	2. Obtain an OpenAI API Key
	•	3. Set Up the Project Directory
	•	4. Create a Virtual Environment
	•	5. Activate the Virtual Environment
	•	6. Install Required Python Packages
	•	7. Securely Store Your OpenAI API Key
	•	Using the Script
	•	1. Place Your HTML Files
	•	2. Run the Script
	•	3. View the Output
	•	Troubleshooting
	•	Additional Notes

Overview

This script automates the process of:

	1.	Scanning a directory for HTML files containing transcripts.
	2.	Extracting the transcript text from each HTML file.
	3.	Formatting the transcript using OpenAI’s GPT-4o model.
	4.	Saving the formatted transcript as a text file.
	5.	Renaming the processed HTML files to prevent reprocessing.

Prerequisites

Before you begin, ensure you have the following:

	•	Python 3.7 or later installed on your computer.
	•	An OpenAI API key. You can obtain one by creating an account on the OpenAI website.
	•	Basic familiarity with using the command line (Terminal on macOS/Linux or Command Prompt on Windows).

Setup Guide

1. Install Python

If you don’t have Python installed:

	•	Windows: Download and install Python from the official website.
	•	macOS/Linux: Python 3 is usually pre-installed. Verify by running python3 --version in Terminal.

2. Obtain an OpenAI API Key

	1.	Sign up or log in to your account at OpenAI.
	2.	Navigate to the API Keys section.
	3.	Create a new secret key.
	4.	Important: Keep this key secure and do not share it publicly.

3. Set Up the Project Directory

Choose a location on your computer for the project.

	1.	Create a new folder (e.g., TranscriptProject).
	2.	Inside this folder, create two subfolders:
	•	html – to store your HTML files.
	•	txt – where the processed transcripts will be saved.

Your folder structure should look like:

TranscriptProject/
├── html/
├── txt/

4. Create a Virtual Environment

A virtual environment isolates the project’s dependencies.

	1.	Open your command line interface (CLI).
	2.	Navigate to your project directory:

cd /path/to/TranscriptProject


	3.	Create a virtual environment named venv:

python3 -m venv venv

	•	Windows: Use python instead of python3 if necessary.

5. Activate the Virtual Environment

	•	macOS/Linux:

source venv/bin/activate


	•	Windows (Command Prompt):

venv\Scripts\activate


	•	Windows (PowerShell):

venv\Scripts\Activate.ps1



After activation, your command prompt should show (venv) at the beginning.

6. Install Required Python Packages

With the virtual environment activated, install the necessary packages:

pip install openai beautifulsoup4 lxml tiktoken nltk

7. Securely Store Your OpenAI API Key

It’s essential to keep your API key secure.

	1.	Set the API Key as an Environment Variable
	•	macOS/Linux:

export OPENAI_API_KEY='your-api-key-here'


	•	Windows (Command Prompt):

set OPENAI_API_KEY=your-api-key-here


	•	Windows (PowerShell):

$env:OPENAI_API_KEY="your-api-key-here"


Replace 'your-api-key-here' with your actual API key.

	2.	Ensure the Environment Variable Is Set
	•	macOS/Linux:

echo $OPENAI_API_KEY


	•	Windows (Command Prompt or PowerShell):

echo %OPENAI_API_KEY%


The command should display your API key (or part of it).

Using the Script

1. Place Your HTML Files

Copy all your HTML files containing transcripts into the html folder inside your project directory.

2. Run the Script

Create a new file named extract_transcript.py in your project directory with the following content:

import os
import string
import re
from datetime import datetime
from bs4 import BeautifulSoup
from openai import OpenAI

# Instantiate the OpenAI client
api_key = os.getenv('OPENAI_API_KEY')
if api_key is None:
    raise ValueError("Please set the OPENAI_API_KEY environment variable.")
client = OpenAI(api_key=api_key)

# Directory containing the HTML files
html_directory = os.path.join(os.getcwd(), 'html')

# Directory to save the extracted transcripts
transcript_directory = os.path.join(os.getcwd(), 'txt')

# Ensure the transcript directory exists
if not os.path.exists(transcript_directory):
    os.makedirs(transcript_directory)

# Function to sanitize file names
def sanitize_filename(name):
    valid_chars = "-_.() %s%s" % (string.ascii_letters, string.digits)
    sanitized = ''.join(c for c in name if c in valid_chars)
    sanitized = sanitized.replace(' ', '_')  # Replace spaces with underscores
    return sanitized[:100]  # Limit filename length to 100 characters

# Function to extract transcript text from the HTML soup
def extract_transcript_text(soup):
    # Try the original selector
    transcript_elements = soup.find_all('span', {'data-purpose': 'cue-text'})

    # If not found, try alternative selectors
    if not transcript_elements:
        # Example alternative: find all <div> elements with class 'transcript-text'
        transcript_elements = soup.find_all('div', {'class': 'transcript-text'})

    if not transcript_elements:
        # Try extracting from <div> with id 'transcript'
        transcript_div = soup.find('div', {'id': 'transcript'})
        if transcript_div:
            transcript_elements = transcript_div.find_all('p')

    if not transcript_elements:
        # Try extracting from a script tag or other location
        script_tags = soup.find_all('script')
        for script in script_tags:
            if 'transcript' in script.text:
                match = re.search(r'"transcript":"(.*?)"', script.text, re.DOTALL)
                if match:
                    transcript_text = match.group(1)
                    # Clean up the transcript text if necessary
                    transcript_text = transcript_text.encode('utf-8').decode('unicode_escape')
                    return transcript_text

    # If transcript elements were found, extract text
    if transcript_elements:
        transcript = [element.get_text(strip=True) for element in transcript_elements]
        transcript_text = ' '.join(transcript)
        return transcript_text
    else:
        return None

# Function to format transcript using OpenAI API
def format_transcript_with_openai(transcript_text):
    import tiktoken

    prompt = (
        "From the following transcript, generate a concise title (max 10 words) that summarizes the main topic, "
        "and then format the transcript into markdown with the title as an H2 heading (## Title), "
        "followed by the transcript written coherently. Do not change details of the transcript, just format."
    )

    full_prompt = f"{prompt}\n\nTranscript:\n{transcript_text}"

    # Choose the appropriate encoding based on your model
    try:
        encoding = tiktoken.encoding_for_model('gpt-4o')
    except KeyError:
        # If 'gpt-4o' is not recognized, default to a compatible encoding
        encoding = tiktoken.get_encoding('cl100k_base')

    # Calculate the number of tokens in the full prompt
    prompt_tokens = len(encoding.encode(full_prompt))

    # Add 300 tokens for the response
    additional_tokens = 300

    # Ensure the total tokens do not exceed the model's limit
    max_model_tokens = 8192  # Adjust according to your model's limit

    # Calculate max_tokens for the API call
    available_tokens = max_model_tokens - prompt_tokens
    max_tokens = min(available_tokens, additional_tokens)

    # If max_tokens is less than or equal to zero, the prompt is too long
    if max_tokens <= 0:
        print("Error: The prompt is too long for the model to process.")
        return None

    try:
        # Use the client to create a chat completion
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {
                    "role": "system",
                    "content": "You are a helpful assistant that formats transcripts into markdown and generates concise titles."
                },
                {
                    "role": "user",
                    "content": full_prompt
                }
            ],
            max_tokens=max_tokens,
            temperature=0.5
        )

        formatted_transcript = response.choices[0].message.content.strip()
        return formatted_transcript
    except Exception as e:
        print(f"An error occurred during the OpenAI API call: {e}")
        return None

# Scan the directory for HTML files, excluding those starting with 'processed_'
html_files = [
    f for f in os.listdir(html_directory)
    if f.endswith('.html') and not f.startswith('processed_')
]

# Process each HTML file
for html_file in html_files:
    print(f"Processing file: {html_file}")

    # Full path to the HTML file
    file_path = os.path.join(html_directory, html_file)

    # Read the HTML content from the file
    with open(file_path, 'r', encoding='utf-8') as file:
        html_content = file.read()

    # Parse the HTML content with BeautifulSoup
    soup = BeautifulSoup(html_content, 'lxml')

    # Extract transcript text (if available)
    transcript_text = extract_transcript_text(soup)

    if transcript_text:
        # Format the transcript using OpenAI API
        formatted_transcript = format_transcript_with_openai(transcript_text)

        if formatted_transcript:
            # Extract the title from the formatted transcript
            lines = formatted_transcript.split('\n')
            title_line = lines[0].strip()

            if title_line.startswith('##'):
                title = title_line.lstrip('#').strip()
            else:
                title = os.path.splitext(html_file)[0]

            # Sanitize the title to create a safe filename
            sanitized_title = sanitize_filename(title)

            # Get current timestamp
            current_time = datetime.now()
            timestamp = current_time.strftime('%Y-%m-%d-%H-%M-%S_')

            # Define new filenames using 'processed_' prefix before the timestamp and sanitized title
            txt_file_new_name = f"{timestamp}{sanitized_title}.txt"
            html_file_new_name = f"processed_{timestamp}{sanitized_title}.html"

            # Paths for the new files
            transcript_file_path = os.path.join(transcript_directory, txt_file_new_name)
            processed_html_file_path = os.path.join(html_directory, html_file_new_name)

            # Save the formatted transcript to a .txt file in the transcript directory
            with open(transcript_file_path, 'w', encoding='utf-8') as transcript_file:
                transcript_file.write(formatted_transcript)

            print(f"Transcript extracted and formatted. Saved to {transcript_file_path}")

            # Rename the original HTML file by adding 'processed_' prefix before the timestamp and sanitized title
            os.rename(file_path, processed_html_file_path)
            print(f"Renamed '{html_file}' to '{os.path.basename(processed_html_file_path)}'")
        else:
            print(f"Failed to format transcript for '{html_file}'")
    else:
        print(f"No transcript found in '{html_file}'")

print("Processing complete.")

Save the file and return to your command line interface.

3. View the Output

	•	Processed Transcripts:
	•	Check the txt folder for the formatted transcript files.
	•	Renamed HTML Files:
	•	The original HTML files are renamed with a processed_ prefix to prevent reprocessing.

Troubleshooting

	•	OpenAI API Key Error:
	•	If you see an error about the API key, ensure you’ve set the OPENAI_API_KEY environment variable correctly.
	•	Missing Packages:
	•	If you encounter an error about a missing package, make sure you’ve installed all required packages in the virtual environment.
	•	SSL Warnings on macOS:
	•	You might see warnings related to SSL. These can often be ignored, but if they cause issues, consider downgrading the urllib3 package:

pip uninstall urllib3
pip install 'urllib3<2'


	•	Script Does Not Find Transcripts:
	•	Ensure that your HTML files contain transcripts and that the script’s extraction logic matches the HTML structure of your files.

Additional Notes

	•	Keep Your API Key Secure:
	•	Never share your OpenAI API key publicly.
	•	Virtual Environment Activation:
	•	Remember to activate your virtual environment each time you start a new terminal session before running the script.
	•	Extending the Script:
	•	The script is designed to be extensible. If your HTML files have a different structure, you may need to adjust the extract_transcript_text function.
	•	Processing Large Transcripts:
	•	For very large transcripts, the script may need adjustments to handle token limits. This can involve splitting the transcript into smaller chunks.

Thank you for using the Transcript Extraction and Formatting Script!

If you have any questions or need further assistance, feel free to reach out.
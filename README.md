# HTML Text Extraction Script

This script extracts text from all HTML files in a specified directory and removes repeated phrases longer than 10 words. The resulting text is saved to an output file.

## Requirements

- Python 3.x
- `beautifulsoup4` library

## Installation

1. Ensure you have Python 3.x installed on your machine.
2. Install the `beautifulsoup4` library if you haven't already:

    ```sh
    pip install beautifulsoup4
    ```

    - **Usage**
      - Save the script as `scraper.py`.
      - Update the `directory` variable in the script to point to the directory containing your HTML files:

        ```python
        # Directory containing the HTML files
        directory = r"Your Directory Here"
        ```

      - Run the script:

        ```sh
        python scraper.py
        ```

      - The script will process all HTML files in the specified directory and its subdirectories, remove repeated phrases longer than 10 words, and save the extracted text to `extracted_text.txt` in the same directory.

## How It Works

1. The script walks through the specified directory and collects all HTML files.
2. For each HTML file, it uses BeautifulSoup to parse the file and extract text from `<p>`, `<div>`, `<article>`, and `<section>` tags.
3. It removes repeated phrases longer than 10 words.
4. It combines the unique text from each file and adds it to the final output.
5. The resulting text is saved to `extracted_text.txt`.

## Script Details

```python
import os
from bs4 import BeautifulSoup
from collections import defaultdict

def extract_text(file_path):
    with open(file_path, "r", encoding="utf-8") as file:
        soup = BeautifulSoup(file, "html.parser")
        
        # Remove unnecessary sections (common classes/IDs for menus, headers, footers)
        for tag in soup(["nav", "header", "footer", "aside", "menu"]):
            tag.decompose()

        # Targeting common content tags
        content = soup.find_all(['p', 'div', 'article', 'section'])
        
        # Extract text from the filtered content
        text = "\n".join([element.get_text(separator="\n") for element in content])
        return text

def get_file_paths(directory):
    file_paths = []
    for root, dirs, files in os.walk(directory):
        for file in files:
            if file.endswith(".html") or file.endswith(".htm"):
                file_paths.append(os.path.join(root, file))
    return file_paths

def remove_repeated_phrases(text, max_words=10):
    phrases = text.split()
    unique_phrases = []
    seen_phrases = set()

    for i in range(len(phrases) - max_words + 1):
        phrase = " ".join(phrases[i:i + max_words])
        if phrase not in seen_phrases:
            seen_phrases.add(phrase)
            unique_phrases.append(phrase)

    return " ".join(unique_phrases)

def extract_all_text(directory):
    all_text = ""
    file_paths = get_file_paths(directory)

    for file_path in file_paths:
        file_text = extract_text(file_path)
        unique_text = remove_repeated_phrases(file_text)
        all_text += f"File: {file_path}\n"  # Add the file path to the extracted text
        all_text += unique_text
        all_text += "\n\n"  # Add a separator between files

    return all_text

# Directory containing the HTML files
directory = r"Your Directory Here"

all_text = extract_all_text(directory)

# Save the extracted text to a file
output_file_path = os.path.join(directory, "extracted_text.txt")
with open(output_file_path, "w", encoding="utf-8") as output_file:
    output_file.write(all_text)

print(f"Text extracted and saved to {output_file_path}")

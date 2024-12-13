---
layout: page
title: LLM Web Scraper
parent: LLMs
description: Using LLMs to scrape useful information from web pages
nav_order: 1
tags: [llm, huggingface, web-scraping, python]
---

# Using Large Language Models (LLMs) for Detecting Values from Web-Scraped HTML Using Hugging Face
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

Large Language Models (LLMs), such as those available through Hugging Face, can be powerful tools for extracting specific values from web-scraped HTML files. In this guide, we'll explore how you can leverage these models to detect values like product prices, dates, or specific text patterns from HTML data using Hugging Face transformers.

---

### 1. Business Use Case Overview

In this scenario, your business needs to extract certain values (e.g., prices, product names, or dates) from web-scraped HTML files. While traditional scraping techniques might rely on rules and regular expressions, an LLM-based approach can generalize better to a wide range of HTML structures by leveraging the model's ability to understand natural language and structure.

Using Hugging Face models, we can automate this task by using transformer-based models to parse and understand HTML content, extract relevant data, and ensure accurate results across different pages and layouts.

### 2. Setting Up the Environment

To start, you’ll need to set up your environment to handle Hugging Face models and basic HTML processing libraries.

1. **Install the required libraries:**

   ```bash
   pip install transformers beautifulsoup4 requests torch
   ```

2. **Sign up for Hugging Face API access (optional but recommended)**:

   - If you want to access large models directly or fine-tune them using the Hugging Face Hub, create an account at [Hugging Face](https://huggingface.co/) and get an API key.
   - Install the CLI to manage models:

```bash
pip install huggingface_hub
huggingface-cli login
```

### 3. Processing the HTML Data

Before feeding the HTML content to the LLM, we need to clean and extract the relevant text. We can use `BeautifulSoup` to parse the HTML and convert it into plain text.

Here’s how you can scrape HTML and preprocess it:

```python
import requests
from bs4 import BeautifulSoup

# Fetch the web page
url = 'https://example.com'
response = requests.get(url)
html_content = response.text

# Use BeautifulSoup to parse the HTML
soup = BeautifulSoup(html_content, 'html.parser')

# Extract text content (basic cleaning)
text_content = soup.get_text(separator=' ', strip=True)
print(text_content)
```

This will give you the cleaned-up text from the HTML file, which is more suitable for processing by a language model.

### 4. Using Pre-Trained Models from Hugging Face

Hugging Face offers a variety of models that can help with text extraction tasks, such as Named Entity Recognition (NER) or custom token classification models. Depending on the type of values you want to extract (e.g., dates, prices), you can choose a pre-trained model from the Hugging Face Hub.

For example, you might choose a pre-trained NER model to extract entities like dates or monetary values:

```python
from transformers import pipeline

# Load a pre-trained NER model
ner_pipeline = pipeline('ner', model='dbmdz/bert-large-cased-finetuned-conll03-english')

# Use the model to extract named entities from the text content
ner_results = ner_pipeline(text_content)

# Display the extracted values
for entity in ner_results:
	print(f"{entity['word']} -> {entity['entity']}, Confidence: {entity['score']:.2f}")
```

This will give you named entities like "DATE", "MONEY", "PERSON", etc., based on the pre-trained model.

### 5. Customizing the Model for Your Use Case

If the pre-trained models don’t extract the specific values you’re interested in (e.g., custom fields or product information), you can fine-tune a model using your own labeled data.

#### Steps to Fine-Tune a Model:

1. **Prepare Your Data**: 
   Label the HTML data with the values you want to detect. For example, if you want to extract product prices, annotate the HTML files to highlight the prices.

   Example annotated format:
   ```json
   {
     "text": "This product costs $49.99 and is available on September 23.",
     "labels": [
       {"entity": "PRICE", "start": 20, "end": 26},
       {"entity": "DATE", "start": 44, "end": 55}
     ]
   }
   ```

2. **Fine-Tune with Hugging Face**: 
   Fine-tuning requires a dataset and a Hugging Face transformer model. You can use the `Trainer` API to handle the training process.

   Here’s a simplified outline of how to fine-tune:

   ```python
   from transformers import AutoModelForTokenClassification, Trainer, TrainingArguments
   from datasets import load_dataset

   # Load a pre-trained model and fine-tune it for token classification
   model = AutoModelForTokenClassification.from_pretrained("bert-base-cased", num_labels=2)

   # Load your dataset
   dataset = load_dataset('json', data_files='labeled_data.json')

   # Define training arguments
   training_args = TrainingArguments(
       output_dir='./results', 
       evaluation_strategy="epoch",
       learning_rate=2e-5,
       per_device_train_batch_size=16,
       num_train_epochs=3
   )

   # Set up the Trainer
   trainer = Trainer(
       model=model,
       args=training_args,
       train_dataset=dataset['train'],
       eval_dataset=dataset['validation']
   )

   # Train the model
   trainer.train()
   ```

   After fine-tuning, you can use the trained model to detect specific values in any web-scraped HTML content.

### 6. Example Code

Here’s a full example that combines HTML scraping and using a pre-trained model for value extraction:

```python
import requests
from bs4 import BeautifulSoup
from transformers import pipeline

# Fetch web page and scrape HTML
url = 'https://example.com'
response = requests.get(url)
html_content = response.text
soup = BeautifulSoup(html_content, 'html.parser')
text_content = soup.get_text(separator=' ', strip=True)

# Load a pre-trained NER model from Hugging Face
ner_pipeline = pipeline('ner', model='dbmdz/bert-large-cased-finetuned-conll03-english')

# Extract values using the NER model
ner_results = ner_pipeline(text_content)

# Print the results
for entity in ner_results:
    print(f"{entity['word']} -> {entity['entity']}, Confidence: {entity['score']:.2f}")
```

This example extracts named entities from the web-scraped HTML and outputs them with confidence scores.

### 7. Conclusion

LLMs offer a powerful and flexible approach to extracting values from web-scraped HTML. By leveraging Hugging Face's pre-trained models or fine-tuning your own, you can handle diverse and unstructured data more effectively than with traditional scraping methods. This approach is particularly beneficial for extracting nuanced or variable information like dates, prices, or product details from inconsistent HTML structures.

Hugging Face provides an easy-to-use interface for deploying state-of-the-art models, allowing businesses to quickly scale their information extraction workflows. Whether using off-the-shelf NER models or fine-tuning a custom model, you can integrate advanced AI capabilities into your web scraping pipeline.
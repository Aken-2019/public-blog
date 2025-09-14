+++ 
title = "Building a BPE Tokenizer with TDD - Part 1: Project Setup and First Test"
date = "2025-09-13 6:00:00 +0000"
tags = ["python", "nlp", "bpe", "tdd", "tokenization"]
draft = false
showtoc = true
summary = "First part of a series on building a Byte Pair Encoding tokenizer using Test-Driven Development. We set up our project structure, create a virtual environment, and write our first test for the tokenizer's initialization."
+++

Welcome to the first part of our series on building a Byte Pair Encoding (BPE) tokenizer using Test-Driven Development (TDD). In this tutorial, we'll set up our project structure, create a virtual environment, and write our first test for the tokenizer's initialization.

> **Source Code**: The complete source code for this tutorial series is available on my GitHub repo at [daily-dev-notes/2025/bpe-tokenizer-tdd](https://github.com/Aken-2019/daily-dev-notes/tree/main/2025/bpe-tokenizer-tdd). Feel free to clone the repository and follow along with the implementation.

## Prerequisites

- Tested with Python 3.12 (should work with Python 3.7+)
- `pytest` for testing
- Basic knowledge of Python and unit testing

## Project Structure

Let's start by creating the following directory structure for our project:

```
bpe-tokenizer/
├── src/
│   └── __init__.py
│   └── BPETokenizer.py      # Our main tokenizer implementation
├── tests/
│   ├── __init__.py
│   └── test_bpe_tokenizer.py # Our test file
├── requirements.txt
└── README.md
```

## Setting Up the Development Environment

1. First, create and activate a virtual environment:

```bash
# Create a virtual environment
python -m venv venv

# Activate the virtual environment
# On Windows:
# .\venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

2. Install the required packages:

```bash
pip install -r requirements.txt
```

## Writing Our First Test

Let's start by writing a test for the basic initialization of our BPE tokenizer. We'll use TDD, so we'll write the test first and then implement the functionality.

Create a new file `tests/test_tokenizer.py` with the following content:

```python
import pytest
from src.tokenizer import BPETokenizer


def test_tokenizer_initialization():
    """Test that the tokenizer initializes with empty vocabularies and merges."""
    # Arrange
    tokenizer = BPETokenizer()
    
    # Assert
    assert isinstance(tokenizer.vocab, dict), "Vocab should be a dictionary"
    assert isinstance(tokenizer.inverse_vocab, dict), "Inverse vocab should be a dictionary"
    assert isinstance(tokenizer.bpe_merges, dict), "BPE merges should be a dictionary"
    assert len(tokenizer.vocab) == 0, "Initial vocab should be empty"
    assert len(tokenizer.inverse_vocab) == 0, "Initial inverse vocab should be empty"
    assert len(tokenizer.bpe_merges) == 0, "Initial BPE merges should be empty"
```

## Implementing the Basic Tokenizer Class

Now, let's create the initial implementation of our tokenizer in `src/tokenizer.py`:

```python
class BPETokenizer:
    """A simple implementation of Byte Pair Encoding (BPE) tokenizer."""
    
    def __init__(self):
        """Initialize the BPE Tokenizer with empty vocabularies and merges."""
        # Maps token_id to token_str (e.g., {11246: "some"})
        self.vocab = {}
        # Maps token_str to token_id (e.g., {"some": 11246})
        self.inverse_vocab = {}
        # Dictionary of BPE merges: {(token1, token2): merged_token_id}
        self.bpe_merges = {}
```

## Running the Tests

Let's run our test to make sure everything is working:

```bash
pytest tests/
```

You should see output indicating that the test passed. If you encounter any issues, make sure:

1. Your virtual environment is activated
2. You've installed pytest
3. The file structure matches exactly what's shown above

## What We've Accomplished

In this first part of the series, we've:

1. Set up our project structure
2. Created a virtual environment
3. Installed necessary dependencies
4. Written our first test for tokenizer initialization
5. Implemented the basic tokenizer class structure

## Next Steps

In the next part of this series, we'll:

1. Add functionality to train the tokenizer on text data
2. Implement the BPE algorithm
3. Add methods to encode and decode text

In [Part 2]({{< ref "20250913-building-bpe-tokenizer-part2-train-method.md" >}}) we'll dive deeper into implementing the BPE algorithm!

## Resources

- [BPE Tokenizer from Scratch by Sebastian Raschka](https://sebastianraschka.com/blog/2025/bpe-from-scratch.html) - The reference implementation this tutorial is based on
- [Let's build the GPT Tokenizer](https://www.youtube.com/watch?v=zduSFxRajkE)


Happy coding!

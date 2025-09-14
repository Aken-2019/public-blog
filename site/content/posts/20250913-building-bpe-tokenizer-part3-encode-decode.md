+++ 
title = "Building a BPE Tokenizer with TDD - Part 3: Implementing Encode and Decode Methods"
date = "2025-09-13 8:00:00 +0000"
tags = ["python", "nlp", "bpe", "tdd", "tokenization"]
draft = false
showtoc = true
summary = "Final part of our BPE tokenizer series, where we implement encoding and decoding capabilities. We'll write comprehensive tests for token conversion, handle special tokens, and ensure proper error handling for edge cases."
+++

Welcome to Part 3 of our series on building a Byte Pair Encoding (BPE) tokenizer using Test-Driven Development (TDD). In [Part 2]({{< ref "20250913-building-bpe-tokenizer-part2-train-method.md" >}}), we implemented the training logic for our tokenizer. Now, we'll add encoding and decoding capabilities to make our tokenizer fully functional.

> **Source Code**: The complete source code for this tutorial series is available on my GitHub repo at [daily-dev-notes/2025/bpe-tokenizer-tdd](https://github.com/Aken-2019/daily-dev-notes/tree/main/2025/bpe-tokenizer-tdd). Feel free to clone the repository and follow along with the implementation.

> **Note to Readers**: The complete solution to all tests can be found in `src/solution_BPETokenizer.py`. We encourage you to try implementing the solutions yourself first, then compare with the reference implementation.

## What Do the Encode and Decode Methods Do?

The encode and decode methods are essential for using our trained tokenizer:

1. `encode`: Converts text into a sequence of token IDs using the learned BPE merges
2. `decode`: Converts a sequence of token IDs back into text

Here's an example of how they work:

```python
tokenizer = BPETokenizer()
tokenizer.train("hello world", vocab_size=10)

# Encoding without special tokens
ids = tokenizer.encode("hello")  # might return [1, 2, 3, 3, 4] 
# where each number represents a token learned during training

# Encoding with special tokens
special_tokens = {"<|endoftext|>"}
ids = tokenizer.encode("hello <|endoftext|>", allowed_specials=special_tokens)
# Returns token IDs for text and special token

# Decoding
text = tokenizer.decode([1, 2, 3, 3, 4])  # returns "hello"
```

## The TDD Workflow

As before, we'll follow the red-green-refactor cycle:
1. **Red**: Write a failing test
2. **Green**: Write minimal code to make the test pass
3. **Refactor**: Clean up while keeping tests green

## Step 1: Testing Basic Encoding

Let's start with the simplest encoding case - encoding a string that only uses tokens we've already learned:

```python
def test_encode_basic(self):
    """Test encoding a simple string with learned tokens."""
    tokenizer = BPETokenizer()
    text = "hello"
    tokenizer.train(text, vocab_size=5)  # Only character tokens
    
    # Encoding the same text we trained on
    token_ids = tokenizer.encode(text)
    
    # Check that each character maps to its assigned token ID
    expected_ids = [tokenizer.inverse_vocab[c] for c in text]
    assert token_ids == expected_ids

def test_encode_with_merges(self):
    """Test encoding when BPE merges are involved."""
    tokenizer = BPETokenizer()
    text = "ababab"
    tokenizer.train(text, vocab_size=3)  # Will learn 'ab' as a merged token
    
    # The string "ababab" should now encode to fewer tokens
    # since 'ab' will be treated as a single token
    token_ids = tokenizer.encode(text)
    assert len(token_ids) < len(text), "Encoding should be shorter than original text"
```

## Step 2: Placeholder Implementation (Red)

Let's add the placeholder function to our `BPETokenizer` class:

```python
def encode(self, text: str) -> list[int]:
    """
    Encode text into a sequence of token IDs using the trained BPE merges.
    
    Args:
        text (str): The text to encode
        
    Returns:
        list[int]: A sequence of token IDs
        
    Raises:
        ValueError: If the tokenizer hasn't been trained
    """
    pass

def decode(self, token_ids: list[int]) -> str:
    """
    Decode a sequence of token IDs back into text.
    
    Args:
        token_ids (list[int]): The sequence of token IDs to decode
        
    Returns:
        str: The decoded text
        
    Raises:
        ValueError: If any token ID is not in the vocabulary
    """
    pass
```

Running the tests at this point should fail since we haven't implemented the methods yet.

## Step 3: Testing Edge Cases and Error Handling

Before implementing the core functionality, let's add tests for important edge cases:

```python
def test_encode_empty_string(self):
    """Test encoding an empty string."""
    tokenizer = BPETokenizer()
    tokenizer.train("hello", vocab_size=5)
    assert tokenizer.encode("") == [], "Empty string should encode to empty list"

def test_encode_unknown_chars(self):
    """Test encoding text with characters not seen during training."""
    tokenizer = BPETokenizer()
    tokenizer.train("hello", vocab_size=5)
    
    with pytest.raises(ValueError):
        tokenizer.encode("world")  # 'w', 'r', 'd' not in training data

def test_decode_empty_sequence(self):
    """Test decoding an empty sequence."""
    tokenizer = BPETokenizer()
    tokenizer.train("hello", vocab_size=5)
    assert tokenizer.decode([]) == "", "Empty sequence should decode to empty string"

def test_decode_unknown_tokens(self):
    """Test decoding with token IDs not in vocabulary."""
    tokenizer = BPETokenizer()
    tokenizer.train("hello", vocab_size=5)
    
    with pytest.raises(ValueError):
        tokenizer.decode([999])  # Token ID 999 doesn't exist
```

## Step 4: Testing Special Token Handling

We should also test how our encoder and decoder handle special tokens:

```python
def test_encode_with_special_tokens(self):
    """Test encoding with special tokens in the vocabulary."""
    tokenizer = BPETokenizer()
    special_token = "<|endoftext|>"
    tokenizer.train("hello", vocab_size=10, allowed_special={special_token})
    
    # Special token should encode to a single token ID
    token_ids = tokenizer.encode(special_token)
    assert len(token_ids) == 1, "Special token should encode to single ID"
    assert token_ids[0] == tokenizer.inverse_vocab[special_token]

def test_decode_with_special_tokens(self):
    """Test decoding sequences containing special tokens."""
    tokenizer = BPETokenizer()
    special_token = "<|endoftext|>"
    tokenizer.train("hello", vocab_size=10, allowed_special={special_token})
    
    # Get the token ID for the special token
    special_id = tokenizer.inverse_vocab[special_token]
    
    # Decode a sequence with the special token
    text = tokenizer.decode([special_id])
    assert text == special_token, "Special token should decode correctly"
```

## Implementation Tips

To implement these methods efficiently, consider:

1. For `encode`:
   a. Pre-processing and Validation:
      - Check if tokenizer is trained (vocabulary exists)
      - Handle empty string case
      - Validate special tokens if provided

   b. Special Token Handling:
      - If allowed_specials is provided:
        * Check all special tokens exist in vocabulary
        * Split text into segments at special token boundaries
        * Keep track of both regular text and special token segments
      - If no special tokens, treat entire input as regular text

   c. Character Validation:
      - Extract all regular text segments
      - Verify all characters exist in vocabulary

   d. Encoding Process:
      - For special token segments:
        * Use their predefined token IDs directly
      - For regular text segments:
        * Convert characters to initial token IDs
        * Apply BPE merges using the apply_merges helper
        * Efficiently handle merges using a deque for sequential processing

2. For `decode`:
   - Simply map each token ID to its string representation
   - Join the resulting strings together
   - No need to "undo" merges since the vocabulary maps directly to the final strings

Here's a suggested helper method that might be useful:

```python
def apply_merges(self, token_ids: list[int]) -> list[int]:
    """
    Apply learned BPE merges to a sequence of token IDs.
    
    Args:
        token_ids (list[int]): Initial sequence of token IDs
        
    Returns:
        list[int]: Sequence after applying all possible merges
    """
    pass
```

## Conclusion

You now have a full suite of tests for the encoding and decoding functionality of your BPE tokenizer! Try implementing the methods yourself, starting with the simplest possible code that makes each test pass. Remember:

1. Start with the basic cases
2. Add error handling once the core functionality works
3. Finally, optimize your code if needed

When you're done, you'll have a fully functional BPE tokenizer that can:
- Train on input text to learn token merges
- Encode new text using learned merges
- Decode token sequences back to text
- Handle special tokens correctly
- Provide appropriate error messages for edge cases

+++ 
title = "Building a BPE Tokenizer with TDD - Part 2: Implementing the Train Method"
date = "2025-09-13 7:00:00 +0000"
tags = ["python", "nlp", "bpe", "tdd", "tokenization"]
draft = false
showtoc = true
summary = "Second part of our BPE tokenizer series, focusing on implementing the train method. We'll cover the core BPE algorithm, write tests for training functionality, and implement vocabulary management and pair merging logic."
+++

Welcome to Part 2 of our series on building a Byte Pair Encoding (BPE) tokenizer using Test-Driven Development (TDD). In [Part 1]({{< ref "20250913-building-bpe-tokenizer-part1-project-setup.md" >}}), we set up our project and the basic `BPETokenizer` class. In this tutorial, we'll implement the `train` method, which is the core of the BPE algorithm.

> **Source Code**: The complete source code for this tutorial series is available on my GitHub repo at [daily-dev-notes/2025/bpe-tokenizer-tdd](https://github.com/Aken-2019/daily-dev-notes/tree/main/2025/bpe-tokenizer-tdd). Feel free to clone the repository and follow along with the implementation.


> **Note to Readers**: The complete solution to all tests can be found in `src/solution_BPETokenizer.py`. We encourage you to try implementing the solutions yourself first, then compare with the reference implementation.

## What Does the `train` Method Do?

The `train` method updates three key properties of the tokenizer:

1. `vocab`: Maps token IDs to token strings
2. `inverse_vocab`: Maps token strings to token IDs
3. `bpe_merges`: Records which pairs of tokens were merged

Here are examples of what these properties look like after training:

```python
# After training on "hello" with vocab_size=10:
tokenizer.vocab == {
    0: 'e',  # Each unique character gets an ID
    1: 'h',
    2: 'l',
    3: 'o'
}

tokenizer.inverse_vocab == {
    'e': 0,  # Inverse mapping of the vocab
    'h': 1,
    'l': 2,
    'o': 3
}

tokenizer.bpe_merges == {}  # Empty because no merges were needed

# After training on "ababab" with vocab_size=5:
tokenizer.vocab == {
    0: 'a',
    1: 'b',
    2: 'ab'  # New token from merging 'a' and 'b'
}

tokenizer.inverse_vocab == {
    'a': 0,
    'b': 1,
    'ab': 2
}

tokenizer.bpe_merges == {
    (0, 1): 2  # Maps the pair ('a', 'b') to the new token ID
}
```

## The TDD Workflow

We will follow a red-green-refactor cycle:
1.  **Red**: Write a test that fails because the feature doesn't exist yet.
2.  **Green**: Write the simplest code possible to make the test pass.


## Step 1: Understanding the Main Training Flow

Let's start with the main `train` method test that verifies the core BPE training functionality. This will help us understand what helper methods we'll need.

### The Main Test (Red)

```python
def test_train_learns_merges_and_respects_vocab_size(self):
    """Test that the tokenizer learns BPE merges correctly and respects vocab_size."""
    tokenizer = BPETokenizer()
    text = "ababab"  # Most frequent pair is ('a', 'b')
    
    initial_chars = sorted(list(set(text)))
    vocab_size = len(initial_chars) + 1  # Allows for exactly one merge
    
    tokenizer.train(text, vocab_size)

    assert len(tokenizer.vocab) == vocab_size, f"Vocabulary size should be {vocab_size}"
    assert len(tokenizer.bpe_merges) == 1, "Should have recorded exactly one merge"
    
    assert "ab" in tokenizer.inverse_vocab, "Merged token 'ab' should be in the vocabulary"
    
    a_id = tokenizer.inverse_vocab['a']
    b_id = tokenizer.inverse_vocab['b']
    ab_id = tokenizer.inverse_vocab['ab']
    
    assert (a_id, b_id) in tokenizer.bpe_merges, "The pair ('a', 'b') should be in merges"
    assert tokenizer.bpe_merges[(a_id, b_id)] == ab_id, "The merge should map to the correct new token ID"
```

From this test, we can see that we need:
1. A way to initialize the vocabulary from characters
2. A method to find frequent pairs
3. A method to merge pairs and update the vocabulary

Before tackling this main test, let's first write and pass unit tests for two essential helper functions in the next Step. These helpers will make our implementation of the `train` method much cleaner and easier to reason about.

## Step 2: Breaking Down into Helper Methods

Looking at the main test, we'll need two key helper methods:
1. `find_freq_pair`: To identify which pair of tokens to merge next
2. `replace_pair`: To perform the actual merging of tokens

Let's write tests for these helper methods:

### Testing `find_freq_pair` (Red)

First, let's test the method that finds the most frequent pair in a sequence of token IDs:

```python
def test_find_freq_pair_basic(self):
    """Test finding most frequent pair in a simple sequence."""
    tokenizer = BPETokenizer()
    token_ids = [1, 2, 3, 1, 2]  # (1,2) appears twice
    most_freq = tokenizer.find_freq_pair(token_ids)
    assert most_freq == (1, 2), "Should find (1,2) as most frequent pair"

def test_find_freq_pair_empty(self):
    """Test finding pairs in empty or single-token sequences."""
    tokenizer = BPETokenizer()
    assert tokenizer.find_freq_pair([]) is None, "Empty sequence should return None"
    assert tokenizer.find_freq_pair([1]) is None, "Single token should return None"

def test_find_freq_pair_ties(self):
    """Test that when multiple pairs have same frequency, consistently picks one."""
    tokenizer = BPETokenizer()
    token_ids = [1, 2, 3, 4, 3, 4]  # (3,4) appears twice
    most_freq = tokenizer.find_freq_pair(token_ids)
    assert most_freq == (3, 4), "Should find (3,4) as most frequent pair"
```

### Testing `replace_pair` (Red)

Next, let's test the method that replaces occurrences of a pair with a new token:

```python
def test_replace_pair_basic(self):
    """Test basic pair replacement."""
    tokenizer = BPETokenizer()
    token_ids = [1, 2, 3, 1, 2]
    pair = (1, 2)
    new_id = 4
    result = tokenizer.replace_pair(token_ids, pair, new_id)
    assert result == [4, 3, 4], "Should replace both occurrences of (1,2) with 4"

def test_replace_pair_no_matches(self):
    """Test replacement when pair isn't found."""
    tokenizer = BPETokenizer()
    token_ids = [1, 3, 2, 4]
    pair = (1, 2)
    new_id = 5
    result = tokenizer.replace_pair(token_ids, pair, new_id)
    assert result == token_ids, "Should not modify sequence when pair isn't found"

def test_replace_pair_adjacent_pairs(self):
    """Test replacement of adjacent pairs."""
    tokenizer = BPETokenizer()
    token_ids = [1, 2, 1, 2]  # Two adjacent (1,2) pairs
    pair = (1, 2)
    new_id = 3
    result = tokenizer.replace_pair(token_ids, pair, new_id)
    assert result == [3, 3], "Should handle adjacent pairs correctly"
```

Run these tests and watch them fail:
```bash
pytest -v tests/test_bpe_tokenizer.py -k "test_find_freq_pair or test_replace_pair"
```

### Implementing Helper Methods (Green)

Now that we have our helper method tests, let's implement them. These implementations should be as simple as possible while passing the tests:

```python
@staticmethod
def find_freq_pair(token_ids: list[int]) -> tuple[int, int] | None:
    """
    Find the most frequent adjacent pair of token IDs in the sequence.
    
    Args:
        token_ids (list[int]): A list of token IDs to search for pairs.
    
    Returns:
        tuple[int, int] | None: The most frequent pair of adjacent token IDs,
            or None if no pairs exist.
    """
    pass

@staticmethod
def replace_pair(token_ids: list[int], pair_to_replace: tuple[int, int], new_id: int) -> list[int]:
    """
    Replace all occurrences of a token pair with a new token ID.
    
    Args:
        token_ids (list[int]): The list of token IDs to process.
        pair_to_replace (tuple[int, int]): The pair of token IDs to replace.
        new_id (int): The new token ID to use as replacement.
    
    Returns:
        list[int]: A new list with all occurrences of the pair replaced.
    """
    pass
```

Now it's your turn! Before looking at the solution, try to implement the helper functions yourself.

If you get stuck, you can peek at the reference implementation in `src/solution_BPETokenizer.py` for inspiration. But try to solve it on your own first—it's great practice and will help you understand the BPE algorithm more deeply!

### Implementing the Main Training Method

Now that we have our helper methods ready, we can implement the main training logic for our BPE tokenizer. The training process should:

1. Initialize the vocabulary (starting with individual characters)
2. Iterate and find most frequent token pairs
3. Merge pairs to create new tokens
4. Update vocabulary and training data
5. Continue until reaching target vocabulary size


```python
def train(self, text_sequences: list[str], vocab_size: int) -> None:
    """
    Train the BPE tokenizer on a list of text sequences.
    
    Args:
        text_sequences (list[str]): List of text sequences to train on
        vocab_size (int): Target vocabulary size to achieve
    
    The training process follows these steps:
    1. Initialize vocabulary with unique characters
    2. Convert text to token IDs using current vocabulary
    3. Find and merge most frequent pairs until reaching target vocab size
    4. Update the vocabulary mapping as new tokens are created
    """
    pass
```


## Step 4: Testing the Full Implementation

Now let's write some comprehensive tests to ensure our implementation works as expected:

```python
    def test_train_full_functionality(self):
        """Test the full training process with a single concatenated sequence."""
        tokenizer = BPETokenizer()
        text = "hello worldhello world"
        vocab_size = 15

        # Train the tokenizer
        tokenizer.train(text, vocab_size)

        # Verify vocabulary size
        assert len(tokenizer.vocab) <= vocab_size, "Vocab should not exceed target size"

        # Check that common character pairs were merged
        common_pairs = [('h', 'e'), ('l', 'l'), ('r', 'l')]
        merged = False
        for first, second in common_pairs:
            if first + second in tokenizer.inverse_vocab:
                merged = True
                break
        assert merged, "Should have merged at least one common pair"

        # Test handling of empty sequence
        tokenizer = BPETokenizer()
        tokenizer.train("", 10)
        assert len(tokenizer.vocab) == 0, "Empty sequence should result in empty vocab"

        # Test training with special tokens
        tokenizer = BPETokenizer()
        special_tokens = {"<|endoftext|>", "<|pad|>"}
        tokenizer.train(text, vocab_size, allowed_special=special_tokens)

        for token in special_tokens:
            assert token in tokenizer.inverse_vocab, f"Special token {token} should be in vocab"

```

Run the tests:
```bash
pytest tests/test_bpe_tokenizer.py -v
```

## Conclusion

You've now implemented a complete BPE tokenizer training process! The implementation:

1. Handles initialization from raw text input
2. Correctly identifies and merges frequent pairs
3. Maintains vocabulary mappings
4. Respects the target vocabulary size
5. Supports special tokens

In [Part 3]({{< ref "20250913-building-bpe-tokenizer-part3-encode-decode.md" >}}), we'll implement the encoding and decoding methods to make our tokenizer fully functional.

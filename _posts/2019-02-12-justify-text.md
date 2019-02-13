---
layout: post
title:  "Daily coding problem #28: Write an algorithm to justify text"
date:   2019-02-13 23:11:00 +0200
categories: python, daily coding problem
---
Problem:

> Write an algorithm to justify text. Given a sequence of words and an integer line length k,
> return a list of strings which represents each line, fully justified.
>
> More specifically, you should have as many words as possible in each line. There should be
> at least one space between each word. Pad extra spaces when necessary so that each line has
> exactly length k. Spaces should be distributed as equally as possible, with the extra spaces,
> if any, distributed starting from the left.
>
> If you can only fit one word on a line, then you should pad the right-hand side with spaces.
>
> Each word is guaranteed not to be longer than k.
>
> For example, given the list of words
> `["the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog"]` and `k = 16`,
> you should return the following:
>
> `["the  quick brown",` # 1 extra space on the left
> ` "fox  jumps  over",` # 2 extra spaces distributed evenly
> ` "the   lazy   dog"]` # 4 extra spaces distributed evenly

This looks like a problem that can be quickly solved in a few lines of codes, but it would
actually require a little more lines than expected. While the case of a word longer than the
given integer `k` is not specifically covered in the problem description, I assume than such
word has to be split into k sized chunks.

This is how I am going to call the function:

````python
if __name__ == '__main__':
	print("\n".join(justify(["the", "quick", "brown", "fox",
		"jumps", "over", "the", "lazy", "dog"], 16)))
````

To make things easier at a later time, I am going to appends all words from the array `words` into the new array `w`,
eventually splitting words longer than k into k sized chunks. This to make sure that any word in `w` would not be
longer than `k`:

````python
w = []

for word in words:
	if len(word)<=k:
		# no need to split, just appends
		w.append(word)
	else:
		# split by length
		for i in range(0, len(word), k):
			w.append(word[i:i+k])
````

Improvements needed:

1. While it makes things easier at a later time, I don't really like the approach of creating a copy of the array
2. How to make the code look more pythonic?

Once I have all words (no longer than `k`) in my array, the logic then is simple: I keep a buffer, empty at first,
where I add each word, one by one, if there's still space for it. But if the current
word is going to make the line to overflow, I'll just output the words on the current buffer in justified format,
then I'll empty the buffer and start the process again.

The code is this:

````python
def justify(words, k):
	# w will containg words, making sure that the maximum length of each word is <= k
	w = []

	for word in words:
		if len(word)<=k:
			w.append(word)
		else:
			# split by length
			for i in range(0, len(word), k):
				w.append(word[i:i+k])

	res = []

	current_length = 0
	current_words = []
	for i in range(0, len(w)):
		# try to add a new word to current_words / current_length
		if len(w[i]) + current_length + len(current_words) <= k:
			# can fit
			current_words.append(w[i])
			current_length += len(w[i])
		else:
			# won't fit
			res.append(justify_line(current_words, current_length, k))
			current_words = [w[i]]
			current_length = len(w[i])

	if current_words:
		res.append(justify_line(current_words, current_length, k))

	return res
````

Then I just need the `justify_line` function, that accept as input the array `w` of words that fits in the current line,
the length `l` of the current words (not considering spaces), and the width of the like `k`.

(I could calculate the total length `l` inside my function, but since this is already available I'll just pass to it)

````python
def justify_line(w, l, k):
	if len(w)==1:
		return w[0].ljust(k)
	else:
		narrow_spaces, wide_words = divmod(k - l, len(w)-1)
		ns = " " * narrow_spaces
		ws = " " * (narrow_spaces + 1)

		return ns.join([ws.join(w[0:wide_words+1]), *w[wide_words+1:]])
````

If we have just 1 word then I'll just return this word left-justified with spaces at the right (this is from the problem specs).

If we have more than 1 word then we have to calculate how many spaces to put between each word. The total number of spaces needed
would be `k - l` that has to be divided by the number of words minus one (as spaces has to be put between each word).

The minimum number of spaces is the integer division `k-l // len(w)-1` but there might be some words that need one extra space.

For example if I call `justify_line(['the', 'quick', 'brown'], 13, 16)`, `narrow_spaces` will be set to 1 but `wide_words` would be
set to 1 as well meaning that the first word needs one more extra space than `narrow_spaces` before the next word.

- `ws.join(w[0:wide_words+1])` will join the first and second words with wider spaces
- `ns.join()` will then join the wider-spaced-words with all remaining words

To make things easier during developement, and to make sure that a little fix won't mess up everything, I wrote this testing unit:

````python
import unittest
import re
from justify_text import justify

class TestStringMethods(unittest.TestCase):

	def test_given(self):
		self.assertEqual(
			justify(
				["the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog"], 16),
				["the  quick brown",
        		 "fox  jumps  over",
        		 "the   lazy   dog"])

	def test_lengths(self):
		for i in range(1,80):
			j = justify(["the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog"], i)
			lengths = list(map(lambda x: len(x), j))
			for n in range(0, len(lengths)):
				self.assertEqual(lengths[n], i)

	def test_decrease(self):
		for i in range(1,80):
			j = justify(["the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog"], i)

			spaces_split = list(map(lambda x: re.split('[^ ]+', x), j))
			for n in range(0, len(spaces_split)):
				self.assertEqual(spaces_split[n][0], '')
				for t in range(1, len(spaces_split[n])-2):
					self.assertTrue(len(spaces_split[n][t])>=len(spaces_split[n][t+1]))

if __name__ == '__main__':
    unittest.main()
````

- test_given: the result of the function has to match the result from the problem specification
- test_lengths: all strings have to be of length k (k from 1 to 80)
- test_decrease: all strings have to start with a word, not a space, and the number of spaces from left to right has not to increase

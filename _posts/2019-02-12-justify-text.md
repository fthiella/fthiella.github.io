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
>
> ` "fox  jumps  over",` # 2 extra spaces distributed evenly
>
> ` "the   lazy   dog"]` # 4 extra spaces distributed evenly

This looks like a problem that can be quickly solved in a few lines of codes, but it would
actually require a little more lines than expected. While the case of a word longer than the
given integer `k` is not specifically covered in the problem description, I assume than such
word has to be split into k sized chunks.

The logic is going to be like this:

- I keep a buffer that holds all words that can fit into a single line. This buffer is empty at first.
- From the given array I read each word, one by one. I eventually split each word into k-sized chunks.
- Is the current word / word chunk going to fit into the current line? If there's still space for it I am going
  to add it to the current line buffer.
- If the current word / chunk won't fit, I am going to output the current line (all words in the buffer, with
  proper space justification). The buffer will then be set to contain only the current word.
- When I am finished reading all words, I am going to output the current line (unless the buffer is empty, which
  will happen only if the array of words is empty as well)

This is how I am going to call the function:

{% highlight python %}
if __name__ == '__main__':
	print("\n".join(justify(["the", "quick", "brown", "fox",
		"jumps", "over", "the", "lazy", "dog"], 16)))
{% endhighlight %}

And this is how my `justify` function looks like:

{% highlight python %}
def justify(words, k):
	res = []
	current_length = 0
	current_words = []

	for word in words:
		# read each word
		for i in range(0, len(word), k):
			# split each row into k-sized chuncks
			chunk = word[i:i+k]

			# try to add a new word to current_words / current_length
			if len(chunk) + current_length + len(current_words) <= k:
				# can fit
				current_words.append(chunk)
				current_length += len(chunk)
			else:
				# won't fit
				res.append(justify_line(current_words, current_length, k))
				current_words = [chunk]
				current_length = len(chunk)

	if current_words:
		res.append(justify_line(current_words, current_length, k))

	return res
{% endhighlight %}

The final part is handled by the `justify_line` function, that accept as input the array `w` of words that fits in the current line,
the length `l` of the current words (not considering spaces), and the width of the like `k`.

(`l` is not strictly necessary, I have all elements to calculate it inside the function, but since it's alreay available I'll just pass it to the function)

{% highlight python %}
def justify_line(w, l, k):
	if len(w)==1:
		return w[0].ljust(k)
	else:
		narrow_spaces, wider_words = divmod(k - l, len(w)-1)
		ns = " " * narrow_spaces
		ws = " " * (narrow_spaces + 1)

		return ns.join([ws.join(w[0:wider_words+1]), *w[wider_words+1:]])
{% endhighlight %}

- If the current line contains only one word then I'll just return this
  word left-justified with spaces at the right (as from the problem specs).

- If instead the line is composed by more words, we have to calculate:
  - the total number of spaces needed (`k - l`)
  - the minimum number of spaces between each word (total numer of spaces DIV number of words minus one)
  - the number of words that require an additional space (total number of spaces MOD number of words minus one)

For example if I call `justify_line(['the', 'quick', 'brown'], 13, 16)`:
- `narrow_spaces` will be set to 1 (one space between each word)
- `wider_words` will be set to 1 (the first word needs one more extra space after it)
- `ws.join(w[0:wide_words+1])` will join the first and second words with wider spaces
- `ns.join()` will then join the wider-spaced-words with all remaining words

To make things easier during developement, and to make sure that a little fix won't mess up everything, I wrote this testing unit:

{% highlight python %}
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
{% endhighlight %}

- test_given: the result of the function has to match the result from the problem specification
- test_lengths: all strings have to be of length k (k from 1 to 80)
- test_decrease: all strings have to start with a word, not a space, and the number of spaces from left to right has not to increase

---
date: "2010-04-16T05:43:53"
lastmod: "2010-04-16T05:43:53"
title: "Huffman Coding, Unicode, and CJKV Data"
---
Today I wrote a little utility in Java that compresses a file using [Huffman coding](http://en.wikipedia.org/wiki/Huffman_coding).
Normally Huffman coding works on 8-bit bytes.
However, because of my experience dealing with Chinese, Japanese, Korean, and other non-English text I wondered how well the coding method would work on double byte character sets.
Specifically, I was curious about compressing [UTF-8](http://en.wikipedia.org/wiki/UTF-8) text.

UTF-8 is a variable length encoding for Unicode data that stores characters using between one and four bytes per character.
This works great when the text is written mostly (or completely) with the latin alphabet, but other alphabets require two or more bytes per character, which causes the file to become bloated very quickly.
(This is one of the reasons that Asian countries have been slow to adopt Unicode.)
Also, UTF-8 is designed to allow random access to characters in a file despite each character taking up a different number of bytes.
This is a traditional problem with double byte encodings that use shift in/out characters.
With these older encodings, in order to know if a pair of bytes is a single character, the program must also know what the last shift character was.
UTF-8 solves this problem by using the high bit in the byte to mark a byte as part of a multibyte sequence.
The tradeoff for this is that each byte can only store 7 bits worth of data, which is why UTF-8 uses four bytes in the worst case instead of only two.
(Unicode can also have surrogate character pairs regardless of the underlying encoding that is used, but I'm not going to bother with that in this post.)

Huffman coding is also a variable length encoding.
It uses the distribution of bytes in a given document to assign shorter bit sequences to more common bytes and longer bit sequences to less common bytes.
This means that the most common bytes only take up a few bits (as few as 1 or 2), whereas very uncommon bytes can take up more than 8 bits (and thus take more space to store than they did in the original document.)
Although it is possible, if the input data is sufficiently random, for the output to be larger than the input, in most cases the output will be much smaller than the original input.

Because the Huffman coding algorithm doesn't know about multibyte sequences in UTF-8 documents, it doesn't take into account the fact that certain bytes (those with the high bit set) only occur together with other bytes and therefore their distribution is not independent.
This means that the [entropy](http://en.wikipedia.org/wiki/Entropy_(information_theory)) of the file is actually smaller than the algorithm thinks it is and therefore the algorithm needs fewer bits to represent the file than it realizes.
In order to make use of multibyte sequences - not only in UTF-8, but in other multibyte encodings as well - I modified the algorithm to think in terms of Unicode characters instead of bytes.
By converting the file from its native encoding into a stream of characters before performing the Huffman coding calculations, the entropy of the file is reduced and the file can be stored more compactly.

Since a file written in an Asian language will consist mostly of Unicode characters that translate to two or more bytes in UTF-8, it stands to reason that the standard Huffman coding algorithm won't produce as compact a file as a Huffman coding algorithm that works in terms of characters instead of in terms of bytes.
To test this I compressed a Chinese text file with both the standard and improved algorithms and compared the results.
Although my encoding method didn't save as much space as other common compression utilities such as GZip, I still saw an improvement when encoding characters instead of raw bytes.

Compressing the raw UTF-8 bytes of a 69KB Chinese file created a 48KB file.
Compressing the underlying characters created a 32KB file, which is a significant improvement.
Likewise, starting with a 46KB [GBK](http://en.wikipedia.org/wiki/GBK) encoded version of the same file created a 34KB file when compressing the raw bytes and a 32KB file when compressing characters instead.
(As a matter of fact, compressing characters creates the same file in both cases because the input characters are converted to Unicode internally.)

Here is the raw data from the experiment:

|File Description                                          |File Size|
|----------------------------------------------------------|:-------:|
|Original File - UTF-8 Encoded                             |69 KB    |
|Compressed File - Huffman Bytes (UTF-8 Source)            |48 KB    |
|Original File - GBK Encoded                               |46 KB    |
|Compressed File - Huffman Bytes (GBK Source)              |34 KB    |
|Compressed File - Huffman Characters (GBK or UTF-8 Source)|32 KB    |
|Compressed File - GZiped UTF-8                            |27 KB    |
|Compressed File - GZiped GBK                              |24 KB    |

<span style="font-size: 75%">Note: The "Huffman Bytes" files were created using the same algorithm as the "Huffman Characters" files, except the input encoding was set to ISO-8859-1 so that the algorithm would ignore multibyte sequences in the input files and work purely with 8-bit bytes.  This produces the same result that the original Huffman coding would produce.</span>

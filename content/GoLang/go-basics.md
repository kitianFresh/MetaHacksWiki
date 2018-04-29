---
title: "go-basics"
date: 2018-01-28 20:30
---
[TOC]

# Streaming IO in Go
[Streaming IO in Go](https://medium.com/learning-the-go-programming-language/streaming-io-in-go-d93507931185)

>bytes.NewReader creates a Reader from a byte slice (that is, a chunk of memory you already have in your program.) Useful if you want to pass a byte slice to some other API that expects a Reader. bufio.NewReader on the other hand is for wrapping existing Readers (usually ones whose Read method is relatively expensive, like a TCP connection or a file) to coalesce a bunch of easy-to-program small Read calls into a few larger ones.

>A bytes.Buffer is a buffer with two ends; you can only read from the 
start of it, and you can only write to the end of it. No seeking. 
There's no need to use both sides though; I often use it to construct 
a chunk of encoded stuff in memory (by passing it to serialization 
libraries expecting a Writer) then pull out the []byte at the end 
(with (*bytes.Buffer).Bytes()), then stop using the bytes.Buffer and 
pass the []byte I got to another layer for more work. 

A bytes.Reader is more explicitly only for turning a []byte into 
something that's a Reader; since no modifications are possible, it can 
also implement Seek and ReadAt without ambiguity, thus being more 
useful to pass to other functions expecting a ReaderAt or a ReadSeeker 
or whatever. 
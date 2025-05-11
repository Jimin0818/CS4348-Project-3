# CS4348 Project 3: B-Tree Index File Manager

## Overview
This project implements a command line B Tree index file manager using Python. It allows users to create, manage, and query B-Tree-based index files on disk with minimal memory usage. The application supports basic commands for creating, inserting, searching, printing, and extracting index data. Each index file follows a strict binary format using fixed-size 512-byte blocks.

## Features
- B-Tree implementation with minimum degree 10
- Stores key/value pairs and child pointers in structured binary format
- File-backed index using fixed 512-byte blocks
- Supports up to 3 B-Tree nodes in memory at a time
- Big-endian 8-byte integer serialization
- Command-line interface with the following operations:
  - create — Create a new index file
  - insert — Insert a single key/value pair
  - search — Search for a key and display its value
  - load — Load key/value pairs from a CSV file
  - print — Print all key/value pairs
  - extract — Export all key/value pairs to a CSV

## Command Examples

Create new index file:  python project3.py create test.idx

Insert:  python project3.py insert test.idx 20 100

Search:   python project3.py search test.idx 20

Bulk insert:  python project3.py load test.idx input.csv

Print:  python project3.py print test.idx

Export:  python project3.py extract test.idx output.csv

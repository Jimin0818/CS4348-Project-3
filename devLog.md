### April 27
- **4:30 PM** – Created the GitHub repository and initialized the project with a `README.md`.  
  Reviewed project instructions to understand the B-Tree specifications, 512-byte block structure, and command-line interface requirements.  
  Took notes on memory constraints and binary encoding strategies in Python.

### May 4
- **9:30 PM** – Began active development after planning the architecture.
- **9:42 PM** – Set up the main Python script and class structure for `BTreeNode`, `BTree`, and helper utilities.
- **10:19 PM** – Started implementing the core logic for B-Tree structure and node relationships.
- **10:54 PM** – Fully defined the `BTreeNode` class and its constructor.
- **11:30 PM** – Created internal variables for keys, values, children, and block tracking.
- **11:58 PM** – Took a break for the night.

### May 5
- **10:53 PM** – Returned to development. Reviewed rubric, refactored class structures, and resolved syntax errors.
- **11:27 PM** – Implemented `read_block` and `write_block` functions to handle 512-byte I/O.

### May 7
- **12:45 AM** – Implemented `create_index` to initialize index files with the correct header.
- **12:33 PM** – Implemented `insert_key` and enhanced `read_header` to load root and next block ID correctly.
- **1:07 PM** – Continued building recursive tree logic for insertions and node splitting.

### May 9
- **1:37 AM** – Finalized `insert` and `search` functions. Debugged tree navigation and off-by-one errors.
- **2:06 AM** – Verified core functionality including `insert`, `search`, and node serialization.  
  Started implementing `load` and `extract` functions.
- **2:30 AM** – Finished command parsing logic and internal structure for `load` and `extract`.  
  Noted final tasks for completion.

### May 11
- **3:04 PM** – Ran full test suite. Finalized logic for `search`, `insert`, `load`, `extract`, and `print`.  
  Verified memory usage and B-Tree traversal.
- **3:46 PM** – Completed final `README.md` including command usage, file format explanation, and implementation notes.
- **4:05 PM** – Cleaned and commented the code. Removed debug statements, improved naming, and finalized files for submission.

import sys
import os
import struct

BLOCK_SIZE = 512
MAGIC = b'4348PRJ3'
DEGREE = 10
MAX_KEYS = 2 * DEGREE - 1
MAX_CHILDREN = 2 * DEGREE

def int_to_bytes(n):
    return n.to_bytes(8, 'big')

def bytes_to_int(b):
    return int.from_bytes(b, 'big')

def pad_block(data):
    return data + b'\x00' * (BLOCK_SIZE - len(data))

class BTreeNode:
    def __init__(self, block_id=0, parent=0, keys=None, values=None, children=None):
        self.block_id = block_id
        self.parent = parent
        self.keys = keys or []
        self.values = values or []
        self.children = children or [0] * MAX_CHILDREN

    def to_bytes(self):
        b = int_to_bytes(self.block_id)
        b += int_to_bytes(self.parent)
        b += int_to_bytes(len(self.keys))
        b += b''.join([int_to_bytes(k) for k in self.keys] + [b'\x00' * 8] * (MAX_KEYS - len(self.keys)))
        b += b''.join([int_to_bytes(v) for v in self.values] + [b'\x00' * 8] * (MAX_KEYS - len(self.values)))
        b += b''.join([int_to_bytes(c) for c in self.children])
        return pad_block(b)

    @staticmethod
    def from_bytes(data):
        block_id = bytes_to_int(data[0:8])
        parent = bytes_to_int(data[8:16])
        num_keys = bytes_to_int(data[16:24])
        keys = [bytes_to_int(data[24+i*8:32+i*8]) for i in range(MAX_KEYS)]
        values = [bytes_to_int(data[176+i*8:184+i*8]) for i in range(MAX_KEYS)]
        children = [bytes_to_int(data[328+i*8:336+i*8]) for i in range(MAX_CHILDREN)]
        return BTreeNode(block_id, parent, keys[:num_keys], values[:num_keys], children)

def read_block(file, block_id):
    file.seek(block_id * BLOCK_SIZE)
    return file.read(BLOCK_SIZE)

def write_block(file, block_id, data):
    file.seek(block_id * BLOCK_SIZE)
    file.write(data)

def create_index(filename):
    if os.path.exists(filename):
        print("Error: File already exists.")
        return
    with open(filename, 'wb') as f:
        header = MAGIC + int_to_bytes(0) + int_to_bytes(1)
        f.write(pad_block(header))
    print(f"{filename} created.")

def read_header(file):
    file.seek(0)
    header = file.read(BLOCK_SIZE)
    if header[:8] != MAGIC:
        raise Exception("Invalid index file")
    root = bytes_to_int(header[8:16])
    next_block = bytes_to_int(header[16:24])
    return root, next_block

def write_header(file, root, next_block):
    file.seek(0)
    header = MAGIC + int_to_bytes(root) + int_to_bytes(next_block)
    file.write(pad_block(header))

def split_child(f, parent, index, child, next_block):
    mid = DEGREE - 1
    right = BTreeNode(
        block_id=next_block,
        parent=parent.block_id,
        keys=child.keys[mid+1:],
        values=child.values[mid+1:],
        children=child.children[mid+1:] + [0] * (MAX_CHILDREN - len(child.children[mid+1:]))
    )
    promoted_key = child.keys[mid]
    promoted_value = child.values[mid]
    child.keys = child.keys[:mid]
    child.values = child.values[:mid]
    child.children = child.children[:mid+1] + [0] * (MAX_CHILDREN - (mid + 1))
    parent.keys.insert(index, promoted_key)
    parent.values.insert(index, promoted_value)
    parent.children.insert(index + 1, right.block_id)
    parent.children = parent.children[:MAX_CHILDREN]
    write_block(f, child.block_id, child.to_bytes())
    write_block(f, right.block_id, right.to_bytes())
    write_block(f, parent.block_id, parent.to_bytes())
    _, nb = read_header(f)
    write_header(f, root=read_header(f)[0], next_block=nb + 1)
    print(f"Split child node {child.block_id} into {child.block_id} and {right.block_id}, promoted to {parent.block_id}")

def insert_into_node(f, node, key, value, next_block):
    if all(c == 0 for c in node.children[:len(node.keys)+1]):
        i = 0
        while i < len(node.keys) and key > node.keys[i]:
            i += 1
        if i < len(node.keys) and node.keys[i] == key:
            print("Error: Duplicate key.")
            return
        node.keys.insert(i, key)
        node.values.insert(i, value)
        write_block(f, node.block_id, node.to_bytes())
        print(f"Inserted into leaf node {node.block_id}.")
        return
    i = 0
    while i < len(node.keys) and key > node.keys[i]:
        i += 1
    child_id = node.children[i]
    child = BTreeNode.from_bytes(read_block(f, child_id))
    if len(child.keys) >= MAX_KEYS:
        split_child(f, node, i, child, next_block)
        if key > node.keys[i]:
            child_id = node.children[i + 1]
        else:
            child_id = node.children[i]
        child = BTreeNode.from_bytes(read_block(f, child_id))
    insert_into_node(f, child, key, value, next_block=read_header(f)[1])

def insert_key(filename, key, value):
    key = int(key)
    value = int(value)
    with open(filename, 'r+b') as f:
        root_id, next_block = read_header(f)
        if root_id == 0:
            root_node = BTreeNode(block_id=1)
            root_node.keys.append(key)
            root_node.values.append(value)
            write_block(f, 1, root_node.to_bytes())
            write_header(f, root=1, next_block=2)
            print("Inserted into new root.")
            return
        root_block = read_block(f, root_id)
        root_node = BTreeNode.from_bytes(root_block)
        if len(root_node.keys) >= MAX_KEYS:
            split_root(f, root_node, next_block)
            root_id, _ = read_header(f)
            root_block = read_block(f, root_id)
            root_node = BTreeNode.from_bytes(root_block)
        insert_into_node(f, root_node, key, value, next_block)

def split_root(file, root_node, next_block):
    mid = DEGREE - 1
    left = BTreeNode(
        block_id=next_block,
        parent=next_block + 2,
        keys=root_node.keys[:mid],
        values=root_node.values[:mid]
    )
    right = BTreeNode(
        block_id=next_block + 1,
        parent=next_block + 2,
        keys=root_node.keys[mid+1:],
        values=root_node.values[mid+1:]
    )
    promoted_key = root_node.keys[mid]
    promoted_value = root_node.values[mid]
    new_root = BTreeNode(
        block_id=next_block + 2,
        parent=0,
        keys=[promoted_key],
        values=[promoted_value],
        children=[left.block_id, right.block_id] + [0] * (MAX_CHILDREN - 2)
    )
    write_block(file, left.block_id, left.to_bytes())
    write_block(file, right.block_id, right.to_bytes())
    write_block(file, new_root.block_id, new_root.to_bytes())
    write_header(file, root=new_root.block_id, next_block=new_root.block_id + 1)
    print(f"Root split: new root = {new_root.block_id}, children = {left.block_id}, {right.block_id}")

def search_node(f, node, key):
    i = 0
    while i < len(node.keys) and key > node.keys[i]:
        i += 1

    if i < len(node.keys) and key == node.keys[i]:
        print(f"Found: {key} -> {node.values[i]}")
        return True

    if all(c == 0 for c in node.children):
        print("Key not found.")
        return False

    child_id = node.children[i]
    if child_id == 0:
        print("Key not found.")
        return False

    child = BTreeNode.from_bytes(read_block(f, child_id))
    return search_node(f, child, key)

def search_key(filename, key):
    key = int(key)
    with open(filename, 'rb') as f:
        root_id, _ = read_header(f)
        if root_id == 0:
            print("Empty tree.")
            return
        root = BTreeNode.from_bytes(read_block(f, root_id))
        search_node(f, root, key)


def load_csv(filename, csv_filename):
    if not os.path.exists(csv_filename):
        print("Error: CSV file does not exist.")
        return
    with open(csv_filename, 'r') as csvfile:
        for line in csvfile:
            if ',' not in line:
                continue
            key_str, value_str = line.strip().split(',', 1)
            insert_key(filename, key_str, value_str)

def print_node(f, node):
    if all(c == 0 for c in node.children):
        for k, v in zip(node.keys, node.values):
            print(f"{k} -> {v}")
        return

    for i in range(len(node.keys)):
        if node.children[i] != 0:
            child = BTreeNode.from_bytes(read_block(f, node.children[i]))
            print_node(f, child)
        print(f"{node.keys[i]} -> {node.values[i]}")
    if node.children[len(node.keys)] != 0:
        child = BTreeNode.from_bytes(read_block(f, node.children[len(node.keys)]))
        print_node(f, child)

def print_index(filename):
    with open(filename, 'rb') as f:
        root_id, _ = read_header(f)
        if root_id == 0:
            print("Index is empty.")
            return
        root = BTreeNode.from_bytes(read_block(f, root_id))
        print_node(f, root)


def extract_csv(indexfile, out_csv):
    if os.path.exists(out_csv):
        print("Error: Output file already exists.")
        return
    with open(indexfile, 'rb') as f:
        root_id, _ = read_header(f)
        if root_id == 0:
            print("Index is empty.")
            return
        node = BTreeNode.from_bytes(read_block(f, root_id))
        with open(out_csv, 'w') as out:
            for k, v in zip(node.keys, node.values):
                out.write(f"{k},{v}\n")

if __name__ == '__main__':
    args = sys.argv
    if len(args) < 3:
        print("Usage: project3 <command> <filename> [args...]")
        sys.exit(1)
    cmd = args[1]
    if cmd == 'create':
        create_index(args[2])
    elif cmd == 'insert':
        if len(args) < 5:
            print("Usage: project3 insert <file> <key> <value>")
        else:
            insert_key(args[2], args[3], args[4])
    elif cmd == 'search':
        if len(args) < 4:
            print("Usage: project3 search <file> <key>")
        else:
            search_key(args[2], args[3])
    elif cmd == 'load':
        if len(args) < 4:
            print("Usage: project3 load <indexfile> <csvfile>")
        else:
            load_csv(args[2], args[3])
    elif cmd == 'print':
        if len(args) < 3:
            print("Usage: project3 print <indexfile>")
        else:
            print_index(args[2])
    elif cmd == 'extract':
        if len(args) < 4:
            print("Usage: project3 extract <indexfile> <csvfile>")
        else:
            extract_csv(args[2], args[3])
    else:
        print(f"Unknown command: {cmd}")

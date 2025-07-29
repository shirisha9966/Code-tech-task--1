# Code-tech-task--1
import hashlib
import os
import json

# Function to calculate SHA256 hash of a file
def calculate_hash(filepath):
    sha256_hash = hashlib.sha256()
    try:
        with open(filepath, "rb") as f:
            # Read and update hash string value in blocks of 4K
            for byte_block in iter(lambda: f.read(4096), b""):
                sha256_hash.update(byte_block)
        return sha256_hash.hexdigest()
    except FileNotFoundError:
        return None

# Function to create a hash record for all files in a directory
def create_hash_record(directory, output_file='hashes.json'):
    hash_record = {}
    for root, _, files in os.walk(directory):
        for file in files:
            filepath = os.path.join(root, file)
            hash_record[filepath] = calculate_hash(filepath)
    
    with open(output_file, 'w') as f:
        json.dump(hash_record, f, indent=4)
    print(f"[✔] Hash record saved to '{output_file}'.")

# Function to verify file integrity using saved hash record
def verify_integrity(hash_file='hashes.json'):
    with open(hash_file, 'r') as f:
        original_hashes = json.load(f)

    modified_files = []
    missing_files = []
    for filepath, original_hash in original_hashes.items():
        current_hash = calculate_hash(filepath)
        if current_hash is None:
            missing_files.append(filepath)
        elif current_hash != original_hash:
            modified_files.append(filepath)

    if not modified_files and not missing_files:
        print("[✔] All files are intact. No changes detected.")
    else:
        if modified_files:
            print("[✘] Modified files:")
            for file in modified_files:
                print(f"  - {file}")
        if missing_files:
            print("[✘] Missing files:")
            for file in missing_files:
                print(f"  - {file}")

# Example Usage
if __name__ == "__main__":
    print("1. Create Hash Record")
    print("2. Verify File Integrity")
    choice = input("Enter choice (1/2): ")

    if choice == '1':
        folder = input("Enter directory to monitor: ")
        create_hash_record(folder)
    elif choice == '2':
        hash_file = input("Enter path to hash file (default: hashes.json): ") or 'hashes.json'
        verify_integrity(hash_file)
    else:
        print("Invalid choice.")

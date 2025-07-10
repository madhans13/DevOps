# Linux Foundation Course Notes

## Overview
This repository contains my notes and key learnings from the Linux Foundation course, covering essential Linux operations, file management, and search techniques.

## 📚 Topics Covered

### 1. Basic Operations

#### System Management
- **Logging In and Out**: Understanding user authentication and session management
- **Rebooting and Shutting Down**: Safe system restart and shutdown procedures
- **Locating Applications**: Finding and accessing installed programs

#### Directory Navigation
- **Accessing Directories**: Moving between folders using command line
- **Absolute vs Relative Paths**: 
  - Absolute: Full path from root (`/home/user/documents`)
  - Relative: Path from current location (`./documents` or `../parent`)
- **Exploring the Filesystem**: Understanding Linux directory structure

#### File System Links
- **Hard Links**: Multiple directory entries pointing to the same file data
- **Soft (Symbolic) Links**: Shortcuts that point to another file or directory
- **Directory History Navigation**: Using command history to move efficiently

### 2. Working with Files

#### File Operations
- **Viewing Files**: 
  - `cat` - Display entire file content
  - `less`/`more` - Page through large files
  - `head`/`tail` - Show beginning/end of files

#### File and Directory Management
- **Creating Files**: `touch filename` - Create empty files or update timestamps
- **Creating Directories**: `mkdir dirname` - Make new directories
- **Removing Directories**: `rmdir dirname` - Remove empty directories
- **Moving/Renaming**: `mv source destination` - Move or rename files/directories
- **Removing Files**: `rm filename` - Delete files (use `-rf` for directories)

#### Command Line Customization
- **Modifying Command Prompt**: Personalizing the shell prompt display
- **Command Line Best Practices**: Efficient file and directory operations

### 3. Searching for Files

#### I/O Operations
- **Standard File Streams**: 
  - stdin (0) - Input
  - stdout (1) - Output  
  - stderr (2) - Error output
- **I/O Redirection**: 
  - `>` - Redirect output to file
  - `>>` - Append output to file
  - `<` - Redirect input from file
- **Pipes**: `|` - Send output of one command as input to another

#### File Location Tools
- **locate**: Fast file searching using database
  - `locate filename` - Find files by name
  - `updatedb` - Update locate database

#### Advanced Search with Wildcards
- **Wildcard Patterns**:
  - `*` - Match any characters
  - `?` - Match single character
  - `[abc]` - Match any character in brackets
  - `[a-z]` - Match range of characters
- **Using Wildcards**: `ls *.txt` - List all .txt files

#### The find Command
- **Basic Usage**: `find /path -name "filename"`
- **Advanced Options**:
  - `-type f` - Find files only
  - `-type d` - Find directories only
  - `-size +100M` - Find files larger than 100MB
  - `-mtime -7` - Find files modified in last 7 days
  - `-exec command {} \;` - Execute command on found files

#### Time and Size-Based Searches
- **Finding by Time**:
  - `-mtime` - Modified time
  - `-atime` - Access time
  - `-ctime` - Change time
- **Finding by Size**:
  - `-size +1G` - Larger than 1GB
  - `-size -10M` - Smaller than 10MB

## 🛠️ Practical Labs Completed

1. **Lab 8.2**: Locating Applications
2. **Lab 8.3**: Creating, Moving and Removing Files  
3. **Lab 8.4**: Finding Directories and Creating Symbolic Links

## 📝 Key Commands Summary

```bash
# Basic Navigation
cd /path/to/directory    # Change directory
pwd                      # Print working directory
ls -la                   # List files with details

# File Operations
touch filename           # Create empty file
mkdir dirname           # Create directory
mv source dest          # Move/rename
rm filename             # Remove file
rm -rf dirname          # Remove directory recursively

# Viewing Files
cat filename            # Display file content
less filename           # Page through file
head -n 10 filename     # Show first 10 lines
tail -n 10 filename     # Show last 10 lines

# Searching
locate filename         # Quick file search
find /path -name "*.txt" # Find .txt files
find /path -size +100M  # Find large files
find /path -mtime -1    # Find recently modified files

# I/O Operations
command > file          # Redirect output
command >> file         # Append output
command1 | command2     # Pipe output
```

## 💡 Best Practices

1. Always use tab completion to avoid typos
2. Use relative paths when working within a project directory
3. Be careful with `rm -rf` - it's irreversible
4. Use `find` for complex searches, `locate` for simple filename searches
5. Update locate database regularly with `updatedb`
6. Use wildcards to work with multiple files efficiently

## ⚠️ Important: rm vs rm -rf

### `rm folder1` (Basic Command)
- **Result**: FAILS with error "cannot remove 'folder1': Is a directory"
- **Safety**: ✅ Safe - protects against accidental directory deletion
- **Purpose**: Only removes files, not directories

### `rm -rf folder1` (With Flags)
- **Result**: SUCCEEDS - completely removes directory and ALL contents
- **Safety**: ❌ Dangerous - deletes everything silently without confirmation
- **Flags Explained**:
  - `-r` (recursive): Removes directories and their contents recursively
  - `-f` (force): Forces deletion without prompting for confirmation

### Safety Comparison Table
| Command | Safety Level | Behavior |
|---------|--------------|----------|
| `rm folder1` | ✅ Safe | Fails, protects from accidents |
| `rm -r folder1` | ⚠️ Moderate | Asks for confirmation before deleting |
| `rm -rf folder1` | ❌ Dangerous | Deletes everything silently |

### Safer Alternatives
```bash
rmdir folder1          # Only removes EMPTY directories
rm -ri folder1         # Interactive mode - asks for confirmation
rm -r folder1          # Recursive but asks before removing each file
```

### Critical Safety Tips
- **No recovery**: Deleted files don't go to trash - they're gone forever
- **Double-check paths**: Always verify the directory path before using `-rf`
- **Use interactive mode**: Consider `alias rm='rm -i'` for confirmation prompts
- **Test first**: Use `ls` to verify what you're about to delete

## 🔗 Additional Resources

- [Linux Foundation Training](https://training.linuxfoundation.org/)
- [Linux Command Line Cheat Sheet](https://www.linuxtrainingacademy.com/linux-commands-cheat-sheet/)
- [Advanced Bash Scripting Guide](https://tldp.org/LDP/abs/html/)

---

*Last updated: [Current Date]*
*Course: Linux Foundation Essentials*
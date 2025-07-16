🐧 Bash Scripting Basics: String Manipulation, Conditionals & Looping
This repository documents my learning journey on foundational Bash scripting concepts. I've wrapped up practice and notes on string manipulation, conditional statements, and looping, which are core building blocks for automating tasks in Linux — an essential skill on my path toward DevOps.

📌 Topics Covered
1. 🔤 String Manipulation

${#string} – Get string length
${string:position:length} – Substring extraction
${string#pattern} / ${string##pattern} – Remove prefix
${string%pattern} / ${string%%pattern} – Remove suffix
${string/search/replace} – Replace substring
echo "$string" | tr 'a-z' 'A-Z' – Change case

2. ✅ Conditional Statements

[ -d filename ] – Check if a directory exists
[ -f filename ] – Check if a file exists
[ "$str1" = "$str2" ] – String comparison
if...elif...else and case statements

Example:
bashif [ "$user" = "admin" ]; then
  echo "Welcome, admin"
else
  echo "Access denied"
fi
3. 🔁 Looping

for, while, and until loops
Looping over files, arguments, and ranges

Example:
bashfor file in *.txt; do
  echo "File: $file"
done
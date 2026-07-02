---
name: gen-imglist
description: Generate an imglist YAML block for a Hugo post by finding all *0096.jpg files in a given static/images subdirectory. Pass the subdirectory path (e.g. "static/images/2023") as the argument.
---

# Generate imglist

The user wants an `imglist` YAML block for a Hugo post.

## Steps

1. The user will pass a directory path as the argument (e.g. `static/images/2023`). If no argument is given, ask the user which directory to scan.

2. Run the following to find matching files, sorted:

```bash
ls <directory>/ | grep '0096\.jpg$' | sort
```

3. For each file found:
   - Take the **parent directory name** (last path component of the directory) as the prefix — e.g. `2023`.
   - Take the **filename**, strip the trailing `-0096.jpg` suffix (everything from the last `-0096.jpg` onward).
   - The `root:` value is `<prefix>/<stripped-filename>`.
   - Leave `title:` empty.

4. Output the block in this exact format:

```yaml
imglist:
- root: 2023/1000-L1000435-Enhanced-NR
  title:
- root: 2023/1001-L1000401
  title:
```

## Rules

- Do **not** strip `-Enhanced-NR` or any other middle portions of the filename — only remove the `-0096.jpg` suffix at the very end.
- Do **not** modify any files. Only produce the text for the user to copy.
- Sort entries in the same order `ls` returns them (numeric/lexicographic).

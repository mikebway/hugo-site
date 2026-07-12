---
name: number-imglist
description: Prefix each title in a Hugo post's imglist front matter with its position number, in the form "~#~ ". Pass the content file path (e.g. "content/post/al-andalus.md") as the argument.
---

# Number imglist titles

The user wants each `title:` value inside a post's `imglist:` front matter block
prefixed with its 1-based position in the list, in the form `~#~ ` (tilde,
number, tilde, space), where `#` is replaced by that entry's sequence number.

For example, the 3rd entry's title `Nimes, France, 2026` becomes
`~3~ Nimes, France, 2026`.

## Steps

1. The user will pass a content file path as the argument (e.g.
   `content/post/al-andalus.md`). If no argument is given, ask which file to
   process.

2. Write and run the following Python script against that file (it uses only
   the standard library — no PyYAML dependency needed):

```python
import re, sys

path = sys.argv[1]
with open(path) as f:
    lines = f.readlines()

# Front matter is delimited by the first two "---" lines.
dashes = [i for i, l in enumerate(lines) if l.rstrip('\n') == '---']
if len(dashes) < 2:
    sys.exit(f"Could not find front matter delimiters in {path}")
fm_start, fm_end = dashes[0], dashes[1]

imglist_line = next(
    (i for i in range(fm_start + 1, fm_end) if lines[i].rstrip('\n') == 'imglist:'),
    None,
)
if imglist_line is None:
    sys.exit(f"No top-level 'imglist:' key found in {path}")

root_re = re.compile(r'^- root:')
title_re = re.compile(r'^(\s*title:\s*)(.*?)\s*$')
prefix_re = re.compile(r'^~\d+~\s*')

count = 0
awaiting_title = False
for i in range(imglist_line + 1, fm_end):
    line = lines[i]
    if root_re.match(line):
        count += 1
        awaiting_title = True
        continue
    if awaiting_title:
        m = title_re.match(line)
        if m:
            prefix, value = m.groups()
            value = prefix_re.sub('', value)  # idempotent: drop any existing ~N~ prefix first
            lines[i] = f"{prefix}~{count}~ {value}\n"
            awaiting_title = False

with open(path, 'w') as f:
    f.writelines(lines)

print(f"Numbered {count} title(s) in {path}")
```

Run it as:

```bash
python3 /path/to/script.py <content-file>
```

3. Report back how many entries were numbered, and show the user a `git diff`
   of the file so they can review the change before it's committed.

## Rules

- Only touch `title:` lines that immediately follow a `- root:` line inside
  the `imglist:` block of the front matter (between the first two `---`
  delimiters) — never touch body content, `break:` lines, or anything outside
  `imglist:`.
- Numbering is 1-based and counts every `- root:` entry in order, including
  ones followed by `break: true`.
- The script is idempotent: if a title already starts with `~<number>~ `, that
  prefix is stripped before the current, correct number is applied — so
  re-running after reordering entries fixes the numbers rather than stacking
  prefixes.
- Preserve the trailing space after the closing `~` even when the original
  title is empty (the result is `~5~ ` with nothing after it) — this is
  intentional, per the requested format.
- Do not reformat, reorder, or otherwise touch any other part of the file.

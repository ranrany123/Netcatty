# Autocomplete description overlay: touch pilot

The terminal autocomplete popup keeps the bundled Fig spec in English and overlays
only suggestion descriptions for the active UI locale. The overlay is keyed by the
exact English description string and ships in the renderer bundle; it performs no
runtime network or AI request.

## Sources

| Source | Version / location | Used for |
| --- | --- | --- |
| `@withfig/autocomplete` | `2.692.3`, npm `gitHead` `aef52acff84c45edde61ae610cc2c964802b9a38`; `src/touch.ts` and compiled `build/touch.js` | English description keys and option structure |
| `jaywcjlove/linux-command` | `command/touch.md` at `db88f1b96abf561fb43e6960c2e6b4367f83fdce` (2024-12-26) | Simplified-Chinese terminology and GNU behavior |
| GNU Coreutils manual | 9.12, `touch invocation` (`https://www.gnu.org/software/coreutils/manual/html_node/touch-invocation.html`) | GNU/Linux option semantics |
| macOS `touch(1)` | macOS 26.7 man page (June 1, 2018) | BSD/macOS manual wording |
| Apple `file_cmds` | `main` checked 2026-11-04; latest `touch/touch.c` change `a1a0d101147af8494c96a855506fc0fd38469092` (tag `file_cmds-400`) | Current macOS `-f` implementation |
| FreeBSD `freebsd-src` | commit `b8d55a86995b5a8db5d1651c8dc9fc5093b67d2c` (2026-02-04), `usr.bin/touch/touch.c` | Confirms the BSD-family `-f` no-op |

Checked on 2026-11-04. All English keys below occur only in the `touch` spec
across the bundled `@withfig/autocomplete@2.692.3` specs and local
`electron/specs`; changing any of them does not affect another command's overlay.

## Adopted entries

| Entry | English description | zh-CN |
| --- | --- | --- |
| command | Change file access and modification times | 更改文件的访问时间和修改时间 |
| `-A` | Adjust the access and modification time stamps for the file by the specified value | 按指定值调整文件的访问时间和修改时间 |
| `-a` | Change the access time of the file | 仅修改文件的访问时间 |
| `-c` | Do not create the file if it does not exist | 文件不存在时不创建文件 |
| `-f` | Attempt to force the update, even if the file permissions do not currently permit it | 兼容性选项：即使当前文件权限不允许也不会强制更新（GNU Coreutils 与 macOS 当前实现为空操作） |
| `-h` | If the file is a symbolic link, change the times of the link itself rather than the file that the link points to | 如果文件是符号链接，则修改链接本身的时间，而不是链接所指向文件的时间 |
| `-m` | Change the modification time of the file | 仅修改文件的修改时间 |
| `-r` | Use the access and modifications times from the specified file instead of the current time of day | 使用指定文件的访问时间和修改时间，而不是当前时间 |
| `-t` | Change the access and modification times to the specified time instead of the current time of day | 将访问时间和修改时间设为指定时间，而不是当前时间 |
| `-A` argument | `[-][[hh]mm]SS` | `[-][[hh]mm]SS（时间格式）` |
| `-t` argument | `[[CC]YY]MMDDhhmm[.SS]` | `[[CC]YY]MMDDhhmm[.SS]（时间格式）` |

zh-TW is generated from these values with
`node scripts/generate-autocomplete-descriptions-zh-TW.mjs` (OpenCC s2twp), not
edited by hand. The committed zh-TW file reproduces the generator output.

## Semantic notes

- `-A` is a BSD/macOS option. GNU Coreutils has no `-A`; the Fig spec and the
  translation follow the BSD/macOS form. On macOS, `-A` also implies `-c`, so
  missing files are silently ignored. No platform detection was added.
- `-f` is a compatibility no-op in current implementations. The Fig description
  repeats the historical BSD/macOS man-page wording ("force the update"), but
  Apple's `file_cmds/touch/touch.c` currently handles `case 'f': /* No-op for
  compatibility. */`, and FreeBSD's current `usr.bin/touch/touch.c` does the
  same. GNU Coreutils 9.12 also ignores `-f`. The zh-CN text therefore says the
  option does not force an update and notes the no-op behavior. The macOS man
  page still describes the historical behavior, so the implementation source is
  the authoritative reference here. No platform detection was added.
- `-h` works on both. macOS makes `-h` imply `-c`; GNU does not create empty files
  with `-h`, but still needs `-c` to suppress warnings about missing files. The
  translation only describes changing the link's own timestamps.
- `-t` uses the same documented format. The two implementations express the
  two-digit year window differently, but both map 69-99 to 19xx and the rest to
  20xx.
- The Fig `touch` spec does not include GNU `-d`/`--time`, `--help`, or
  `--version`. They were not added because this round only fills missing
  descriptions and preserves the upstream spec structure.
- `touch` creates an empty file when the operand does not exist unless `-c` or
  `-h` applies; the descriptions above do not change that behavior.

## Other commands still missing

The existing overlay scope in `scripts/list-untranslated-autocomplete-descriptions.mjs`
is complete after `touch`: every covered command with a spec has 0 missing
descriptions. `ip`, `netstat`, and `ss` are listed there but have no bundled Fig
spec in `2.692.3`.

Preloaded common commands that are not in the overlay yet, with missing/total
description strings:

| Command | Missing | Total |
| --- | ---: | ---: |
| `zip` | 4 | 4 |
| `unzip` | 3 | 3 |
| `make` | 24 | 24 |
| `python` / `python3` | 23 / 23 | 23 / 23 |
| `node` | 13 | 13 |
| `vim` | 39 | 40 |
| `nano` | 1 | 1 |
| `sort` | 37 | 37 |
| `awk` | 23 | 24 |
| `sed` | 42 | 42 |

Across the 770 app-visible specs (excluding the filtered `aws`/`gcloud`/`az`
trees), 709 specs still contain at least one missing description: 47,173
occurrences, 38,485 unique English strings. The table above is the practical
next batch. The maintenance script intentionally scans only `COVERED_COMMANDS`;
expand that list one command at a time when starting a new round.

## License notices

The English description strings are derived from `@withfig/autocomplete`, and the
terminology review used `jaywcjlove/linux-command`. Both are MIT licensed:

```text
MIT License

Copyright (c) 2021 Hercules Labs Inc. (Fig)
Copyright © 2019 小弟调调™ (linux-command)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

`@withfig/autocomplete` package metadata declares `ISC`, while the package's
`LICENSE` file is the MIT text reproduced above. The upstream repository is
`https://github.com/withfig/autocomplete`; the linux-command repository is
`https://github.com/jaywcjlove/linux-command`.

# enex2notion

[![PyPI version](https://img.shields.io/pypi/v/enex2notion?label=version)](https://pypi.python.org/pypi/enex2notion)
[![Python Version](https://img.shields.io/pypi/pyversions/enex2notion.svg)](https://pypi.org/project/enex2notion/)
[![tests](https://github.com/vzhd1701/enex2notion/actions/workflows/test.yml/badge.svg)](https://github.com/vzhd1701/enex2notion/actions/workflows/test.yml)
[![codecov](https://codecov.io/gh/vzhd1701/enex2notion/branch/master/graph/badge.svg)](https://codecov.io/gh/vzhd1701/enex2notion)

Easy way to import [Evernote's](https://www.evernote.com/) `*.enex` files to [Notion.so](https://notion.so)

Notion's native Evernote importer doesn't do it for me, so I decided to write my own. Thanks to **Cobertos** and [md2notion](https://github.com/Cobertos/md2notion) for inspiration and **Jamie Alexandre** for [notion-py](https://github.com/jamalex/notion-py).

You can either use Evernote native export or try out my other tool, [evernote-backup](https://github.com/vzhd1701/evernote-backup), to export `*.enex` files from Evernote.

### What is preserved

- Embedded files and images are uploaded to Notion
  - nested images will appear after paragraph
- Text formatting (**bold**, _italic_, etc) and colors
- Tables are converted to the new format (no colspans though)
- Web Clips
  - as plain text or PDFs, see [below](#web-clips)
- Everything else basically

### What is lost

- Paragraph alignment
- Subscript and superscript formatting
- Custom fonts and font sizes
- Tasks
- Encrypted blocks
  - just decrypt them before export

## Installation

If you are not familiar with command line programs, take a look at these step-by-step guides:

### [Step-by-step guide for Windows](https://vzhd1701.notion.site/How-to-use-enex2notion-on-Windows-6fa980b489ab4414a5317e631e7f6bc6)

### [Step-by-step guide for macOS](https://vzhd1701.notion.site/How-to-use-enex2notion-on-macOS-a912dd63e3d14da886a413d3f83efb67)

### Using portable binary

[**Download the latest binary release**](https://github.com/vzhd1701/enex2notion/releases/latest) for your OS.

### With [Homebrew](https://brew.sh/) (Recommended for macOS)

```bash
$ brew install enex2notion
```

### With [**PIPX**](https://github.com/pipxproject/pipx) (Recommended for Linux & Windows)

```shell
$ pipx install enex2notion
```

### With PIP

```bash
$ pip install --user enex2notion
```

**Python 3.7 or later required.**

### From source

This project uses [poetry](https://python-poetry.org/) for dependency management and packaging. You will have to install it first. See [poetry official documentation](https://python-poetry.org/docs/) for instructions.

```shell
$ git clone https://github.com/vzhd1701/enex2notion.git
$ cd enex2notion/
$ poetry install --no-dev
$ poetry run enex2notion
```

## Usage

```shell
$ enex2notion --help
usage: enex2notion [-h] [--token TOKEN] [OPTION ...] FILE/DIR [FILE/DIR ...]

Uploads ENEX files to Notion

positional arguments:
  FILE/DIR                   ENEX files or directories to upload

optional arguments:
  -h, --help                 show this help message and exit
  --token TOKEN              Notion token, stored in token_v2 cookie for notion.so [NEEDED FOR UPLOAD]
  --root-page NAME           root page name for the imported notebooks, it will be created if it does not exist (default: "Evernote ENEX Import")
  --mode {DB,PAGE}           upload each ENEX as database (DB) or page with children (PAGE) (default: DB)
  --mode-webclips {TXT,PDF}  convert web clips to text (TXT) or pdf (PDF) before upload (default: TXT)
  --add-pdf-preview          include preview image with PDF webclips for gallery view thumbnail (works only with --mode-webclips=PDF)
  --add-meta                 include metadata (created, tags, etc) in notes, makes sense only with PAGE mode
  --tag TAG                  add custom tag to uploaded notes
  --condense-lines           condense text lines together into paragraphs to avoid making block per line
  --condense-lines-sparse    like --condense-lines but leaves gaps between paragraphs
  --done-file FILE           file for uploaded notes hashes to resume interrupted upload
  --log FILE                 file to store program log
  --verbose                  output debug information
  --version                  show program's version number and exit
```

### Input

You can pass single `*.enex` files or directories. The program will recursively scan directories for `*.enex` files.

### Token & dry run mode

The upload requires you to have a `token_v2` cookie for the Notion website. For information on how to get it, see [this article](https://vzhd1701.notion.site/Find-Your-Notion-Token-5f57951434c1414d84ac72f88226eede).

The program can run without `--token` provided though. It will not make any network requests without it. Executing a dry run with `--verbose` is an excellent way to check if your `*.enex` files are parsed correctly before uploading.

### Upload continuation

The upload will take some time since each note is uploaded block-by-block, so you'll probably need some way of resuming it. `--done-file` is precisely for that. All uploaded note hashes will be stored there, so the next time you start, the upload will continue from where you left off.

All uploaded notebooks will appear under the automatically created `Evernote ENEX Import` page. You can change that name with the `--root-page` option. The program will mark unfinished notes with `[UNFINISHED UPLOAD]` text in the title. After successful upload, the mark will be removed.

### Upload modes

The `--mode` option allows you to choose how to upload your notebooks: as databases or pages. `DB` mode is the default since Notion itself uses this mode when importing from Evernote. `PAGE` mode makes the tree feel like the original Evernote notebooks hierarchy.

Since `PAGE` mode does not benefit from having separate space for metadata, you can still preserve the note's original meta with the `--add-meta` option. It will attach a callout block with all meta info as a first block in each note [like this](https://imgur.com/a/lJTbprH).

### Web Clips

Due to Notion's limitations Evernote web clips cannot be uploaded as-is. `enex2notion` provides two modes with the `--mode-webclips` option:

- `TXT`, converting them to text, stripping all HTML formatting \[Default\]

  - similar to Evernote's "Simplify & Make Editable"

- `PDF`, converting them to PDF, keeping HTML formatting as close as possible

  - web clips are converted using [wkhtmltopdf](https://wkhtmltopdf.org/), see [this page](https://github.com/JazzCore/python-pdfkit/wiki/Installing-wkhtmltopdf) on how to install it

Since Notion's gallery view does not provide thumbnails for embedded PDFs, you have the `--add-pdf-preview` option to extract the first page of generated PDF as a preview for the web clip page.

### Banned file extensions

Notion prohibits uploading files with certain extensions. The list consists of extensions for executable binaries, supposedly to prevent spreading malware. `enex2notion` will automatically add a `bin` extension to those files to circumvent this limitation. List of banned extensions: `apk`, `app`, `com`, `ear`, `elf`, `exe`, `ipa`, `jar`, `js`, `xap`, `xbe`, `xex`, `xpi`.

### Misc

The `--condense-lines` option is helpful if you want to save up some space and make notes look more compact. [Example](https://imgur.com/a/sV0X8z7).

The `--condense-lines-sparse` does the same thing as `--condense-lines`, but leaves gaps between paragraphs. [Example](https://imgur.com/a/OBzeqn7).

The `--tag` option allows you to add a custom tag to all uploaded notes. It will add this tag to existing tags if the note already has any.

## Examples

### Checking notes before upload

```shell
$ enex2notion --verbose my_notebooks/
```

### Uploading notes from a single notebook

```shell
$ enex2notion --token <YOUR_TOKEN_HERE> "notebook.enex"
```

### Uploading with the option to continue later

```shell
$ enex2notion --token <YOUR_TOKEN_HERE> --done-file done.txt "notebook.enex"
```

## Getting help

If you found a bug or have a feature request, please [open a new issue](https://github.com/vzhd1701/enex2notion/issues/new/choose).

If you have a question about the program or have difficulty using it, you are welcome to [the discussions page](https://github.com/vzhd1701/enex2notion/discussions). You can also mail me directly, I'm always happy to help.

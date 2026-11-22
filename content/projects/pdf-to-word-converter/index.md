+++
title = "Converting Huge PDFs Without Uploading Them Anywhere"
date = 2026-02-24
lastmod = 2026-11-22
draft = false
description = "A small local web app that turns PDFs of any size into editable Word documents on your own computer, built because online converters crashed on big files or wanted my documents."
summary = "Online PDF converters either choked on my large files or wanted me to upload them to a stranger's server. So I built a drag-and-drop converter that runs entirely on my laptop. It turned a 2,000-page PDF into a Word document in four seconds."

featureImage = "feature.png"
featured = true

tags = ["Python", "Flask", "PDF", "Privacy", "Tools"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
showDateUpdated = true
+++

{{< lead >}}
I needed the text out of some very large PDFs for another project. The free online converters crashed on them, the paid ones wanted a subscription, and all of them wanted my files on their servers. So I built my own converter that runs entirely on my laptop.
{{< /lead >}}

The code is [on GitHub](https://github.com/suni639/pdf-to-text).

## The problem

Getting text out of a PDF sounds trivial until the PDF has hundreds of pages. The free online tools I tried either timed out or failed partway through large files. The ones that coped charged a monthly fee for what I needed once or twice.

Then there's the question of trust. Uploading a document to a converter means handing a copy to a company you know nothing about, to be stored for an unknown time on servers you can't see. For a public report that might not matter. For contracts, statements or anything with personal details, it does.

What I wanted was simple: drop in a PDF, get an editable Word document back, and have nothing leave my machine.

---

## What I built

A small web app that runs on my own computer. Double-click a desktop shortcut and it starts a local server and opens a page in the browser. You drag a PDF onto the page, and a few seconds later a Word document downloads.

{{< figure src="feature.png" alt="The converter's page: a card titled PDF to Word Converter with a dashed drop zone reading Drag and drop your PDF here, a Browse Files button and the footer Simple, Fast, Local" caption="The whole interface. Drop a PDF on the box and a Word file comes back." >}}

The page looks like any website, but its address is `localhost`: your own computer. The file goes from your browser to a program on the same machine and back again.

---

## How it works

{{< mermaid >}}
graph LR
    A[Drop a PDF<br>on the page] --> B[Local server<br>saves a temp copy]
    B --> C[Read the text<br>page by page]
    C --> D[Write a<br>Word document]
    D --> E((Download,<br>temp files deleted))
{{< /mermaid >}}

1. **Upload, locally.** The page sends the PDF to a small Python server (built with Flask) running on the same computer, with a progress bar while it goes. It accepts files up to 1 GB.
2. **Read the text.** PyMuPDF, a fast open-source PDF library, opens the file and pulls out the text one page at a time.
3. **Write Word.** python-docx builds a Word document, one section per PDF page with a page break between them.
4. **Download and tidy up.** The Word file downloads with the same name as the PDF, and the temporary copies are deleted.

A small PowerShell script puts a "PDF Converter" icon on the desktop, so starting it is a double-click rather than a command.

---

## How well it works

When I came back to the project for this write-up, I tested it with some generated PDFs.

| Test | Result |
|---|---|
| 3-page text PDF | Converted instantly |
| **2,000-page PDF, 644,000 words** | **Converted in about 4 seconds** |
| A file that isn't a PDF | Rejected with a clear message |
| A scanned page (text only in an image) | No text, now with a warning |

Speed was never the problem the online tools made it look like. On a laptop, a library like PyMuPDF reads thousands of pages in seconds. The online converters were limited by uploads, queues and free-tier caps, not by the work itself.

It's worth being clear about what it doesn't do:

- **Text only.** Images, tables and formatting don't come across. You get the words, in reading order, page by page. That was exactly what I needed, but it isn't a layout-perfect conversion.
- **No scanned documents.** A scanned PDF is a picture of text, not text. Reading it needs **OCR** (optical character recognition), which this tool doesn't do.
- **No locked files.** Password-protected PDFs fail.

---

## What I fixed on the way

Testing it again turned up three things that didn't match the promise of "nothing leaves your machine".

- **Leftover copies.** On Windows, the converted Word file was supposed to be deleted after download, but the app tried to delete it while it was still being sent, and Windows refused. Every conversion left a copy in the temporary folder. It now sends the document from memory and deletes both temporary files every time, even if a conversion fails.
- **Silent empty files.** A scanned PDF produced a blank Word document with no explanation. It now says that no text was found and that the PDF may be scanned.
- **A call to Google.** The page loaded its font from Google Fonts. Your PDF never went there, but the page did contact Google each time it opened. It now uses the computer's own fonts and makes no outside requests at all.

None of these were dramatic, but a privacy tool should keep its promise in the small print too.

---

## Try it yourself

{{< alert "circle-info" >}}
**You'll need:** Windows with a recent version of Python installed. It takes about five minutes to set up.
{{< /alert >}}

1. **Download the code** from [GitHub](https://github.com/suni639/pdf-to-text) and put the folder at `C:\Projects\dev\pdf-to-text`, or update the paths in the shortcut script.
2. **Set it up** in a terminal in that folder: `python -m venv venv`, then `venv\Scripts\activate`, then `pip install -r requirements.txt`.
3. **Add the shortcut** by right-clicking `create_shortcut.ps1` and choosing *Run with PowerShell*.
4. **Double-click "PDF Converter"** on your desktop and drop in a PDF.

---

## Key takeaways

1. **Small tools can solve real annoyances.** A few hundred lines of code replaced a subscription I'd have used twice.
2. **Local beats uploaded for private documents.** If a job can run on your own machine, there's no reason to send your files anywhere.
3. **The bottleneck often isn't the computing.** Four seconds for 2,000 pages. The online tools were slow for business reasons, not technical ones.
4. **Check the small print of your own promises.** "Local" only counts if nothing is left behind and nothing calls out.

Drop in a PDF, get the words back, and nothing leaves the laptop.

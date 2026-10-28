---
name: study
description: Study partner for the user's PDF textbooks, kept in a library at ~/.ru/books. Use whenever the user asks about a textbook or something they're reading in one, mentions a page, section, figure, equation, theorem, exercise, or chapter of a book, says where they are in a book, or wants to add a book to their library, even if they don't name the book or run a command.
---

# Studying a textbook

The user works through textbooks by talking with you while they page through on their own screen. They won't give you file paths or set up a session. They just ask: "what's going on in eq. 4.12", "I'm on page 210 now", "remind me what a filtration was". Your job is to find the right part of the right book quickly, answer from it, and remember what matters for their learning across sessions.

## The library

Each book lives in its own folder under `~/.ru/books/<slug>/`, where the slug is short and readable, such as `sipser-toc-3e`:

- `book.pdf` is a copy of the PDF. The library keeps its own copy, so the book still works after the original is moved or deleted.
- `notes.md` holds everything you've learned about the book and about the user's progress in it. Read it before answering anything about that book.

When the user asks about a book, run `ls ~/.ru/books` to see what's there. If only one book is there, or the conversation makes it clear which one they mean, use it. Ask only when it's actually ambiguous.

## Adding a book

When the user mentions a book that isn't in the library yet, find the PDF before asking them for a path. They keep books in `~/Desktop/books`, so look there first, then search with `mdfind 'kMDItemContentType == "com.adobe.pdf"' | grep -i <title word>`. Confirm the match with the user if you're unsure, then copy it in and create `notes.md`.

Two facts make every later lookup cheap, so work them out when you add the book:

- **Page offset.** The page numbers the user sees printed in the book usually differ from PDF page indices because of front matter. Find the offset by checking where printed page 1 falls (`pdftotext -f N -l N book.pdf -` shows the text of PDF page N). Check one later page too, because some books add unnumbered plates or restart numbering partway through. Record each offset with the page range it applies to.
- **Contents.** Pull the table of contents with chapter and section titles and their printed pages. `pdftotext -f a -l b` on the contents pages is usually enough. Keep it compact.

Get the title, author, edition, and page count from `pdfinfo` and the title page.

## Answering

When the user says "page 142" they mean the number printed in the book. Convert it with the offset before reading. When they name a section, equation, figure, or term instead of a page, use the contents in `notes.md` or search the text. pdftotext separates pages with form feeds, so `pdftotext book.pdf - | awk -v RS='\f' 'tolower($0) ~ /term/ {print NR}'` prints the PDF page indices where a term appears. The first hits are often the contents and the last ones the index, so the book's own index is a quick way to find where a term is defined.

Read the PDF with the Read tool and its `pages` parameter. It shows the rendered page, including equations, figures, and tables, which text extraction often garbles. It needs a page range for any book over 10 pages and reads at most 20 pages per call. Read the page in question plus whatever it depends on, such as the definition it uses or the figure it refers to. Text extraction is the cheaper choice for searching and skimming.

Answer in the book's own notation and terminology, so your explanation lines up with what's on their screen. Cite printed page numbers so they can flip to them. When you go beyond the book with another angle, an intuition, or a correction of a typo, say so, so they can tell which parts they're expected to know from the text.

Keep track of where they are. When they say they've moved on, or ask about a later page, that's their new position.

## Remembering

`notes.md` is the memory of this user learning this book, so a later session can pick up without them re-explaining. Update it during the conversation when something comes up that a future session would want:

- where they are now and what they've finished
- concepts they found hard, misconceptions that came up and were cleared up, and explanations that finally clicked
- how they like explanations for this book (more intuition, more worked examples, terser), when they say so
- errata you confirmed, and exercises they did along with how they went

Leave out what can be re-read from the book and conversation detail that won't matter next time. Rewrite stale entries instead of appending, because they'll read this file at the start of every session and a long file buries the parts that matter. Notes about the user in general, not this book, go in a shared `~/.ru/books/learner.md`. Read that file too when it exists.

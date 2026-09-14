# NoteSeek — a personal search engine for your notes

## What it does
Indexes a folder of `.txt` files using an inverted index (word -> which files
it appears in, and how often), then lets you search across them with results
ranked by relevance and shown with context snippets.

## How to run it
1. Install Streamlit if you don't have it: `pip install streamlit`
2. From this folder, run: `streamlit run noteseek.py`
3. In the sidebar, point it at the `sample_notes` folder (or your own folder
   of .txt files) and click "Build / Rebuild Index"
4. Type a search query and see ranked results with snippets

## Try it with your own notes
Just export or save any of your notes as plain .txt files into a folder and
point NoteSeek at it. It works with lecture notes, journal entries, project
ideas, anything in plain text.

## What to say about it on a resume / in an interview
- You designed and implemented an inverted index (the core data structure
  behind real search engines) from scratch using plain Python dictionaries
- You implemented a basic relevance-ranking algorithm (frequency-based
  scoring across matched query terms)
- You built a snippet-extraction feature to show *why* a result matched,
  which is what real search engines do too
- Possible next steps to mention as "future work": stemming (treating
  "run"/"running" as the same word), TF-IDF ranking instead of raw
  frequency, or supporting PDF/docx files in addition to .txt

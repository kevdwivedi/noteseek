"""
NoteSeek — a personal search engine for your own notes.

How it works (the core CS idea):
1. INDEXING: read every .txt file in a folder, break each one into words,
   and build an "inverted index" — a dictionary that maps each word to
   the list of files it appears in (and how many times). This is the same
   basic idea real search engines use, just at a tiny personal scale.
2. SEARCHING: break the user's query into words, look each one up in the
   index, and add up how often those words appear in each file. Files
   with higher total word-match counts are ranked higher.
3. SNIPPETS: for the top files, find where a query word actually appears
   in the original text and show a bit of surrounding context, so you can
   see *why* a result matched.
"""

import os
import re
import streamlit as st

st.set_page_config(page_title="NoteSeek", page_icon="🔎", layout="wide")

# A small list of very common words that aren't useful to search on.
# Filtering these out keeps the index focused on meaningful words.
STOPWORDS = {
    "the", "a", "an", "and", "or", "but", "is", "are", "was", "were", "be",
    "been", "to", "of", "in", "on", "at", "for", "with", "as", "by", "it",
    "this", "that", "these", "those", "i", "you", "he", "she", "they", "we",
    "my", "your", "his", "her", "their", "our", "from", "not", "so", "if",
}


def tokenize(text):
    """Break text into a list of lowercase words, stripping punctuation."""
    return re.findall(r"[a-zA-Z']+", text.lower())


def build_index(folder_path):
    """
    Build an inverted index from every .txt file in folder_path.

    Returns:
        index: dict mapping word -> dict mapping filename -> count
               e.g. {"python": {"notes1.txt": 3, "notes2.txt": 1}}
        documents: dict mapping filename -> full raw text (kept so we can
                   pull snippets later)
    """
    index = {}
    documents = {}

    for filename in os.listdir(folder_path):
        if not filename.endswith(".txt"):
            continue

        filepath = os.path.join(folder_path, filename)
        with open(filepath, "r", encoding="utf-8", errors="ignore") as f:
            text = f.read()

        documents[filename] = text
        words = tokenize(text)

        for word in words:
            if word in STOPWORDS:
                continue
            if word not in index:
                index[word] = {}
            index[word][filename] = index[word].get(filename, 0) + 1

    return index, documents


def search(query, index, documents):
    """
    Search the index for a query and return ranked results.

    Returns a list of (filename, score, matched_words) tuples,
    sorted by score (highest first).
    """
    query_words = [w for w in tokenize(query) if w not in STOPWORDS]
    if not query_words:
        return []

    scores = {}       # filename -> total relevance score
    matched_words = {}  # filename -> set of query words found in it

    for word in query_words:
        if word not in index:
            continue
        for filename, count in index[word].items():
            scores[filename] = scores.get(filename, 0) + count
            matched_words.setdefault(filename, set()).add(word)

    results = [
        (filename, score, matched_words[filename])
        for filename, score in scores.items()
    ]
    # Rank by score first, then by how many distinct query words matched
    results.sort(key=lambda r: (r[1], len(r[2])), reverse=True)
    return results


def make_snippet(text, query_words, context_chars=80):
    """Find the first occurrence of any query word and show text around it."""
    lower_text = text.lower()
    for word in query_words:
        idx = lower_text.find(word)
        if idx != -1:
            start = max(0, idx - context_chars // 2)
            end = min(len(text), idx + len(word) + context_chars // 2)
            snippet = text[start:end].replace("\n", " ").strip()
            return ("…" if start > 0 else "") + snippet + ("…" if end < len(text) else "")
    return text[:context_chars].replace("\n", " ").strip() + "…"


# ---------------- Streamlit UI ----------------

st.title("🔎 NoteSeek")
st.write("A personal search engine for your own notes — built on an inverted index, just like a real search engine.")

st.sidebar.header("Setup")
folder_path = st.sidebar.text_input("Folder path containing your .txt notes:", "sample_notes")

if "index" not in st.session_state:
    st.session_state.index = None
    st.session_state.documents = None

if st.sidebar.button("Build / Rebuild Index"):
    if os.path.isdir(folder_path):
        with st.spinner("Indexing your notes..."):
            index, documents = build_index(folder_path)
            st.session_state.index = index
            st.session_state.documents = documents
        st.sidebar.success(f"Indexed {len(documents)} file(s), {len(index)} unique word(s).")
    else:
        st.sidebar.error("That folder doesn't exist.")

if st.session_state.index is not None:
    query = st.text_input("Search your notes:", "")

    if query:
        results = search(query, st.session_state.index, st.session_state.documents)

        if not results:
            st.warning("No matches found.")
        else:
            st.subheader(f"Results for \"{query}\" ({len(results)} file(s) matched)")
            for filename, score, matched in results:
                text = st.session_state.documents[filename]
                snippet = make_snippet(text, matched)
                with st.container(border=True):
                    st.markdown(f"**{filename}**  ·  relevance score: {score}")
                    st.caption(f"Matched words: {', '.join(sorted(matched))}")
                    st.write(snippet)
else:
    st.info("Point the sidebar at a folder of .txt files and click 'Build / Rebuild Index' to get started.")

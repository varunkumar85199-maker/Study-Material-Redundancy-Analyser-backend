# StudySync Backend v4

> Smart PDF analysis, topic merging, flashcards, quiz, aur bahut kuch — ek hi file mein.

---

## Quick Start

```bash
# 1. Libraries install karo
pip install flask flask-cors pdfplumber reportlab scikit-learn python-docx

# 2. Server chalao
python backend_v4.py

# 3. Browser mein kholo
http://localhost:5000/health
```

---

## Folder Structure

```
C:\StudySync\
├── backend_v4.py       ← Sirf yahi ek file chahiye
├── index_v4.html       ← Frontend (isi folder mein rakho)
├── uploads/            ← Auto-create hoga
└── outputs/            ← Auto-create hoga
```

---

## Requirements

| Library | Kaam | Install |
|---------|------|---------|
| flask | Web server | `pip install flask` |
| flask-cors | Frontend connect karne ke liye | `pip install flask-cors` |
| pdfplumber | PDF se text nikalna | `pip install pdfplumber` |
| reportlab | PDF generate karna | `pip install reportlab` |
| scikit-learn | Duplicate detection, TF-IDF | `pip install scikit-learn` |
| python-docx | Word file read/write | `pip install python-docx` |

**Ek saath install karo:**
```bash
pip install flask flask-cors pdfplumber reportlab scikit-learn python-docx
```

---

## API Endpoints — Poori List

### Server

| Method | Endpoint | Kaam |
|--------|----------|------|
| `GET` | `/health` | Server online check, sab libraries ka status |

**Response:**
```json
{
  "status": "online",
  "version": "v4",
  "pdfplumber": true,
  "sklearn": true,
  "reportlab": true,
  "python_docx": true,
  "time": "18 Apr 2026 04:30 PM"
}
```

---

### File Management

| Method | Endpoint | Kaam |
|--------|----------|------|
| `POST` | `/upload` | Files upload karo |
| `GET` | `/files` | Uploaded files ki list |
| `DELETE` | `/files/<filename>` | File delete karo |
| `POST` | `/clear` | Saare uploads + outputs delete karo |

**Upload example:**
```js
const fd = new FormData();
fd.append('files', file1);
fd.append('files', file2);
fetch('http://localhost:5000/upload', { method: 'POST', body: fd });
```

**Response:**
```json
{
  "uploaded": [
    { "name": "Physics.pdf", "size_kb": 420.5 }
  ],
  "count": 1
}
```

---

### Analysis

| Method | Endpoint | Body | Kaam |
|--------|----------|------|------|
| `POST` | `/analyze` | `{ "files": ["a.pdf"] }` | Topic-wise analysis |
| `POST` | `/merge` | `{ "files": [...], "format": "pdf" }` | Type 1 notes banao |
| `POST` | `/smart-merge` | `{ "files": [...], "format": "pdf" }` | Type 2 smart merge |
| `POST` | `/summary` | `{ "files": [...], "sentences": 5 }` | Auto summary |
| `POST` | `/keywords` | `{ "files": [...], "top": 15 }` | Top keywords |
| `POST` | `/important-lines` | `{ "files": [...], "top": 20 }` | Important sentences |
| `POST` | `/search` | `{ "files": [...], "query": "Newton" }` | Text search |
| `POST` | `/flashcards` | `{ "files": [...], "count": 10 }` | Q&A flashcards |
| `POST` | `/quiz` | `{ "files": [...], "count": 5 }` | MCQ quiz |
| `POST` | `/stats` | `{ "files": [...] }` | File statistics |
| `POST` | `/compare` | `{ "file1": "a.pdf", "file2": "b.pdf" }` | 2 files compare |

---

### Download

| Method | Endpoint | Kaam |
|--------|----------|------|
| `POST` | `/export-docx` | `{ "files": [...] }` — Word doc banao |
| `GET` | `/outputs` | Output files ki list |
| `GET` | `/download/<filename>` | File download karo |

---

## Do Types ke Notes

### Type 1 — Topic-Wise Notes (`/merge`)
- PDF headings detect karta hai
- Same topic ki lines sab files se group karta hai
- Duplicate sentences hatata hai (TF-IDF)
- Har topic ka overlap % calculate karta hai
- Output: `topic_wise_notes.pdf` ya `topic_wise_notes.txt`

### Type 2 — Smart Merged Notes (`/smart-merge`)
- Saari PDFs ki har line ek saath leta hai
- Junk lines automatically remove karta hai:
  - Page numbers, blank lines, URLs
  - 4 words se chhoti lines
  - Copyright lines, figure labels
- Duplicate / same meaning lines TF-IDF se hatata hai
- Koi topic grouping nahi — clean bullet points
- Output: `smart_merged_notes.pdf` ya `smart_merged_notes.txt`

---

## Format Options

`format` parameter mein yeh dono values de sakte ho:

| Value | Output |
|-------|--------|
| `"pdf"` | Formatted PDF with headings, table, colors |
| `"txt"` | Plain text — WhatsApp/email pe share karo |

---

## Frontend ke liye Quick Reference

```js
const API = 'http://localhost:5000';

// Upload
const fd = new FormData();
files.forEach(f => fd.append('files', f));
await fetch(`${API}/upload`, { method: 'POST', body: fd });

// Analyze
await fetch(`${API}/analyze`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ files: ['Physics.pdf', 'Notes.docx'] })
});

// Type 1 merge
await fetch(`${API}/merge`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ files: ['Physics.pdf'], format: 'pdf' })
});

// Type 2 smart merge
await fetch(`${API}/smart-merge`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ files: ['Physics.pdf'], format: 'pdf' })
});

// Download
window.open(`${API}/download/topic_wise_notes.pdf`);
```

---

## Common Errors aur Fix

| Error | Reason | Fix |
|-------|--------|-----|
| `ModuleNotFoundError: flask` | Library install nahi | `pip install flask flask-cors` |
| `ModuleNotFoundError: pdfplumber` | PDF library nahi | `pip install pdfplumber` |
| `PermissionError: index.html` | OneDrive folder mein hai | Files `C:\StudySync\` mein rakho |
| `Address already in use` | Port 5000 busy hai | `taskkill /f /im python.exe` phir restart |
| `python not recognized` | PATH set nahi | `py backend_v4.py` try karo |
| `CORS error` | Frontend alag port pe hai | CORS pehle se enabled hai, browser console check karo |

---

## Output Files

Server chalne ke baad yeh files `outputs/` folder mein banti hain:

```
outputs/
├── topic_wise_notes.pdf      ← Type 1 PDF
├── topic_wise_notes.txt      ← Type 1 TXT
├── smart_merged_notes.pdf    ← Type 2 PDF
├── smart_merged_notes.txt    ← Type 2 TXT
└── studysync_notes.docx      ← Word document (dono types)
```

---

## Server Start hone pe yeh dikhega

```
=======================================================
  StudySync Backend  v4
  URL      : http://localhost:5000
  Health   : http://localhost:5000/health
  pdfplumber : ✓ OK
  sklearn    : ✓ OK
  reportlab  : ✓ OK
  python-docx: ✓ OK
=======================================================
 * Running on http://127.0.0.1:5000
 * Debugger is active!
```

**Yeh window band mat karna jab tak kaam kar rahe ho!**

---

## Notes

- Supported file types: `.pdf`, `.docx`, `.txt`
- CORS enabled hai — kisi bhi frontend se connect ho sakta hai
- Agar koi library missing ho to woh feature skip ho jaata hai, server crash nahi karta
- `uploads/` aur `outputs/` folders automatically ban jaate hain

---

*StudySync v4 — Made for students, by students* 🎓

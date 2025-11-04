# 🧠 AI-Based Automotive Test Procedure Generator

## 📘 Problem Statement

In automotive validation, requirement documents are lengthy, complex, and often reference multiple technical standards or related documents.  
Manually converting these requirements into **detailed, structured test procedures** is time-consuming, error-prone, and inconsistent across engineers.

This tool automates that process using **AI-driven text understanding** — generating **structured Excel test cases** directly from a requirements document (PDF, DOCX, XLSX, or TXT).  
It ensures consistency in **naming, step sequence, action/expected result formatting**, and supports contextual understanding via **RAG (Retrieval-Augmented Generation)**.

---

## 🧩 Approach

### 🔹 Step 1: Requirement Document Ingestion
- Input requirement documents (main + related references) are read from the `requirements_docs/` folder.
- Supported formats: `.pdf`, `.docx`, `.xlsx`, `.txt`
- Each document is **text-extracted**, cleaned, and **chunked** into overlapping sections for contextual embeddings.

### 🔹 Step 2: RAG Index Construction
- Uses **SentenceTransformer** (`all-MiniLM-L6-v2`) to embed each chunk.
- Stores embeddings in a **FAISS** index for efficient semantic retrieval.
- During generation, the tool retrieves the top contextually relevant chunks for the given requirement.

### 🔹 Step 3: AI Test Case Generation
- Uses **Mistral’s API (e.g., `mistral-large-latest`)** for natural language understanding.
- The prompt guides Mistral to generate:
  - Structured JSON output with keys: `test_name`, `test_description`, `steps`
  - Each step includes `step_name`, `action`, and `expected_result`
- The generation process runs **chunked** to handle long documents robustly.
- If partial JSON or incomplete response is returned, the script:
  - Saves failed chunks to `failed_chunk_X.json`
  - Re-requests Mistral to repair or complete the JSON automatically

### 🔹 Step 4: Output Formatting
- Parses AI responses safely (via custom `safe_json_parse`)
- Merges results from all chunks
- Converts structured JSON into a final Excel sheet:
  ```
  | Test Name | Test Description | Step Name | Action Description | Expected Results |
  ```
- Ensures:
  - Numbered test names (e.g., `001_...`)
  - Step names reset per test (`Step 1`, `Step 2`, …)
  - No missing expected results

---

## ⚙️ Tech Stack & Libraries

| Library | Purpose | Why It’s Used |
|----------|----------|---------------|
| **PyPDF2** | PDF text extraction | Handles automotive requirement PDFs with embedded text |
| **docx**, **openpyxl** | Reading `.docx` and `.xlsx` files | Many requirements are shared as Word or Excel documents |
| **SentenceTransformers** | Sentence embeddings | Enables semantic chunking and similarity-based RAG |
| **FAISS** | Vector similarity search | Efficient retrieval of most relevant requirement chunks |
| **Requests** | Mistral API calls | Clean, lightweight HTTP requests |
| **json / re** | Parsing AI outputs | Robust JSON repair and cleanup |
| **Pandas** | Writing Excel output | Generates readable structured test procedure Excel |
| **Torch** | Backend for SentenceTransformer | Utilizes GPU acceleration if available |

---

## 🧠 Why RAG (Retrieval-Augmented Generation)?

Automotive requirement documents are often:
- Spread across multiple files (e.g., functional, HMI, CAN signal specs)
- Contain cross-references (e.g., “as defined in ISO15118”)
- Updated incrementally

RAG ensures that the model:
- Retrieves **relevant background context** dynamically  
- Generates **accurate and consistent test cases** even if input doc lacks all details

---

## 🚀 How to Run

### **1️⃣ Environment Setup**
```bash
git clone https://github.com/<your-username>/TestProcedureGenerator.git
cd TestProcedureGenerator
python -m venv venv
venv\Scripts\activate  # (Windows)
# or
source venv/bin/activate  # (Linux/Mac)
pip install -r requirements.txt
```

### **2️⃣ Folder Structure**
```
TestProcedureGenerator/
│
├── requirements_docs/
│   ├── ISO15118_requirements.pdf
│   ├── PlugNCharge_DesignSpec.docx
│   └── ...
│
├── PlugNCharge_requirements_doc.xlsx
├── main_notebook.ipynb
├── main.py
├── output/
│   └── generated_tests.xlsx
└── failed_chunk_X.json
```

### **3️⃣ Run via Command Line**
```bash
python main.py   --index_dir ./requirements_docs   --query_doc ./PlugNCharge_requirements_doc.xlsx   --mistral_api_key <YOUR_MISTRAL_API_KEY>   --out_excel ./output/generated_tests.xlsx
```

### **4️⃣ Run via Notebook**
- Open `main_notebook.ipynb`
- Execute all cells sequentially
- Failed chunks will be automatically retried and logged

---

## 🧾 Output Example

| Test Name | Test Description | Step Name | Action Description | Expected Results |
|------------|------------------|------------|--------------------|------------------|
| 001_To Check Plug & Charge Activation | To Check Plug & Charge Activation | Step 1 | Set below CAN signals… | Plug & Charge should activate |
| 002_To Verify HMI Display | To Verify HMI Display | Step 1 | Open Plug & Charge HMI page | HMI displays expected fields |

---

## 🧰 Troubleshooting

| Issue | Likely Cause | Fix |
|--------|--------------|-----|
| ❌ 400/401 Error | Invalid or expired Mistral API key | Regenerate API key |
| ❌ JSON parsing failed | Model returned incomplete JSON | Check `failed_chunk_X.json` |
| ⚠️ Long runtime | Large doc or CPU embedding | Use GPU or smaller embedding model |
| 🚫 Missing output Excel | File path issue | Ensure correct `--out_excel` path |

---

## 🧑‍💻 Author & Credits

Developed by **Kanishk Mishra**  
AI Intern at **RNTBCI (Renault Nissan Technology & Business Centre India)**  
Focus: *AI for Automotive Test Automation & Validation*

---

## 📜 License

Licensed under the **MIT License** — free to use, modify, and distribute with attribution.

---

### ✅ Summary

This project demonstrates how **AI + RAG + prompt engineering** can automate traditionally manual engineering processes — transforming **requirement analysis** into **automated, structured test case generation**.

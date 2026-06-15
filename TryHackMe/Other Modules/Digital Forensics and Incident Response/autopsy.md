# Autopsy

## Overview

Autopsy is an open-source digital forensics platform used by law enforcement, corporate investigators, and security professionals worldwide. It provides a graphical interface for analysing disk images, mobile devices, and logical file systems. This room covers the Autopsy workflow — from case creation and data source ingestion through ingest modules, the user interface, and reporting — along with the visualization tools available for timeline analysis.

---

## Topics Covered

- What Autopsy is and what it analyses
- Basic workflow: case creation, data source selection, ingest modules
- Supported disk image formats
- User interface: Tree Viewer, Result Viewer, Contents Viewer, Keyword Search
- Data Sources Summary
- Report generation
- Visualization tools: Timeline, Images/Videos, Communications

---

## Key Concepts

### What is Autopsy?

Autopsy is a free, open-source forensics platform originally developed for the Sleuth Kit toolkit. It provides a graphical interface for forensic analysis of:
- Hard drive disk images
- Mobile devices
- USB drives and removable media
- Logical file collections

It supports plug-in ingest modules that automate extraction of specific artifacts — keyword hits, deleted files, browser history, email, metadata, and more.

---

### Basic Workflow

```
1. Create or open a case
2. Add a data source (disk image, logical files, etc.)
3. Configure ingest modules to run
4. Review extracted artifacts in the Tree/Result/Contents viewers
5. Generate a report
```

---

### Case Creation

When creating a new case:
- **Case Name** — identifies the investigation
- **Base Directory** — root folder for all case files
- **Case Type** — Single-User (local) or Multi-User (server, multiple analysts)

Autopsy case files use the `.aut` extension.

---

### Data Sources

Autopsy supports multiple input formats:

**Disk Image formats:**
| Format | Extensions |
|--------|-----------|
| Raw Single | `.img`, `.dd`, `.raw`, `.bin` |
| Raw Split | `.001`, `.002`, `.aa`, `.ab` |
| EnCase | `.e01`, `.e02` |
| Virtual Machine | `.vmdk`, `.vhd` |

For split images (E01, E02, E03...) — point Autopsy to the first file only; it handles the rest automatically.

Other data source options include: local disk, logical files, unallocated space image, and Autopsy logical imager results.

---

### Ingest Modules

Ingest Modules are Autopsy plug-ins that automatically extract specific artifact types from the data source. They can be configured:
- **While adding the data source** — selected during the import wizard
- **After adding the data source** — right-click the data source → "Run Ingest Modules"

By default, modules run on all files, directories, and unallocated space.

Results from ingest modules populate the **Results** node in the Tree Viewer. Progress is shown in the Status Area (bottom right).

---

## User Interface

### Five Primary Areas

#### 1. Tree Viewer (left pane)

Five top-level nodes:
- **Data Sources** — file system organised like Windows Explorer
- **Views** — files organised by type, MIME type, file size
- **Results** — output from ingest modules
- **Tags** — files and results manually or automatically tagged
- **Reports** — generated reports

#### 2. Result Viewer (top right)

Displays additional information about the item selected in the Tree Viewer. Shows files and metadata in a tabular format.

#### 3. Contents Viewer (bottom right)

Displays detailed content of items selected in the Result Viewer — file hex view, text view, metadata, and application-specific views.

**Column indicators in the Result Viewer:**
| Column | Meaning |
|--------|---------|
| S (Score) | Red exclamation = notable; yellow triangle = suspicious |
| C (Comment) | Yellow page = comment exists for this file |
| O (Occurrence) | How many times this file has been seen in previous cases (requires Central Repository) |

#### 4. Keyword Search

Top right — allows ad-hoc keyword searches across all indexed content during or after ingestion.

#### 5. Status Area

Bottom right — shows ingest module progress bar and percentage. Click for detailed module status.

---

### Data Sources Summary

Provides a high-level summary across nine categories before deep investigation — useful for quickly understanding the scale and nature of the data. Available via right-clicking the data source in the Tree Viewer.

---

### Report Generation

Autopsy can export investigation findings as:
- HTML report
- Excel spreadsheet
- Tab-delimited text
- Body file (for timeline tools)
- STIX (threat intelligence format)

**Generate Report:** File menu → Generate Report → select format and scope.

Reports contain all Result Viewer data and can be used for continuing analysis without needing Autopsy running — useful on low-resource systems where the tool may be slow.

---

## Visualization Tools

### Timeline

A three-panel view for chronological event analysis:

| Panel | Content |
|-------|---------|
| Filters | Narrow events by type, date range, and data source |
| Events | Chronological display of file system and artifact events |
| Files/Contents | Additional detail about selected events |

**Three view modes:**
- **Counts** — bar chart showing event volume over time
- **Details** — detailed event listing, collapsed/clustered to avoid UI overload
- **List** — tabular event view

Timeline is especially useful for identifying when an attacker first appeared on the system and reconstructing the sequence of their actions.

### Images/Videos

Displays recovered image and video files in a gallery view — useful for media-related investigations.

### Communications

Visualises extracted communication data (email, messages, call records) as a graph showing relationships between participants.

---

## Important Terminology

| Term | Meaning |
|------|---------|
| Ingest Module | Autopsy plug-in that automatically extracts a specific artifact type |
| Tree Viewer | Left-pane navigation showing the data source file structure and results |
| Result Viewer | Tabular display of items matching a selection in the Tree Viewer |
| Contents Viewer | Detailed view of a selected file's content, metadata, and hex data |
| `.aut` | Autopsy case file extension |
| Logical Files | A data source type importing files from a folder rather than a disk image |
| Unallocated space | Storage space not currently assigned to any file — recoverable deleted data may be here |
| Score column | Visual indicator of whether a file was flagged as notable or suspicious |
| Central Repository | Optional shared database enabling cross-case artifact correlation |

---

## Workflow / Process

```
Launch Autopsy → Create new case or open existing .aut file
        |
        v
Add data source (disk image, logical files, etc.)
        |
        v
Configure ingest modules for the investigation type
        |
        v
Wait for ingestion and module processing to complete
        |
        v
Review Data Sources Summary for high-level overview
        |
        v
Investigate via Tree Viewer:
  - Data Sources: browse the file system
  - Views: filter by file type, size, MIME
  - Results: review ingest module findings (deleted files, browser history, etc.)
        |
        v
Use Keyword Search for targeted artifact hunting
        |
        v
Use Timeline visualization for chronological event analysis
        |
        v
Tag items of interest
        |
        v
Generate Report for documentation and sharing
```

---

## Real-World Relevance

- Autopsy is used by law enforcement agencies worldwide for evidence processing in criminal investigations
- The Timeline view is critical during incident response for establishing when a compromise began and what the attacker did in sequence
- Ingest modules automate artifact extraction that would take hours manually — keyword search, deleted file recovery, extension mismatch detection, browser history, email extraction
- Reports provide legally defensible documentation of findings that can be presented to management, legal teams, or law enforcement
- Autopsy's open-source nature makes it accessible for organisations without budget for commercial forensics platforms

---

## Key Learnings

- Autopsy analyses disk images, mobile devices, and logical file collections
- Basic workflow: create case → add data source → configure ingest modules → review results → report
- Tree Viewer organises data into Data Sources, Views, Results, Tags, and Reports
- Ingest modules run automatically to extract artifacts — results appear in the Results node
- Timeline visualization provides three views: Counts (bar chart), Details (clustered), List (tabular)
- Reports can be generated in multiple formats for documentation and sharing

---

## Conclusion

Autopsy provides a comprehensive graphical environment for forensic analysis that scales from simple file recovery to full incident timeline reconstruction. Its modular architecture means it can be tailored to any investigation type, and its open-source availability makes it accessible to analysts at all levels. Proficiency with Autopsy's interface — knowing where artifacts land and how to navigate to them efficiently — is a foundational skill for Windows and media forensics work.

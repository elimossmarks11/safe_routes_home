_This repository contains a project completed as part of LSE's DS105 course. It was built in December 2025_

**Team Members:** [Eli](https://github.com/elimossmarks11), [Ariana](https://github.com/ariana-alexander), [Deyaan](https://github.com/DeyaanMistry)

**Team Name:** CtrlAltElite

**Research Question:** 'What are the safest routes for women walking home at night in London?'

## How to Run This Code

### Before Running the Notebooks

**Prerequisites**

Python Version 3.12+

**Install Dependencies**

`pip install requests pandas`

**API Keys**

| API | Authentication | Setup |
|-----|----------------|-------|
| Google Places | Required | Enable in Google Cloud Console |
| Google Routes | Required | Enable in Google Cloud Console |
| UK Police | None | — |
| TfL | None | — |

**Create a `.env` file in the project root with your Google API key:**

```
API_KEY=your_google_api_key
```

### Step-by-Step Execution

Run the notebooks sequentially. Each notebook depends on the output of the previous one.

#### Step 1: Run `NB01_Data_Collection.ipynb`

Collects raw data from external APIs.

| Section | Description | API |
|---------|-------------|-----|
| 1 | Gather nightclub data for Dalston, Hackney Wick, Bermondsey | Google Places |
| 2 | Collect stations within 1km of each nightclub | Google Places |
| 3 | Filter stations by direction (8 sectors, max 2 per sector) | — |
| 4 | Generate walking routes from nightclubs to stations | Google Routes |
| 5 | Convert routes to Police API polygon format | — |
| 6 | Fetch crime data for each route corridor (Jan–Jun 2025) | UK Police |
| 7 | Check night service availability at each station | TfL |

**Notes:**
- Sections 1–4 cache API responses. Delete JSON files to force a refresh.
- TfL API (Section 7) is rate-limited; the code includes automatic retry logic.

#### Step 2: Run `NB02_Data_Transformation.ipynb`

Transforms raw JSON data into a normalised SQLite database.

| Section | Description |
|---------|-------------|
| 1 | Filter routes by night service viability |
| 2 | Create SQLite database schema |
| 3 | Batch-insert data (areas → clubs → destinations → routes → crimes) |

**Output:** `data/processed/nightclub_safety.db`

**Database Schema:**

| Table | Description |
|-------|-------------|
| `areas` | London areas (Dalston, Hackney_Wick, Bermondsey) |
| `clubs` | Nightclubs/bars within each area |
| `destinations` | Stations reachable from each club |
| `routes` | Walking routes from club to destination |
| `crimes` | Crime incidents along route corridors |
| `route_crimes` | Junction table linking routes to crimes |

---

### Output Directory Structure

After running both notebooks:

```
data/
├── raw/
│   ├── dalston/
│   │   ├── dalston_nightclubs.json
│   │   ├── stations/
│   │   │   ├── {club}_stations.json
│   │   │   └── filtered_stations/
│   │   │       └── {club}_stations_filtered.json
│   │   ├── routes/
│   │   │   └── {club}_routes.json
│   │   ├── police_params/
│   │   │   └── {club}_route_{n}_police_api_param.txt
│   │   ├── metadata/
│   │   │   └── {club}_route_{n}_metadata.json
│   │   ├── crime_data/
│   │   │   └── {YYYY-MM}/
│   │   │       └── {club}_route_{n}_crime_data_{YYYY-MM}.json
│   │   └── night_service/
│   │       └── {club}_route_{n}_night_service.json
│   ├── hackney_wick/
│   │   └── ...
│   └── bermondsey/
│       └── ...
└── processed/
    └── nightclub_safety.db
```

#### Step 3: Run `NB03_Data_Visualisation.ipynb`

Generates 4 visualisations and an interactive map visualising route safety in Bermondsey, Dalston, and Hackney Wick. 

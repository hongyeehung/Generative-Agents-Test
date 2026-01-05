# Agent Hospital Demo (Minimal)

This repo now includes a lightweight “Agent Hospital” style demo with 5 roles (DrChen, NurseLi, DrWang, PatientA, PatientB) and medical prompts. Names are URL-safe (no spaces or symbols) to avoid Django reverse lookup issues. Use it as a starting point to reproduce the paper-style flow (triage -> consult/exam -> diagnosis/treatment) at small scale.

## Key assets
- Base simulation folder: `environment/frontend_server/storage/base_agent_hospital_demo`
  - `reverie/meta.json` lists the 5 personas.
  - `environment/0.json` sets their starting tiles on `the_ville` map.
  - `personas/*/bootstrap_memory/` holds per-agent scratch & spatial memory.
- Medical prompts: `reverie/backend_server/persona/prompt_template/simple_medical_templates.py` (triage/respiratory/dermatology templates).
- Medical history seeding (optional): `environment/frontend_server/static_dirs/assets/the_ville/agent_history_init_med5.csv` gives each agent a small background/intent.

## Quick start
1) Install deps (recommend venv):
   ```
   pip install -r requirements.txt
   ```
2) Set API key (one of):
   - Export `DASHSCOPE_API_KEY` (or override model via `LLM_MODEL`), **or**
   - Export `OPENAI_API_KEY` (and optional `OPENAI_API_BASE`).
   - Set `key_owner` in `reverie/backend_server/utils.py` to your name.
3) Start frontend server:
   ```
   cd environment/frontend_server
   python manage.py runserver
   ```
4) Start backend simulation:
   ```
   cd reverie/backend_server
   python reverie.py
   ```
   - When prompted for forked simulation, enter: `base_agent_hospital_demo`
   - Enter a new simulation name, e.g., `med-demo-1`
5) (Optional) Load medical seed history at the backend prompt:
   ```
   call -- load history the_ville/agent_history_init_med5.csv
   ```
6) Run steps:
   ```
   run 50
   ```
7) View in browser:
   - Live: http://localhost:8000/simulator_home
   - Replay: http://localhost:8000/replay/<simulation-name>/<start-step>

## How to use the medical prompts
If you have a custom dialog/LLM call stack, import templates and route by role/department:
```python
from persona.prompt_template.simple_medical_templates import (
    triage_prompt, resp_doctor_prompt, derm_doctor_prompt
)
# pick prompt based on persona role/department before calling ChatCompletion
```

## Notes / tweaks
- Sprites: update `environment/frontend_server/static_dirs/assets/characters/atlas.json` and drop PNGs if you want unique looks; otherwise reuse existing sprites with new names.
- Starting positions: adjust tiles in `environment/frontend_server/storage/base_agent_hospital_demo/environment/0.json` if you want them co-located.
- Add more cases: extend `agent_history_init_med5.csv` or create new CSVs in the same folder and load them via `call -- load history ...`.

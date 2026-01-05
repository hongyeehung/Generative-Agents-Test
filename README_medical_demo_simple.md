# Medical Simulation Quick Guide (5 Roles)

Applies to the current setup: 2 doctors, 1 nurse, 2 patients (DrChen / DrWang / NurseLi / PatientA / PatientB).

## How many steps is one day?
- Step size is in `environment/frontend_server/storage/base_agent_hospital_demo/reverie/meta.json`:
  - `sec_per_step = 10` means 1 step = 10 seconds of game time
- Steps per day = 24 * 60 * 60 / 10 = `8640`
- If you only run 08:00-20:00 (12 hours) = `4320` steps

## Minimal run flow
1) Install deps: `pip install -r requirements.txt`
2) Set LLM key (`DASHSCOPE_API_KEY` or `OPENAI_API_KEY`)
3) Frontend: `cd environment/frontend_server && python manage.py runserver`
4) Backend: `cd reverie/backend_server && python reverie.py`
5) Choose base: `base_agent_hospital_demo`, then enter a new sim name
6) Optional seed history: `call -- load history the_ville/agent_history_init_med5.csv`
7) Run steps: `run 1000`
8) Browser: `http://localhost:8000/simulator_home`

## Key directories
- `reverie/backend_server/reverie.py`: main backend loop (read env -> call agents -> write movement)
- `reverie/backend_server/persona/persona.py`: per-agent core flow
- `reverie/backend_server/persona/cognitive_modules/`: perceive/retrieve/plan/reflect/execute
- `reverie/backend_server/persona/prompt_template/`: LLM prompts and wrappers
- `environment/frontend_server/`: Django frontend and map
- `environment/frontend_server/storage/base_agent_hospital_demo`: base sim (personas, start tiles)

## What happens each step (core logic)
1) Frontend writes `environment/<step>.json` (all agent positions)
2) Backend reads it and syncs positions into `Maze`
3) For each agent, call `persona.move(...)`:
   - `perceive` -> `retrieve` -> `plan` -> `reflect` -> `execute`
4) Backend writes `movement/<step>.json` for the frontend
5) `step += 1`, `curr_time += sec_per_step`

## Why it slows down after 08:00 (common causes)
- 08:00 is typically the wake-up/start-activity time: `plan.py` triggers many LLM calls
  - schedule decomposition, location/object/action generation, dialog decisions
- `debug=True` plus many `print` calls in `plan.py` cause heavy log IO
- LLM latency/rate limits cause retries in `safe_generate_response`
- If the browser tab is throttled, the frontend stops writing new env files and the backend waits

## Speed tips (no logic changes)
- Set `debug = False` in `reverie/backend_server/utils.py`
- Wrap or comment out DEBUG prints in `plan.py`
- Increase `sec_per_step` (for example 60 seconds) to cut steps per day to 1440
- Run in chunks: `run 200` and check progress
- Keep `simulator_home` in the foreground

## Find movement steps with chat records
If you do not have `rg` installed, use `grep` to list all movement files that
contain chat records:

```bash
grep -l "\"chat\": \\[" environment/frontend_server/storage/<sim-name>/movement/*.json
```

To include line numbers:

```bash
grep -n "\"chat\": \\[" environment/frontend_server/storage/<sim-name>/movement/*.json
```

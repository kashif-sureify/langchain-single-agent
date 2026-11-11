## How to run ?

1. uv venv --python 3.11
2. source .venv/bin/activate
3. uv pip install -r requirements-dev.txt   (app + notebooks + tests; `requirements.txt` is the app only)
4. streamlit run main.py

The app needs these keys in `.env`: `GEMINI_API_KEY`, `GEMINI_MODEL`, `TAVILY_API_KEY`, `WEATHER_API_KEY` (Weatherstack).

## Deploy on Streamlit Community Cloud

1. Go to [share.streamlit.io](https://share.streamlit.io), sign in with GitHub and click **Create app**, then **Deploy a public app from GitHub**.
2. Repository `kashif-sureify/langchain-single-agent`, branch `main`, main file path `main.py`.
3. Under **Advanced settings**, pick Python **3.11** and paste the secrets (TOML):

   ```toml
   GEMINI_API_KEY = "..."
   GEMINI_MODEL = "gemini-3.5-flash-lite"
   TAVILY_API_KEY = "..."
   WEATHER_API_KEY = "..."
   ```

4. Click **Deploy**. Every push to `main` redeploys the app.

Cloud only installs `requirements.txt`. The keys reach the app as environment variables, so no code change is needed between local and Cloud.

## Project layout

```
main.py            Streamlit entry point. Wires core/ and ui/ together
core/              the agent itself (no Streamlit)
  config.py        loads .env
  weather.py       get_weather tool (Weatherstack: weather + AQI)
  search.py        tavily_search tool (langchain-tavily)
  agent.py         Gemini agent + memory, run_agent() / load_history() events
ui/                Streamlit UI
  styles.py        CSS, AQI colours
  components.py    hero, weather card, sources, tool-step lines
  theme.py         Light / Dark / System switch
  session.py       chat thread (kept in the URL), history, queued prompts
  sidebar.py       sidebar
  chat.py          transcript + live streaming of a reply
tests/             pytest suite
.streamlit/        theme config (separate light / dark palettes)
```

`core/` can be used without the UI, e.g. from a notebook:

```python
from core.agent import build_agent, run_agent
from core.config import load_config

agent = build_agent(load_config())
for event in run_agent(agent, "Weather and AQI in Delhi, India?", thread_id="demo"):
    print(event)
```

## Tests

```bash
pytest                          # offline: core logic + UI (AppTest with a fake agent), ~3s
RUN_LIVE_TESTS=1 pytest -m live # real Gemini / Tavily / Weatherstack calls
```

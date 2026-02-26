# AGENTS.md

## Cursor Cloud specific instructions

### Project overview

AlphaGPT is a crypto quantitative trading system for Solana meme tokens. It uses a Transformer model to generate interpretable factor formulas, evaluates them via backtesting, and deploys for live trading. See `CATREADME.md` for a detailed architecture walkthrough.

### Services

| Service | Purpose | How to run |
|---|---|---|
| **PostgreSQL** | Central data store (`crypto_quant` DB with `tokens` and `ohlcv` tables) | `sudo pg_ctlcluster 16 main start` |
| **model_core** | Train AlphaGPT factor generator | `cd /workspace && python3 -m model_core.engine` |
| **data_pipeline** | Fetch Solana token data from Birdeye API into PostgreSQL | `cd /workspace && python3 -m data_pipeline.run_pipeline` (requires `BIRDEYE_API_KEY`) |
| **dashboard** | Streamlit web UI for monitoring | `cd /workspace && streamlit run dashboard/app.py --server.port 8501 --server.headless true` |
| **execution** | Solana trading via Jupiter DEX | Requires `SOLANA_PRIVATE_KEY` and `QUICKNODE_RPC_URL` in `.env` |

### Key gotchas

- **PostgreSQL must be started manually** before running any module that touches the database (`model_core`, `data_pipeline`, `dashboard`). Use `sudo pg_ctlcluster 16 main start`.
- **`execution/config.py` raises `ValueError` at import time** if `SOLANA_PRIVATE_KEY` is not set. This means any code that imports from the `execution` package will crash without that env var. The `dashboard` module handles this gracefully with a try/except.
- **No `__init__.py` files exist** in any package directory. Python 3.3+ namespace packages make relative imports work when using `python3 -m package.module` syntax from the workspace root.
- The `.env` file at `/workspace/.env` holds database credentials and API keys. Default DB credentials are `postgres`/`password` on `localhost:5432`, database `crypto_quant`.
- `model_core` training requires data in the `tokens` and `ohlcv` PostgreSQL tables. Without real Birdeye data, you can seed synthetic data for testing (see database seeding script patterns in the setup session).
- Lint: `flake8 --max-line-length=120` from the workspace root. There are ~400 pre-existing style warnings; no lint config file exists in the repo.
- The `dashboard/app.py` imports from `data_service` and `visualizer` using bare module names (not relative), so Streamlit must be run from or with `dashboard/` as the working context (the `streamlit run dashboard/app.py` command handles this).
- `data_loader.py` uses deprecated `DataFrame.fillna(method='ffill')` which emits FutureWarning on pandas 2.x. This is cosmetic and does not affect functionality.

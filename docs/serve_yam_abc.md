# Serving `cortexairobot/pack-groceries-openpi-15k-steps`

A pi0.5 policy for the bimanual YAM (XDOF/ABC) setup, using absolute joint actions. It is served with the `pi05_abc` config.

## 1. Download the checkpoint

```bash
hf download cortexairobot/pack-groceries-openpi-15k-steps \
  --local-dir checkpoints/pack-groceries-openpi-15k-steps
```

## 2. Start the server

```bash
uv run scripts/serve_policy.py policy:checkpoint \
  --policy.config=pi05_abc \
  --policy.dir=checkpoints/pack-groceries-openpi-15k-steps
```

The server listens on port 8000 by default; use `--port` to change it. Norm stats are loaded from the checkpoint's `assets/pi05_yam_abc/jdsample-pack-groceries-merged/norm_stats.json`.

## 3. Query from the robot

Install the client (`uv pip install -e packages/openpi-client`), then:

```python
from openpi_client import websocket_client_policy

client = websocket_client_policy.WebsocketClientPolicy(host="<server-ip>", port=8000)

obs = {
    "images": {
        "top": top_img,                  # HxWx3 uint8 (or 3xHxW, or float in [0, 1])
        "left_wrist": left_wrist_img,    # "left" also accepted
        "right_wrist": right_wrist_img,  # "right" also accepted
    },
    "state": state,                      # (14,) absolute joints: [left 6 joints, left gripper, right 6 joints, right gripper]
    "prompt": "pack the groceries",
}
actions = client.infer(obs)["actions"]  # (30, 14) absolute joint targets, same layout as state
```

Images are resized to 224x224 on the server. Each call returns an action chunk of 30 steps. Execute some or all of them before you query again.

See [remote_inference.md](remote_inference.md) for general details on the client/server setup.

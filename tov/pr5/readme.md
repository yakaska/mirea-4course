```bash
source venv/bin/activate
pip install fastapi[all]
```

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
import subprocess

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
async def read_root():
    return """
    <html>
    <head>
        <title>Deploy VMs</title>
    </head>
    <body>
        <h1>Deploy Virtual Machines</h1>
        <form action="/deploy" method="post">
            <button type="submit">Deploy VMs</button>
        </form>
    </body>
    </html>
    """

@app.post("/deploy")
async def deploy():
    # Запускаем Ansible плейбук
    result = subprocess.run(['ansible-playbook', 'pve_create_vm.yml', '-i', 
                             'hosts', '--user=ansible'], capture_output=True, text=True)
    if result.returncode == 0:
        return {"message": "VMs deployed successfully!"}
    else:
        return {"message": "Error deploying VMs", "details": result.stderr}

if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='0.0.0.0', port=8000)
```

```bash
python main.py
```
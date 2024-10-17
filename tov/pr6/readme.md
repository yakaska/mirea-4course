```bash
pip install fastapi uvicorn proxmoxer jinja2
```

template.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Управление нодами и ВМ Proxmox</title>
<!-- Подключение Bootstrap -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-
alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
<div class="container mt-5">
<h1 class="text-center mb-4">Управление нодами и ВМ Proxmox</h1>
<h2 class="text-center mb-4">Информация о нодах и виртуальных
машинах</h2>
<div class="table-responsive">
<table class="table table-bordered table-hover">
<thead class="table-success">
<tr>
<th scope="col">Нода</th>
<th scope="col">ID ВМ</th>
<th scope="col">Имя ВМ</th>
<th scope="col">Статус</th>
<th scope="col">Память</th>
<th scope="col">CPU</th>
<th scope="col">Действия</th>
</tr>
</thead>
<tbody>
{% for node in nodes %}
{% for vm in node.vms %}
<tr>
<td>{{ node.name }}</td>
<td>{{ vm.vmid }}</td>
<td>{{ vm.name }}</td>
<td>{{ vm.status }}</td>
<td>{{ vm.memory }} МБ</td>
<td>{{ vm.cpu }}%</td>
<td>
<form action="/delete_vm" method="post" class="d-inline">
<input type="hidden" name="node" value="{{ node.name
}}">
<input type="hidden" name="vmid" value="{{ vm.vmid }}">
<button type="submit" formaction="/start_vm" class="btn
btn-success btn-sm">Запустить</button>
<button type="submit" formaction="/stop_vm" class="btn
btn-warning btn-sm">Остановить</button>
<button type="submit" class="btn btn-danger btnsm">Удалить</button>
</form>
</td>
</tr>
{% endfor %}
{% endfor %}
</tbody>
</table>
</div>
<h2 class="text-center mt-5">Создание новой виртуальной машины</h2>
<form action="/create_vm" method="post" class="mt-4">
<div class="mb-3">
<label for="node" class="form-label">Выберите ноду:</label>
<select name="node" id="node" class="form-select">
{% for node in nodes %}
<option value="{{ node.name }}">{{ node.name }}</option>
{% endfor %}
</select>
</div>
<div class="mb-3">
<label for="vm_name" class="form-label">Имя ВМ:</label>
<input type="text" id="vm_name" name="vm_name" class="form-control"
required>
</div>
<button type="submit" class="btn btn-primary">Создать ВМ</button>
</form>
</div>
<!-- Подключение Bootstrap JS и Popper.js -->
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-
alpha1/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

connect proxmox

main.py

```python
from fastapi import FastAPI, Form
from fastapi.responses import HTMLResponse
from proxmoxer import ProxmoxAPI
from jinja2 import Template

app = FastAPI()

# Подключение к Proxmox API
proxmox = ProxmoxAPI('192.168.89.11', user='root@pam', password='123123', verify_ssl=False)

# Чтение HTML-шаблона
with open("temp.html") as f:
    html_template = f.read()

@app.get("/", response_class=HTMLResponse)
async def read_root():
    nodes_info = []
    nodes = proxmox.nodes.get()
    for node in nodes:
        node_name = node['node']
        vms = proxmox.nodes(node_name).qemu.get()
        vm_info = []
        for vm in vms:
            status = proxmox.nodes(node_name).qemu(vm['vmid']).status.current.get()
            vm_info.append({
                "vmid": vm['vmid'],
                "name": vm['name'],
                "status": status['status'],
                "memory": round(status['mem'] / 1024**2, 2),
                "cpu": round(status['cpu'] * 100, 2)
            })
        nodes_info.append({"name": node_name, "vms": vm_info})

    template = Template(html_template)
    return template.render(nodes=nodes_info)

@app.post("/create_vm")
async def create_vm(node: str = Form(...), vm_name: str = Form(...)):
    # Получаем следующий доступный VM ID
    vm_id = proxmox.cluster.nextid.get()

    # Создаем виртуальную машину на указанной ноде
    proxmox.nodes(node).qemu.post(
        vmid=vm_id,
        name=vm_name,
        cores=2,
        memory=2048,
        net0='virtio,bridge=vmbr0',
        ostype='l26'  # Тип ОС для Linux
    )
    return {"message": f"VM '{vm_name}' created on node {node}"}

@app.post("/delete_vm")
async def delete_vm(node: str = Form(...), vmid: str = Form(...)):
    # Удаление ВМ по ID на указанной ноде
    proxmox.nodes(node).qemu(vmid).delete()
    return {"message": f"VM {vmid} deleted from node {node}"}

@app.post("/start_vm")
async def start_vm(node: str = Form(...), vmid: str = Form(...)):
    # Добавьте код для старта виртуальной машины
    #
    #
    #
    return {"message": f"VM {vmid} started on node {node}"}

@app.post("/stop_vm")
async def stop_vm(node: str = Form(...), vmid: str = Form(...)):
    # Добавьте код для остановки виртуальной машины
    #
    #
    #
    return {"message": f"VM {vmid} stopped on node {node}"}

if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='0.0.0.0', port=8000)

```

```bash
python main.py
```
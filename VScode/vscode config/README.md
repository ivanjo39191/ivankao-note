# vscode config

Created: February 1, 2022 5:22 PM
Tags: VScode

```bash
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more info, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Django",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/path/src/manage.py",
      "args": [
        "runserver",
        "0.0.0.0:8899",
        "--noreload"
      ],
      "django": true
    }
  ]
}
```
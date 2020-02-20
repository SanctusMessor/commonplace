# Windows

### Remove Windows' Path from WSL $PATH

Edit or Create `/etc/wsl.conf`

{% code title="/etc/wsl.conf" %}
```text
[interop]
appendWindowsPath = false
```
{% endcode %}


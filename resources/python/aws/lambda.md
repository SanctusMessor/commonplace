# Lambda

## Working with Lambda Layers

When creating a zip for a Lambda Layer \(Python 3.6+\) you need a full path to site-packages as shown below, using python3.8 as an example. Despite the common guidance, `python -m pip install packagename -t folder/path` does **not** work.

{% hint style="info" %}
python/lib/python3.8/site-packages
{% endhint %}

```text
python
└── lib
    └── python3.8
        └── site-packages
            ├── aiohttp
            │   ├── __init__.py
            │   ├── ...
            ├── slack
            │   ├── __init__.py
            │   ├── ...
            ├── slackclient-2.5.0.dist-info
            │   ├── ...
```

The folder will be unzipped at `/opt` 


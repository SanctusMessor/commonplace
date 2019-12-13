---
description: Level up your AWS CloudFormation game
---

# Cloudformation

{% embed url="https://marketplace.visualstudio.com/items?itemName=SirTori.indenticator" %}

{% embed url="https://marketplace.visualstudio.com/items?itemName=oderwat.indent-rainbow" %}

{% hint style="info" %}
 Add the following [user setting](https://code.visualstudio.com/docs/getstarted/settings) to enable the extra highlighting of the block located at the current cursor position.
{% endhint %}

```yaml
{
    "indenticator.inner.showHighlight": true
}
```

{% embed url="https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig" caption="Also see: https://editorconfig.org/" %}

{% hint style="info" %}
 Inside your Git repositories that contain your CloudFormation templates, create a `.editorconfig` file at the root directory that looks like this:
{% endhint %}

```bash
# .editorconfig

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
end_of_line = lf
insert_final_newline = true

# Keeps CloudFormation YAML files standard
[*.yaml]
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
```

{% embed url="https://marketplace.visualstudio.com/items?itemName=Tyriar.sort-lines" %}

{% hint style="warning" %}
Prerequisite: Install `cfn-lint` before `vscode-cfn-lint`
{% endhint %}

```bash
python -m pip install cfn-lint
```

{% embed url="https://marketplace.visualstudio.com/items?itemName=kddejong.vscode-cfn-lint" %}

{% embed url="https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml" %}




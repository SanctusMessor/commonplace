---
description: >-
  ref: (new) Windows Terminal -
  https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701
---

# Windows Terminal

## WSL

Preferred flavor: Ubuntu  
ref: [Creating Additional WSL2 Instances](https://commonplace.sanctus.dev/resources/windows/wsl2/creating-additional-wsl2-instances)  
font: Cascadoa Mono PL from [Microsoft Cascadia Code](https://github.com/microsoft/cascadia-code/releases)

oh-my-zsh  
Theme: agnoster  
Plugins:

```bash
plugins=(
	git
	kubectl
	kube-ps1
	docker
	docker-compose
	helm
	aws
	golang
	python
)
```

Install:  
- kubectl  
- krew  
Krew Install:  
- ns  
- ctx

Add this to the end of `~/.oh-my-zsh/themes/agnoster.zsh-theme`

```bash
prompt_kube_context() {
	if [[ $KUBE_PS1_ENABLED == "on" ]]; then
    echo $(kube_ps1)"\n% " 
	fi
}

PROMPT='$(prompt_kube_context)%{%f%b%k%}$(build_prompt) '
```

Refresh your session: `source ~/.zshrc`  
You can now use `kubeon` and `kubeoff`


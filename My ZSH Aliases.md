# Zsh Aliases

```
alias zshconfig="nvim ~/.zshrc"
alias ohmyzsh="nvim ~/.oh-my-zsh"
alias pbcopy='xsel --clipboard --input'
alias pbpaste='xsel --clipboard --output'

alias mypubkey="cat ~/.ssh/id_rsa.pub|pbcopy && echo 'ssh public key copied in buffer'"
alias myprivatekey="cat ~/.ssh/id_rsa|pbcopy && echo 'ssh private key copied in buffer'"

alias wg="wget"
alias kb="kubectl"
alias mk="minikube"
alias hl="helm"
alias tf="terraform"
alias vg="vagrant"
alias pt="poetry"
alias mng="python3 manage.py"
alias listen="lsof -i -P -n | grep LISTEN"
alias listen_port="lsof -i -P -n | grep LISTEN | grep $1"
alias dc="docker-compose"
alias dk="docker"
alias gp="git add -A; git commit -m '$1'; git push" 
alias dos="dosbox -conf $HOME/.dosbox.conf $HOME/DOS"
alias -s {md,json,rs,css,ts,js,html,yaml,yml,conf}=nvim
alias vi="nvim"
alias ec="nvim $HOME/.zshrc"
alias sc="source $HOME/.zshrc"

alias ytd="youtube-dl"
alias wg-up="sudo wg-quick up wg0"
alias wg-down="sudo wg-quick down wg0"
```
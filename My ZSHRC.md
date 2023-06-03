
```  
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="powerlevel10k/powerlevel10k"
plugins=(
adb
ag
alias-finder
aliases
ansible
autopep8
aws
battery
branch
brew
cake
coffee
colorize
cp
docker
docker-compose
dotenv
encode64
gem
gcloud
gh
git-flow
github
gitignore
golang
gradle
gulp
helm
heroku
iterm2
jira
kubectl
mvn
minikube
nmap
node
npm
oc
operator-sdk
pep8
pip
pipenv
poetry
postgres
pyenv
pylint
python
react-native
rsync
rust
sdk
sudo
vault
vagrant
virtualenv
wp-cli
yarn
vscode
zsh-autosuggestions
zsh-syntax-highlighting
git)

alias zshconfig="nvim ~/.zshrc"
alias ohmyzsh="nvim ~/.oh-my-zsh"
alias pbcopy='xsel --clipboard --input'
alias pbpaste='xsel --clipboard --output'
alias mypubkey="cat ~/.ssh/id_rsa.pub|pbcopy && echo 'ssh public key copied in buffer'"
alias myprivatekey="cat ~/.ssh/id_rsa|pbcopy && echo 'ssh private key copied in buffer'"
alias wg="wget"
alias kb="kubectl"
alias kbx="kubectx"
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
alias vi="nvim"
alias ec="nvim $HOME/.zshrc"
alias sc="source $HOME/.zshrc"
alias ytd="youtube-dl"
alias wg-up="sudo wg-quick up wg0"
alias wg-down="sudo wg-quick down wg0"
alias -s {md,json,rs,css,ts,js,html,yaml,yml,conf}=nvim

source $ZSH/oh-my-zsh.sh
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh

# The next line updates PATH for Yandex Cloud CLI.
if [ -f '/home/oleg/yandex-cloud/path.bash.inc' ]; then source '/home/oleg/yandex-cloud/path.bash.inc'; fi

# The next line enables shell command completion for yc.
if [ -f '/home/oleg/yandex-cloud/completion.zsh.inc' ]; then source '/home/oleg/yandex-cloud/completion.zsh.inc'; fi
```
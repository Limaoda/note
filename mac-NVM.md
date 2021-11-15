```shell
# nvm更换镜像源
# 方式一
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node

# 方式二
nvm node_mirror: https://npm.taobao.org/mirrors/node/
nvm npm_mirror: https://npm.taobao.org/mirrors/npm/

# nvm环境变量配置（永久）
vi ./.bash_profile

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm

vim ~/.zshrc
# 在文件最后一行
# source ~/.bash_profile

# npm更换镜像源
npm config set registry https://registry.npm.taobao.org
```


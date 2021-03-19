# awscli
## awscli installation
```shell
brew install awscli
```
- refer: https://aws.amazon.com/cli/
## aws_completer installation
**zshrc**
```shell
which aws_completer
export PATH=/usr/local/bin:$PATH
source ~/.bash_profile
autoload bashcompinit && bashcompinit
complete -C '/usr/local/bin/aws_completer' aws
```
Add the command to ~/.zshrc to run it each time you open a new shell.
```
cat <<EOF >>~/.zshrc
complete -C '/usr/local/bin/aws_completer' aws
EOF
```
- refer: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-completion.html
## aws configure 
```shell
$ aws configure
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json

$ cd ~/.aws
$ tree                                           
├── config
├── credentials
└── ops-hub
    └── logs
        ├── main.2020-10-25.log
        ├── main.2020-10-28.log
        ├── renderer.2020-10-25.log
        ├── renderer.2020-10-28.log
        ├── worker.2020-10-25.log
        └── worker.2020-10-28.log

2 directories, 8 files
```
AWS prompts you to enter your Access Key ID and Secret Access Key and stores them in ~/.aws/credentials:
```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAdPLE
aws_secret_access_key=wJalrXUtnFEMI/K7sdENG/bPxRfiCYEXAMPLEKEY
```
It also stores the other settings you entered in ~/.aws/config:
```
[default]
region=us-west-2
output=json
```

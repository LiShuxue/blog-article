## 批量修改Git所有提交记录的作者和邮箱

1. 先进入要修改的repo下面，设置正确的Arthur 和 Email
```sh
git config user.name 'LiShuxue '
git config user.email '1149926505@qq.com'
```

2. 执行以下命令，批量修改
```sh
git filter-branch --env-filter '

OLD_EMAIL="your old email"
CORRECT_NAME="new arthur name"
CORRECT_EMAIL="new arthur email"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

3. 执行以下命令，将改动推送到远程
```sh
git push origin --force --all
```

4. 以后记住公司git账号跟私人的分开，免得再用公司邮箱提交到自己的GitHub
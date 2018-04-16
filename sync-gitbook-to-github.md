最近因為gitbook改版，讓我有點心驚驚，所以想要把gitbook都遷到github上，至少當個備份。

要做的事情其實很單純：

1. git clone原本的gitbook repo

2. git push到github上的repo

3. 啟動sync

要git clone gitbook有些眉角，首先要先創出gitbook的credential\(`~/.netrc`\)。

```
machine git.gitbook.com
  login USERNAME-or-EMAIL
  password API-TOKEN-or-PASSWORD
```

然後就可以

> git clone [https://git.gitbook.com/{{UserName}}/{{Book}}.git](https://git.gitbook.com/{{UserName}}/{{Book}}.git)

接著加入github的remote

```bash
git remote rm origin ; git remote add origin git@github.com:{{UserName}}/{{Book}}.git
git push origin master
```

最後啟動gitbook內的github sync，只要選對repo，就會看到結果是IN SYNC。






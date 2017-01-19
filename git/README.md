# Git

## Rebase

From [documentation](https://git-scm.com/docs/git-rebase).

Current situation:

```
      A---B---C topic
     /
D---E---F---G master
```

Do the rebase:

```bash
$ git checkout topic
$ git rebase master
```

New state:

```
              A'--B'--C' topic
             /
D---E---F---G master
```

## Merge

From [documentation](https://git-scm.com/docs/git-merge).

Current situation:

```
      A---B---C feature
     /
D---E---F---G devel
```

Merge a feature into `devel` branch:

```bash
$ git checkout devel
$ git merge --no-ff feature
$ git push origin devel
```

New state:

```
      A---B---C feature
     /         \
D---E---F---G---H devel
```

## Tags

After a merge into `master` branch:

```bash
$ git checkout master
$ git tag
# 0.0.1
# 0.0.2
# 0.1.0

# Create a changelog from last tag 0.1.0
$ git log --pretty=oneline 0.1.0...HEAD > /tmp/file.txt

# Create the new tag
$ git tag -a 0.1.1 -F /tmp/file.txt
$ git push --tags
```
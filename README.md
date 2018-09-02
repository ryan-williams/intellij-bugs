# IDEA-195908

## Repro Steps:

### Clone + checkout branch

```bash
git clone git@github.com:ryan-williams/intellij-bugs.git
cd intellij-bugs
git checkout IDEA-195908
```

### Backup existing local Maven repository

```bash
mv ~/.m2/repository{,.bak}
```

### Import project to IntelliJ, verify "Source Sets" in `:foo` module:




### Extract [`repository.tar.gz`](./repository.tar.gz) into `~/.m2/repository`:

```bash
tar xvzf repository.tar.gz -C ~/.m2
```

<details><summary>#### Check contents</summary>
<p>

```bash
pushd ~/.m2
find repository
```
```
repository
repository/org
repository/org/apache
repository/org/apache/hadoop
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.jar
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.pom.sha1
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.pom
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.jar.sha1
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/_remote.repositories
```
</p>
</details>

#### Verify SHAs

```bash
cat <<EOF | sha1sum -c -
d4898ad4355427b81c9b17bba876aacfdecf4924 repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.pom
5fa80f09065a661227da50ba7df3ec5a0e9425d3 repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.jar
EOF
```
```
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.pom: OK
repository/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/hadoop-mapreduce-client-jobclient-2.7.3.jar: OK
```

- [see also: Maven Central](https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3/)
- On OSX, `sha1sum` may be available as `gsha1sum` via `brew install coreutils` 

### Reload IntelliJ project, observe "Source Sets" are gone

### Restore "Source Sets" via any of several tweaks

When these things are all true:
- `mavenLocal` comes before `mavenCentral` in `project.repositories`
- `foo` depends on `hadoop-minicluster`, and
- `hadoop-mapreduce-client-jobclient:2.7.3` is present in `~/.m2/repository`,

then the `foo` module is not loaded correctly.

Breaking any one of them makes "Source Sets" come back:
- remove `mavenLocal` from `foo/build.gradle`
  - or just put it after `mavenCentral` 
- remove the `hadoop-minicluster` dependency in `foo/build.gradle`
- remove `~/.m2/repository`
  - this restores the state from [the beginning](#backup-existing-local-maven-repository)
  - even in the presence of lots of other artifacts in `~/.m2/repository`, removing just `org/apache/hadoop/hadoop-mapreduce-client-jobclient/2.7.3` brings back "Source Sets"


### Restore local Maven repository

```bash
mv ~/.m2/repository{.bak,}
```

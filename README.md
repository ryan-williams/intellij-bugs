# IDEA-195908

When a certain (valid) artifact is in the local Maven repository, the IntelliJ Gradle plugin silently fails to load a module (that transitively depends on the artifact) correctly

See [screencast here](https://www.youtube.com/watch?v=wLEckYEX-_c), or steps below:

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

![](https://cl.ly/f4f8551b6356/Screen%20Shot%202018-09-02%20at%203.01.23%20PM.png)

![](https://cl.ly/7523e20871eb/Screen%20Shot%202018-09-02%20at%203.01.56%20PM.png)

![](https://cl.ly/7cf8887a9bb1/Screen%20Shot%202018-09-02%20at%203.03.22%20PM.png)

### Extract [`repository.tar.gz`](./repository.tar.gz) into `~/.m2/repository`:

```bash
tar xvzf repository.tar.gz -C ~/.m2
```

`~/.m2/repository` now contains only the artifact `org.apache.hadoop:hadoop-mapreduce-client-jobclient:2.7.3`

<details><summary>Check contents</summary>
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

<details><summary>Verify SHAs</summary>
<p>

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

</p>
</details>

### Reload IntelliJ project, observe "Source Sets" are gone

![](https://cl.ly/7df12d595c93/Screen%20Shot%202018-09-02%20at%203.04.33%20PM.png)

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


### Cleanup: restore local Maven repository

```bash
rm -rf ~/.m2/repository
mv ~/.m2/repository{.bak,}
```

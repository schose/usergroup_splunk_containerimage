---

marp: true
theme: default
paginate: true
backgroundImage: url(img/background-std-header.png)

---
<!-- Global style -->
<style>
header {
  color: white;
  background-color: black;
  text-align: right;
  font-size: 30px;
  left: 450px;
  height: 50px;
  top: 10px;
  position: absolute;
  
}

img[alt~="center"] {
  display: block;
  margin: 0 auto;
}

h1 {
  text-align: right;
  font-size: 60px;
}

</style>
<!-- Global style -->

<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: true,
  theme: "default",
  });
</script>

                     build your Dev Environment in 10 Minutes.. 


--- 

<!-- _header: 'Container Lifecycle' --> 

<div class="mermaid" style="text-align:center">
flowchart LR;
    A[Developer] --> B[Dockerfile];
    B --> C[build Image];
    C --push--> G[Docker Hub/Registry<br>Image];
    G --pull--> User;
    User --run--> F[Container];
    style A fill:#f9f,stroke:#333,stroke-width:1px;
    style F fill:#bbf,stroke:#333,stroke-width:1px;

</div>

--- 

<!-- _header: 'Splunk Container Image' --> 

- [Docker Hub](https://hub.docker.com/r/splunk/splunk)

- [Github Repo](https://github.com/splunk/docker-splunk)

- [Deployment splunk-ansible](https://github.com/splunk/splunk-ansible)

- "Support": 
https://www.splunk.com/blog/2018/10/24/announcing-splunk-on-docker.html

- today Splunk Enterprise Core: linux/amd64 only(!)

---

<!-- _header: 'hello world' --> 

```bash
docker run splunk/splunk:9.3.4 

WARNING: No password ENV var.  Stack may fail to provision if splunk. \
password is not set in ENV or a default.yml\
License not accepted, please adjust SPLUNK_START_ARGS to indicate you \
have accepted the license.
The license you are accepting is the Splunk General Terms, available \
here: https://www.splunk.com/en_us/legal/splunk-general-terms.html
Unless you have jointly executed with Splunk a negotiated version of \
these General Terms that explicitly supersedes this agreement, by \
accessing or using Splunk software, you are agreeing to the Splunk \
General Terms.
Please read and make sure you agree to the Splunk General Terms before \
you access or use this software.
only once you've done so should you include the '--accept-license' \
flag to indicate your acceptance of the Splunk General Terms and \
launch this software.\

For example: docker run -e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=Password01 splunk/splunk
```

--- 

<!-- _header: 'start me!' --> 

<div class="mermaid" style="text-align:center">
graph LR;
    SplunkContainer[Splunk Docker Container v9.3.4];
    subgraph EnvVars [Environment Variables];
        envPassword[envPassword];
        envRole[envRole];
    end;
    subgraph Volumes [Volumes];
            VolumeEtc[myvol_etc -> etc];
            VolumeVar[myvol_var -> var];
    end;
    subgraph PortMappings [PortMappings];
            User;
    end;
    User <-->|8101 ➝ 8000 <br> 8111 ➝ 8089| SplunkContainer;
    SplunkContainer --> VolumeEtc;
    SplunkContainer --> VolumeVar;
    envPassword --> SplunkContainer;
    envRole --> SplunkContainer;
</div>

```bash
docker run -p 8101:8000 -p 8111:8089 \
-v myvol_etc:/opt/splunk/etc -v myvol_var:/opt/splunk/var \
-e SPLUNK_START_ARGS=--accept-license -e SPLUNK_PASSWORD=Password01 \ 
splunk/splunk:9.3.4
```
---

<!-- _header: 'vars vars vars!!' --> 


- https://splunk.github.io/splunk-ansible/ADVANCED.html#supported-environment-variables
- SPLUNK_ROLE=splunk_standalone,splunk_indexer,splunk_deployer..
  - SPLUNK_SECRET=ABCDEFG..
  - JAVA_VERSION=openjdk:11
  - JAVA_DOWNLOAD_URL=https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz
  - SPLUNKBASE_USERNAME=myuser SPLUNKBASE_PASSWORD=
  - SPLUNK_APPS_URL=https://github.com/schose/fixedissues/releases/download/v0.9.1/fixedissues_v091.tgz,https://splunkbase.splunk.com/app/2686/release/3.18.2/download
  - SPLUNK_LICENSE_URI= #URI or local file path
  

---

<!-- _header: 'docker compose' --> 

```yaml
services:
  single:
    image: docker.io/splunk/splunk:9.3.4
    ports:
      - "8101:8000"
      - "8102:8089"
    volumes:
      - myvol_etc:/opt/splunk/etc
      - myvol_var:/opt/splunk/var
    hostname: single
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=Password01
      - SPLUNK_ROLE=splunk_standalone
      - JAVA_VERSION=openjdk:11
      - JAVA_DOWNLOAD_URL=https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz
      - SPLUNK_APPS_URL=https://github.com/schose/fixedissues/releases/download/v0.9.1/fixedissues_v091.tgz
volumes:
  myvol_etc:
  myvol_var:
```

```bash
docker-compose up -d
```

--- 
<!-- _header: 'build index cluster - cluster manager' --> 

<div class="mermaid" style="text-align:center">
graph TD;
    A[Cluster Manager] --> IDX1[Indexer Peer 1];
    A --> IDX2[Indexer Peer 2];
    A --> IDX3[Indexer Peer 3];
    %% Apply red border only;
    style A stroke:#ff0000,stroke-width:8px;
</div>


```yaml
- SPLUNK_START_ARGS="--accept-license"
- SPLUNK_PASSWORD=Password01
- SPLUNK_ROLE=splunk_cluster_master
- SPLUNK_INDEXER_URL=idx1,idx2,idx3
- SPLUNK_IDXC_SECRET=mypass4splunk
```

--- 
<!-- _header: 'build index cluster - cluster peers' --> 

<div class="mermaid" style="text-align:center">
graph TD;
    A[Cluster Manager] --> IDX1[Indexer Peer 1];
    A --> IDX2[Indexer Peer 2];
    A --> IDX3[Indexer Peer 3];
    %% Apply red border only;
    style IDX1 stroke:#ff0000,stroke-width:8px;
    style IDX2 stroke:#ff0000,stroke-width:8px;
    style IDX3 stroke:#ff0000,stroke-width:8px;
</div>

```yaml
- SPLUNK_START_ARGS="--accept-license"
- SPLUNK_PASSWORD=Password01
- SPLUNK_ROLE=splunk_indexer
- SPLUNK_CLUSTER_MASTER_URL=clm
- SPLUNK_IDXC_SECRET=mypass4splunk
```
--- 
<!-- _header: 'Searchhead cluster' --> 

<div class="mermaid" style="text-align:center">
graph TD;
    A[Deployer] --> SHD1[Captain];
    A --> SHD2[Search Peer 1];
    A --> SHD3[Search Peer 2];
    %% Apply red border only;
    style A stroke:#ff0000,stroke-width:8px;
</div>

```yaml
      - SPLUNK_START_ARGS="--accept-license"
      - SPLUNK_PASSWORD=Password01
      - SPLUNK_ROLE=splunk_deployer
      - SPLUNK_CLUSTER_MASTER_URL=clm
      - SPLUNK_INDEXER_URL=idx1,idx2,idx3      
      - SPLUNK_SEARCH_HEAD_URL=search-0,search-1
      - SPLUNK_DEPLOYER_URL=deployer
      - SPLUNK_SEARCH_HEAD_CAPTAIN_URL=captain
      - SPLUNK_SHC_SECRET=mypass4splunk
      - SPLUNK_IDXC_SECRET=mypass4splunk
```
--- 
<!-- _header: 'Searchhead cluster' --> 

<div class="mermaid" style="text-align:center">
graph TD;
    A[Deployer] --> SHD1[Captain];
    A --> SHD2[Search Peer 1];
    A --> SHD3[Search Peer 2];
    %% Apply red border only;
    style SHD1 stroke:#ff0000,stroke-width:8px;
</div>

```yaml
      - SPLUNK_START_ARGS="--accept-license"
      - SPLUNK_PASSWORD=Password01
      - SPLUNK_ROLE=splunk_search_head_captain
      - SPLUNK_CLUSTER_MASTER_URL=clm
      - SPLUNK_INDEXER_URL=idx1,idx2,idx3
      - SPLUNK_SEARCH_HEAD_URL=search-0,search-1
      - SPLUNK_DEPLOYER_URL=deployer
      - SPLUNK_SEARCH_HEAD_CAPTAIN_URL=captain
      - SPLUNK_SHC_SECRET=mypass4splunk
      - SPLUNK_IDXC_SECRET=mypass4splunk
```
--- 
<!-- _header: 'Searchhead cluster' --> 

<div class="mermaid" style="text-align:center">
graph TD;
    A[Deployer] --> SHD1[Captain];
    A --> SHD2[Search Peer 1];
    A --> SHD3[Search Peer 2];
    %% Apply red border only;
    style SHD2 stroke:#ff0000,stroke-width:8px;
    style SHD3 stroke:#ff0000,stroke-width:8px;
</div>

```yaml
      - SPLUNK_START_ARGS="--accept-license"
      - SPLUNK_PASSWORD=Password01
      - SPLUNK_ROLE=splunk_search_head
      - SPLUNK_CLUSTER_MASTER_URL=clm
      - SPLUNK_INDEXER_URL=idx1,idx2,idx3
      - SPLUNK_SEARCH_HEAD_URL=search-0,search-1
      - SPLUNK_DEPLOYER_URL=deployer
      - SPLUNK_SEARCH_HEAD_CAPTAIN_URL=captain
      - SPLUNK_SHC_SECRET=mypass4splunk
      - SPLUNK_IDXC_SECRET=mypass4splunk
```
--- 
<!-- _header: 'commandline things..' --> 

- copy something into a container

```bash
docker cp apps/testapp/default/indexes.conf splunk_clm:/opt/splunk/etc/manager-apps/_cluster/local/
```

- run a command as splunk user

```bash
docker exec -ti splunk_clm sudo -u splunk /opt/splunk/bin/splunk apply cluster-bundle -auth admin:Password01
```

- open a shell in the container

```bash
docker exec -ti splunk_clm /bin/bash

[ansible@splunk_clm splunk]$ sudo -s
[root@splunk_clm splunk]# exit
exit
[ansible@splunk_clm splunk]$ sudo -u splunk -s
[splunk@splunk_clm splunk]$
```

--- 
<!-- _header: 'known issues' --> 

- `docker-compose` (pip) vs. `docker compose` (native)
- `podman` vs. `podman-compose` (pip)
- `SPLUNK_LICENSE_URI
- splunktcp ingest
- for LB of SHC - traefik, nginx..
- **High Availability** - e.g. use kubernetes

--- 
![bg](img/background-chapter.png)


# Fragen? # 

